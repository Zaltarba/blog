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

## A pratical implementation 

### Importing the data 

In this project we will use the yahoo Finance API to get daily values. Using python, we can get the historical values of the Tesla stock for instance : 

```python
import yfinance as yahooFinance
TSLA_ticker = yahooFinance.Ticker("TSLA")
TSLA_ticker_history = TSLA_ticker.history(period="max")
```

### Coding the strategy 

We first make the required imports : 

```python
from tqdm import tqdm
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import yfinance as yahooFinance
from yfinance.tickers import Tickers
from typing import List, Tuple
```

We then define our constants 

```python
N_DAYS = 2518
HOLDING_PERIOD = 2
PERSISTENT_THRESHOLD = 0.60
HURST_PERIOD = 20
MISE = 10
tickers = [
  "JNJ", "XOM", "IBM",
  "C", "GE", "F", "T",
  "MMM", "WMT", "JPM",
  "MCD", "DIS", "PG", "KO",
  ]
```

We define our hurst estimator 

```python
def hurst_estimator(log_prices:np.ndarray)->float:
  """
  Computes the Hurst exponent of a log_prices
  Input:
    - log_prices (np.ndarray): historical log prices
  Output:
    - hurst (float): the hurst exponent
  """
  numerator = np.sum((log_prices[1:]-log_prices[:-1])**2)
  log_prices_bis = np.array([log_prices[2 * i] for i in range(0, len(log_prices)//2)])
  denominator = np.sum((log_prices_bis[1:]-log_prices_bis[:-1])**2)
  hurst = -1/2 * np.log2(numerator / (2 * denominator))
  return hurst
```

We then implement our momentum trading strategy :

```python
def strategy_positions(log_prices:np.ndarray, hurst_exponents:List[float])->np.ndarray:
  """
  Compute the position of the momentum strategy for serie
  Inputs:
    - log_prices (np.ndarray): the stock log prices
    - hurst_exponents (List[float]): the hurst exponent at each day
  Output:
    - positions (np.ndarray): the strategy positions 
  """
  n = len(log_prices)
  positions = np.zeros(n)
  for day, hurst in zip(range(HOLDING_PERIOD, n-HOLDING_PERIOD), hurst_exponents):
    if hurst > PERSISTENT_THRESHOLD:
      if log_prices[day-HOLDING_PERIOD]<log_prices[day]:
        positions[day] = 1
      else:
        positions[day] = -1
  return(positions)

def strategy_exposure(positions:np.ndarray)->np.ndarray:
  """
  Computes the exposition of the strategy
  Input:
    - positions (np.ndarray): the strategy positions 
  Output:
    - exposure (np.ndarray): the strategy exposure 
  """
  exposure = np.zeros(positions.shape)
  exposure = np.abs(positions) * MISE
  exposure[HOLDING_PERIOD:] -= np.abs(positions[:-HOLDING_PERIOD]) * MISE
  return exposure

def strategy_capital_gains(
    close_prices:np.ndarray,
    open_prices:np.ndarray,
    positions:np.ndarray
    )->np.ndarray:
  """
  Computes the capital gain of the strategy for one stock
  Inputs:
    - open_prices (np.ndarray): the serie of the open prices
    - close_prices (np.ndarray): the serie of the close prices
    - positions (np.ndarray): the positions taken
  Output:
    - capital_gain (np.ndarray): the capital gain
  """
  capital_gains = np.zeros(len(close_prices))
  for i, position in enumerate(positions):
    if position == 1:
      capital_gains[i] = MISE * (open_prices[i+HOLDING_PERIOD] - close_prices[i])/close_prices[i]
    elif position == -1:
      capital_gains[i] = MISE * (close_prices[i] - open_prices[i+HOLDING_PERIOD])/close_prices[i]
    else:
      pass
  return capital_gains

def strategy_yields(
    close_prices:np.ndarray,
    open_prices:np.ndarray,
    positions:np.ndarray
    ):
  """
  Compute the strategy yields
  Inputs:
    - close_prices (np.ndarray): the serie of the close prices
    - open_prices (np.ndarray): the serie of the open prices
    - positions (np.ndarray): the positions taken
  Output:
    - yields (np.ndarray): the serie of the yields
  """
  yields = []
  for i, position in enumerate(positions):
    if position == 1:
      yields.append(
          100*(open_prices[i+HOLDING_PERIOD] - close_prices[i])/close_prices[i]
          )
    elif position == -1:
      yields.append(
          100*(close_prices[i] - open_prices[i+HOLDING_PERIOD])/close_prices[i]
          )
    else:
      pass
  return yields

def strategy(open_prices:pd.Series, close_prices:pd.Series)->Tuple[np.ndarray]:
  """
  Computes the global strategy
  Inputs:
    - open_prices (pd.Series): the open prices serie 
    - close_prices (pd.Series): the close prices serie 
  Outputs:
    - exposure (np.ndarray): the exposition 
    - capital_gains (np.ndarray): the capital gains
    - yields (np.ndarray): the yields 
  """
  log_prices = np.log(close_prices.shift(1).dropna())
  hurst_exponents = [
      hurst_estimator(
          log_prices.iloc[i-HURST_PERIOD:i].values
          ) for i in range(
              HURST_PERIOD, len(close_prices)-HURST_PERIOD
              )
          ]
  hurst_exponents = pd.Series(hurst_exponents)
  hurst_exponents = hurst_exponents.rolling(4).mean()
  positions = strategy_positions(log_prices, hurst_exponents)
  exposure = strategy_exposure(positions)
  capital_gains = strategy_capital_gains(close_prices, open_prices, positions)
  yields = strategy_yields(close_prices, open_prices, positions)
  return (exposure, capital_gains, yields)
```

Finally we run the momentum trading strategy for each of the portfolio's stock : 

```python
portfolio = {stock:{} for stock in tickers}
for stock in tqdm(portfolio):
  df = yahooFinance.Ticker(stock).history(period="10y")
  open_prices = df["Open"]
  close_prices = df["Close"]
  close_prices.plot()
  portfolio[stock]["exposure"], portfolio[stock]["capital_gains"], portfolio[stock]["yields"] = strategy(open_prices, close_prices)
```

### Results

```python
portfolio_capital_gain = np.zeros(N_DAYS)

fig, ax = plt.subplots(1, 1, figsize=(7, 5))
for stock in tqdm(portfolio):

  stock_capital_gain = np.cumsum(portfolio[stock]["capital_gains"])
  portfolio_capital_gain += stock_capital_gain
  sns.lineplot(x=df.index, y=stock_capital_gain, ax=ax)
ax.set_xlabel("Date")
ax.set_ylabel("Capital gains")
fig.suptitle("Capital gains of our portfolio's stocks")
plt.show()
```

[Insert graph]

```python
fig, ax = plt.subplots(1, 1, figsize=(7, 5))
sns.lineplot(x=df.index, y=portfolio_capital_gain,)
ax.set_xlabel("Date")
ax.set_ylabel("Capital gains")
fig.suptitle("Capital gains of our portfolio")
plt.show()
```

[Insert graph]

```python
portfolio_exposure = np.zeros(N_DAYS-1)

for stock in tqdm(portfolio):

  stock_exposure = np.cumsum(portfolio[stock]["exposure"])
  portfolio_exposure += stock_exposure
  #sns.lineplot(x=df.index[1:], y=stock_exposure, ax=ax1)

sns.lineplot(x=df.index[1:], y=portfolio_exposure,)
ax.set_xlabel("Date")
ax.set_ylabel("Exposure")
fig.suptitle("Portfolio exposure")
plt.show()
```




