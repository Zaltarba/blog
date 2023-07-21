---
layout: post
title:  A momentum trading strategy with the Hurst exponent 
categories: [Personnal Project, Coding]
excerpt: Just for fun 
---

# A momentum trading strategy with the Hurst exponent

## Introduction

In this blog post, we will compute the Hurst exponent, and see possible use case. By harnessing data from Yahoo Finance, we'll embark on a journey to unveil valuable insights that can aid in making informed investment decisions.

Whether you are an experienced quantitative analyst or a novice with a passion for financial mathematics, this guide will equip you with the knowledge and technical skills necessary to calculate and interpret the Hurst exponent effectively. So, let's dive into the world of fractal Brownian motion and uncover the power of the Hurst exponent in analyzing financial time series data.

## Importing the data 

In this project we will use the yahoo Finance API to get daily values. Using python, we can get the historical values of the Tesla stock for instance : 

```python
import yfinance as yahooFinance
TSLA_ticker = yahooFinance.Ticker("TSLA")
TSLA_ticker_history = TSLA_ticker.history(period="max")
```
