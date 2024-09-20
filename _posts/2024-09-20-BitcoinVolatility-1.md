---
layout: post
title: Estimating Bitcoin's Volatility Using EWMA
categories: [Algo Trading, Personal Project, Quantitative Analysis]
excerpt: Dive into the first part of our three-part series on computing Bitcoin's volatility with Binance data.
image: /images/BayesianClustering.png
hidden: False
---

## Introduction

In our [previous post](https://zaltarba.github.io/blog/DataBaseCreation/), we explored how to fetch and store Binance candlestick data using HDF5, laying the groundwork for efficient data management in cryptocurrency research. Efficient data handling is crucial when dealing with large datasets, especially in the fast-paced world of crypto trading.

Now, we're taking the next step in our journey by delving into the analysis of this data. This is the first part of a three-part series where we'll dive deep into estimating Bitcoin's volatility using different methods. 

**Terminology Alert** : Volatility is a fundamental concept in finance, representing the degree of variation in the price of a financial asset over time. In simpler terms, it measures how much and how quickly the value of an asset changes. 

It's important to note that volatility is not directly observable—it's a **hidden variable** that must be estimated from market data. Its value can vary depending on the estimation model used, making it a crucial yet complex component in financial analysis.

There are two main types of volatility:

- **Historical Volatility**: Calculated from past market prices, it reflects the asset's actual price movements over a specified period. Since volatility is hidden and not directly measurable, we estimate historical volatility using statistical models applied to historical return data. The choice of model and time frame can significantly impact the volatility estimate.

- **Implied Volatility**: Derived from the market price of a traded derivative, especially options. It represents the market's expectation of future volatility. Implied volatility is extracted using option pricing models like the Black-Scholes model. Because it depends on the model's assumptions and inputs, implied volatility is also an estimate and can vary between models.

Why this difference ? Well, it happens volatility is far easier to predict than returns themselves. For options or other financial products, the future value of the volatility (meaning the future behaviour of the asset) is more usefull. Thus, the market will use the volatility expectancy for those products. In this post, and the other to follow, **we will focus on the historical volatility**.

### Why Study Historical Volatility?

Understanding historical volatility is crucial, especially when dealing with highly volatile assets like Bitcoin. Historical volatility measures the degree of variation in an asset's past price movements over a specific period. Studying it provides valuable insights into the asset's risk profile and helps in making informed decisions. Here's why analyzing historical volatility is essential:

1. Risk Assessment and Management : Historical volatility is a key indicator of the risk associated with an asset. By analyzing how much Bitcoin's price has fluctuated in the past, traders and investors can gauge the potential risk of future price movements. This understanding allows for:
2. Strategic Trading Decisions : Studying historical volatility can help traders develop effective trading strategies by recognizing periods of high or low volatility to choose appropriate trading approaches (e.g., trend following in high volatility, mean reversion in low volatility). 
3. Option Pricing and Derivative Valuation: Since volatility is a hidden variable and not directly observable, historical volatility serves as an essential estimate.
4. Getting Risk-Adjusted Returns: Evaluating whether the potential returns justify the risks based on historical volatility.

In essence, studying historical volatility is **not just an academic** exercise—it's a practical necessity. It equips traders and investors with the knowledge to navigate the complexities of the financial markets, particularly the dynamic and often unpredictable cryptocurrency landscape. By acknowledging that volatility is a hidden variable dependent on estimation models, we can appreciate the importance of selecting appropriate methods to measure and interpret it. This understanding ultimately leads to better risk management, more effective trading strategies, and informed decision-making in the pursuit of financial goals.

## Preparing the Data

Before we can estimate volatility, we need to prepare our data. The database we will use is the one we created in [this post](https://zaltarba.github.io/blog/DataBaseCreation/).

### Loading Data from HDF5

First, let's load the Bitcoin data we previously stored in our HDF5 file.

```python
import pandas as pd

# Load the data from the HDF5 file
df = pd.read_hdf('data/crypto_database.h5', key='BTCUSDT')

# Display the first few rows
print(df.head())
```

This will give us access to the Bitcoin candlestick data with 1-minute granularity.

### Data Cleaning and Preprocessing

Ensuring data quality is paramount. We'll check for missing values and ensure that the data types are appropriate.

```python
# Check for missing values
print(df.isnull().sum())

# Convert 'Open_Time' to datetime if not already done
df.reset_index(inplace=True)
df['Open_Time'] = pd.to_datetime(df['Open_Time'])

# Set 'Open_Time' as the index
df.set_index('Open_Time', inplace=True)
```

## Calculating Log Returns

### Why Use Log Returns?

Logarithmic returns, or log returns, are preferred over simple returns for several reasons:

- **Time Additivity**: Log returns are additive over time, which simplifies calculations.
- **Normality**: They often better approximate a normal distribution, which is a common assumption in financial models.
- **Symmetry**: Log returns treat upward and downward movements symmetrically.

The formula for log returns is:

$$
r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
$$

Where:

- $ r_t $ is the log return at time $ t $.
- $ P_t $ is the price at time $ t $.

### Implementing Log Returns Calculation

Let's calculate the log returns of the 'Close' price.

```python
import numpy as np

# Calculate log returns
df['log_returns'] = np.log(df['Close'] / df['Close'].shift(1))

# Drop the NaN value created by the shift
df.dropna(subset=['log_returns'], inplace=True)

# Display the first few log returns
print(df['log_returns'].head())
```

Now, let's visualize the log returns to get a sense of their behavior.

```python
import matplotlib.pyplot as plt

# Plot the log returns
plt.figure(figsize=(12, 6))
df['log_returns'].plot()
plt.title('Bitcoin Log Returns')
plt.xlabel('Date')
plt.ylabel('Log Return')
plt.show()
```

## Introducing the EWMA Method

### Understanding EWMA

The Exponentially Weighted Moving Average (EWMA) is a method that gives more weight to recent observations while not discarding older data entirely. This makes it highly responsive to recent changes in volatility, which is particularly useful in volatile markets like cryptocurrency.

The EWMA volatility is calculated using the formula:

$$
\sigma_t^2 = \lambda \sigma_{t-1}^2 + (1 - \lambda) r_t^2
$$

Where:

- $ \sigma_t^2 $ is the variance at time $ t $.
- $ \lambda $ is the decay factor (0 < $ \lambda $ < 1).
- $ r_t $ is the return at time $ t $.

### Reference to Hull's Methodology

John Hull's book *"Options, Futures, and Other Derivatives"* discusses EWMA as an effective method for volatility estimation, particularly in the context of RiskMetrics developed by J.P. Morgan. The method's ability to adapt quickly to market changes makes it valuable for risk management and derivative pricing.

## Estimating Volatility Using EWMA

### Setting the Decay Factor (Lambda)

The choice of the decay factor $ \lambda $ determines how quickly the weights decrease. A commonly used value is $ \lambda = 0.94 $, as suggested by RiskMetrics. A higher $ \lambda $ gives more weight to older observations, while a lower $ \lambda $ emphasizes recent returns more heavily.

### Step-by-Step Calculation

Using pandas, we can implement the EWMA volatility estimation easily.

```python
# Set the decay factor
lambda_ = 0.94
alpha = 1 - lambda_

# Calculate the EWMA variance
df['ewma_variance'] = df['log_returns'].ewm(alpha=alpha, adjust=False).var()

# Calculate the EWMA volatility
df['ewma_volatility'] = np.sqrt(df['ewma_variance'])

# Display the first few EWMA volatility values
print(df['ewma_volatility'].head())
```

### Visualization

Let's plot the EWMA volatility to visualize how it changes over time.

```python
# Plot the EWMA volatility
plt.figure(figsize=(12, 6))
df['ewma_volatility'].plot()
plt.title('EWMA Volatility of Bitcoin Returns')
plt.xlabel('Date')
plt.ylabel('Volatility')
plt.show()
```

## Interpreting the Results

### Analyzing Volatility Patterns

By examining the plot, we can observe periods where volatility spikes, indicating higher risk and potentially higher reward opportunities. These spikes often correspond to significant market events or news affecting Bitcoin.

### Benefits of Using EWMA

- **Responsiveness**: EWMA adjusts quickly to changes, making it suitable for volatile markets.
- **Simplicity**: It's straightforward to implement and doesn't require complex modeling.
- **Weighted Data**: By giving more importance to recent data, EWMA provides a more current view of market volatility.

## Conclusion and Next Steps

In this post, we've introduced the concept of volatility and its importance in financial markets, particularly for Bitcoin. We've shown how to estimate volatility using the EWMA method, which provides a responsive and practical approach for traders and analysts.

This is just the beginning. In the next posts of this series, we'll explore more sophisticated methods like GARCH models and the use of high-frequency data for volatility estimation.

Stay tuned for deeper insights into the fascinating world of financial volatility analysis!

## Additional Resources

- **Code Repository**: [GitHub Link](#) *(Replace with actual link)*
- **Further Reading**:
  - John Hull's *Options, Futures, and Other Derivatives*
  - J.P. Morgan's RiskMetrics Technical Document

## Closing Remarks

Feel free to share your thoughts or ask questions in the comments below. I encourage you to experiment with the code, try different decay factors, and see how it affects the volatility estimates.

Happy analyzing, and may your trading decisions be ever informed!
