---
layout: post
title: Linking Cointegration and the Multi Factor Model 
categories: [Personnal project, Practical paper]
excerpt: This post is the second of our serie about pair trading and market neutral strategies ...
---

## Introduction

Just so you know, this article will be a direct follow-up to my [first article](https://zaltarba.github.io/blog/PairsTrading-1/) about pairs trading. After providing some global context, motivations, and information on pairs trading, I will go more in-depth about something I teased: the link between the beta(s) of a pair of stocks and cointegration.

## About Multi Factor Model

When we speak about the multi-factor model, we generally refer to a generalization of the CAPM model. The main idea behind it is: the market return isn't the only factor with a shared influence on the stocks. Fama and French developed a famous three-factor model. We will be working with it for the following, but keep in mind other models exist using, for instance, ETFs as factors. You can refer to [Analysis of Financial Time Series](https://cpb-us-w2.wpmucdn.com/blog.nus.edu.sg/dist/0/6796/files/2017/03/analysis-of-financial-time-series-copy-2ffgm3v.pdf) for more information on multi-factor models.

### The Fama French Model

The Fama-French model expands on the CAPM by adding two more factors to better capture the returns of a portfolio. Instead of just looking at the market return, it also considers the size of firms and the book-to-market values. The model is typically represented by the following equation:

$$
R_i = R_f + \beta_i (R_m - R_f) + s_i \cdot SMB + h_i \cdot HML + \alpha_i + \epsilon_i
$$

Here's a quick breakdown:
- $R_i$: Expected return of the portfolio
- $R_f$: Risk-free rate
- $\beta_i$: Sensitivity to market return
- $R_m$: Market return
- $SMB$: Size premium (Small Minus Big)
- $HML$: Value premium (High Minus Low)
- $s_i$ and $h_i$: Sensitivities to SMB and HML respectively
- $\alpha_i$: Alpha, representing the stock-specific return
- $\epsilon_i$: Error term

So, instead of just betting on the market, the Fama-French model gives us a more nuanced view by adding these size and value factors. It's like getting a more detailed map for navigating the stock market! Good news about this model, French makes available to us those regressors (not live frequently updated). If you are a fellow quant doing some research you can look it [here](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html#Research).

## Link with the Cointegration 

## Conclusion 
