---
layout: post
title: URSA Minor Cryptography Challenge | BlackHat MEA 2022
tags: CTF
---


Hi Crypto Folks,

This is writeup for Crypto Challege "URSA Minor" from BlackHat MEA 2022 CTF, it was really a great CTF, my team got the 28th place and qualified to join final round in KSA.

```
Level: Easy.
Category: Cryptography - RSA.
```


A fast exploration for the code, you will find the flag is saved into environment variable called "FLAG" on the target CTF server.
Also, the RSA implementation of this challenge is like the standard one, but differs from the generation of Prime factors.

Firstly, I've downloaded the challenge file, created fake env variable FLAG and tested the callenege locally, to easly understand how the script works.
```
$ export FLAG="BlackHatMEA{_______fake_flag_for_testing______}"
```

Let's have a close look into prime_gen function, it yields both P,Q modulus factors by generating two lists of a so-called smooth primes and multiply elements together for each list, then obtaining the prime factors by these formulas: Q = 2 * q + 1, P = 2 * p + 1.


Now, let's run the script.
```
$ python ursaminor.py
|
|  ~ Welcome to URSA decryption services
|    Press enter to start key generation...
|
|
|    Please hold on while we generate your primes...
|
|
|  ~ You are connected to an URSA-256-12 service, public key ::
|    id = 4a21cb29fd1c29a2e72a17d0202a9e26adb4fb54e881e37079827f3e7cd5cb82
|    e  = 65537
|
|  ~ Here is a free flag sample, enjoy ::
|    366683454045997880071275453810557595464925805240016257635220608467951753434957841435519660289301181545123313872084417998465869332952029193663612651
|
|  ~ Key updated succesfully ::
|    id = 4a21cb29fd1c29a2e72a17d0202a9e26adb4fb54e881e37079827f3e7cd5cb82
|    e  = 30257182642005449276899975292576172041198937532728427352786359129115764207172352786325227556490097175227043549012305391020732117816072965452259951
|
|  ~ Menu (key updated after 3 requests)::
|    [E]ncrypt
|    [D]ecrypt
|    [U]pdate key
|    [Q]uit
|
|  > 

```

From the output above, here're some observatinos:
- the N modulus is not given, but sha256 hash value instead, where it is impossible to recover from the hash.
```
"|    id = {}".format(hashlib.sha256(str(oracle.public['n']).encode()).hexdigest())
```
- both key exponents: e,d are changed at the most begining of running the script and after every 3 trials of encryption.
- the flag ciphertext is returned at running the script every time.
- the e parameter is so big after updating the key
- when choose to update RSA keys, "id" hash value remains the same, hence the public modulus remains fixed for every script running session.



Now, it's time to explore update_key() function:
```
def update_key(self):
	# Prime generation is expensive, so we'll just update d and e instead ^w^
	self.private['d'] ^= int.from_bytes(hashlib.sha512((str(self.private['d']) + str(time.time())).encode()).digest(), 'big')
	self.private['d'] %= self.private['f']
	self.public['e'] = inverse(self.private['d'], self.private['f'])
```
```
new_d = old_d ^ sha512(old_d + current_timestamp) % phi
e = new_d ^ -1 % phi
```

From equations above, we should know it's impossible to predict the old private key "d", which is by the way not the only requirement to solve the challenge but also N is required. After multiple failed trials to abuse the code above to recover the private exponent "d", I've choosen another to path for the solution.


# Solution 

After failed trials and unproven thinking pathes, the solution seems to be as follow:
1. Deduce the public Modulus N
2. Check the integrity of the modulus by computing the sha256 hash value and compare both values 
3. Trying to factor the modulus abusing the smooth prime implementation in the challenge
4. Constructing the private exponent 
5. Recover the flag

I've decided to deduce the modulus N from multiple pairs of messages & ciphertexts, after some google searches, I've discoverd that it's mathemtically applicable without even using any of RSA key exponents.

