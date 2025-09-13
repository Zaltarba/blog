---
layout: post
title: "Random Walk vs Martingale: What’s the Difference"
categories: [Statistics, Quantitative Finance, ]
excerpt: "Learn the key differences between martingales and random walks in finance. Includes intuitive examples, asset pricing theory, and Python code."
image: /thumbnails/RandomWalkvsMartingale.jpeg
hidden: False
tags: [random walk vs martingale, difference between random walk and martingale, martingale finance explained, random walk stock prices, risk neutral measure in finance, stochastic discount factor tutorial, radon nikodym derivative finance, martingale vs random walk in asset pricing, martingale property in economics, python simulation random walk]
---

# Random Walk vs Martingale : What’s the Difference ? 

Finance students often encounter both [*random walks*](https://en.wikipedia.org/wiki/Random_walk) and [*martingales*](https://fr.wikipedia.org/wiki/Martingale_(calcul_stochastique)) in their studies. At first sight, the two concepts look almost the same : both can be applied to describe the process, that “do not drift on average”. But both concepts are not identical. Understanding the distinction is crucial.  

In this article, we’ll clarify the concepts, illustrate them with Python simulations, and explain why the distinction matters in finance.  

## Table of Contents

1. [Random Walk: A Simple Model of Prices](#Random-Walk)  
2. [Martingale: A Broader Mathematical Concept](#Martingale)  
3. [Random Walk vs Martingale: Key Differences](#KeyDifferences)  
4. [Conclusion](#Conclusion)  
5. [Further Reading](#Further_Reading)

## Random Walk: A Simple Model of Prices  {#Random-Walk}

A **random walk** is one of the simplest [stochastic processes](https://en.wikipedia.org/wiki/Stochastic_process). In discrete time, it is defined as:

$$
X_{t+1} = X_t + \epsilon_{t+1}
$$

where $\epsilon_{t+1}$ are independent and identically distributed (i.i.d.) shocks, often assumed to be Gaussian with mean zero and variance $\sigma^2$. The noise distribution can however take different for, including distribution with heavy tails.

### Intuition

At each step, the process moves up or down randomly, with no predictable trend. The past brings no information to the future.

### Financial interpretation 

If stock prices followed a random walk, tomorrow’s price would equal today’s price plus a random shock. This aligns with the **Efficient Market Hypothesis** : if markets are efficient, new information arrives randomly, and so do price changes.  

### Python example

```python
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)

N, T = 20, 252
eps = np.random.normal(0, 1, (T, N))
X = np.cumsum(eps, axis=0)  # random walk

plt.plot(X)
plt.title("Random Walk Simulations")
plt.show()
```

![Random Walk Simulation with python](/blog/images/RWvsM_fig_1.png)

## Martingale: A Broader Mathematical Concept  {#Martingale}

A martingale is defined in probability theory as a process $X_t$ such that:

$$
\mathbb{E}[X_{t+1} \mid \mathcal{F}_t] = X_t
$$

where $\mathcal{F}_t$ is the information available up to time $t$.

### Intuition

The conditional expectation of tomorrow’s value, given today’s information, is just today’s value. In other words, knowing the past gives you no edge in predicting the future.

### Key difference from random walk 

A martingale is defined by a conditional expectation property under a probability measure, when a random walk is model of the behavior of $X_t$. This makes the martingale a far more general concept.

### Financial Interpretation: The Role of the SDF and Change of Measure

Under the **real-world probability measure** $\mathbb{P}$, a stock price is generally **not** a martingale due to the presence of risk premia—investors demand compensation for bearing risk. However, in **risk-neutral pricing**, we use a different probability measure, called the **risk-neutral measure** $\mathbb{Q}$, under which **discounted asset prices become martingales**. This makes the concept universal.

#### What is the risk neutral probability 

This probability transformation is made possible by the **Stochastic Discount Factor (SDF)**, also known as the **pricing kernel**. The SDF captures how investors value future payoffs relative to today, adjusting for both **time value** and **risk preferences**. 

If we consider no dividends from $t$ to $t+1$ the connection is formalized as:

$$
\forall i \in \Omega : P_t^i  = \mathbb{E}^{\mathbb{P}}_t \left[ m_{t+1} \cdot P_{t+1}^i \right] 
$$

Here, $m_{t+1}$ acts as a **weighting function** that adjusts future payoffs based on states of the world, giving less weight to "good" states (where marginal utility is low) and more to "bad" states (where it's high). 

But how is this possible ? Let's define the **[Radon-Nikodym](https://en.wikipedia.org/wiki/Radon%E2%80%93Nikodym_theorem) derivative**:

$$
\begin{equation}
\frac{d\mathbb{Q}}{d\mathbb{P}} = \frac{m_{t+1}}{\mathbb{E}^{\mathbb{P}}[m_{t+1}]}
\tag{1}
\end{equation}
$$

using this ratio we can **shift from the real-world measure to the risk-neutral measure**:

$$
\begin{aligned}
\mathbb{E}^{\mathbb{P}}_t \!\left[ m_{t+1}\, P_{t+1}^i \right] 
&= \mathbb{E}^{\mathbb{P}}_t \!\left[ \frac{d\mathbb{Q}}{d\mathbb{P}} \; \mathbb{E}^{\mathbb{P}}[m_{t+1}] \; P_{t+1}^i \right] \\[1em]
&= \mathbb{E}^{\mathbb{P}}_t \!\left[ \frac{d\mathbb{Q}}{d\mathbb{P}} \, P_{t+1}^i \right] \, \mathbb{E}^{\mathbb{P}}[m_{t+1}] \\[1em]
&= \mathbb{E}^{\mathbb{Q}}_t \!\left[ P_{t+1}^i \right] \, \mathbb{E}^{\mathbb{P}}[m_{t+1}].
\end{aligned}
$$

Using equation (1) we can apply it to the risk free asset such that wih return $e^{r}$:

$$
\mathbb{E}^{\mathbb{P}}[m_{t+1}] = e^{-r}
$$

We end up with :

$$
P_t = e^{-r} \mathbb{E}^{\mathbb{Q}}_t \left[ P_{t+1}^i \right]
$$

If we use $e^{rt} P_t$ as the discounted price, we get 

$$
e^{-rt} P_t^i = e^{-r(t+1)} \mathbb{E}^{\mathbb{Q}}_t \left[ P_{t+1}^i \right]
$$

$$
e^{-rt} P_t^i = \mathbb{E}^{\mathbb{Q}}_t \left[ e^{-r(t+1)} P_{t+1}^i \right]
$$

Thus if we note $\tilde{P}_t^i = e^{rt} P_t^i$ we have :

$$
\mathbb{E}^{\mathbb{Q}}_t[ \tilde{P}_{t+1}^i] = \tilde{P}_t^i
$$

In this new measure $ \mathbb{Q} $, the discounted price of any asset is a **martingale**, meaning its expected future value (adjusted for time) equals its current price. 

This is the mathematical foundation of modern asset pricing and derivatives valuation: we **price assets as if investors are risk-neutral**, using a change of measure justified by the SDF, even though in reality they are not.

## Random Walk vs Martingale: Key Differences  {#KeyDifferences}

| Feature         | Random Walk                                  | Martingale                                      |
|-----------------|---------------------------------------------|------------------------------------------------|
| Definition      | $X_{t+1} = X_t + \epsilon_{t+1}$, iid shocks | $\mathbb{E}[X_{t+1} \mid \mathcal{F}_t] = X_t$ |
| Distribution    | Specific (often Gaussian)                    | Any distribution, as long as conditional expectation holds |
| Predictability  | No trend, but variance accumulates           | No conditional drift; more general           |
| Finance usage   | Descriptive model of price series           | Mathematical framework for pricing, based on no-arbitrage arguments |

## When is Martingale not a Random Walk ?

The key difference between a random walk and a martingale process lies in the distributional assumptions. A random walk assumes i.i.d. (independent and identically distributed) increments, while a martingale only requires that the conditional expectation of tomorrow’s value equals today’s value.

To see this distinction, consider the following stochastic process:

$$
X_{t+1} = X_t + X_t\epsilon_{t+1}
$$

where $\epsilon_{t+1}$ is a zero-mean random shock.  

- This process is a **martingale**, because  
  $$
  \mathbb{E}[X_{t+1} \mid \mathcal{F}_t] = X_t.
  $$  
- But it is **not a random walk**, since the increments depend on the current level $X_t$ rather than being i.i.d.  

This example also highlights why **log prices** are often modeled instead of raw prices. Taking logs, we get:  

$$
\log(X_{t+1}) = \log\!\big(X_t(1+\epsilon_{t+1})\big) 
= \log(X_t) + \log(1+\epsilon_{t+1}).
$$  

Now the process $\log(X_t)$ evolves as a **random walk with innovations $\log(1+\epsilon_{t+1})$**, which are i.i.d. under suitable conditions.  

### Python illustration

```python
# Simple martingale simulation
X = [10]  # start at 0
for t in range(1, 252):
    # steps are fair, zero expected change
    X.append(X[-1] + np.random.choice([-0.1, 0.1])*X[-1])  

plt.plot(X, label="Martingale-like Process")
plt.title("Martingale Simulation")
plt.legend()
plt.show()
```

![Martingale Simulation with python](/blog/images/RWvsM_fig_2.png)

## Conclusion  {#Conclusion}

Random walks and martingales may seem similar at first glance, but the distinction is subtle and important. Random walks describe how prices move step by step, while martingales capture a broader property of **fair game** conditional expectations. Understanding both is fundamental to quantitative finance, stochastic modeling, and financial mathematics. To see a pratical use of random walk models and martingales properties to forecast volatility, check this [previous post](https://zaltarba.github.io/blog/BitcoinVolatility-2/) ! 

## Further Reading  {#FurtherReading}

To deepen your understanding of martingales, stochastic discount factors, and their role in asset pricing, consider exploring the following resources:

- **John H. Cochrane – Asset Pricing**  
  A comprehensive and rigorous treatment of asset pricing using stochastic discount factors, risk-neutral valuation, and the martingale approach. Some free content is available on [Cochrane's website](https://www.johnhcochrane.com/).

- **Darrell Duffie – Dynamic Asset Pricing Theory**  
  A mathematically advanced reference focusing on continuous-time models and martingale techniques in finance.

- **Steven Shreve – Stochastic Calculus for Finance I & II**  
  Volume I introduces discrete-time models; Volume II covers continuous-time finance using Brownian motion and martingales. Great for building intuition and formal understanding.

- **Martin Baxter & Andrew Rennie – Financial Calculus**  
  A practical introduction to arbitrage pricing and martingale methods in continuous time.

- **Wikipedia articles**:  
  - [Martingale (probability theory)](https://en.wikipedia.org/wiki/Martingale_(probability_theory))  
  - [Stochastic discount factor](https://en.wikipedia.org/wiki/Stochastic_discount_factor)  
  - [Radon–Nikodym theorem](https://en.wikipedia.org/wiki/Radon%E2%80%93Nikodym_theorem)

These readings offer both the theoretical foundation and practical insights needed for modern quantitative finance.




