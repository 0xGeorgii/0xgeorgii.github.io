# Elliptic curves smalltalk #1 #

_please visit the new revision of this article [https://www.georgeplotnikov.com/elliptic-curves-smalltalk-vol-1/](https://www.georgeplotnikov.com/elliptic-curves-smalltalk-vol-1/)_


Factorization on the elliptic curves was my diploma subject, so I decided to write some short note simply describes that the elliptic curves idea. If I had this note at the beginning of my research, I would save a lot of time 😃

Elliptic curves uses are:

- Bitcoin
- SSH
- TLS
- Citizen Card (Austria)

What the elliptic curves are? Simply talk - that is the set of solutions of an equation of the form <img src="https://latex.codecogs.com/gif.latex?\inline&space;Y^{2}=X^{3}&plus;AX&plus;B" title="Y^{2}=X^{3}+AX+B" />

<img src="https://upload.wikimedia.org/wikipedia/commons/d/d0/ECClines-3.svg" />

Why they are interesting and why are they use in cryptography? The first fascinating fact is that we are able to summarize two points on a curve: if we have two points on a curve (P and Q), we may associate them to the third point and to call it as their sum. To do that we should intersect P and Q. That line will cross a curve at the only one point R, which then we will reflect R from X-axis[1]. That's it.

The problem might be if we would try to summarize a point with itself. It's not a problem, actually: that means the line becomes the tangent curve[2].

The second problem if we will try to summarize two symmetric points. In that case we assumes the solution is a point somewhere in infinity <img src="https://latex.codecogs.com/gif.latex?\inline&space;\varnothing"/>[3].

<img src="https://upload.wikimedia.org/wikipedia/commons/c/c1/ECClines.svg" />

Basically we can easily sum a points programmatically with the following steps:

- <img src="https://latex.codecogs.com/gif.latex?\inline&space;P_{1}=(x_{1},y_{1})&space;P_{2}=(x_{2},&space;y_{2})"/>
- suppose both points are not a <img src="https://latex.codecogs.com/gif.latex?\inline&space;\varnothing" />
- <img src="https://latex.codecogs.com/gif.latex?\inline&space;L:Y=\lambda&space;X&plus;\nu" title="L:Y=\lambda X+\nu" /> - line, intersected <img src="https://latex.codecogs.com/gif.latex?\inline&space;P_{1}" />, <img src="https://latex.codecogs.com/gif.latex?\inline&space;P_{2}" />
    - <img src="https://latex.codecogs.com/gif.latex?\inline&space;\lambda&space;=(y_{2}-y_{1})/(x_{2}-x_{1})" /> or
    - <img src="https://latex.codecogs.com/gif.latex?\lambda&space;=(3x_{1}^{2}&plus;A)/(2y_{1})" />
    - <img src="https://latex.codecogs.com/gif.latex?\nu&space;=&space;y_{1}&space;-&space;\lambda&space;x_{1}" />
- substitute <img src="https://latex.codecogs.com/gif.latex?Y&space;=&space;\lambda&space;X&space;&plus;&space;\nu"/> into line equtation
- <img src="https://latex.codecogs.com/gif.latex?P_{1}&space;\oplus&space;P_{2}&space;=&space;(x_{3},y_{3})" />
    - <img src="https://latex.codecogs.com/gif.latex?x_{3}=&space;\lambda&space;^{2}&space;-&space;x_{1}&space;-&space;x_{2},&space;y_{3}&space;=&space;\lambda&space;(x_{1}&space;-&space;x_{3})-y_{1}"/>
    
    as you see, we event don't need to resolve cubic equitation 😃
    Now, lets clarify the definition of the elliptic curves: <img src="https://latex.codecogs.com/gif.latex?Y^{2}=X^{3}&plus;AX&plus;B,&space;3A^{3}&plus;27B^{2}\neq&space;\varnothing"/>, plus the addition point <img src="https://latex.codecogs.com/gif.latex?\inline&space;\varnothing"/>
    
    This kind of summarizing is very useful and looks similar to sum the numbers:

- <img src="https://latex.codecogs.com/gif.latex?(P\oplus&space;Q)\oplus&space;R=P&space;\oplus&space;(Q&space;\oplus&space;R)" />
- <img src="https://latex.codecogs.com/gif.latex?P&space;\oplus&space;\varnothing&space;=&space;\varnothing&space;\oplus&space;P"/>
- <img src="https://latex.codecogs.com/gif.latex?P&space;\oplus&space;-P&space;=&space;\varnothing"/>
- <img src="https://latex.codecogs.com/gif.latex?P&space;\oplus&space;Q&space;=&space;Q&space;\oplus&space;P"/>

p.s. other words it means this is an [Abelian](https://en.wikipedia.org/wiki/Abelian_group) group over sum.

On practice, coordinates of a point are in [Finite field](https://en.wikipedia.org/wiki/Finite_field). for example:
- [Real number](https://en.wikipedia.org/wiki/Real_number)
- [Complex number](https://en.wikipedia.org/wiki/Complex_number)
- [Finite fields](https://en.wikipedia.org/wiki/Finite_field) <img src="https://latex.codecogs.com/gif.latex?F_{p}" /> and <img src="https://latex.codecogs.com/gif.latex?F_{p^t}" />
    
important note:  A finite field of order q exists if and only if the order q is a prime power <img src="https://latex.codecogs.com/gif.latex?p^{k}" /> , basically a fields of order 17 or 239 contains just of excees of division by modulus prime number.

Example <img src="https://latex.codecogs.com/gif.latex?F_{p}" />:
- <img src="https://latex.codecogs.com/gif.latex?Y^{2}=X^{3}&plus;AX&plus;B" />
- replace R with <img src="https://latex.codecogs.com/gif.latex?F_{p}" />, <img src="https://latex.codecogs.com/gif.latex?p=q^{t}" />, <img src="https://latex.codecogs.com/gif.latex?q&space;\neq&space;2,3"/>
- <img src="https://latex.codecogs.com/gif.latex?E:Y^{2}-X^{3}-3X&plus;3"/> over <img src="https://latex.codecogs.com/gif.latex?F_{5}" />
    - <img src="https://latex.codecogs.com/gif.latex?(1,1)&space;\in&space;E,&space;(4,1)&space;\notin&space;E"/>
- use the previous algotithm

note: if p is 2 or 3

- <img src="https://latex.codecogs.com/gif.latex?p=2^{t}" />

    - <img src="https://latex.codecogs.com/gif.latex?Y^{2}&plus;CY=X^{3}&plus;AX&plus;B" /> or    
    - <img src="https://latex.codecogs.com/gif.latex?Y^{2}&plus;CXY=X^{3}&plus;AX^{2}&plus;B" />
- <img src="https://latex.codecogs.com/gif.latex?p=3^{t}" />

    - <img src="https://latex.codecogs.com/gif.latex?Y^{2}=X^{3}&plus;AX^{2}&plus;BX&plus;C"/>
    
In the real task it uses in [DL](https://en.wikipedia.org/wiki/Discrete_logarithm)
suppose we have:

- <img src="https://latex.codecogs.com/gif.latex?G=\{g,g^{2},...,g^{n-1},g^{n}=e\}" /> - circle group (all elements are the power one of the element in this group)
- <img src="https://latex.codecogs.com/gif.latex?h&space;\in&space;G"/>

need to find <img src="https://latex.codecogs.com/gif.latex?x,&space;g^{x}=h" title="x, g^{x}=h" />
this is the [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) [DL](https://en.wikipedia.org/wiki/Discrete_logarithm) base.

[DL](https://en.wikipedia.org/wiki/Discrete_logarithm) task separates to the two:

Classic task of [DL](https://en.wikipedia.org/wiki/Discrete_logarithm):

- p - Prime
- <img src="https://latex.codecogs.com/gif.latex?g,h&space;\in&space;Z" title="g,h \in Z" />

need to find <img src="https://latex.codecogs.com/gif.latex?g^{x}&space;\equiv&space;\mod&space;p"/> (not nulls excees of [Finite field](https://en.wikipedia.org/wiki/Finite_field))

[ECDLP](https://ru.wikipedia.org/wiki/ECDLP):

- E - elliptic curve
- <img src="https://latex.codecogs.com/gif.latex?P,Q&space;\in&space;E(F_{p})"/>

need to find <img src="https://latex.codecogs.com/gif.latex?n,nP=Q"/>
this is <img src="https://latex.codecogs.com/gif.latex?n&space;=&space;log_{P}&space;Q" />

The complexity in this case - is that the group of the point on an elliptic curve might be not cyclic. There is no generatrix point there. But we can select a point on the elliptic curve use of which we are able to generate Many points on the elliptic curve. If in the cyclic group we can power the generatrix number to get others non zero elements of the field, in elliptic curve case - we cannot multiple points, but can only sum them.

<img src="https://latex.codecogs.com/gif.latex?nP=mP"/>
<img src="https://latex.codecogs.com/gif.latex?\varnothing&space;=&space;(m-n)P&space;=&space;P&plus;...m...&plus;P-P...n...P"/>
count of pointer restricted by square field size plus one. As [Hasse's theorem on elliptic curves](https://en.wikipedia.org/wiki/Hasse%27s_theorem_on_elliptic_curves) tells us - the amount of pointers is about an order of field. Consequently - if the pointer generated big enough field, need to find what P was multiplied on to get Q.

important: if P and n is known - the calculation of Q might be done quickly <img src="https://latex.codecogs.com/gif.latex?O(\sqrt&space;p)"/>:
<img src="https://latex.codecogs.com/gif.latex?5P=(P&plus;P)&plus;(P&plus;P)&plus;P"/>

to be continued...


----------


**Useful links**:
 - [https://en.wikipedia.org/wiki/Abelian_group](https://en.wikipedia.org/wiki/Abelian_group)
 - [https://en.wikipedia.org/wiki/Finite_field](https://en.wikipedia.org/wiki/Finite_field)
 - [https://en.wikipedia.org/wiki/Real_number](https://en.wikipedia.org/wiki/Real_number)
 - [https://en.wikipedia.org/wiki/Complex_number](https://en.wikipedia.org/wiki/Complex_number)
 - [https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
 - [https://en.wikipedia.org/wiki/Discrete_logarithm](https://en.wikipedia.org/wiki/Discrete_logarithm)
 - [https://ru.wikipedia.org/wiki/ECDLP](https://ru.wikipedia.org/wiki/ECDLP)
 - [https://en.wikipedia.org/wiki/Hasse%27s_theorem_on_elliptic_curves](https://en.wikipedia.org/wiki/Hasse%27s_theorem_on_elliptic_curves)
 
