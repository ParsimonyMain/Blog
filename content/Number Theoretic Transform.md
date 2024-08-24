> *I will not explain every detail and the definition of every word, however, I will do thorough computation and examples. My goal is to explain the key details that simplify conceptual understanding. The other loose ends will either be simple or irrelevant enough for the reader to learn on their own.*

The Number Theoretic Transform(NTT) is the Fast Fourier Transform(FFT) within the context of [[finite fields]]. It is used to evaluate polynomials over a specific domain(x-coordinates). Please read the [[finite fields]] post to understand the field terminology, notation, and functionality. 
## Schoolbook Algorithm
In school, you learn to evaluate polynomials using the following algorithm:
```python
def eval_poly_at(poly, x, modulus):
    y = 0
    power_of_x = 1
    for coefficient in poly:
        y += power_of_x * coefficient
        power_of_x *= x
    return y % modulus
```
Let $\space p(x) = 3 +2x + x^2$ over the field $F_5$ and $x=3$. Then we would use 
```python
print(eval_poly_at([3, 2, 1], 3, 5))
# 3
```` 
 $p(3) = 3 + 2(3) + (3)^2 \pmod 5 = 18 \equiv 3 \pmod 5$. Awesome. It works.
This algorithm runs in $O(n)$ for one point evaluation as it has to go through each coefficient and runs in $O(n^2)$ for multipoint evaluation as it has to also evaluation each point.
## NTT Algorithm
NTT is fundamentally based on the cyclic nature of the multiplicative group of a field(denoted as $F^*$). $F^*$ removes the $0$ element from the field which makes its order potentially composite and a cyclic group. If the order of $F^*$ is a power of $2$, and we use it as an evaluation domain, then we can orchestrate redundant computations within our polynomial evaluation. Redundancy is the key to optimization. If we can reuse the computations and eliminate the redundancy, then we will do less work to get the same result. 

Let's use the field $F_5$ which has $F_5^* = \{1, 2, 3, 4 \}$. Its order is $4$ which is a power of $2$. The multiplicative group of a field has a special property where its elements can be written in terms of a primitive root/generator element who's powers will generate every element in the group. $2$ is a generator for $F^*_5$ as $2^0 = 1, 2^1=2, 2^2=4, 2^3=3, \pmod 5$. We can rewrite $F^*_5$ as $\{2^0, 2^1, 2^2, 2^3\}$.  Let $p(x) = x^2$. If we evaluate $p(x)$ using $F^*_5$ as the domain we get:
$$
p(2^0) = (2^0)^2 = 1 \pmod 5 \newline
p(2^1) = (2^1)^2 = 4 \pmod 5 \newline
p(2^2) = (2^2)^2 = 1 \pmod 5 \newline
p(2^3) = (2^3)^2 = 4 \pmod 5 \newline
$$
As you can see there is a symmetry/redundancy in the results. This is because when we square each element the domain is halved by the [[Cyclic Group Structure Theorem]]. This notion is sort of revered in the proof. Nevertheless, we first try to generate a cyclic subgroup of half the size of the current one(lets use order $10$ for clarity. $10/2 = 5$). Now trying to generate subgroup of size $5$ we use the $q/n$ exponent $10/5 = 2$. This is applied to the generator for exponents $0...5$ which is only the first half of the elements in the initial group. This is because the new generator $g^{q/n}$ will cycle at half the initial amount of elements which eliminates the need to use the rest. So technically we are trying to halve the size of the group and this is done through squaring each element. The point of this is to create redundancy which we will now exploit.

In the initial example, we can cut our computation in half by simply returning the first half($[1, 4]$) of our evaluations twice reusing them for the latter half($[1,4,1,4]$). Before moving on, its important to note that we will be recursively reusing evaluations over many levels of $x^2$. For example $x^8 = ((x^2)^2)^2$  . However, the only way I've seen people articulate the shrinking domain is to only evaluate the squared portion of the polynomial over a halved domain. So for $p(x) = x^2$ we would rewrite this as $p(x^2) = x$ making the input domain $\{2^0, 2^2, 2^4, 2^6\} = \{2^0, 2^2, 2^0, 2^2\} = \{2^0, 2^2\}$. This way we reduce the domain before we even try to evaluate it rather than just inserting the initial domain and stopping halfway as we might do intuitively(from an algorithmic perspective). 
> To clarify $p(expression) = a_1 + a_2x + a_3x^2 + ...$  just means that whatever the expression evaluates to will get substituted into $x$. So $p(3x^2 + 1) = x + 1$ where $x=1$ will first compute $3(1)^2+1 = 4$ and then substitute $4$ for $x$ which gives $5$ for the final evaluation of $p$. Some people will write $p(x^2) = y$ or $p(y) = x$ where $y = x^2$ to delineate between the input computation of $x$ verses its substitution value within the polynomial.

Now let's say $p(x) = x^3$. We can easily convert it into a form of $x^2$ by simply factoring out an $x$. So $p(x) = x(x^2)$. The $x^2$ can be converted into it's own polynomial in order to take advantage of the halved domain. So $p(x) = xp_1(x^2)$ where $p_1(x^2)$ will only be evaluated on half the original domain. From the first example, we already know that $p_1(x^2) = [1, 4]$ over $\{2^0, 2^2\}$. We return that plus the duplicate $[1, 4, 1, 4]$ and finally we multiply each evaluation by $x$ within the original domain:
$$
p(x) = [2^0(1), 2^1(4), 2^2(1), 2^3(4)] = [1, 3, 4, 2]
$$
over $F^*_5$.

What about multiple term polynomials? Let's take $p(x) = 4 + 3x + 2x^2 + x^3$. We can factor the even and odd exponents accordingly:
$$
p(x) = (4 + 2x^2) + x(3 + x^2) = p_{even}(x^2) + x \cdot p_{odd}(x^2) \space (\text{over } \{2^0, 2^1, 2^2, 2^3\})\newline
	p_{even}(x^2) = 4 + 2x , \space \space p_{odd}(x^2)=3 + x
$$
Now from our perspective at this point we can just plug in a value for $x$ and return the results. However, for the recursive algorithm we will split one more time until our polynomials are only constants. Then we just return the constants.
$$
p_{even}(x) = p_{even1}(x^2) + x\cdot p_{odd1}(x^2) \space (\text{over } \{2^0, 2^2\})\newline
p_{even1}(x^2) = 4, \space \space p_{odd1}(x^2) = 2 \newline
p_{even}(x) = [4 + 2^0(2), 4 + 2^2(2)] = [1, 2]
$$
$$
p_{odd}(x) = p_{even2}(x^2) + x\cdot p_{odd2}(x^2) \space (\text{over } \{2^0, 2^2\})\newline
p_{even2}(x^2) = 3, \space \space p_{odd2}(x^2) = 1 \newline
p_{odd}(x) = [3 + 2^0(1), 3 + 2^2(1)] = [4, 2]
$$
> Note that $p_{even}$ changed from using $x^2$ to $x$. This was because the domain of consideration changed. Initially we working over $F^*_5$ but later working over $\{2^0, 2^1\}$. This is why the $x^2$ was needed initially but not subsequently. 

So now(verify with algorithm from beginning):
$$
p(x) = [1, 2, 1, 2] + x \cdot[4, 2, 4, 2] \text{ for } x \in F^*_5 \newline
 = [0, 1, 2, 3]
$$
This is the NTT algorithm in a nutshell. Here is the code for it:
```python
def myfft(vals, modulus, domain):
    if len(vals) == 1:
        return vals
    L = fft(vals[::2], modulus, domain[::2])
    R = fft(vals[1::2], modulus, domain[::2])
    o = [0 for i in vals]
    for i, (x, y) in enumerate(zip(L, R)):
        y_times_root = y*domain[i]
        o[i] = (x+y_times_root) % modulus
    for i, (x, y) in enumerate(zip(L, R)):
        y_times_root = y*domain[i+len(L)]
        o[i+len(L)] = (x+y_times_root) % modulus
    return o
