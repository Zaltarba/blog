---
layout: post
title: Estimating Bitcoin's Volatility Using A GARCH model
categories: [Algo Trading, Personal Project, Quantitative Analysis]
excerpt: Dive into the second part of our three-part series on computing Bitcoin's volatility with Binance data.
image: /images/BayesianClustering.png
hidden: False
---

## Introduction
In our [last post](https://zaltarba.github.io/blog/BitcoinVolatility-1/), we discussed the Exponentially Weighted Moving Average (EWMA) method for estimating Bitcoin’s volatility. We used historical data from Binance and implemented the EWMA model to track volatility. In this post, we’ll take it a step further by introducing the **GARCH(p,q)** model—a more sophisticated method used to estimate volatility that can capture both short-term changes and long-term volatility trends.

The [**GARCH**](https://en.wikipedia.org/wiki/Autoregressive_conditional_heteroskedasticity) (Generalized Autoregressive Conditional Heteroskedasticity) model, introduced by Tim Bollerslev in [this article](https://public.econ.duke.edu/~boller/Published_Papers/joe_86.pdf), is a popular choice for financial volatility modeling, especially in markets where volatility tends to cluster over time. Let’s dive in!

### What is a GARCH Model?

While EWMA focuses only on recent data to estimate volatility, **GARCH** integrates both recent returns and a long-term average variance, capturing more information from the dataset. A GARCH(p,q) model uses the following equation to calculate volatility:

$$
\sigma_t^2 = \omega + \alpha \cdot r_{t-1}^2 + \beta \cdot \sigma_{t-1}^2
$$

Where:
- $\sigma_t^2$ is the current variance estimate (volatility squared),
- $\omega$ is the long-run variance,
- $\alpha$ weights the impact of recent price changes ($(r_{t-1}$),
- $\beta$ weights the impact of the previous day's variance.

**GARCH(1,1)** allows us to model both **volatility clustering** and **mean reversion**, which are key characteristics of financial time series, particularly in crypto markets.

**Terminology Alert** : Volatility Clustering is a well documented phenomenum in markets returns distribution where ...

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

