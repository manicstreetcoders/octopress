---
layout: post
title: "RSA Sample Number"
date: 2016-04-26 15:24:39 +0900
comments: true
categories: 
---

RSA sample number.

``` 
# 두개의 소수를 정한다. 
p = 3
q = 11

# 소수의 곱으로 큰 수를 하나 만든다.
N = p*q

# 이제 이 큰 N 의 order 를 구한다.
z = (p-1)*(q-1)


# 이제 소수를 하나 정해서 개인키로 쓰다. (실제로는 coprime to z 이기만 하면 된다.)
e = 7

print "z: %d" % z
print "N: %d" % N
print "e: %d" % e

# e 의 인버스를 구한다. 인버스라 함은 곱해서 1 이 되는 수. (모듈로 z)
for d in range(1,z):
    if (e*d) % z  == 1:
        print "Found d!!!: %d" % d
        break

# e is the private key (in this case, 7)
# d is the public key (in this case, 3)

plain = 2
cipher = pow(plain,e) % N # 암호화: (plain)^e mod N
print "Encrypting %d -> %d" % ( plain, cipher )

plain = pow(cipher,d) % N # 복호화: (cipher)^d mod N
print "Decrypting %d -> %d" % ( cipher, plain )

# $ python rsa.py
# z: 20
# N: 33
# e: 7
# Found d!!!: 3
# Encrypting 2 -> 29
# Decrypting 29 -> 2
```
