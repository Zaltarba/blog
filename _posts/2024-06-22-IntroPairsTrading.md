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

Pairs trading is specific framework with the purpose of creating market neutral strategies. The way to do so used by HF quants is the folowing : a use a linear combination of two stock to remove the beta from the market. Let's considers two stocks : stock A and stock B.

Under the CAPM model (again this approach is today outdated and if you are looking for pratical advise you should more work with a multi factor model at least) you get :

$ beta_B * r_t^A - beta_A * r_t^B = beta_B * alpha_B - beta_B * alpha_A + eps_t^A + eps_t^B $

with 

- $beta_A$, $alpha_A$, $r_t^A$ and $eps_t^A$ the capm components for stock A
- $beta_B$, $alpha_B$, $r_t^B$ and $eps_t^B$ the capm components for stock B

As you can see, the market part has here disapeared. Theorecally, buying $beta_B$ stock A and shorting $beta_A$ stock B should give you an market neutral portfolio for any stock. Where the hitch ? Well warning here, an assumption of the CAPM model is here that eps_t is an idiosyncratic risk, meaning the correlation between $eps_t^A$ and $eps_t^B$ is null. That's a big step to take and why have being warning some much about this framework. We will see more usefull frameworks in next posts.

**Terminology Alert** : in practice, strategies tend to go long on stock A and short $frac{beta_A}{beta_B}$. Thus we introduce the hedge ratio $rhau =  frac{beta_A}{beta_B}$.