```
Vitalik Buterin [posted](https://vitalik.eth.limo/general/2019/05/12/fft.html) a slightly more efficient algorithm but it achieves the same result:
```python
def fft(vals, modulus, domain):
    if len(vals) == 1:
        return vals
    L = fft(vals[::2], modulus, domain[::2])
    R = fft(vals[1::2], modulus, domain[::2])
    o = [0 for i in vals]
    for i, (x, y) in enumerate(zip(L, R)):
        y_times_root = y*domain[i]
        o[i] = (x+y_times_root) % modulus
        o[i+len(L)] = (x-y_times_root) % modulus
    return o
```
His uses the symmetry of the cyclic group to multiply by $-x$ as opposed to $x$ for the latter half of the evaluation domain. $\{2^0, 2^1, 2^2, 2^3\} = \{2^0, 2^1, -2^0, -2^1\}$

### Time Complexity $O(n\cdot log(n))$
![[NTTTimeComp.svg]]
Let $n$ refer to the number of coefficients within the polynomial.  There are $log(n)$ recursive levels of functions called. The loops represent the number of evaluations for $x$ within the formula $p(x) = p_{even}(x^2) + x \cdot p_{odd}(x^2)$ for the associated domain. As you can see the number of functions in each level and their number of loops always adds up to $n$. This means that each level's complexity is on the order of $n$. So we have $log(n)$ levels times $n$ cycles per level which gives us $O(n \cdot log(n))$.