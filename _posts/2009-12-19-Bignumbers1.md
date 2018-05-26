---
layout: post
title: Arithmetic operations with arbitrary precision
---

I decided to write this small library in 2000, after watching a rerun of _“The Simpsons”_ episode [“Treehouse of Horror VI”](https://en.wikipedia.org/wiki/Treehouse_of_Horror_VI).


<div style='float: right;margin-left: 20px'>
  <center><img src="/images/bignumbers1_homer3.png"></center>
  <center><font size="2">Figure 1: Homer Simpson in a three-dimensional world.</font></center>
</div>

In the last story of this episode, Homer Simpson jumps into a three-dimensional world. The full story, and specially this parallel world, are full of mathematical tributes: geometrical shapes, [Euler’s identity](https://en.wikipedia.org/wiki/Euler%27s_identity), a reference to the [P versus NP problem](https://en.wikipedia.org/wiki/P_versus_NP_problem), and so on. Among these mathematical references, a disturbing numerical formula attracted my attention: 

 $$1782^{12} + 1841^{12} = 1922^{12}$$

The previous statement is obviously a joke, since it contradicts [Fermat’s Last Theorem](https://en.wikipedia.org/wiki/Fermat%27s_Last_Theorem). However, evaluating both sides of the equation with a scientific calculator, one can fall for it. Try with Google, for example:

* [$$1782^{12} + 1841^{12} = 2.5412103\,.10^{39}$$](https://www.google.com/search?q=1782%5E12+%2B+1841%5E12)
* [$$1922^{12} = 2.5412103\,.10^{39}$$](https://www.google.com/search?q=1922%5E12)

The problem with is that a normal calculator doesn’t handle the enough amount of [significant figures](https://en.wikipedia.org/wiki/Significant_figures). Given the fact that both sides of the equality share the 9 first digits, the result is, apparently, the same. Here is the full 40-digit result:

* [$$1782^{12} + 1841^{12} = 2541210258614589176288669958142428526657$$](https://www.wolframalpha.com/input/?i=1782%5E12+%2B+1841%5E12)
* [$$1922^{12} = 2541210259314801410819278649643651567616$$](https://www.wolframalpha.com/input/?i=1922%5E12)

The main trouble stems obviously from a spatial limitation of the display, unable to represent all the computed figures in [scientific notation](https://en.wikipedia.org/wiki/Scientific_notation). But even if our device had a larger screen, the above formulas couldn’t be properly evaluated due to the internal [floating point](https://en.wikipedia.org/wiki/Floating_point) storage precision.

Even though there are numerous tools able to perform arithmetic operations with arbitrary big integers (like the [phyton language](https://www.python.org/), or [GNU bc](https://www.gnu.org/software/bc/)), I found interesting, as part of my learning process, to write a lightweight and generic library to deal with this and other similar problems.

In this post I will present the library code and its basic structure, and in future publications we will try to improve its efficiency and show some practical examples.

## Library specification

The library works with natural [binary-coded decimal](https://en.wikipedia.org/wiki/Binary-coded_decimal) (NBCD) format, with sign and module agreement, so an arbitrarily big integer can be encoded in a - long enough - BCD digit sequence.

<div style='margin-bottom: 15px'>
<center><img src="/images/bignumbers1_1.png"></center>
<center><font size="2">Figure 2: BCD sequence.</font></center>
</div>

In order to handle fractional numbers, the library uses [fixed-point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic), therefore the format - the BCD sequence - is divided into integer and fractional parts. This way, fractional numbers can be approximated with arbitrary precision by simply allocating enough memory in the fractional part. This scheme can deal with arbitrary-sized numbers, but in a static way, i.e., having previously allocated the needed memory.

<div style='float: left;margin-right: 30px;margin-top: 15px;margin-bottom: 15px'>
  <center><img src="/images/bignumbers1_2.png"></center>
  <center><font size="2">Figure 3: Multiplication with fixed-point arithmetic.</font></center>
</div>


Once the format has been chosen, let us describe some simple arithmetic operations:

* **Addition** and **subtraction**: These operations are made by a point-wise addition or subtraction of both BCD sequences, applying a [carry propagation](https://en.wikipedia.org/wiki/Carry_(arithmetic)) from each digit to the next one.
* **Multiplication**: The operation is performed by a typical [long multiplication algorithm](http://mathworld.wolfram.com/LongMultiplication.html).
* **Division**: The algorithm used is the [long division algorithm](http://mathworld.wolfram.com/LongDivision.html).

## The code

The library is coded in **C++**, but without exploiting many of the features of the language.

A number is coded in [`BigNumber`](https://github.com/ibancg/bignumbers/blob/1.0.0/bignum.h) class, which consists basically in a byte array and a sign flag. For simplicity and clarity, only one decimal digit is coded in each byte, instead of two per byte.

The file [`config.h`](https://github.com/ibancg/bignumbers/blob/1.0.0/config.h) is a configuration point, where we can set the number of digits assigned to the integer and fractional parts in the format.

The file [`test.cc`](https://github.com/ibancg/bignumbers/blob/1.0.0/test.cc) contains a main function with an example application: it evaluates both sides of the former equation $$1782^{12} + 1841^{12} = 1922^{12}$$, and verifies its falseness. At least 40 digits are needed to carry out the operation, so the default configuration, which reserves 50 digits for both the integer and fractional parts, is enough for our purposes.

The rest of the files implement several operations.

* [`addsub.cc`](https://github.com/ibancg/bignumbers/blob/1.0.0/addsub.cc): addition and subtraction.
* [`mul.cc`](https://github.com/ibancg/bignumbers/blob/1.0.0/mul.cc): multiplication.
* [`div.cc`](https://github.com/ibancg/bignumbers/blob/1.0.0/div.cc): division.
* [`convert.cc`](https://github.com/ibancg/bignumbers/blob/1.0.0/convert.cc): conversion between bignumbers and floating point numbers.

## Links

* [Library hosted on github.com](https://github.com/ibancg/bignumbers) (the code explained here is tagged as [1.0.0](https://github.com/ibancg/bignumbers/releases/tag/1.0.0), the posterior code can contain improvements explained in future posts)
* [Source code](https://github.com/ibancg/bignumbers/archive/1.0.0.zip)

<p><span style="color: blue; font-size: x-small;">Legal Notice:</span> <span style="font-size: x-small;"><em>The Simpsons</em> TM and (C) Fox and its related companies. All rights reserved. Any reproduction, duplication, or distribution in any form is expressly prohibited.</span></p>

<p><span style="color: blue; font-size: x-small;">Disclaimer:</span> <span style="font-size: x-small;"> This web site, its operators, and any content contained on this site relating to <em>The Simpsons</em> are not specifically authorized by Fox.</span></p>

