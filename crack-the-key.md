# crack-the-key

## Overview

We're given a RSA public key (pub.pem), an encoded flag file (flag.enc), and a python script that was used to encrypt the flag file and provides a decrypt function.

## Solution

This was my solution. I'm really interested in any faster solutions, because some teams solved this in like 7 minutes.

### Get e and n (and other info) from the public key

```python
# requires pycryptodome to run
from Crypto.PublicKey import RSA

public_key = RSA.import_key(open('pub.pem', 'r').read())

n = public_key.n
e = public_key.e

print(f"n: {n}\n")
print(f"e: {e}\n")

# factors of n from factordb.com (website provided by challenge description)
p = 106824314365456746562761668584927045312727977773444260463553547734415788806571
q = 109380489566403719014973591337211389488057388775161611283670009403393352513149

print(f"p: {p}\n")
print(f"q: {q}\n")
totient_n = (p - 1) * (q - 1)

d = pow(public_key.e, -1, totient_n)
print(f"d: {d}\n")

e1 = d % (p - 1)
e2 = d % (q - 1)
coeff = pow(q, -1, p)

print(f"e1: {e1}\n")
print(f"e2: {e2}\n")
print(f"coeff: {coeff}\n")
```

### Getting the private key from these numbers

I searched for how to get a private key from the integer d (representing the private key), and I found https://stackoverflow.com/a/19855935/14095682.

```
asn1=SEQUENCE:rsa_key

[rsa_key]
version=INTEGER:0
modulus=INTEGER:187
pubExp=INTEGER:7
privExp=INTEGER:23
p=INTEGER:17
q=INTEGER:11
e1=INTEGER:7
e2=INTEGER:3
coeff=INTEGER:14
```

Except, we replace it with the numbers we found with the python script above.

asn1=SEQUENCE:rsa_key

```python
[rsa_key]
version=INTEGER:0
modulus=INTEGER:11684495802889072585203310515250083572285658052270998153007378254694580706620837521287604089276341404868210594675627429508088431073125103913482926295102079
pubExp=INTEGER:65537
privExp=INTEGER:9502600695762465168053942767479655832876898370813310504861990228374299358868616133592676596783265831918643917621798668021116791177628509774827019838861193
p=INTEGER:106824314365456746562761668584927045312727977773444260463553547734415788806571
q=INTEGER:109380489566403719014973591337211389488057388775161611283670009403393352513149
e1=INTEGER:46146499900826189052570999577458070705840086954605497016702999678182483130183
e2=INTEGER:4142429087443948160507262366281011775170643131970507029709461271331038969401
coeff=INTEGER:59439371951810104817485400014452457559729940472205620133799763087399217636726
```

I saved the file above as a text file (private_key.txt), and ran the following command from the stackoverflow post:

`openssl asn1parse -genconf private_key.txt -out newkey.der`

And then to convert that key into PEM format, I ran  
`openssl rsa -in newkey.der -outform PEM -out private_key.pem`

Then, following the commands from https://stackoverflow.com/a/50416013/14095682, we can decrypt `flag.enc`.

`openssl pkeyutl  -decrypt -in flag.enc -out FLAG.txt -inkey private_key.pem`  
Note: I used pkeyutl instead of rsautl because command line said it was deprecated.

Uh oh! We get an error:

```
Public Key operation error
4037DF2D027F0000:error:0200006C:rsa routines:rsa_ossl_private_decrypt:data greater than mod len:../crypto/rsa/rsa_ossl.c:406:
```

I was lucky that stackoverflow post also included a fix!

We decode the file from base64 with `cat flag.enc | base64 -d > flag_raw.enc`

And we use `openssl` to now decrypt `flag_raw.enc`

`openssl pkeyutl  -decrypt -in flag_raw.enc -out FLAG.txt -inkey private_key.pem`

And now we see the flag in FLAG.txt!

```
dam{4lw4y5_u53_l4r63_r54_k3y5}
```
