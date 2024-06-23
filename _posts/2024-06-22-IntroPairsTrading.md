---
layout: post
title: An Introduction to Pair Trading and Market Neutral Strategies
categories: [Personnal project, Practical paper]
excerpt: This post is the first of a serie about pair trading and market neutral strategies ...
---

## Introduction

I have started working on pairs trading and market-neutral trading strategies in the US equity market over the last few months. Man, what a challenge! Here, I will try to give a clear and concise introduction to the topic and introduce the frameworks I will be delving into more deeply in the next posts.

## General Overview

Before speaking of pairs trading, let's first define market-neutral strategies. A market-neutral strategy aims to provide uncorrelated sources of returns from the market. The Capital Asset Pricing Model (CAPM), even if now outdated, provides a nice framework to explain the intuition:

The return of an asset can be expressed as $r_t - r_f = \alpha + (\beta \cdot (r_m - r_f)) + \epsilon_t$, with:
- $r_f$ the risk-free return
- $r_m$ the market return
- $\alpha$ the expected excess return
- $\beta$ the sensitivity of the asset's returns to the market returns

With this framework, if you manage to construct a portfolio with a null beta (i.e., the linear combination of the assets within is such that the sum of their betas is null), you then get a portfolio whose returns are uncorrelated from those of the market! Bear and bull markets don't matter anymore! What's the point? Why not just buy OTM calls with one week until expiration and pray to get rich? Well, to that, I would say, based on my understanding of Harry Markowitz's works, that you surely should do both. I am sure an efficient frontier exists mixing those strategies. 

Anyway, when working on a market-neutral strategy, the quant researcher's role is thus to reduce its strategy's beta as much as possible and to maximize its alpha (i.e., its expected outperformance).

## About Pairs Trading

Pairs trading is a specific framework with the purpose of creating market-neutral strategies. The way to do this, used by hedge fund quants, is as follows: use a linear combination of two stocks to remove the beta from the market. Let's consider two stocks: stock A and stock B.

Under the CAPM model (again, this approach is outdated and if you are looking for practical advice, you should work with a multi-factor model at least) you get:

$$ 
\beta_B \cdot r_t^A - \beta_A \cdot r_t^B = \beta_B \cdot \alpha_A - \beta_A \cdot \alpha_B + \epsilon_t^A + \epsilon_t^B 
$$

with

- $\beta_A$, $\alpha_A$, $r_t^A$, and $\epsilon_t^A$ being the CAPM components for stock A
- $\beta_B$, $\alpha_B$, $r_t^B$, and $\epsilon_t^B$ being the CAPM components for stock B

As you can see, the market part has disappeared. Theoretically, buying $\beta_B$ stock A and shorting $\beta_A$ stock B should give you a market-neutral portfolio for any stock. Where's the hitch? Well, a warning here: an assumption of the CAPM model is that $\epsilon_t$ is an idiosyncratic risk, meaning the correlation between $\epsilon_t^A$ and $\epsilon_t^B$ is null. That's a big assumption and why I have been warning so much about this framework. We will see more useful frameworks in the next posts.

**Terminology Alert**: In practice, strategies tend to go long on stock A and short $\frac{\beta_A}{\beta_B}$. Thus, we introduce the hedge ratio $\rho = \frac{\beta_A}{\beta_B}$.

Pair trading is all about working on spreads between the two assets and to construct getting positive returns from it (modern pair trading can of course be involving 3, 4, 5, or even more stock but we will keep simple for now).

Now that we are clear on what market neutral means, let's get into some of the basic tools.

## Some Technical Aspects

Let's dive into some technical aspects. I will stop using the CAPM model from now on. Despite it being great for gaining some common sense/intuitions, it's not so good for modeling today's market.

A common framework for years has been **cointegration**. Basically, you consider stocks A and B being linearly correlated in prices:
$$P_t^A = cst + \rho \cdot P_t^B + \epsilon_t$$
You are then able to compute the hedge ratio from a simple regression. Moreover, you can get a confidence interval and p-value to ensure it is significantly non-null. Great!

But the devil hides in the details. I won't delve into all linear regression temporal aspects and its application for sequential data (refer to a handbook on time series prediction for more knowledge), but one has to be very careful.

**Terminology Alert**: A spurious regression is a regression where the p-value indicates a statistically significant relationship between variables, but this relationship is actually meaningless or random, often due to underlying trends in the data rather than a true correlation.

Let's consider two stock prices, $(P_t^A)$ and $(P_t^B)$. Let's model both of them with a random walk, and with uncorrelated noise. So we get 

$$P_t^A = P_0^A + \sum_{i=1}^{t} \epsilon_i^A$$
$$P_t^B = P_0^B + \sum_{i=1}^{t} \epsilon_i^B$$

with :

$$\epsilon_t^A \sim WN(0, \sigma_A^2)$$
and 
$$\epsilon_t^B \sim WN(0, \sigma_B^2)$$

With issue here is $Var(P_t^A) = t \dot sigma_A^2$ which means we have a non stationnary variable. Thus the linear regression requirements are not meet. If you attemps to do one the issue is that the usual test statistics (like t-statistics) do not follow their standard distributions under the null hypothesis. P-value will not be interpretable since it has to follow a uniform law between 0 and 1 under the null hypothesis. 

Doing some simulation with the folowing code : 

```python
    import numpy as np
    import pandas as pd
    import statsmodels.api as sm
    import matplotlib.pyplot as plt
    
    def simulate_random_walk(n):
        """ Generate a random walk series. """
        # Random walk starts at zero
        walk = np.zeros(n)
        # Generate random steps, add to previous
        for i in range(1, n):
            walk[i] = walk[i - 1] + np.random.normal()
        return walk
    
    def perform_regression(x, y):
        """ Perform linear regression and return the p-value. """
        x = sm.add_constant(x)  # Adding a constant for the intercept
        model = sm.OLS(y, x)
        results = model.fit()
        return results.pvalues[1]  # p-value for the slope
    
    def main():
        num_simulations = 1000
        num_points = 1000  # Number of points in each random walk
        p_values = []
    
        for _ in range(num_simulations):
            # Generate two independent random walks
            x = simulate_random_walk(num_points)
            y = simulate_random_walk(num_points)
            
            # Perform regression and get the p-value
            p_value = perform_regression(x, y)
            p_values.append(p_value)
    
        # Plotting the distribution of p-values
        plt.hist(p_values, bins=100, edgecolor='black')
        plt.xlabel('P-value')
        plt.ylabel('Frequency')
        plt.title('Distribution of P-values for 100 Regressions of Independent Random Walks')
        plt.show()
    
        # Optionally, return or print p_values or any other statistics
        return p_values
    
    # Call the main function to execute the simulation
    if __name__ == "__main__":
        p_values = main()
```

You get the folowing graph :

![Figure 1](/blog/images/IPT_spurious_pvalues.png)

As you can see, despite have two unrelated random walk we get an significant relationship most of the time.

So to resume, cointegration could a great tool, allowing us to compute the hedge ratio, but can be complety spurious and hypothesis testing is required in order to use it. See, the p-value will be correct only under some conditions, one being that the errors have to be stationary. That's where the cointegration's best friend comes into play: the **stationarity test**.
