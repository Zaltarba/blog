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

$$ \beta_B \cdot r_t^A - \beta_A \cdot r_t^B = \beta_B \cdot \alpha_A - \beta_A \cdot \alpha_B + \epsilon_t^A + \epsilon_t^B $$

with

- $\beta_A$, $\alpha_A$, $r_t^A$, and $\epsilon_t^A$ being the CAPM components for stock A
- $\beta_B$, $\alpha_B$, $r_t^B$, and $\epsilon_t^B$ being the CAPM components for stock B

As you can see, the market part has disappeared. Theoretically, buying $\beta_B$ stock A and shorting $\beta_A$ stock B should give you a market-neutral portfolio for any stock. Where's the hitch? Well, a warning here: an assumption of the CAPM model is that $\epsilon_t$ is an idiosyncratic risk, meaning the correlation between $\epsilon_t^A$ and $\epsilon_t^B$ is null. That's a big assumption and why I have been warning so much about this framework. We will see more useful frameworks in the next posts.

**Terminology Alert**: In practice, strategies tend to go long on stock A and short $\frac{\beta_A}{\beta_B}$. Thus, we introduce the hedge ratio $\rho = \frac{\beta_A}{\beta_B}$.

Pair trading is all about working on spreads between the two assets and to construct getting positive returns from it (modern pair trading can of course be involving 3, 4, 5, or even more stock but we will keep simple for now).

Now that we are clear on what market neutral means, let's get into some of the basic tools.

## Some Technical Aspects

Let's do some. I will stop using the CAPM model from now, despite it being great to get some common sense / intuition, it's not so good for modelling todays market. 

A common framework for years has been the **cointegration**. Basically you consider stock A and B being linearly correlated in prices : 
$$
P_t^A = cst + rhau * P_t^B + eps_t
$$
You are then able to compute the hedge ratio from a simple regression. Moreover, you can get a interval confidence and p value to be sure it is signatiffically non null. Great !  

But devils hides in the details. I won't do into all linear regression temporal and it's application for sequential data (go to a handbook a time series prediction for some knowledge) but one has to verry carrefull.

**Terminology Alert** : a spurious regression is a regression where the p value for your component is bellow the significance threshold but still is complemetly random. 

When you work with prices and not return, this happens verry frequently because of the temporal trend in the data.

See, p value is correct only under some conditions, one being that the errors have to be stationnary. That's where comes into play the cointegration best friend : the **stationnary test**. 
