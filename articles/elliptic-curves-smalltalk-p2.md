# Elliptic curves smalltalk #2 #

Consider [Diffie–Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) key exchange.

[DH](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) uses final field. For example <img src="https://latex.codecogs.com/gif.latex?\{1,2,3...238\}=\{1,g,g^2,...&space;g^{238}\;\}(mod\;239)"/>

user <img src="https://latex.codecogs.com/gif.latex?A"/> select <img src="https://latex.codecogs.com/gif.latex?x"/> in <img src="https://latex.codecogs.com/gif.latex?\mathbb{Z}"/> as a key

user <img src="https://latex.codecogs.com/gif.latex?B"/> select <img src="https://latex.codecogs.com/gif.latex?y"/> in <img src="https://latex.codecogs.com/gif.latex?\mathbb{Z}"/> as a key

int this case key exchange will looks:
user <img src="https://latex.codecogs.com/gif.latex?A"/> send <img src="https://latex.codecogs.com/gif.latex?g^x"/> to <img src="https://latex.codecogs.com/gif.latex?B"/>
user <img src="https://latex.codecogs.com/gif.latex?B"/> send <img src="https://latex.codecogs.com/gif.latex?g^y"/> to <img src="https://latex.codecogs.com/gif.latex?A"/>

their public key is <img src="https://latex.codecogs.com/gif.latex?g^{xy}"/> = <img src="https://latex.codecogs.com/gif.latex?g^{x^{y}}"/>. How do they calculate it?
user A takes <img src="https://latex.codecogs.com/gif.latex?g^{x^{y}}"/> and vice versa.
there we see that for finding a key, cursed user <img src="https://latex.codecogs.com/gif.latex?C"/> should solve discrete logarithm task.

where here we can find an elliptic curve?
we should replace exponentiation with multiplication. each participant selects some number <img src="https://latex.codecogs.com/gif.latex?n"/> and to summarize a point <img src="https://latex.codecogs.com/gif.latex?p"/> (on an elliptic curve) with itself <img src="https://latex.codecogs.com/gif.latex?n"/> times.
the point is already known, also known that it produces big enough sub-group in elliptic curve points group. Each user sends his <img src="https://latex.codecogs.com/gif.latex?n"/> to the participant and keeps <img src="https://latex.codecogs.com/gif.latex?p"/> in secret. 
Consequently, a public key will be calculated as summarizing the point from the other participant with itself his <img src="https://latex.codecogs.com/gif.latex?p"/> times.

## Encryption ##

### [El-Gamal](https://en.wikipedia.org/wiki/ElGamal_encryption) system on EC ###

Suppose we have the pre-defined curve and a point. Curve, as known, is over some final field. 
User <img src="https://latex.codecogs.com/gif.latex?A"/> wants to send a message to user <img src="https://latex.codecogs.com/gif.latex?B"/>. Assume message is also a point on a curve. 
- <img src="https://latex.codecogs.com/gif.latex?E"/> elliptic curve over <img src="https://latex.codecogs.com/gif.latex?F_p"/>, <img src="https://latex.codecogs.com/gif.latex?P\inE(F_p)"/>
- <img src="https://latex.codecogs.com/gif.latex?d_A"/> - private key <img src="https://latex.codecogs.com/gif.latex?A"/>
  <img src="https://latex.codecogs.com/gif.latex?Q_A"/> = <img src="https://latex.codecogs.com/gif.latex?d_aP"/> - public key Assume
- <img src="https://latex.codecogs.com/gif.latex?M"/> message
- <img src="https://latex.codecogs.com/gif.latex?k"/> random number

