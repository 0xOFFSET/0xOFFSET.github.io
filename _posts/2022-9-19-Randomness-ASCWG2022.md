---
layout: post
title: Randomness Cryptograpgy challenge | ASCWG CTF 2022
tags: CTF Cryptograpgy
---

It's an amazing and awesome experience to join CTF competitions, it's honor to participate in Arab Cyber Security Waregame 2022.
This's a quick walkthrough of "Randomness" cryptogrpahy challenge we've solved.

I was amazed by the level an ideas of the challenges, by the author Ahmed Sayed @Y4mm1.

```
Level: Medium
Points: 600
Category: Cryptography - Digital Signature 
```



It seems Elgamal Signature scheme , so let's figure out how it works at a glance and what're its weaknesses in order to reveal the flag.

## Elgamal Digital Signature Scheme
Elgamal DSA is a digital signature scheme that relies on the complexity of computing discerete lograrithms and the algebraic properties of modular exponentiation.

The algorithm uses a randomoly choosen private key to sign the message, and public key for verification, so let's declare its operations into four steps and know about the parameters of the algotithm.

1- Key generation
- choose prime number: p of the same key lenght: N
- choose a cryptographic function: H
- choose a generator g < p of the multiplicative group of integers modulo p, (Z*)p.

Hence, the shared parameters of the the alogrithm between users & systems envolved are (p, g) (which are found the challenge code).

- choose private key x randomly from {1...p - 2}
- compute the public key y := g ^ x mod p

Broadly speaking, the private and public keys are (p,g,x) and (p,g,y), repectively.


2- Key distribution

