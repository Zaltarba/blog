---
layout: post
title: Estimating Bitcoin's Volatility Using A GARCH model
categories: [Algo Trading, Personal Project, Quantitative Analysis]
excerpt: Dive into the second part of our three-part series on computing Bitcoin's volatility with Binance data.
image: /images/BayesianClustering.png
hidden: False
---

## Introduction

In our [last post](https://zaltarba.github.io/blog/BitcoinVolatility-1/), we discussed the Exponentially Weighted Moving Average (EWMA) method for estimating Bitcoin’s volatility. We used historical data from Binance and implemented the EWMA model to track volatility. In this post, we’ll take it a step further by introducing the **GARCH** model, a more sophisticated method used to estimate but also forecast volatility. Let’s dive in!

## Motivations 

When modeling financial volatility, it’s crucial to choose a model that captures the unique characteristics of asset price returns without unnecessary complexity. While simple models like the random walk with gaussian increments might seem sufficient for short-term forecasts, they miss important features of financial time series. 

Let's consider the random walk model :

$$
p_{t+1} = p_t + \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0, \sigma^2)
$$

Then the following empirical properties cannot be modelized :

1. **Volatility Clustering** : One of the most well-documented phenomena in financial markets. This refers to the empirical observation that large price changes tend to be followed by large price changes, and small changes tend to be followed by small changes, regardless of the direction of the price movement. 
2. **Heavy-Tailed Distributions** : Another well-known stylized fact about financial returns. Extreme events (large price changes) happen more often than a normal distribution would predict. Simple volatility models that assume normality tend to underestimate the likelihood of these extreme events.

From a mathematical standpoint:

$$
\text{Cov}(\epsilon_t^2, \epsilon_{t+k}^2) > 0 \quad \text{for small } k
$$

$$
\text{Kurt}(\epsilon_t) = \frac{\mathbb{E}[(\epsilon_t - \mu)^4]}{\sigma^4} > 3
$$

For any financial time series exhibiting these characteristics (volatility clustering and heavy tails) using a more sophisticated model becomes necessary. Fortunately, we can identify these properties through rigorous statistical tests :

