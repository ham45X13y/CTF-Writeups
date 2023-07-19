The Challenge:
- This Challenge is about a AES-CTR KEY-Reuse Attack
- AES-CTR works with the key and then xors it with the message to get the encrypted message
- We were given the original message in four blocks and the encrypted with also four blocks
- to solve this we can use some xor magic because :
1. same xor same = 0 
2. Xor is interchangeably
3. This results in ```a xor b xor a = b```
- We can set up the following equation to solve the challenge:
  + original message: m
  + key: k
  + original flag: f
  + encrypted message: em = k xor m
  + encrypted flag: ef = k xor f

```
em xor ef xor m
= k xor m xor k xor f xor m
= f
 ```

This is the original code:

```
import binascii
from pwn import xor


originalMessage=[
    b'This is some public information that can be read out loud.',
    b'No one can crack our encryption algorithm.',
    b'HTB{?????????????????????????????????????????????}',
    b'Secret information is encrypted with Advanced Encryption Standards.']

output=["76ca21043b5e471169ec20a55297165807ab5b30e588c9c54168b2136fc97d147892b5e39e9b1f1fd39e9f66e7dbbb9d8dffa31b597b53a648676a8d4081a20b",
"6ccd6818755214527bed6da3008600514bad4d62ac83c1c9417ca3136fc97d146d96b3f8cc910a199ed2fc4093b8dcff",
"6af60a0c6e5944432af77ea30682076509ae0873e785c79e026b8c1435c566463d8eadc8cecc0c459ecf8e75e7cdfbd88cedd861771932dd224762854889aa03",
"71c72b057e43145874e522b21f86175304ac1879ffc6cac45077aa1772c377147b93a0ff9eb91a0792929923f19e9f97cee2af1f0d7e53bd0c1a18ea28e3c57fd718b40f5d2c0014a3dbe6a3e5654fe8"]


binOutput=[]
for i in output:
    binOutput.append(binascii.unhexlify(i).strip())

enc_flag=binOutput[2]
enc_message=binOutput[3]
message=b'Secret information is encrypted with Advanced Encryption Standards.'

temp=xor(enc_message,enc_flag)
flag=xor(temp,message)

print(flag)
```

### Possible mitigation
- If aes in ctr mode somehow is the only viable encryption for you dont reuse the key, but change it for every useage
- Also changing the aes type to a less vulnerable mode like cbc
- Additionally if this is checked serverside calculat a MAC on it to check if someone has tampered with the encryption