1. Enum
- The website is rather simple and has, as the challenge description says, the goal to log into the web app
- From the code we can see that the username is admin and the password is randomly generated so we probably need a bypass and cannot guess it

2. Solve
- Looking through the code we can see the following steps during authentication:
  + Check if the password and user parameter are set in the front end
  + Post to */api/login* with Username and Password as json-Data
  + Check if */api/login* recieved a not-empty username and password
  + Check find the user with the provided password in the database and check if the database-answer is not empty
- The function appends the username and the password to a selector

```
const options = {
            selector: {
                username: username,
                password: password,
            },
        };
```
- Reading the Couchdb-docs we can find a bypass to that check
- We set up a burp proxy and route the traffic through that to intercept the api-request
- Then we change the json-payload to the following and we can bypass the login to read the flag at the */overview* Page
```
{
"username":"admin",
"password":
    {
    "$ne":""
    }
}
```
- This works because the database now checks if a user with the username: admin and a password that is not equal to "" exists. Obviously the password differs, so the user is logged in as Admin  