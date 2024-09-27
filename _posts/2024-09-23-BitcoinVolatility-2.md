---
layout: post
title: Estimating Bitcoin's Volatility using a GARCH Model
categories: [Personal Project,Algo Trading]
tags: [binance,trading,garch,volatility]
excerpt: Dive into the second part of our three-part series on computing Bitcoin's volatility with Binance data.
image: /thumbnails/BitcoinVolatility2.jpeg
hidden: false
---

## Table of Contents

- [Introduction](#introduction)
- [Motivations](#motivations)
- [The GARCH Model](#the-garch-model)
- [Preparing the data](#preparing-the-data)
- [Testing for ARCH Effects](#testing-for-arch-effects)
- [Model Selection](#model-selection)
- [Interpreting the GARCH Parameters](#interpreting-the-garch-parameters)
- [Goodness-of-fit Check](#goodness-of-fit-check)
- [Estimating and Forecasting Volatility](#estimating-and-forecasting-volatility)
- [Conclusion](#conclusion)

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

1. **Autocorrelation Tests**: We can apply tests like the [**Ljung-Box Q-test**](https://en.wikipedia.org/wiki/Ljung%E2%80%93Box_test) on the squared residuals $\epsilon_t^2$ to detect volatility clustering. 
2. **Kurtosis Test**: We can perform a [**Jarque-Bera test**](https://en.wikipedia.org/wiki/Jarque%E2%80%93Bera_test) to check whether the distribution of the residuals significantly deviates from normality.

Moreover we also have the GARCH model, which is specifically designed to capture and replicate such behaviors. When applied, the GARCH model accounts for these empirical features, allowing the time series to exhibit the volatility clustering and non-normal distributions that are often observed in financial markets. But enough teasing and let's explain it !

## The GARCH Model 

### What is a GARCH Model?

The [**GARCH**](https://en.wikipedia.org/wiki/Autoregressive_conditional_heteroskedasticity) (Generalized Autoregressive Conditional Heteroskedasticity) model was developped by [Tim Bollerslev](https://public.econ.duke.edu/~boller/Published_Papers/joe_86.pdf) and is a popular choice for financial volatility modeling, especially in markets where volatility tends to cluster over time. Let’s dive in.

We modelize the log returns with : 

$$
r_t = \log(p_t) - \log(p_{t-1}) \quad \text{where} \quad \epsilon_t \sim \mathcal{N}(0, \sigma_t^2)
$$

Here, $\sigma_t^2$ is the time-varying volatility, which is gonna be modeled by the GARCH model. A **GARCH** model need two parameters for it's definition : p and q. It considers a combination of **p** lagged variances (past periods’ volatility) and **q** lagged squared returns (recent price changes), giving it the flexibility to model volatility with a deeper memory. The general form of the **GARCH(p, q)** model is given by:

$$
\sigma_t^2 = \omega + \sum_{i=1}^{q} \alpha_i \cdot r_{t-i}^2 + \sum_{j=1}^{p} \beta_j \cdot \sigma_{t-j}^2
$$

Where:
- $\sigma_t^2$ is the current variance estimate (volatility squared),
- $\omega$ represents the long-term variance (baseline level of volatility),
- $\alpha_i$ weights the impact of past squared returns $(r_{t-i}^2)$, capturing how recent market shocks influence volatility,
- $\beta_j$ weights the impact of past variances $(\sigma_{t-j}^2)$, reflecting how long-term volatility persists over time.

By adjusting the **p** and **q** parameters, the **GARCH(p, q)** model becomes highly versatile. It can capture both **volatility clustering** (the tendency for large changes to follow large changes) and **mean reversion** (volatility eventually returning to a long-term average), which are essential characteristics in financial markets, especially in assets with erratic price movements like cryptocurrencies.

### Why Use a GARCH Model Here?

The decision to make a blog post about using **GARCH model** for estimating Bitcoin’s volatility stems from both academic motivations and empirical necessities. Fitting a GARCH model is not just an exercise in financial theory, but is also one let's be honnest here. 

But the model’s ability to incorporate past variances and shocks into future volatility estimates makes it a compelling choice for researchers and practitioners alike. Academically, it’s exciting to explore how well the GARCH model fits different time series data, especially in markets as volatile as cryptocurrencies, where price swings happen frequently.

In financial time series like Bitcoin’s returns, there are well-documented **ARCH effects**. Volatility changes over time and exhibits clustering. This means simple models that assume constant volatility will fail to adequately describe the data. Moreover, it seems legit to assume Bitcoin’s return distribution exhibits **fat tails**, meaning extreme price changes occur more frequently than predicted by a normal distribution. This heavy-tailed behavior is another reason to use a GARCH model. 

## Preparing the Data

Before we fit a GARCH model, let’s load and clean our data, just like we did in the previous post with EWMA. We’ll again use the Bitcoin data we stored in HDF5 format and ensure the dataset is free of missing values. Because we are gonna use plenty of statistical test, we are gonna have to tackle some hardware limitations. For this modelling we will work with 3 months historic and keep one months for out of sample testing.

### Loading Data from HDF5

First, let's load the Bitcoin data we previously stored in our HDF5 file.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Define the date range 
start_date = '2019-01-01'
end_date = '2019-05-01'
# Load the data from the HDF5 file
df = pd.read_hdf('data/crypto_database.h5', key='BTCUSDT', where=f"index >= '{start_date}' and index <= '{end_date}'")
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

### Taking a look at the data 

Let's ensure we have good quality data by plotting the evolution of the Bitcoin Price.

```python
# Plot the log returns
plt.figure(figsize=(12, 6))
df['Close'].plot()
plt.title('Bitcoin Price')
plt.xlabel('Date')
plt.ylabel('BTC USDT')
plt.show()
```

![figure 1](/blog/images/BitcoinVolatility-2-figure-1.png)

### Calculating Log Returns

Let's calculate here again the log returns using the 'Close' price. 

```python
# Calculate log returns
df['log_returns'] = np.log(df['Close'] / df['Close'].shift(1))

# Drop the NaN value created by the shift
df.dropna(subset=['log_returns'], inplace=True)
```

Now, let's visualize the log returns to get a sense of their behavior.

```python
# Plot the log returns
plt.figure(figsize=(12, 6))
df['log_returns'].plot()
plt.title('Bitcoin Log Returns')
plt.xlabel('Date')
plt.ylabel('Log Return')
plt.show()
```

![figure 2](/blog/images/BitcoinVolatility-2-figure-2.png)

## Testing for ARCH Effects

In financial time series, it’s crucial to test for **ARCH effects** (Autoregressive Conditional Heteroskedasticity), which means that the volatility of the series changes over time and is not constant. If ARCH effects are present, we can use models like **GARCH** to capture the volatility dynamics. For the folowing statistical tests, we will as an example a 3 months period, keeping the last month as an out of sample dataset.

```python
split_date = pd.to_datetime("2019-04-01")
training_data_mask = df.index < split_date
returns = df.loc[training_data_mask, 'log_returns']
```

### Stationarity Test 

Before testing for ARCH effects, we must first check if the series is **stationary**. This is indeed a required features to fit a GARCH model on the serie. We typically can use the [**Augmented Dickey-Fuller (ADF) test**](https://en.wikipedia.org/wiki/Augmented_Dickey%E2%80%93Fuller_test). The null hypothesis of the ADF test is that the series has a unit root, i.e., it is non-stationary.

```python
import pandas as pd
from statsmodels.tsa.stattools import adfuller

# Assuming 'returns' is a Pandas Series of your financial returns
result = adfuller(returns)

print(f'ADF Statistic: {result[0]}')
print(f'p-value: {result[1]}')
```
```bash
ADF Statistic: -53.002111501058444
p-value: 0.0
```

The **p-value** is below 0.05, we reject the null hypothesis, implying that the series is stationary.
  
### Autocorrelation Test

Once stationarity is confirmed, we check for autocorrelation in the **squared returns**. The presence of autocorrelation in squared returns indicates volatility clustering, a key sign of ARCH effects.

To test for autocorrelation, we can use the **Ljung-Box Q-test**. This test checks for autocorrelation at multiple lags.

```python
from statsmodels.stats.diagnostic import acorr_ljungbox

# Check autocorrelation in squared returns
ljung_box_test = acorr_ljungbox(returns**2, lags=[10], return_df=True)

print(ljung_box_test)
```
```bash
         lb_stat  lb_pvalue
10  34331.948673        0.0
```

A significant p-value (below 0.05) for the Ljung-Box test means that there is significant autocorrelation in the squared returns, suggesting the presence of ARCH effects.

### Normality Test

The **Jarque-Bera test** assesses whether the skewness and kurtosis of the series significantly deviate from those of a normal distribution. The null hypothesis is that the data follows a normal distribution.

The test checks both **skewness** and **kurtosis**:

- **Skewness** indicates asymmetry in the distribution.
- **Kurtosis** measures the tail heaviness of the distribution.

A high Jarque-Bera test statistic suggests that the series is not normally distributed, which is often the case with financial returns, where we observe fat tails and non-symmetric behavior.

```python
from scipy.stats import jarque_bera

# Perform Jarque-Bera test on returns
jb_test_stat, jb_p_value = jarque_bera(returns)

print(f'Jarque-Bera Test Statistic: {jb_test_stat}')
print(f'P-value: {jb_p_value}')
```
```bash
Jarque-Bera Test Statistic: 85728124.76566969
P-value: 0.0
```

A **p-value** lower than 0.05 indicates that we reject the null hypothesis, implying that the returns do not follow a normal distribution. This further strengthens the case for using models like GARCH, which can handle non-normal characteristics such as fat tails and volatility clustering.

## Model Selection

We now have to select the p (the lag of past volatilities) and q (the lag of past squared returns) orders of our GARCH model. Goal here is thus to find the configuration that best fits the data. To do this, we'll estimate multiple models by varying the parameters p and q in the GARCH model, ranging from simple to more complex configurations, and compare their AIC and BIC values. The model with the **lowest [Akaike Information Criterion](https://en.wikipedia.org/wiki/Akaike_information_criterion) (AIC)** is generally favored when focusing on model fit, while the **lowest [Bayesian Information Criterion](https://en.wikipedia.org/wiki/Bayesian_information_criterion) (BIC)** is preferred when penalizing more complex models to avoid overfitting.

Some pratical considerations now : 

1. We’ll use Python’s `arch` package to fit the GARCH model.
2. We'll apply a **scaling factor** of 1440 to the log returns.
3. We'll we used a **zero mean** assumption for the log returns.

### Why scale the data ? 

This is necessary because GARCH models tend to perform better when the input data is scaled to more manageable levels, especially with high-frequency data like minute-by-minute returns. Without scaling, the small magnitude of log returns can lead to numerical instability in the estimation process. Scaling helps to improve the model's convergence and interpretability of the parameter estimates. Here we choose to multiply the returns by the number of minutes in the day : 1440.

### Why assume a zero drift ?

The argument I will give here was presented by Collin Bennett in his book *Trading Volatility*. Bennett explains that when calculating volatility, it is often **best to assume zero drift**. The calculation for standard deviation measures the deviation from the average log return (drift), which must be estimated from the sample. This estimation can lead to misleading volatility calculations if the sample period includes unusually high or negative returns. 

For example, if a stock rises by 10% every day for ten days, the standard deviation would be zero because there is no deviation from the 10% average return. However, such trends are unrealistic over the long term, and using the sample log return as the expected future return can distort the volatility estimate. By assuming a **zero mean** or drift, we prevent the volatility calculation from being influenced by extreme sample returns. In theory, over the long term, the expected return should be close to zero, as the forward price of an asset should reflect this assumption. This is why volatility calculations are typically more accurate when assuming **zero drift**.

### Results

```python
from arch import arch_model

scaling_factor = 60*24 # We get daily returns
min_bic = 1e10
selected_orders = (0, 0)

for p in range(1, 4):
    for q in range(0, 4):
        # GARCH(p,q) model
        model = arch_model(
            df['log_returns']*scaling_factor, 
            mean='Zero', 
            vol='GARCH', 
            p=p, q=q
            )
        res = model.fit(disp='off', last_obs=split_date,)
        print(f'GARCH({p},{q}) AIC: {res.aic}, BIC: {res.bic}')
        if res.bic < min_bic:
            min_bic = res.bic
            selected_orders = (p, q)
p, q = selected_orders
print(f'Selected model with BIC : p={p} and q={q}')
```
```bash
GARCH(1,0) AIC: 269793.01931055624, BIC: 269812.558147921
GARCH(1,1) AIC: 245265.1323021801, BIC: 245294.44055822722
GARCH(1,2) AIC: 244480.07736779848, BIC: 244519.15504252794
GARCH(1,3) AIC: 244168.23045229167, BIC: 244217.0775457035
GARCH(2,0) AIC: 260475.38690308636, BIC: 260504.69515913347
GARCH(2,1) AIC: 245268.00954894855, BIC: 245307.087223678
GARCH(2,2) AIC: 244482.07737045927, BIC: 244530.9244638711
GARCH(2,3) AIC: 244170.230456519, BIC: 244228.8469686132
GARCH(3,0) AIC: 255222.99234444185, BIC: 255262.0700191713
GARCH(3,1) AIC: 245269.1326656593, BIC: 245317.97975907114
GARCH(3,2) AIC: 244484.07805419483, BIC: 244542.69456628902
GARCH(3,3) AIC: 244172.23049541027, BIC: 244240.61642618684
Selected model with BIC : p=1 and q=3
```

After evaluating multiple GARCH(p, q) models, we first select the **GARCH(1, 3)** model based on both information criterions. Let's take a look at the estimated parameters then :

```python
# GARCH(1, 3) model
model = arch_model(
    df['log_returns']*scaling_factor,
    mean='Zero',
    vol='GARCH',
    p=p, q=q
    )
res = model.fit(last_obs=split_date, disp='off')
print(res.summary())
```
```bash
                       Zero Mean - GARCH Model Results                        
==============================================================================
Dep. Variable:            log_returns   R-squared:                       0.000
Mean Model:                 Zero Mean   Adj. R-squared:                  0.000
Vol Model:                      GARCH   Log-Likelihood:               -122079.
Distribution:                  Normal   AIC:                           244168.
Method:            Maximum Likelihood   BIC:                           244217.
                                        No. Observations:               129239
Date:                Thu, Sep 26 2024   Df Residuals:                   129239
Time:                        10:34:30   Df Model:                            0
                              Volatility Model                              
============================================================================
                 coef    std err          t      P>|t|      95.0% Conf. Int.
----------------------------------------------------------------------------
omega          0.0180  3.464e-03      5.200  1.991e-07 [1.122e-02,2.480e-02]
alpha[1]       0.1996  1.736e-02     11.497  1.362e-30     [  0.166,  0.234]
beta[1]        0.3958      0.103      3.859  1.140e-04     [  0.195,  0.597]
beta[2]        0.1163      0.127      0.919      0.358     [ -0.132,  0.364]
beta[3]        0.2756  3.697e-02      7.455  9.010e-14     [  0.203,  0.348]
============================================================================

Covariance estimator: robust
```

This provides us with the key parameters of our model, that we can analyze now !

## Interpreting the GARCH Parameters

The **GARCH(1, 3)** model provides several important parameters that give insight into Bitcoin’s volatility. Below is a breakdown of each key parameter and its interpretation:

<table style="width:100%; border-collapse:collapse;">
  <thead>
    <tr>
      <th style="text-align:center; vertical-align:middle;"><strong>Parameter</strong></th>
      <th style="text-align:center; vertical-align:middle;"><strong>Value</strong></th>
      <th style="text-align:center; vertical-align:middle;"><strong>Description</strong></th>
      <th style="text-align:center; vertical-align:middle;"><strong>Interpretation</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center; vertical-align:middle;">$\omega$</td>
      <td style="text-align:center; vertical-align:middle;">0.0180</td>
      <td style="text-align:center; vertical-align:middle;">Represents the <strong>long-term or baseline variance</strong>.</td>
      <td style="text-align:center; vertical-align:middle;">A small, positive, and significant $\omega$ indicates a constant underlying volatility in the returns. Spikes in volatility are largely driven by recent shocks and historical persistence.</td>
    </tr>
    <tr>
      <td style="text-align:center; vertical-align:middle;">$\alpha_1$</td>
      <td style="text-align:center; vertical-align:middle;">0.1996</td>
      <td style="text-align:center; vertical-align:middle;">Captures the impact of <strong>recent squared returns</strong> ($r_{t-1}^2$) on current volatility.</td>
      <td style="text-align:center; vertical-align:middle;">About 20% of recent price movements influence current volatility. This suggests that large, sudden price changes significantly impact volatility.</td>
    </tr>
    <tr>
      <td style="text-align:center; vertical-align:middle;">$\beta_1$</td>
      <td style="text-align:center; vertical-align:middle;">0.3958</td>
      <td style="text-align:center; vertical-align:middle;">Reflects the <strong>persistence of past volatility</strong> on current volatility.</td>
      <td style="text-align:center; vertical-align:middle;">Around 40% of previous volatility persists into the current period. This explains the <strong>volatility clustering</strong> phenomenon, where high volatility tends to follow high volatility.</td>
    </tr>
    <tr>
      <td style="text-align:center; vertical-align:middle;">$\beta_2$</td>
      <td style="text-align:center; vertical-align:middle;">0.1163</td>
      <td style="text-align:center; vertical-align:middle;">Represents the influence of the <strong>second lag of volatility</strong> on current volatility.</td>
      <td style="text-align:center; vertical-align:middle;">$\beta_2$ is <strong>not statistically significant</strong> (p-value = 0.358), suggesting the second lag has a minimal effect on current volatility.</td>
    </tr>
    <tr>
      <td style="text-align:center; vertical-align:middle;">$\beta_3$</td>
      <td style="text-align:center; vertical-align:middle;">0.2756</td>
      <td style="text-align:center; vertical-align:middle;">Measures the impact of the <strong>third lag of volatility</strong> on current volatility.</td>
      <td style="text-align:center; vertical-align:middle;">The third lag is statistically significant with a <strong>t-statistic</strong> of <strong>7.455</strong>. This reinforces the idea that Bitcoin volatility has a <strong>long memory</strong>.</td>
    </tr>
  </tbody>
</table>

### Key Insights:

1. **Long-term volatility ($\omega$)** is relatively low, suggesting that Bitcoin’s baseline volatility is not extreme but driven by recent market shocks.
2. **Recent shocks ($\alpha_1$)** have a notable effect, with around 20% of volatility attributed to recent returns.
3. **Volatility persistence ($\beta_1$)** is high, reinforcing the **clustering of volatility** that is often observed in financial markets.
4. **Second lag ($\beta_2$)** is not significant, indicating that further-back periods do not heavily influence the current volatility.
5. **Third lag ($\beta_3$)** is significant, meaning that some past volatility still impacts the current market, showcasing Bitcoin’s long volatility memory.

## Goodness-of-fit Check 

After fitting our selected GARCH model, it is however highly recommended to perform a **goodness-of-fit check** on the residuals to ensure that the model adequately captures the dynamics of your time series. The main goal of such checks is to assess whether the model residuals behave as expected—ideally, they should resemble white noise, meaning they have no autocorrelation and constant variance (homoscedasticity). We will work on the standardized residuals 

```python
# Standardized residuals from the GARCH model
std_residuals = res.resid / res.conditional_volatility
# Because of the last obs arguments we have NaNs
std_residuals = std_residuals.dropna()
```

Let's vizualize those residuals :

```python
# Plot the standardized residuals from the GARCH model
plt.figure(figsize=(12, 6))
std_residuals.plot()
plt.title('Standardized residuals from the GARCH model')
plt.xlabel('Date')
plt.ylabel('Standardized residuals')
plt.show()
```

![figure 3](/blog/images/BitcoinVolatility-2-figure-3.png)

At first glance, the situation looks concerning: we observe clear volatility clustering, with high volatility in the initial months, and numerous extreme values, suggesting the residuals may deviate from normality. 

<div style="text-align: center;">
We now need to realize some <strong>rigorous statistical testing</strong> to confirm that.
</div>

![figure 4](/blog/images/here_we_go_again.png)

### Autocorrelation of Residuals

The standardized residuals (the residuals divided by their estimated volatility) should no longer show any significant autocorrelation. We can here again use the **Ljung-Box Q-test** to test whether there is any remaining autocorrelation in the residuals or squared residuals.

```python
# Ljung-Box test for residuals (no autocorrelation should be present)
ljung_box_res = acorr_ljungbox(std_residuals, lags=[10], return_df=True)
print('Ljung-Box test for residuals')
print(ljung_box_res)

# Ljung-Box test for squared residuals (no remaining ARCH effects should be present)
ljung_box_sq_res = acorr_ljungbox(std_residuals**2, lags=[10], return_df=True)
print('Ljung-Box test for squared residuals')
print(ljung_box_sq_res)
```
```bash
Ljung-Box test for residuals
       lb_stat     lb_pvalue
10  163.434818  6.326210e-30

Ljung-Box test for squared residuals
      lb_stat  lb_pvalue
10  19.082143   0.039232
```

The **p-values** are bellow 0.05, this indicates that there is statistically significant autocorrelation in the residuals or squared residuals, suggesting that the GARCH model has not effectively captured all the volatility structure.

### Normality of Residuals 

The standardized residuals should follow a normal distribution. The **Jarque-Bera test** can be used for this:

```python
from scipy.stats import jarque_bera

# Perform Jarque-Bera test on standardized residuals
jb_stat, jb_p_value = jarque_bera(std_residuals)
print(f'Jarque-Bera Test Statistic: {jb_stat}')
print(f'P-value: {jb_p_value}')
```
```bash
Jarque-Bera Test Statistic: 5551677.823833454
P-value: 0.0
```

The **p-value** bellow 0.05 indicates that the residuals do significantly deviate from normality. 

After running both test, the results indicate that **the model’s residuals have issues**. 

### What Might Actually Improve the Model?

To improve the fit of the GARCH model, we could consider several adjustments. The following table outlines possible strategies for improving the model:

<table style="width:100%; border-collapse:collapse;">
  <thead>
    <tr>
      <th style="text-align:center; vertical-align:middle;"><strong>Adjustment</strong></th>
      <th style="text-align:center; vertical-align:middle;"><strong>Description</strong></th>
      <th style="text-align:center; vertical-align:middle;"><strong>Benefit</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center; vertical-align:middle;"><strong>Use a higher-order GARCH</strong></td>
      <td style="text-align:center; vertical-align:middle;">Increase the number of lags in the GARCH terms, such as moving to <strong>GARCH(2, 3)</strong>. This adjustment could better capture the autocorrelation if volatility has a longer memory, thereby modeling both short-term and long-term volatility effects more effectively.</td>
      <td style="text-align:center; vertical-align:middle;">Provides more flexibility in capturing extended memory in the volatility series, which could improve the model’s fit in markets with persistent volatility patterns.</td>
    </tr>
    <tr>
      <td style="text-align:center; vertical-align:middle;"><strong>Use an EGARCH or GJR-GARCH model</strong></td>
      <td style="text-align:center; vertical-align:middle;">A <strong>GARCH variant</strong> like <strong>EGARCH</strong> or <strong>GJR-GARCH</strong> can account for <strong>asymmetric volatility</strong> (where negative returns have a greater effect on volatility than positive returns).</td>
      <td style="text-align:center; vertical-align:middle;">Improves performance in modeling volatility clustering in assets with skewed returns, like Bitcoin.</td>
    </tr>
    <tr>
      <td style="text-align:center; vertical-align:middle;"><strong>Change the residual distribution</strong></td>
      <td style="text-align:center; vertical-align:middle;">If the residuals deviate from normality (as indicated by the <strong>Jarque-Bera test</strong>), switch from a normal distribution to a <strong>Student’s t-distribution</strong> or <strong>Generalized Error Distribution (GED)</strong> for the residuals.</td>
      <td style="text-align:center; vertical-align:middle;">Better reflects the <strong>non-normal</strong> characteristics of financial time series, particularly fat tails, improving the model's ability to handle extreme events and outliers.</td>
    </tr>
  </tbody>
</table>

But should we really bother ?

### Should We Discard the Current GARCH Model?

Although the Ljung-Box tests for residuals and squared residuals show statistically significant autocorrelation, and the Jarque-Bera test indicates that the residuals deviate from normality, these results do not necessarily invalidate the use of the GARCH model. This indeed does not imply that the model is entirely ineffective. GARCH models are specifically designed to capture **conditional heteroskedasticity**—volatility that changes over time based on past returns and variances. While the remaining autocorrelation suggests there are still some dynamics unexplained by the model, the GARCH framework remains usefull to capture volatility clustering.

Even though the residuals do not follow a perfect normal distribution, the GARCH model can still provide valuable insights by capturing time-varying volatility. If we expect fat-tailed distributions, we could enhance the model by specifying a **non-normal distribution** (e.g., Student's t-distribution) for the residuals, which might better reflect the empirical properties of the data.

Financial time series are notoriously difficult to model, and violations of normality or remaining autocorrelation are expected in complex markets like cryptocurrencies. GARCH remains a valuable tool in capturing the essential dynamics of volatility, and with potential enhancements—such as using a different distribution for the residuals or exploring higher-order GARCH models—we can further refine the model for better performance.

In summary, while the current model may not be perfect, it provides a solid foundation for understanding volatility patterns. Refining the model rather than discarding it will likely lead to more accurate and insightful results. Thus, we will keep it that way for the moment, and probably improve it in a latter post.

## Estimating and Forecasting Volatility

Once we have select and fit one model, we can use it both to estimate realized volatility and for forecast. 

### Estimating Realized Volatility

In this subsection, we'll estimate **realized volatility** using our **GARCH model** and compare it to an alternative method like the **EWMA** estimator. This comparison will give us a clearer picture of how well the GARCH model captures the dynamics of Bitcoin’s volatility compared to simpler approaches.

Using the following code, we extract and plot the **conditional volatility** from our GARCH model:

```python
# Get conditional volatility
cond_vol = res.conditional_volatility
cond_vol.dropna(inplace=True)

# Plot the conditional volatility
plt.figure(figsize=(12, 6))
cond_vol.plot()
plt.title(f'Conditional Volatility from GARCH({p},{q}) Model')
plt.xlabel('Date')
plt.ylabel('Volatility')
plt.show()
```

![figure 5](/blog/images/BitcoinVolatility-2-figure-4.png)

The plot shows how volatility fluctuates over time, capturing periods of heightened risk (volatility clustering) and more stable periods. 

The **Exponentially Weighted Moving Average (EWMA)** model is another method often used to estimate realized volatility, giving more weight to recent observations. To compute and compare the EWMA volatility, we can use the following code:

```python
# Compute EWMA volatility
lambda_ = 0.94  # Decay factor as used in RiskMetrics
ewma_vol = (scaling_factor*returns).ewm(alpha=(1 - lambda_), adjust=False).std()

# Create a figure with two subplots (2 rows, 1 column)
fig, axs = plt.subplots(2, 1, figsize=(12, 12))

# First subplot: GARCH vs EWMA Volatility
axs[0].plot(cond_vol, label='GARCH Conditional Volatility', color='blue')
axs[0].plot(ewma_vol, label='EWMA Volatility', color='orange')
axs[0].set_title('GARCH vs. EWMA Volatility')
axs[0].set_xlabel('Date')
axs[0].set_ylabel('Volatility')
axs[0].legend()

# Second subplot: GARCH vs EWMA Volatility Ratio
axs[1].plot((cond_vol - ewma_vol) / ewma_vol, label='GARCH Conditional Volatility vs EWMA Volatility Ratio', color='blue')
axs[1].plot(((cond_vol - ewma_vol) / ewma_vol).rolling(60*24).mean(), label='Rolling Mean', color='red')
axs[1].set_title('GARCH vs. EWMA Volatility Ratio')
axs[1].set_xlabel('Date')
axs[1].set_ylabel('Volatility Ratio')
axs[1].legend()

# Adjust layout to prevent overlap
plt.tight_layout()
# Display the figure
plt.show()
```

![figure 6](/blog/images/BitcoinVolatility-2-figure-5.png)

   - **Observation**: The GARCH model consistently shows higher volatility estimates compared to the EWMA model.
   - **Explanation**: This is a typical feature of the GARCH model, which captures both the recent volatility and its persistence over time. Unlike EWMA, which gives more weight to recent data while smoothing out fluctuations, GARCH is designed to account for longer-term volatility clustering, making it more responsive to periods of heightened market stress.

   - **Observation**: The EWMA model displays smoother, more stable volatility estimates over time.
   - **Explanation**: EWMA assigns exponentially decreasing weights to past data, which dampens the impact of older, more volatile periods. This explains why EWMA produces lower volatility values, as it focuses more on recent changes without accounting for the persistent volatility that the GARCH model captures.

### Forecasting Volatility 

The **GARCH(1,3)** model allows us to generate conditional forecasts over future periods. Let's expirement forecasting the volatility on unseen data.

```python
forecasts = res.forecast(horizon=60, start=split_date)
forecasted_variance = forecasts.variance
forecasted_volatility = np.sqrt(forecasted_variance)    

# Plot the conditional volatility
plt.figure(figsize=(12, 6))
forecasted_volatility["h.60"].plot()
plt.title(f'Forecast Volatility from GARCH({p},{q}) Model')
plt.xlabel('Date')
plt.ylabel('Volatility')
plt.show()
```

![figure 6](/blog/images/BitcoinVolatility-2-figure-6.png)

The plot shows how volatility is expected to behave over the next 60 minutes. Spikes in forecasted volatility would indicate heightened market risk, while a more stable line would suggest a calmer market.

## Conclusion

In this post, we explored the use of the **GARCH model** for estimating and forecasting Bitcoin's volatility. Compared to simpler models like **EWMA**, GARCH provides a more robust framework for capturing the dynamic nature of financial markets, particularly in handling **volatility clustering** and **fat tails**—two key characteristics often present in cryptocurrency markets.

We began by discussing the motivations for moving beyond simple models, then delved into the mechanics of the **GARCH(1, 3)** model, explaining how it captures both recent volatility shocks and their persistence. We demonstrated how to fit the GARCH model, perform essential **goodness-of-fit tests**, and interpret the resulting parameters. We encountered some issues with the residuals, but despite these imperfections our GARCH model still provides valuable insights into Bitcoin's volatility structure and that potential improvements, like changing the residual distribution or testing higher-order models, could further refine the model.

We showed how the GARCH model can be used both to estimate **realized volatility** and to generate **volatility forecasts** for the next hour. This ability to project future volatility is incredibly useful for traders, risk managers, and anyone involved in high-frequency trading environments where knowing short-term risk is critical.

In the next post, we will dive deeper into volatility forecasting by exploring alternative data points like Low and High. Stay tuned for the final part of our series as we continue to explore the fascinating world of volatility modeling for Bitcoin !

## Additional Resources

- **Code Repository**: [GitHub Link](https://github.com/Zaltarba/BitcoinVolatilityEstimation/tree/main) 
- **Adviced Reading**: John Hull's *Options, Futures, and Other Derivatives* and Collin Bennet's *Trading Volatility*

Feel free to check out the GitHub repository for the complete code and try experimenting with different parameters to see how they affect volatility estimates.

I'd love to hear your thoughts! Share your feedback, questions, or suggestions for future topics in the comments below.

**Happy analyzing**, and as always, may your trading strategies be **well-informed**!
