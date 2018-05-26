---
layout: post
title: A division and n-root algorithm for big numbers
---

In a [previous post](/Bignumbers1/), we introduced a library for performing arithmetic operations with arbitrary precision, and in [this update](/FFT-Multiplication/) we optimized the multiplication algorithm using the [Fast Fourier Transform](https://en.wikipedia.org/wiki/Fast_Fourier_transform) factorization.

In this post, using the FFT-based multiplication algorithm as the cornerstone of our library, we will optimize the division algorithm and introduce some other methods for computing square and quartic roots. In a previous example, we succeeded to compute the [largest known prime number](https://en.wikipedia.org/wiki/Largest_known_prime_number) at the time - the Mersenne prime $$M_{43112609}$$ - working with integer arithmetic. With the new ingredients, we will be able to compute some other interesting things, such as **millions of decimals of [$$\pi$$](https://en.wikipedia.org/wiki/Pi)** or other [irrational numbers](https://en.wikipedia.org/wiki/Irrational_number) using [convergent series](https://en.wikipedia.org/wiki/Convergent_series).

## A new division algorithm

We have already optimized the multiplication method, moving from a $$O(n^2)$$ [time complexity](https://en.wikipedia.org/wiki/Big_O_notation) algorithm to a $$O(n\,log_2(n))$$ one. But the division is still computed with a [long division algorithm](https://en.wikipedia.org/wiki/Long_division), with a time complexity of $$O(n^2)$$. We will try to find a faster algorithm using the [Newton Raphson](https://en.wikipedia.org/wiki/Division_algorithm#Newton.E2.80.93Raphson_division) method to converge to the solution.

The problem is: given the numbers $$a$$ and $$b$$, we want to compute the quotient $$c = \dfrac{a}{b}$$. The following function has a [zero](https://en.wikipedia.org/wiki/Zero_of_a_function) at the desired quotient:

$$  f(x) = b\,x - a.  $$

Applying the [Newton-Raphson](https://en.wikipedia.org/wiki/Newton's_method) method to it, starting with an initial guess $$x_0$$, we can compute a sequence $$\{x_k\},\,k=1,\,2,\,\ldots$$ that converges quadratically to $$\dfrac{a}{b}$$ in the following way

$$  x_{k+1} = x_{k} - \dfrac{f(x_k)}{f'(x_{k})} = x_{k} - \dfrac{b\,x_{k} - a}{b}  $$

Turns out that we didn’t make any progress yet, as long as we still need to compute a division with the same divisor $$b$$. But, conceiving the division $$\dfrac{a}{b}$$ as a multiplication by the inverse $$a\,\left(b^{-1}\right)$$, and being that inverse $$b^{-1}$$ a zero of the following trivial function:

$$  f(x) = \dfrac{1}{x} - b,  $$

we can build a series that converges quadratically to $$b^{-1}$$

$$  x_{k+1} = x_{k} - \dfrac{f(x_k)}{f'(x_{k})} = x_{k} - \dfrac{\dfrac{1}{x_{k}} - b}{-\dfrac{1}{x_{k}^2}} = x_{k}\left(2 - b\,x_{k}\right)  $$

Thus, we only need multiplications and subtractions to compute every element of the sequence. Once we have converged to the inverse, $$x_{k} \simeq b^{-1}$$, we only need to estimate the division as

$$c \simeq = a\,x_{k}$$

A [quadratic convergence](https://en.wikipedia.org/wiki/Rate_of_convergence) means that the amount of figures found, should double from one iteration to the next one.

## A square root and _n-root_ algorithm

In a similar way, we can build a sequence converging to the square root of a given number, which is a root of the following function

$$f(x) = x^2 - a \qquad \Rightarrow \qquad x_{k+1} = x_{k} - \dfrac{ x_k ^ 2 - a}{2\,x_k}$$

Or, if we want to avoid computing the division in each iteration, we can build a sequence converging to the inverse of the square root, being such number a zero of the function

$$  f(x) = \dfrac{1}{x^2} - a \qquad \Rightarrow \qquad x_{k+1} = \dfrac{x_{k}}{2}\left(3 - a\,{x_k}^2\right).  $$

_Voilà_, we only need multiplications and additions/subtractions in each iteration, and the solution will converge quadratically to $$\dfrac{1}{\sqrt{a}}$$. We can even compute the desired value $${\sqrt{a}}$$ at the end without having to invert the solution, only multiplying by $$a$$

$$  \sqrt{a} = a\,\dfrac{1}{\sqrt{a}} \simeq a\,x_k.  $$

This method can be generalized for computing $${\sqrt[n]{a}}$$ (the two previous examples correspond to $$n = 1$$ and $$n = 2$$):

$$  f(x) = \dfrac{1}{x^n} - a \qquad \Rightarrow \qquad x_{k+1} = \dfrac{x_{k}}{n}\left(n + 1 - a\,{x_k}^n\right).  $$

Once we have the final solution $$x_{k} \simeq \dfrac{1}{\sqrt[n]{a}}$$, we can compute the n-root without inverting the solution, by means of

$$  \sqrt[n]{a} \simeq a\,x_k^{n-1}  $$

## Implementation details

The implementation in the [project page](https://github.com/ibancg/bignumbers) adds the new features to the previous version:

* Inverse computation (file [div.cc](https://github.com/ibancg/bignumbers/blob/1.0.2/div.cc)).
* Division algorithm using the inverse method (file [div.cc](https://github.com/ibancg/bignumbers/blob/1.0.2/div.cc)). The long division algorithm was already implemented.
* Square and quartic roots (n-root not implemented) using the direct methods or the more efficient inverse ones (file [sqrt.cc](https://github.com/ibancg/bignumbers/blob/1.0.2/sqrt.cc)).

You will find more information about how to switch the algorithms in the [README](https://github.com/ibancg/bignumbers/blob/1.0.2/README) file.

The plot below shows a scalability comparison of the methods implemented in the library. Notice the scalability problems of Long Division and Long Multiplication algorithms (the latest takes 10,000 times longer than the FFT multiplication algorithm to compute 1 million figures). The Newton inverse algorithm and its related algorithms (inverse division, square root and quartic root), perform well enough for our purposes (much better than the non inverse versions). The non inverse square and quartic root algorithms are based on an inverse division algorithm (they would perform much worse if based on a long division algorithm). There are many possible optimizations, but they are beyond the scope of this post.

<div>
  <center><img src="/images/nsqrt_compare.png"></center>
  <center><font size="2">Figure 1: Scalability of the algorithms implemented in the library.</font></center>
</div>

Some other important issues we didn’t tackle yet:

* **Initial guesses** for the iterative methods: check the methods `BigNumber::fromDouble(double, int)` and `toDouble(double&, int&)` and their usage for computing an initial guess, i.e. for a [multiplicative inverse](https://github.com/ibancg/bignumbers/blob/1.0.2/div.cc#L31).
* **Convergence criterions**: a typical stop criterion with a convergent sequence would be when $$x_{k+1}$$ and $$x_{k}$$ are very close each other, $$x_{k+1} \simeq x_{k}$$, and that means that all (or almost all) of their digits match up. Check the source code again for further detail.
* **Rounding errors**: using algorithms that converge to multiplicative inverses can incur rounding errors when working with finite arithmetic. These errors can be overcome, but no solution has been implemented in the code, nor discussed in this post. As a hint for the division algorithm, for example, once we get the approximate quotient $$c_1 \simeq \dfrac{a}{b}$$ using the inverse approach, we can compute a new quotient $$c_2 \simeq \dfrac{r}{b}$$ (where $$r = a - b\,c_1$$ is the first remainder), and estimate the final quotient as $$c \simeq c_1 + c_2$$. The second division can be performed with a more accurate Long Division algorithm, not taking too long given the small remainder.

## Application example

I always wanted to write my own program for computing millions of decimals of $$\pi$$, and now we have all the ingredients to do it by using, for instance, a 4th order [Borwein’s algorithm](https://en.wikipedia.org/wiki/Borwein's_algorithm) (disclaimer: this is just an application of this general purpose library, not a specially optimized way to compute decimals of $$\pi$$). Let us start with the following values:

$$  a_0 = 6 - 4\sqrt{2},\qquad y_0 = \sqrt{2} - 1,  $$

and then construct the sequences $$\{y_k\}$$ and $$\{a_k\}$$

$$  y_{k+1} = \dfrac{1-(1-y_k^4)^{1/4}}{1+(1-y_k^4)^{1/4}}, \\ \\ a_{k+1} = a_k(1+y_{k+1})^4 - 2^{2k+3} y_{k+1} (1 + y_{k+1} + y_{k+1}^2),  $$

the sequence $$\{a_{k}\}$$ **converges with quartic order** to $$\dfrac{1}{\pi}$$. You can get this result in a few minutes with a normal computer - as of 2012 - if you **compute $$\pi$$ with 1,038,090 decimals** (the last 7 decimals are wrong due to rounding errors, and the total amount of figures involved in computations is $$2^{20}$$ = 1,048,576). You can find the implementation in the file [pi.cc](https://github.com/ibancg/bignumbers/blob/1.0.2/pi.cc).

## Links

* [Project hosted on github.com](https://github.com/ibancg/bignumbers)
* [Source code](https://github.com/ibancg/bignumbers/archive/1.0.2.zip)

