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
$$
P_t^A = cst + \rho \cdot P_t^B + \epsilon_t
$$
You are then able to compute the hedge ratio from a simple regression. Moreover, you can get a confidence interval and p-value to ensure it is significantly non-null. Great!

But the devil hides in the details. I won't delve into all linear regression temporal aspects and its application for sequential data (refer to a handbook on time series prediction for more knowledge), but one has to be very careful.

**Terminology Alert**: A spurious regression is a regression where the p-value indicates a statistically significant relationship between variables, but this relationship is actually meaningless or random, often due to underlying trends in the data rather than a true correlation.

When you work with prices and not returns, this happens very frequently because of the temporal trend in the data.

Let's consider two time series, $(Y_t)$ and $(X_t)$, both with linear trends. We can express them as:

$ Y_t = \alpha_Y + \beta_Y \cdot t + \epsilon_t^Y $
Then 
$ X_t = \alpha_X + \beta_X \cdot t + \epsilon_t^X $
where $\alpha_Y$ and $\alpha_X$ are constants, $\beta_Y$ and $\beta_X$ are the coefficients of the linear trends, and $(\epsilon_t^Y)$ and $(\epsilon_t^X)$ are the error terms.

Now, let's perform a regression of $(Y_t)$ on $(X_t)$:
$$
Y_t = \gamma + \delta \cdot X_t + u_t
$$
Substituting the expressions for \(Y_t\) and \(X_t\):
$$
\alpha_Y + \beta_Y \cdot t + \epsilon_t^Y = \gamma + \delta \cdot (\alpha_X + \beta_X \cdot t + \epsilon_t^X) + u_t
$$
Rearranging terms, we get:
$$
\alpha_Y + \beta_Y \cdot t + \epsilon_t^Y = \gamma + \delta \cdot \alpha_X + \delta \cdot \beta_X \cdot t + \delta \cdot \epsilon_t^X + u_t
$$
For simplicity, let \(\gamma' = \gamma + \delta \cdot \alpha_X\) and \(u_t' = \epsilon_t^Y - \delta \cdot \epsilon_t^X + u_t\), so:
$$
\alpha_Y + \beta_Y \cdot t + \epsilon_t^Y = \gamma' + \delta \cdot \beta_X \cdot t + u_t'
$$

This equation shows that if both time series \(Y_t\) and \(X_t\) have a linear trend, the regression will reflect this trend rather than a meaningful relationship between the series. The term \(\delta \cdot \beta_X \cdot t\) suggests that the p-value might indicate a significant relationship due to the common trend \(\beta_X \cdot t\) and not due to any genuine correlation between \(\epsilon_t^Y\) and \(\epsilon_t^X\).

This spurious correlation arises because both series share a common linear trend, leading the regression to falsely suggest a meaningful relationship.

See, the p-value is correct only under some conditions, one being that the errors have to be stationary. That's where the cointegration's best friend comes into play: the **stationarity test**.