3- Signing
- choose integer k randomly from {2...p - 2} with k relatively prime to p-1. (It's recommended not choose the same k with different signing operations, we will figure out why! Randomness!? )
- compute r := g ^ k mod p
- compute s := (H(m) - xr)* (K ^ -1) mod (p-1), where s shouln't equal 0.

Hence, (r, s) are the generated signature for each message, keeping in mind that r will be fixed for different signatures in case of relying on the same parameter k.

4- Verification
- verify 0 < r < p and 0 < s < p - 1
- siganture is valid if and only if g ^ H(m) congurent (y ^ r) (r ^ s) mod p

## Security Concerns of this algorithm
In fact, in case of the leakage of the private key or even - in a diffcult scenarior - a collosion occurs in the hashing functions used, attacker can forge the users' signatures.

A one famous case that may be a big security concern, is using the same k for different signing operations, as attackers can deduce the private key from a single signature, hence he/she will sign messages on the behalf of the key owner.

## Solution
Refering to the challenge name, the security concens above and the challenge itself, let's try to abuse the fact of using fixed k value in signing.

By looking at challenge attachments and included script, we figure out:
- given parameters are p,g
- x and A are the private and public keys, respectively.
- the output.txt file contains a list of signatures. As we know signaturs should be the (r,s) pair, but interestingly, it includes multiple s values and a signle r value at its most last line.
- the line s = (pow(self.k, -1, self.p-1) * (m - self.x*r)) % (self.p - 1) indicates no use of cryptographic hash function, but just the plain message.
- there's a single messsage "I wanna look closer, but my glasses are broken!", we should found its singature in output.txt file.


Firstly, let's hopfully reveal the k parameter relying on disceret logarithm problem by searching for any online calculator (I found this helpful: https://www.alpertron.com.ar/DILOG.HTM), where:
- exponent is k
- Power is r value
- Base is g parameters

By submitting the parameters and giving the tool some time, you'll find the k value plus some remainder.
k = 45252513931146808668347235459955660091


Then, it's time to show how we will get the flag, using the list of singatures and the k value.

Suppose we have two singatures (s1, r) & (s2, r) and two messages m1 & m2, let m1 be the message included in the code and mark the flag as m2. We'll suppose that both the signatures of m1 & m2 found in output.txt.

Since, s = ((m - xr) * (k ^ -1)) mod p-1

when we subtract s2 from s1 we have: 
(s1 - s2)k = (m1 - m2) mod p-1

then the message (flag) m2 = ((s1 - s2)k + m1) mod p-1  

since, we don't know which s values map to each messages, let's try all combinations of s values and filter on messages starts with the flag schema.

Finally, the flag is **ASCWG{H17719_7H3_W0r1d_0f_W33k_Pr1m3_E1G4m@1_n47ur3s}**

```
# Challenge Code

from Crypto.Util.number import *
from Crypto.Cipher import AES
from hashlib import sha256
import os 
from secret import plains

plain = b'I wanna look closer, but my glasses are broken!'
class GDSA:
    def __init__(self):
        self.p = self.genPrime() # 71500532584610353335967572228576961622670888049686244244336530474941226340958441170979183862795117652159440106046491327947803659173888001
        self.x = getRandomInteger(256)
        self.g = 7
        self.A = pow(self.g, self.x, self.p)
        self.k = getRandomInteger(128)
        while GCD(self.k, self.p-1) != 1:
            self.k = getRandomInteger(128)

    def sign(self, msg):
        m = bytes_to_long(msg)
        r = pow(self.g, self.k, self.p)
        s = (pow(self.k, -1, self.p-1) * (m - self.x*r)) % (self.p - 1)
        return r, s

    def genPrime(self):
        p = 0
        while not isPrime(p+1):
            p = 2
            for _ in range((2<<2)):
                p *= getRandomInteger(50)
            p *= getRandomInteger(64)
        return p+1

obj = GDSA()
sigs = []
for plain in plains:
    r, s = obj.sign(plain) 
    sigs += [hex(s)]
sigs += [hex(r)]
with open('output.txt', 'wb') as f:
    for s in sigs:
        f.write(s.encode()+b'\n')



        
```

Here're the list of signatures provided (output.txt):
```
0x3ada728da2811b7928cb46cb97d9d905d6ef274f2535a361a549dcd99d26461d7e88ea5178c374339e0ada83487b8a1fc8ef6aea65e3f24b61
0x133c7655470ef0b4d0bb6bcbe741117f1b372e14adc69211a6ef6fbc8fd8d3de0ba19a2214b64876b058cad79707b6f127fadbe16e906f18c8
0x589645936cac16c433f6c15a6929f222d5b58f140009e43a1766cc56a3b5ec859529f3220115302acb51186d5e5e308a60b8891f888fafa552
0x3f01b0e1b8cbc35826bf3948404a271e22a3bb47421d4d63efffaffa8da780bfa9eebe9419d40dde38ae22d74ce69949397f567212f8e98b5f
0x5c2fef672a2ff68fb98dc9b1abd54e8495d041e4ef68fd474ca2557a8e09bfd56544760fd49cdc2fccb67b991d9b4556feda56e576b3d52c1f
0x552cef670becfcc047559f4585e92c0ba21f8d6edb161015837e36747edac0b88c73c77459539071dbe9b74111447d208a0f224b92e6d2aaab
0x20aef033f58ec4b5648fe67e4959273bea4419a150777b70fa703b33653a0e45be7a8af2bba90685df65e9db0a7326c5250ffb407698ae4bee
0x60b81999abbff2d985939740ab3b0fab8610da9a8be853769e3814a62aa422f59edd3c5c811e2a4e3ef0e9b4ebbdfdbc02e9c3a0217978d4c6
0x190cdad9d08c7180248833188c9913620ae411386a27c457e3f94154ddf86ff4fa4a59f0dc82abc18fedf5e8cc8dd4702fce2ce70a633fdc56
0x31afb0bfd0a2ba26cff6f09e32581fb15b6a4d3f6dfdaa2888e387ca56e3775b4a2bd62c7824296a728956bfdd043cea6a50588ee6f2d724a1
0x5c3cfc2062451485bbd4819344d70d85f4dd876aba328414886d6ae532f29f00d565eb7f9e075b12991d8ec94eb301a960a7d8c3472063c68b
0x3b8ac578f909a2ff300ace11fe0a5224cdb2606a23120f283d0df09001ccfb543d305dca1ef6a658ac924cb3a2f433cc0e936c83c512cd1ea2
0x50c32c0be450ad8b3047fb3052733c13d41b048d596100e99eefb850cc8709da0d985882c472ae1940ead9e536010c06c71785de4238fb1509
0x50047f7c26727abf1e0d46ce7afcae9fffd44403cf9af2c297003efee2a1a2751ab33c4c527f5d53ebd11cc8d34c74e8c59ca070bb1c3b0b12
0x4293b32d930dfd1dcd9f2c6346a0f8a39617371ae75076cf9b7966f401be1d335b4a3ddbc730cf6c3a7084c7a14e5fe81ff7a4936d071c0869
0x817d304ead8f4fc52143fb53fcd23b69b4f4cba4d27db0a833e653f3850edf456b29d81f46eb59aa9a026dbc614979ddcea233246d80994fa
0x61be68c9a940bb74dc3eae55d55879c3b876945c77110d984896bc6868848acfa8988726b556784a09db9c5332178804c5504cf2f60fba47f1
0x857466f1fe6eac21ba514c80b93c19bdeed088c005889f7fd91fc18d4712c10ac9f76873d2a9c1fd1e95224c97c817308cca378f81aabf503
0x243fe76954cfd0c536a287ddeba080be0fea25a0f19597a99004646c23ea80954ff2cb9024668454163e7a53f45b49e442f8751b8b61d9da38
0x257bbb1484d3a950c221c76f02b861ef8d223f18affb6a947ad0c5560faaf64aeec737424f8fa77a29fc09f3328b6036aec90a41db0a7d6435
0x354f3ada1bf3e9e3f383695c3cdea2afbcc63c66536d44f56c2c09e66eded8b162857df5f4e8e76635d57fcbf17fb7197c83ec3c2b4db26361
0x483083a3e84df868baccf526c8302dc0e3f58602af0fb7fb94b8d8d7477b35e3f7ce02ad4c2e4bc1c7ba32d6ca91373df5580d0a1416047eea
0x4bd2facae08261f8759378f48f4093edef5bccf043a4370fd2941ad77b159d33543acc28b29a4166e337f02ed3a22436abe4f757ef0d7a9712
0x37cc8538021af3504665000ce301d4dea569cae07e6882e4b9feb50b1c4d37da8f5c8a38ce4a594e531fb8c4d07e499a56474ea1ad13a936a8
0x1b0949857cad14f92a70fea559d50c7d6160a2f1421690a56c5d19b467b4e6d872b1487207f20f995f9b9e96ebb45ded99b6261aec6500781c
0x5835f23fb87e3b264acb75f9b31ed54f9bfd818529e617e963c89f7d3b02e92e85d07d038b9b465f95075bdef328f82be7a13bfb0d5f816896
0x54e66ca98919d363547808527d26fff37a6052db8e5803566ebfb685f187c2d5fb176195f59b001225fc51aad79e91fecf271a01f209ce5d61
0x1f60aa92a64dff1a85dd1831f2c4de783505ce20e7f3ffed9607ac1a9a9b5df659311a3657f2091e3852eb2525367886ac99d75e696331c807
0x109c5d70b578aa209d21666389a6028f0ac86f88dc85c1ffd6aa5b54d3c121dfb5041fe432b5466a3a9326a0e74f549ebf53d044f3076c0ea9
0x4209725ab68ab1820c3826756aee8d41c86f75c17b6615f7efa630880841df066f4b6ecbf073af55a6c331acb583d06b50cfb9832f1b0485ca
0x6073a322142fb76fb978e96aab58f0baabdf71d79fbb20a37c5533d32fd73c20d1e567d09456410a4433d3d60c779b70dcad52db471ce2b300
0x159437291a1b3abb1d7b956feb58e3e91a8d27c4b1d8f94e67f5e71d98568917eb8681dc3bf60ff7009037bcf708b5a3105bee569baf218696
0x52e833262380224f2581ff9df1790ac66736ee763b7cc02cd2bd8cdf3f9ae3c640359caa40182f8eb6f5997f45dba2c1eefcdb5f73193050cf
0x43b4b54b4889e59a8219ccf1547c47127282cb830de115bce4e703ec4603d8ea3157694ce1272ab3a92807eab5d4df9a0fdbaee7700789faeb
0xcc515038cd46814fe859ee9ef24464794f628297d43de6f9ec9797d880d151500487a6cc59cc4cef4aea78cad40c2d8139b652a729f23a136
0x119e7c775b8882a64634165d1fee9ee429bce9a5391bcbb12227f03985cb6470305701c61dec78c11851da3002dad4d88ff7491ff3bc4e3434
0x368987c5d91cd332d7323d454220c5efda081c8acfc162bfc6d5ef8dfd6a7137f27765399202e4a4b7982671f64596847810f8088a8842fd24
0x19c5f6564693e04d213ddbabf3f7ebe71ab76f709cb4c2142a64649782c78519b4c048a11859196592575d92f184efd84b9680256cbbb435f8
0x30c48b7b506dfa3212e101fe8eebff069cafbd6f920a5dd03661107206f885b0a8ebdbb57273e72900bfce4b1a2751efdbb33a0493041fcbc6
0x234271b0a070f0c13413d906f59d9341299459482597374922a3b965f06807bbea36377b6e17e5e0ae29a509b0e6dca7372e86a926136f436e
0x1556b2739f5607c2297e5e6694d12092e9a492a242c9be7bc584957ea7908124bb43351cf1da7782c0be8ee1a5c261cece311fa0bb593e51e1
0x2cdc30475417f062cb1647fe84086d8f90eb1d23c926169f3c13405d15a84ab906d201ef5e10d8b56342e430418631237a0a283f35b950adb5
0x5839e36d3ea5d7912eb9ae0360f173a8b13f37684eea9320ca548e4f952e1b1578f9ccac5d6be698e18b6d764916e18a5b107291b87c4412e1
0x27ac1316bc1171a2ced5c4e9796f7101159d40665dd3e87caf97c5a27d7df77f9752e1cfd88b4059dffa92c3ad6111ea68b332f3a7ae27fbb4
0x14e36bfd9e6072d945ea5bdf3aa62ed0df5389e1d256c3181866f61922ffcf3b35c674e6cf6a415c6895b4d463bb100f8d0ce11d938a6cb1b2
0x18bc07d7ca479dd942dc07798f79782df649a7d2e15a36e12a630b906d92c18b09b8dcb4df3f863c0bd3562f4743ac3c39cbca0ef90c483b0
0x16682d13957b27df995e90c72342ff7768a6528f38fe09af9cd72797ee12a507b6e64f7bbe3c56d857f1ba94deac252a530234fbe834b17d69
0x4f80629ea217d5e41e62a1bf32b4e8dcb2149d0b37f4263cb61f2003cc24e0c08cbe76754c56f9174477c2b547c03ae1d7e272bb9ac9d23c06
0x62378bed1371ea369cb3a212a67ac7df1c5caf3495a2234bbfff6ba775c28bb7f39d6420c2e54b7c0cd059acbf3be63c1804ce5efd8be8a96f
0x2a10bb69276ef92a8d658c6df833197bc2716fee89c44671f963a20e497389ede0c6cc1f40fb5cf9b9c47edb24612a37c69407a472d2a7347e
0x4f52ae16db8fe960ff478b2d213475a1e6c4a878666ff8f3fb3b2d03254d57d75229bafaf0fa89fa3b9322b316cd6440cb07cc7f044f553dfb
0x4c4de74de95d947128eb0168dcc8d5fd70c2b8fe674750300d4a116a9f18abc2fffd2801f921c46770b40bc0ea79f467f2ba78f9ae892e2bb8
0x359241cd9521c58ca2fb0026b0e6741b37f7d5e23d773f1ae2405825cce40fc299d254a4688cfa904784817a0c7b92a42430f918c81f531c8b
0x3a0cf5f21c3f1d43bef9470d40a788c14aafe57afcf0fdddbcf28afb5774133d26896ba4edda8c0662e8b4d3081655a08fd5cdbc0b17ce6f12
0x293eb00e25cd6be2f2be007ea2d37bbc18197f0a523bee692978f58760eb7325863f9bdebaec7fa4d5cc3406dc6a0b458464853e843f75d477
0x1534e712fb505fa9fc1973febb8210f13fb91437b762bcab2764363ada278d2d194399da0bf49cdb7f05e2f93a34266c69c9fbe48bbc00c22b
0x57bd991d7f8d4733cc44bf3651f4c6c81c64dda49c6d5820029b4fd5e8824dc615a04d8e387db703e73a92ad79211d92699180009ee1ea9676
0xe9868ba16183794ef1be0024967044e043fc3929cba55794935db18a413e7a3480ed7b79bdabe6fe2e3f9121e88bd6fb6564b00d0d88fd3cb
0x3e1cf6ca6d58e027eb5f15f17fb6c368679fc8535fe40f8529267da59e5acce744c6acc0e04baf8c06977fa707566d6daefdd0facce7a45282
0x12cb28320166c148f803e564099aa65fbe337d7e1c58d69d1799f8214ea4c30f47584d95082eb71d41c89f54d8f7648b4a42ab3c8582b06cd3
0x1d5098ec29872c91c538afebe682c25e9887de64ad9b9567b0843cdec3770c5736704077325954bc2eccd1967c2a19c099038bdee021fcf9bd
0x2d378c8909db8d97c288e7690e8f80d103880012d684ea169c44aa5fbde1e8b2522aeff3b4a8f8fbca6b6aacc6638fd21668a057400f91211a
0x14e2b92fcd9f380da78b7d12d57a64b2bd62338051afe6e70b1937cdd441231e12f66d49c63b7234213f7d504924c715c671e80c96df3a5bee
```

Happy reading.




# References
- https://crypto.stackexchange.com/questions/1479/elgamal-signature-scheme-recovering-the-key-when-reusing-randomness?rq=1
- https://en.wikipedia.org/wiki/ElGamal_signature_scheme
- https://www.doc.ic.ac.uk/~mrh/330tutor/ch06s02.html#:~:text=The%20discrete%20logarithm%20problem%20is,logarithms%20depends%20on%20the%20groups.
- https://math.mit.edu/classes/18.783/2022/LectureNotes9.pdf
- https://www.alpertron.com.ar/DILOG.HTM