Here's how it works:
Consider you've access to RSA cryptosystem with fixed public modulus, you can choose two messages M1 and M2 then forms M1′= M1 ^ 2 and M2′= M2 ^ 2.
Attacker can construct the corresponding ciphertexts: Ci = Mi ^ e mod N (for 𝑖 ∈ {1,2,...}). Since M1′= M1 ^ 2, then C1′ ≡ C1 ^ 2 mod N , and from the modular arithmetic ((C1 ^ 2)- C1′) is a multiple of N. Similarly, ((Ci ^ 2)- Ci′) for any arbitray i value (it's better to choose i > 2).


Here's a prove of concept

```
from Crypto.Util.number import *
from math import gcd

# Construct a valid Public Modulus and exponent N,e

# I've choosen 3 messages not just 2 ,to yield a more robust result

m1 = bytes_to_long(b'first_message')
m2 = bytes_to_long(b'second_message')
m3 = bytes_to_long(b'third_message')

m1_sqr = m1 ** 2
m2_sqr = m2 ** 2
m3_sqr = m3 ** 2

c1 = pow(m1, e, N)
c2 = pow(m1_sqr, e, N)

c3 = pow(m2, e, N)
c4 = pow(m2_sqr, e, N)

c5 = pow(m3, e, N)
c6 = pow(m3_sqr, e, N)


predicted_N = gcd((c1 ** 2) - c2,
					(c3 ** 2) - c4,
					(c5 ** 2) - c6))

# now, the result shouldn't be exactly N, but may be a multiple of it.
# So, we should get rid of the small factors to recover N, or solve this by choosing more pairs of messages/ciphertexts.

```


I rewrote the script as follow to get the modulus, but I had to do it manually or to use "pwntools" while connecting to the challenge server. 

```
# import required modules

# class URSA:
#	..... 

oracle = URSA(256, 12)
oracle.update_key()

print("|    id = {}".format(hashlib.sha256(str(oracle.public['n']).encode()).hexdigest()))
print(f"|	n= {oracle.public['n']}")
print(f"|	e = {oracle.public['e']}")
print(f"|	d = {oracle.private['d']}")
print(f"|	p: {oracle.private['p']}")
print(f"|	q: {oracle.private['q']}")
print(f"|	phi: {oracle.private['f']}")


messages = [b'first_message_here', b'second_message_here', b'third_message_here']
ciphers = []


ciphers += oracle.encrypt(btl(messages[0]))
ciphers += oracle.encrypt(btl(messages[0])**2)

ciphers += oracle.encrypt(btl(messages[1]))
# exponents are updated every 3 trials
oracle.update_key()
ciphers += oracle.encrypt(btl(messages[1])**2)

ciphers += oracle.encrypt(btl(messages[2]))
ciphers += oracle.encrypt(btl(messages[2])**2)


print(ciphers)
predicted_n = gcd(ciphers[0]**2 - ciphers[1], ciphers[2]**2 - ciphers[3], ciphers[4]**2 - ciphers[5])

print(f"predicted_n : {predicted_n}")
print(f"predicted_n/N : {predicted_n/oracle.public['n']}")
print(f"Check hash of predicted_n: {hashlib.sha256(str(predicted_n).encode()).hexdigest()}")


```

After geting the public modulus, it's time to try to factor it relying on Pollard's p-1 algorithm abusing the case of using smooth primes to construct the prime factors, and finally create the modulus.

I've found "primefac" tool is very useful better than the headache of re-implementing the algorithm from scratch, here's a sample

```
$ python -m primefac 47977600676712306961271055921478510160617402720530883907904941595889755989062340127036337919920650404728693854575944287397259111710277460523610677

47977600676712306961271055921478510160617402720530883907904941595889755989062340127036337919920650404728693854575944287397259111710277460523610677: 10821917219895172982990514376546872602044138733684948945830138455832826339 4433373468105039180591838340895533737808061206923727381999961418656007943

```

Here we go, successfully got the factors p,q,  now simply consruct the private exponent easily by: 
```
from Crypto.Util.number import inverse

d = inverse(e, (p-1)* (q-1))

```

It's now your turn, just decrypt the cipher and get the flag!



```
# ~ the complete challenge code ~

#!/usr/local/bin/python
#
# Polymero
#

# Imports
from Crypto.Util.number import isPrime, getPrime, inverse
import hashlib, time, os

# Local import
FLAG = os.environ.get('FLAG').encode()


class URSA:
    # Upgraded RSA (faster and with cheap key cycling)
    def __init__(self, pbit, lbit):
        p, q = self.prime_gen(pbit, lbit)
        self.public = {'n': p * q, 'e': 0x10001}
        self.private = {'p': p, 'q': q, 'f': (p - 1)*(q - 1), 'd': inverse(self.public['e'], (p - 1)*(q - 1))}
        
    def prime_gen(self, pbit, lbit):
        # Smooth primes are FAST primes ~ !
        while True:
            qlst = [getPrime(lbit) for _ in range(pbit // lbit)]
            if len(qlst) - len(set(qlst)) <= 1:
                continue
            q = 1
            for ql in qlst:
                q *= ql
            Q = 2 * q + 1
            if isPrime(Q):
                break
        while True:
            plst = [getPrime(lbit) for _ in range(pbit // lbit)]
            if len(plst) - len(set(plst)) <= 1:
                continue
            p = 1
            for pl in plst:
                p *= pl
            P = 2 * p + 1
            if isPrime(P):
                break 
        return P, Q
    
    def update_key(self):
        # Prime generation is expensive, so we'll just update d and e instead ^w^
        self.private['d'] ^= int.from_bytes(hashlib.sha512((str(self.private['d']) + str(time.time())).encode()).digest(), 'big')
        self.private['d'] %= self.private['f']
        self.public['e'] = inverse(self.private['d'], self.private['f'])
        
    def encrypt(self, m_int):
        c_lst = []
        while m_int:
            c_lst += [pow(m_int, self.public['e'], self.public['n'])]
            m_int //= self.public['n']
        return c_lst
    
    def decrypt(self, c_int):
        m_lst = []
        while c_int:
            m_lst += [pow(c_int, self.private['d'], self.public['n'])]
            c_int //= self.public['n']
        return m_lst


# Challenge setup
print("""|
|  ~ Welcome to URSA decryption services
|    Press enter to start key generation...""")

input("|")

print("""|
|    Please hold on while we generate your primes...
|\n|""")
    
oracle = URSA(256, 12)
print("|  ~ You are connected to an URSA-256-12 service, public key ::")
print("|    id = {}".format(hashlib.sha256(str(oracle.public['n']).encode()).hexdigest()))
print("|    e  = {}".format(oracle.public['e']))

print("|\n|  ~ Here is a free flag sample, enjoy ::")
for i in oracle.encrypt(int.from_bytes(FLAG, 'big')):
    print("|    {}".format(i))


MENU = """|
|  ~ Menu (key updated after {} requests)::
|    [E]ncrypt
|    [D]ecrypt
|    [U]pdate key
|    [Q]uit
|"""

# Server loop
CYCLE = 0
while True:
    
    try:

        if CYCLE % 4:
            print(MENU.format(4 - CYCLE))
            choice = input("|  > ")

        else:
            choice = 'u'
        
        if choice.lower() == 'e':
            msg = int(input("|\n|  > (int) "))

            print("|\n|  ~ Encryption ::")
            for i in oracle.encrypt(msg):
                print("|    {}".format(i))

        elif choice.lower() == 'd':
            cip = int(input("|\n|  > (int) "))

            print("|\n|  ~ Decryption ::")
            for i in oracle.decrypt(cip):
                print("|    {}".format(i))
            
        elif choice.lower() == 'u':
            oracle.update_key()
            print("|\n|  ~ Key updated succesfully ::")
            print("|    id = {}".format(hashlib.sha256(str(oracle.public['n']).encode()).hexdigest()))
            print("|    e  = {}".format(oracle.public['e']))

            CYCLE = 0
            
        elif choice.lower() == 'q':
            print("|\n|  ~ Closing services...\n|")
            break
            
        else:
            print("|\n|  ~ ERROR - Unknown command")

        CYCLE += 1
        
    except KeyboardInterrupt:
        print("\n|  ~ Closing services...\n|")
        break
        
    except:
        print("|\n|  ~ Please do NOT abuse our services.\n|")


```


Happy reading.






# References:
- https://en.wikipedia.org/wiki/Smooth_number
- https://crypto.stackexchange.com/questions/43583/deduce-modulus-n-from-public-exponent-and-encrypted-data
- https://en.wikipedia.org/wiki/Pollard%27s_p_%E2%88%92_1_algorithm