Encrypted text from <img src="https://latex.codecogs.com/gif.latex?B"/> to <img src="https://latex.codecogs.com/gif.latex?A"/> will consists of the two parts:
1. <img src="https://latex.codecogs.com/gif.latex?C_1"/> = <img src="https://latex.codecogs.com/gif.latex?kP"/>
2. <img src="https://latex.codecogs.com/gif.latex?C_2"/> = <img src="https://latex.codecogs.com/gif.latex?M"/> + <img src="https://latex.codecogs.com/gif.latex?kQ_a"/>
receiver:
<img src="https://latex.codecogs.com/gif.latex?C_2"/> - <img src="https://latex.codecogs.com/gif.latex?d_AC_1=(M&plus;kd_aP)-d_AkP=M"/>

It means anyone can encrypt and send a message, but the only receiver who has both keys can decrypt it. 
Real life example is the post office. You came to send a post to your aunt. You are able to put your envelope into the box, but you cannot open it.

### ECDSA ###

[Elliptic curve digital sign algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)

- <img src="https://latex.codecogs.com/gif.latex?E"/> - elliptic curve over <img src="https://latex.codecogs.com/gif.latex?F_p"/>, <img src="https://latex.codecogs.com/gif.latex?P\inE(F_p)"/> - point order <img src="https://latex.codecogs.com/gif.latex?n"/>
- <img src="https://latex.codecogs.com/gif.latex?d"/> - private key <img src="https://latex.codecogs.com/gif.latex?A"/>
  - <img src="https://latex.codecogs.com/gif.latex?Q_A"/> = <img src="https://latex.codecogs.com/gif.latex?d_AP"/> - public key <img src="https://latex.codecogs.com/gif.latex?A"/>
- <img src="https://latex.codecogs.com/gif.latex?k"/> - random number

User <img src="https://latex.codecogs.com/gif.latex?A"/> wants to send message <img src="https://latex.codecogs.com/gif.latex?M"/> to user <img src="https://latex.codecogs.com/gif.latex?B"/>
<img src="https://latex.codecogs.com/gif.latex?A"/> calculates 
hash <img src="https://latex.codecogs.com/gif.latex?e=H(m)"/> and for the random <img src="https://latex.codecogs.com/gif.latex?k"/>

<img src="https://latex.codecogs.com/gif.latex?kP=(x_1,y_1)"/>
Sign:
<p>
<img src="https://latex.codecogs.com/gif.latex?(r,s)"/>,&nbsp;&nbsp;<img src="https://latex.codecogs.com/gif.latex?r=x_1\;mod\;n"/>,&nbsp;&nbsp;<img src="https://latex.codecogs.com/gif.latex?s=k^{-1}(&plus;d_Ar)\;mod\;n"/>,&nbsp;&nbsp;<img src="https://latex.codecogs.com/gif.latex?r"/>,&nbsp;&nbsp;<img src="https://latex.codecogs.com/gif.latex?s\neq0"/>
</p>

User <img src="https://latex.codecogs.com/gif.latex?B"/> checks the sign:
- <img src="https://latex.codecogs.com/gif.latex?u_1P&plus;u_2Q_A=(x_2,&space;y_2)"/>
- <img src="https://latex.codecogs.com/gif.latex?u_1=s^-1e\;mod\;n"/>, <img src="https://latex.codecogs.com/gif.latex?u_2=s^-1r\;mod\;n"/>
- <img src="https://latex.codecogs.com/gif.latex?if(\;x_2\;mod\;n)=r"/> then the sign is valid

## Elliptic cryptography pros and cons ##

pros:
- for the same security level, using elliptic curves algorithms require key with significantly less length.
4096 bit's RSA' key equal 313 bits in EC system. Performance improvement here takes place because despite each operation consumes more calculation, a key length much less.

cons:
- RSA has been analyzed much more times.

Common problems usually lay in the implementation:
- bad curve
- bad point
- bad random numbers generator (true for the RSA as well)

Let's consider another cases of using EC.

## [Lenstra's Factorization](https://en.wikipedia.org/wiki/Lenstra_elliptic-curve_factorization) ##

