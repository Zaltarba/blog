---
layout: post
title: Estimating the Value of $\pi$ with Python
categories: [Statistics]
excerpt: Let's explore how Monte Carlo methods allow us to approximate $\pi$ using Python.
image: /thumbnails/PiEstimation.jpeg
hidden: False
tags: [Monte Carlo, Python, Pi]
---

## Table of Contents

1. [Introduction](#introduction)  
2. [Monte Carlo in a Nutshell](#montecarlo)  
3. [From Uniform Randomness to $\pi$](#probabilities)  
4. [Practical Implementation in Python](#implementation)  
5. [Conclusion](#conclusion)  
6. [References and Further Reading](#references-and-further-reading)

## Introduction  {#introduction}

How can we estimate the value of $\pi$? While we know $\pi$ is irrational and cannot be expressed as a finite closed formula, approximations like $3.14$ or $22/7$ are widely used. But what if we want a systematic, simulation-based method to approximate $\pi$ with arbitrary precision?

Monte Carlo methods provide exactly this. By exploiting randomness and statistical principles, we can approximate $\\pi$ in a surprisingly elegant way.

## Monte Carlo in a Nutshell  {#montecarlo}

Monte Carlo methods rely on random sampling and the central limit theorem (CLT) to estimate expectations:

$$
\frac{1}{\sqrt{n}}\sum_{i=1}^{n}(X_i - \mu) \xrightarrow[]{d} N(0, \sigma^2)
$$

If we can find a random variable $X$ such that $\mathbb{E}[X] = \pi$, then simulating $X$ repeatedly and averaging gives us an estimate of $\pi$. The more samples we take, the closer we get to the true value, with the rate of convergence governed by the variance of $X$.

More precisely, if $\text{Var}(X) = \sigma^2$, then by the Central Limit Theorem, the sample mean

$$
\hat{\pi}_n = \frac{1}{n} \sum_{i=1}^n X_i
$$

satisfies

$$
\sqrt{n}\,(\hat{\pi}_n - \pi) \xrightarrow[]{d} N(0, \sigma^2).
$$

This means that for large $n$, the estimation error is approximately normal with variance $\frac{\sigma^2}{n}$. Hence, the standard error of our estimator is

$$
\text{SE}(\hat{\pi}_n) = \frac{\sigma}{\sqrt{n}}.
$$

### How many simulations do we need?

Suppose we want the absolute error to be less than $\varepsilon > 0$ with probability at least $1 - \alpha$. Using the normal approximation, we require

$$
\mathbb{P}\big( |\hat{\pi}_n - \pi| < \varepsilon \big) \approx 1 - \alpha.
$$

This leads to the condition

$$
\frac{\sigma}{\sqrt{n}} z_{1-\alpha/2} \leq \varepsilon,
$$

where $z_{1-\alpha/2}$ is the $(1 - \alpha/2)$ quantile of the standard normal distribution (for example, $z_{0.975} \approx 1.96$ for a $95\%$ confidence level).

Rearranging gives the required number of runs:

$$
n \geq \left( \frac{z_{1-\alpha/2} \cdot \sigma}{\varepsilon} \right)^2.
$$

This formula provides a practical guideline: once we know or estimate the variance $\sigma^2$ of the Monte Carlo variable, we can determine how many simulations are needed to achieve a desired accuracy $\varepsilon$ with a given confidence level $1 - \\alpha$.


## From Uniform Randomness to $\pi$  {#probabilities}

In practice, random number generation typically begins with pseudo-random draws from a uniform distribution, which can be transformed into other distributions. In Python, `numpy.random.uniform` provides a straightforward way to simulate such values.

Here is the key idea: let $X_1$ and $X_2$ be two i.i.d. uniform random variables on $[-1, 1]$. The probability that the point $(X_1, X_2)$ falls inside the unit circle is proportional to the ratio between the area of the circle and the area of the square:

$$
\mathbb{P}(X_1^2 + X_2^2 \leq 1) = \frac{\text{Area of Circle}}{\text{Area of Square}} = \frac{\pi r^2}{(2r)^2} = \frac{\pi}{4}.
$$

Thus, if we simulate many points uniformly in the square $[-1, 1]^2$ and check the fraction that lies within the unit circle, multiplying this fraction by 4 yields an approximation of $\pi$.

## Practical Implementation in Python  {#implementation}

The implementation is straightforward:

```python
import numpy as np

n = int(1e6)  # number of simulations
x1 = np.random.uniform(-1, 1, n)
x2 = np.random.uniform(-1, 1, n)

# Check which points fall inside the unit circle
inside_circle = x1**2 + x2**2 <= 1

# Estimate of pi
pi_estimate = 4 * inside_circle.mean()
print(pi_estimate)
```

Running this code with one million points gives a fairly good approximation of $\\pi$. Increasing `n` will further improve accuracy.

## Conclusion  {#conclusion}

Estimating $\pi$ with Monte Carlo is a beautiful demonstration of how probability and geometry intersect. It shows that if we can identify a random variable with an expectation equal to the quantity of interest, then repeated sampling gives us a practical way to approximate it.

## References and Further Reading  {#references-and-further-reading}

- [Monte Carlo Method — Wikipedia](https://en.wikipedia.org/wiki/Monte_Carlo_method)  
- [Central Limit Theorem — Wikipedia](https://en.wikipedia.org/wiki/Central_limit_theorem)  
- Gentle, J. E. (2003). *Random Number Generation and Monte Carlo Methods*. Springer.  