1. **Autocorrelation Tests**: To detect volatility clustering, we can apply tests like the [**Ljung-Box Q-test**](https://en.wikipedia.org/wiki/Ljung%E2%80%93Box_test) on the squared residuals $\epsilon_t^2$. 
2. **Kurtosis Test**: Since heavy tails (excess kurtosis) are often observed in financial returns, we can perform a [**Jarque-Bera test**](https://en.wikipedia.org/wiki/Jarque%E2%80%93Bera_test) to check whether the distribution of the residuals significantly deviates from normality.

Moreover we also have the GARCH model, which is specifically designed to capture and replicate such behaviors. When applied, the GARCH model accounts for these empirical features, allowing the time series to exhibit the volatility clustering and non-normal distributions that are often observed in financial markets. But enough teasing and let's explain it !

## About The GARCH Model 

**GARCH models** are specifically designed to capture this clustering behavior. The model dynamically adjusts the variance based on both past returns and past volatility, allowing it to accommodate periods of high and low volatility naturally. This is crucial in markets like cryptocurrencies, where volatility tends to persist over time after major events. GARCH models are particularly useful here because they don’t impose normality on the returns directly. Instead, by modeling volatility dynamically and accounting for changes in variance over time, they can better accommodate the heavy-tailed nature of returns, which is frequently observed in financial markets. This ability to model volatility fluctuations means GARCH models can better handle extreme price movements, like those often seen in Bitcoin or other cryptocurrencies.

- **Bollerslev (1986)** demonstrated the success of GARCH models in explaining time-varying volatility in exchange rates.
- **Engle (1982)**, who pioneered the ARCH model (the predecessor to GARCH), showed how volatility clustering in financial markets can be modeled effectively with these frameworks.

The [**GARCH**](https://en.wikipedia.org/wiki/Autoregressive_conditional_heteroskedasticity) (Generalized Autoregressive Conditional Heteroskedasticity) model was developped by [Tim Bollerslev](https://public.econ.duke.edu/~boller/Published_Papers/joe_86.pdf) and is a popular choice for financial volatility modeling, especially in markets where volatility tends to cluster over time. Let’s dive in!

### What is a GARCH Model?

While EWMA focuses only on recent data to estimate volatility, **GARCH** (Generalized Autoregressive Conditional Heteroskedasticity) integrates both recent returns and a long-term average variance, capturing more information from the dataset. A **GARCH(p, q)** model extends this idea by considering a combination of **p** lagged variances (past periods’ volatility) and **q** lagged squared returns (recent price changes), giving it the flexibility to model volatility with a deeper memory.

The general form of the **GARCH(p, q)** model is given by:

$$
\sigma_t^2 = \omega + \sum_{i=1}^{q} \alpha_i \cdot r_{t-i}^2 + \sum_{j=1}^{p} \beta_j \cdot \sigma_{t-j}^2
$$

Where:
- $\sigma_t^2$ is the current variance estimate (volatility squared),
- $\omega$ represents the long-term variance (baseline level of volatility),
- $\alpha_i$ weights the impact of past squared returns $(r_{t-i}^2)$, capturing how recent market shocks influence volatility,
- $\beta_j$ weights the impact of past variances $(\sigma_{t-j}^2)$, reflecting how long-term volatility persists over time.

By adjusting the **p** and **q** parameters, the **GARCH(p, q)** model becomes highly versatile. It can capture both **volatility clustering** (the tendency for large changes to follow large changes) and **mean reversion** (volatility eventually returning to a long-term average), which are essential characteristics in financial markets, especially in assets with erratic price movements like cryptocurrencies.

### Why Use GARCH for Bitcoin Volatility?

Bitcoin's notorious price swings make it a perfect candidate for a volatility model that adjusts over time. Where EWMA adapts well to sudden changes, GARCH is more versatile, accounting for periods of high and low volatility over time. By using **GARCH(1,1)**, we can estimate the conditional variance for future price movements while factoring in historical volatility.

### Preparing the Data

Before we fit the GARCH model, let’s load and clean our data, just like we did in the previous post with EWMA. We’ll again use the Bitcoin data we stored in HDF5 format and ensure the dataset is free of missing values.

```python
import pandas as pd
import numpy as np
from arch import arch_model

# Load Bitcoin data
df = pd.read_hdf('data/crypto_database.h5', key='BTCUSDT')
df['log_returns'] = np.log(df['Close'] / df['Close'].shift(1))
df.dropna(subset=['log_returns'], inplace=True)
```

After calculating the log returns, we’re ready to move forward with fitting our GARCH model.

### Fitting the GARCH(1,1) Model

We’ll use Python’s `arch` package to fit the GARCH model. The **ARCH** (Autoregressive Conditional Heteroskedasticity) package is particularly useful for estimating financial volatility models like GARCH.

```python
# Fit the GARCH(1,1) model
model = arch_model(df['log_returns'], vol='Garch', p=1, q=1)
garch_fit = model.fit()
print(garch_fit.summary())
```

This provides us with the key parameters for our model : **omega (ω)**, **alpha (α)**, and **beta (β)**. These parameters give us insight into how much weight the model places on recent volatility and how much on long-term trends.

### Interpreting the GARCH Parameters
The summary output from fitting the model provides us with estimated values for the parameters:

- **Omega (ω)**: Represents the constant or baseline variance. A smaller omega indicates less baseline noise in the market.
- **Alpha (α)**: Controls how much weight recent returns have on future volatility. A higher alpha suggests that recent shocks have a large influence on volatility.
- **Beta (β)**: Reflects how persistent volatility is over time. A higher beta means volatility tends to persist.

For Bitcoin, we often observe high beta values, indicating that periods of volatility tend to last for extended periods.

### Forecasting Volatility Using GARCH
Once we have fitted the model, we can use it to forecast future volatility. The **GARCH(1,1)** model allows us to generate conditional forecasts for the next few periods.

```python
# Forecast future volatility
forecasts = garch_fit.forecast(horizon=5)
print(forecasts.variance[-1:])
```

The result will show us the variance forecast over the next five periods, which we can use to adjust trading strategies or risk management plans.

### Visualization
Finally, let’s plot the realized volatility against the GARCH forecast for a better understanding of how well the model captures Bitcoin's volatility over time.

```python
# Plot the actual vs. forecasted volatility
df['volatility_forecast'] = np.sqrt(forecasts.variance.values[-1, :])
df[['realized_volatility', 'volatility_forecast']].dropna().plot()
```

By visualizing these two metrics, we can see periods where Bitcoin's realized volatility was much higher than expected and how well the GARCH model adapts to these shocks.

## Conclusion

In this post, we’ve implemented a **GARCH(1,1)** model to estimate and forecast Bitcoin’s volatility. Unlike EWMA, GARCH gives us more flexibility by incorporating both recent returns and longer-term volatility trends. This model provides a valuable tool for traders and risk managers dealing with highly volatile assets like Bitcoin.

In the next post, we’ll explore more advanced volatility models such as GARCH extensions that account for asymmetric market shocks. Stay tuned for more insights into the fascinating world of volatility modeling!

## Additional Resources

- **Code Repository**: [GitHub Link](https://github.com/Zaltarba/BitcoinVolatilityEstimation/tree/main) 
- **Adviced Reading**: John Hull's *Options, Futures, and Other Derivatives*

Feel free to check out the GitHub repository for the complete code and try experimenting with different parameters to see how they affect volatility estimates.

Happy analyzing, and as always, may your trading strategies be well-informed!