Assume we have number <img src="https://latex.codecogs.com/gif.latex?N"/> which we want to factorize. This algorithm works fine if we suppose <img src="https://latex.codecogs.com/gif.latex?N"/> has not so big prime divider. In this algorithm, we consider not the EC over final field, 
but the set of points which are satisfied the same equitation as an EC (coordinates are <img src="https://latex.codecogs.com/gif.latex?mod\;n"/>).
We take some threshold number <img src="https://latex.codecogs.com/gif.latex?B"/> and declare that we will find any <img src="https://latex.codecogs.com/gif.latex?N"/> divider lesser or equals <img src="https://latex.codecogs.com/gif.latex?B"/>.
Calculate <img src="https://latex.codecogs.com/gif.latex?mod\;N"/> point <img src="https://latex.codecogs.com/gif.latex?Q"/> which is point <img src="https://latex.codecogs.com/gif.latex?P"/> <img src="https://latex.codecogs.com/gif.latex?j"/>-times summarized with itself. It means at the end of the loop we want to calculate <img src="https://latex.codecogs.com/gif.latex?B!P"/>.
Why do we need this <img src="https://latex.codecogs.com/gif.latex?Q"/>? If we will try to calculate the following <img src="https://latex.codecogs.com/gif.latex?j((j-1)!P)"/> we can find the case with illegal summarize operation. It means we found some number immutable <img src="https://latex.codecogs.com/gif.latex?N"/>. Probably it's multiplicator.
If true, we should take a new random EC (point) and repeat algorithm. But if it is the lesser common divider - we found <img src="https://latex.codecogs.com/gif.latex?N"/>'s divider (<img src="https://latex.codecogs.com/gif.latex?j"/>). <img src="https://latex.codecogs.com/gif.latex?j"/> might not be prime.
The problem causes if <img src="https://latex.codecogs.com/gif.latex?N"/> is a multiplication of two big primes (~ <img src="https://latex.codecogs.com/gif.latex?\sqrt&space;N"/>).

- take <img src="https://latex.codecogs.com/gif.latex?N"/>
- take random <img src="https://latex.codecogs.com/gif.latex?a"/>, <img src="https://latex.codecogs.com/gif.latex?b"/>, <img src="https://latex.codecogs.com/gif.latex?A\;mod\;N"/>
- take <img src="https://latex.codecogs.com/gif.latex?B\equiv&space;b^2-a^3&plus;Aa\;mod\;N"/>
- <img src="https://latex.codecogs.com/gif.latex?E:Y^2=X^3&plus;AX&plus;B"/>

  - for j=2,3...treshold
  - <img src="https://latex.codecogs.com/gif.latex?Q=jP\;mod\;N"/>
  - fail? means <img src="https://latex.codecogs.com/gif.latex?d>1"/> and divider <img src="https://latex.codecogs.com/gif.latex?N"/>
  - <img src="https://latex.codecogs.com/gif.latex?d<N"/> - victory
  - <img src="https://latex.codecogs.com/gif.latex?d=N"/> - should re-generate all and do it once again

Average working time: <img src="https://latex.codecogs.com/gif.latex?O(e\sqrt{2(log\;p)(log\;log\;p)})"/>  


----------


**Useful links**:
- [https://csrc.nist.gov/csrc/media/events/workshop-on-elliptic-curve-cryptography-standards/documents/papers/session6-adalier-mehmet.pdf](https://csrc.nist.gov/csrc/media/events/workshop-on-elliptic-curve-cryptography-standards/documents/papers/session6-adalier-mehmet.pdf)
- [http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)
- [https://en.wikipedia.org/wiki/Dual_EC_DRBG](https://en.wikipedia.org/wiki/Dual_EC_DRBG)
- [https://www.microsoft.com/en-us/research/wp-content/uploads/2013/11/734.pdf](https://www.microsoft.com/en-us/research/wp-content/uploads/2013/11/734.pdf)
- [https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
- [https://en.wikipedia.org/wiki/ElGamal_encryption](https://en.wikipedia.org/wiki/ElGamal_encryption)
