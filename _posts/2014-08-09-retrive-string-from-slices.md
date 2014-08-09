---
layout: post
title: Retrieve String From Sampled Slices
categories: 
- Algorithm
- C/C++
- Statistics
tags:
- Algorithm
- C/C++
- Statistics
status: publish
type: post
published: true
author: Bo Yang
---

1. [Problem]("#problem")
2. [Solution]("#solution")
3. [Source Code]("#code")

### 1. <a name="problem">Problem</a>

Given a string, such as 01001010101001101011, we can randomly sliced multiple substrings. Assume that during the slicing, due to some unexpected noises, some characters may flip(0->1 or 1->0). For example:

<pre>
Position: 0123456789.........
String:   01001010101001101011

slice1:    1001110101000
slice2:       1010111001111
slice3:       10101
slice4:                1101011
</pre>

In above sample, `slice1` starts from position 1(assume that the index of a string begins with 0), `slice2` starts from position 4, `slice3` starts from 4, and `slice4` starts from 13. In slice1, `0` flips to `1` at position 5 and `1` flips to `0` at position 13.

For one specific position in the original string, if it is `1`, then the probability of flipping to 0 in a slicing is 0.1; and vice versa(i.e.Prob(0->1)=0.1).

The problem is: **if we only have multiple slices(the length of each slice may vary) and their starting positions in the string, and we don't know the original string, given an arbitrary position in the original string, how can we calculate the probability that position is a `1`?**

Assume most positions will be covered at least once in slices, and we have following parameters:

	p01=0.1; // Probability a ‘0’ in string but flipped to a ‘1’ in a slice
	p10=0.1; // Probability a ‘1’ in string but flipped to a ‘0’ in a slice
	p1=0.5;  // Prior probability that any given position in string is a ‘1’

We can also assume that the string is a random string of `0`s and `1`s, and during slicing, each position is sampled independently.

For the above example string and four slices, we already have following probabilities for each position:

	Pos Prob
	0   0.500
	1   0.900
	2   0.100
	3   0.100
	4   0.999
	5   0.100
	6   0.999
	7   0.001
	8   0.999
	9   0.500
	10  0.988
	11  0.012
	12  0.012
	13  0.900
	14  0.988
	15  0.500
	16  0.988
	17  0.100
	18  0.900
	19  0.900

### 2. <a name="solution">Solution</a>

Obviously, the numbers of 0s and 1s in all slices for each position can be counted using a hashmap(`unordered_map` in C++). However, the difficult part is to find a model to calculate the probabilities. Simple multiplicaiton or subtraction won't work, because we cannot get `0.999` with the given probabilities and arithmetic operations. Besides, the Bayes' theorem also cannot be used directly, because we don't know the number of slices and the computation would be too complicated. And we also may be tempted to try Markov Model, but Markov Model cannot fit all positions in the above example, such as positions 4(`1,1,1`), 5(`1,0,0`), 9(`0,1`) and 13(`0,1,1`).

The best model to fit this problem is the [Binomial Distribution](http://www.mathworks.com/help/stats/binomial-distribution.html):

> The binomial distribution models the total number of successes in repeated trials from an infinite population under the following conditions:
> 
> * Only two outcomes are possible on each of n trials.
> * The probability of success for each trial is constant.
> * All trials are independent of each other.

The probability density function(pdf) of Binomial Distribution is:

$$
	f(x|n,p)={n \choose x} p^x (1-p)^{n-x}; \quad \quad x=0,1,2,\cdots,n.
$$

As for this problem, given a position in the original string, we first need to find the probability of a `0` occurs in this position(in this case `0`=>`1`, say `bp01`), and then calculate the probability of a `1` occurs in this position(in this case `1`=>`1`, say `bp11`). Finally, we can get the probability of `1` in this position in the original string by: `p=bp11/(bp01+bp11)`.

Take position 5 as an example. We get one `1` and two `0`s in position 5 in three observations. We can calculate `b11` using Matlab `binopdf` function. Since `p11=1-p10=0.9`, therefore we have `bp11=binopdf(1,3,0.9)=0.027`. As for `bp01`, because `p01=0.1`, we have `bp01=binopdf(1,3,0.1)=0.243`. Therefore, we can calc the probability of `1` in position 5 by `p=0.027/(0.243 + 0.027)=0.1`.

### 3. <a name="code">Source Code</a>

The implementation of this problem can be found in [my GitHub Channel](https://github.com/bo-yang/ProgrammingChallenges/tree/master/string_retrieve). 

### References:

1. [http://stackoverflow.com/questions/24967842/how-to-calculate-the-probability-of-a-given-character-in-a-string-using-slices-o](http://stackoverflow.com/questions/24967842/how-to-calculate-the-probability-of-a-given-character-in-a-string-using-slices-o)
2. [http://www.mathworks.com/help/stats/binomial-distribution.html](http://www.mathworks.com/help/stats/binomial-distribution.html)
