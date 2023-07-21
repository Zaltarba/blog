---
layout: post
title: Unveiling the Potential of the Momentum Strategy Enhanced by the Hurst Exponent 
categories: [Personnal Project, Coding]
excerpt: Just for fun 
---

# Unveiling the Potential of the Momentum Strategy Enhanced by the Hurst Exponent

## Introduction

Welcome to this post, where we delve into the captivating domain of finance and investment strategies. Today, we try to enhance the performance of the momentum strategy with the hurst exponent.

The momentum strategy is a widely-used investment approach based on the belief that assets with recent strong performance will continue to perform well, while those with weak performance will continue to underperform. It involves buying assets with positive price trends and selling those with negative trends over a specific time frame. This strategy relies on herding behavior and has shown effectiveness over time, but it comes with inherent risks and requires careful research and risk management.

Within this blog post, we endeavor to unravel a "humble" momentum strategy that augments its power through the application of the lesser-known yet potent metric, the Hurst exponent. Named after the eminent British hydrologist, Harold Edwin Hurst, this mathematical tool has found diverse applications in various disciplines, including finance, owing to its ability to reveal valuable insights into asset price movements.

The Hurst exponent assumes a pivotal role in comprehending underlying trends and persistence within financial time series data, thereby enriching our momentum-based approach. By merging the principles of momentum trading with the analytical prowess of the Hurst exponent, we aim to gain a deeper appreciation of how this strategy may potentially outperform the market and refine our investment decisions.

Throughout this exposition, we shall expound upon the theoretical underpinnings of the humble momentum strategy and delve into the intricacies of calculating and interpreting the Hurst exponent. Additionally, empirical evidence and backtesting results shall be presented to illustrate the strategy's efficacy across diverse market conditions.

So, if you are eager to traverse the realm of momentum trading, heightened by the mathematical prowess of the Hurst exponent, we invite you to join us on this journey of exploring the full potential of this captivating strategy.

## Theorical foundations of the model 

Speak about mBm ect

## Importing the data 

In this project we will use the yahoo Finance API to get daily values. Using python, we can get the historical values of the Tesla stock for instance : 

```python
import yfinance as yahooFinance
TSLA_ticker = yahooFinance.Ticker("TSLA")
TSLA_ticker_history = TSLA_ticker.history(period="max")
```
