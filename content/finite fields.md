> *This is a reference for all theorems and properties of finite fields that are relevant to any of my articles. As of now the formal definition is not necessary to understand the proofs and structure of the protocols I care about.*

For our purposes a finite field is the set of integers modulo some prime number $q$. This is represented as $\mathbb{Z}_q$(integers mod q) or $\mathbb{F}_q$(field that is the integers mod q). These two notations are equivalent but most people use the latter(can use $F_q$ too). This is the set we will be working with:
$$
\{0, 1, 2, ..., q-1\}
$$
## Modular Arithmetic
Modular arithmetic is like a clock. What happens if you add 3 hours past 11 am? Then it would be 2 pm. Because $11 + 3 = 14, 14 - 12 = 2$. What if you had something 28 hours past 11 am? Then $11 + 28 = 39, 39 - 12 = 27 - 12 = 15 - 12 = 3$. So we continually go around the clock or subtract $12$ until we hit a number within $1,...,12$. This is exactly how modular math is done. The set is the numbers on the clock($\mathbb{F}$) and the thing we subtract each step is the modulus($q$). However, rather than waiting for a number between $1,...,q$ we wait for a number between $0,...,q-1$. You may have noticed that the result is the remainder of dividing the number by the modulus($39 / 12 = 12(3) + (3)$). This is how we will treat it implicitly from now on. Let's try a few examples from $\mathbb{F}_5 = \{0, 1, 2, 3, 4\}$:
$$
4 + 2 = 6 \equiv 1 \mod 5 \newline
3 \cdot 3 = 9 \equiv 4 \mod 5 \newline
5(2) - 3(2) = 4 \mod 5 \newline 
\text{ also } 5(2) = 10 \equiv 0, 3(2) = 6 \equiv 1, 0 - 1 = -1 \equiv 4 \mod 5 
$$
>For our purposes, the $=$ and $\equiv$ operator are interchangeable, however, they don't mean the same thing. $=$ indicates that the elements are identical whereas $\equiv$ indicates the numbers are both within the same equivalence class(but not necessarily identical). Regardless we end up mapping the elements into our domain of interest, i.e. the field elements, and continue to perform subsequent computations on those elements. Therefore, there isn't really a distinguishing factor here between the two conventions. All you need to remember is to convert the result of your arithmetic back into the field using mod. I will only be using $=$ from now on for simplicity.

Division works a little differently. Division is technically multiplying by the reciprocal of the number. So $10 / 2 = 10 \cdot 1/2$. The reciprocal is simply the multiplicative inverse of the element(raising it to the power of -1. ($1/2 = 2^{-1}$). We don't have fractions in a field but we do have multiplicative inverses which will perform the same function. Same as in ordinary arithmetic the element times its multiplicative inverse is one($2 \cdot 2^{-1} = 1$). So in a field the multiplicative inverse of an element is such that it fulfills this property. In $\mathbb{F}_5$ the multiplicative inverse of $2$ is $3$($2 \cdot 3 = 6 = 1 \mod 5$). So in a field, $10 / 2 = 10 \cdot 2^{-1} = 10 \cdot 3 = 30 = 0 \mod 5$).
### Polynomials
All the polynomial rules work the same in a field before you evaluate the polynomial. So factoring, distributive property, etc. all work in a field. Just remember that when evaluating a polynomial it is arithmetic and therefore needs to be done modulo $q$. For example, let $p(x) = x^2 - 1 = (x+1)(x-1)$ over the field $F_7$. Then
$$
p(3) = 9-1=8=1 \mod 7 \newline
p(3) = (3 + 1)(3 - 1) = 4 \cdot 2 = 8 = 1 \mod 7
$$



