---
layout: post
title: Estimating Bitcoin's Volatility using a GARCH Model
categories: [Algo Trading, Personal Project, Quantitative Analysis]
excerpt: Dive into the second part of our three-part series on computing Bitcoin's volatility with Binance data.
image: /images/BayesianClustering.png
hidden: true
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

## Implementing our Model 

We’ll use Python’s `arch` package to fit the GARCH model.

```python
from arch import arch_model
```

### Model Selection

In this section, we will systematically evaluate several different GARCH model configurations to find the one that best fits the data. To do this, we'll estimate multiple models by varying the parameters $ p $ (the lag of past volatilities) and $ q $ (the lag of past squared returns) in the GARCH model.

We will evaluate several combinations of GARCH(p, q) models, ranging from simple to more complex configurations, and compare their AIC and BIC values. The model with the **lowest [Akaike Information Criterion](https://en.wikipedia.org/wiki/Akaike_information_criterion) (AIC)** is generally favored when focusing on model fit, while the **lowest [Bayesian Information Criterion](https://en.wikipedia.org/wiki/Bayesian_information_criterion) (BIC)** is preferred when penalizing more complex models to avoid overfitting.

```python
from arch import arch_model
scaling_factor = 1000
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
```
```bash
GARCH(1,0) AIC: 175540.79659658077, BIC: 175560.3354339455
GARCH(1,1) AIC: 151662.94092316425, BIC: 151692.24917921136
GARCH(1,2) AIC: 150227.85465395244, BIC: 150266.9323286819
GARCH(1,3) AIC: 150746.29683750868, BIC: 150795.14393092052
GARCH(2,0) AIC: 166223.16418857573, BIC: 166252.47244462284
GARCH(2,1) AIC: 151015.75193265954, BIC: 151054.829607389
GARCH(2,2) AIC: 153347.75328247808, BIC: 153396.60037588992
GARCH(2,3) AIC: 149918.00773812976, BIC: 149976.62425022395
GARCH(3,0) AIC: 160970.76963022564, BIC: 161009.8473049551
GARCH(3,1) AIC: 155776.01361340753, BIC: 155824.86070681937
GARCH(3,2) AIC: 155058.9626537681, BIC: 155117.5791658623
GARCH(3,3) AIC: 153747.4205368927, BIC: 153815.80646766926
```

In the model selection process, we also applied a **scaling factor** of 1000 to the log returns. This is necessary because GARCH models tend to perform better when the input data is scaled to more manageable levels, especially with high-frequency data like minute-by-minute returns. Without scaling, the small magnitude of log returns can lead to numerical instability in the estimation process. Scaling helps to improve the model's convergence and interpretability of the parameter estimates.

After evaluating multiple GARCH(p, q) models, we select the **GARCH(2, 3)** model based on the **Akaike Information Criterion (AIC)**. The GARCH(2, 3) model has the lowest AIC value, indicating that it provides the best balance between goodness of fit and model complexity. Since the AIC penalizes models with more parameters but still favors those that explain the data well, selecting the model with the lowest AIC helps ensure we avoid overfitting while capturing the essential volatility dynamics in Bitcoin’s returns.

### Model Selection 

We’ll use Python’s arch package to fit the GARCH model. The **ARCH** (Autoregressive Conditional Heteroskedasticity) package is particularly useful for estimating financial volatility models like GARCH.

```python
from arch import arch_model
scaling_factor = 1000
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
```
```bash
GARCH(1,0) AIC: 175540.79659658077, BIC: 175560.3354339455
GARCH(1,1) AIC: 151662.94092316425, BIC: 151692.24917921136
GARCH(1,2) AIC: 150227.85465395244, BIC: 150266.9323286819
GARCH(1,3) AIC: 150746.29683750868, BIC: 150795.14393092052
GARCH(2,0) AIC: 166223.16418857573, BIC: 166252.47244462284
GARCH(2,1) AIC: 151015.75193265954, BIC: 151054.829607389
GARCH(2,2) AIC: 153347.75328247808, BIC: 153396.60037588992
GARCH(2,3) AIC: 149918.00773812976, BIC: 149976.62425022395
GARCH(3,0) AIC: 160970.76963022564, BIC: 161009.8473049551
GARCH(3,1) AIC: 155776.01361340753, BIC: 155824.86070681937
GARCH(3,2) AIC: 155058.9626537681, BIC: 155117.5791658623
GARCH(3,3) AIC: 153747.4205368927, BIC: 153815.80646766926
```

In my GARCH model specification, I used a **zero mean** assumption, following the argument presented by Collin Bennett in his book *Trading Volatility*. Bennett explains that when calculating volatility, it is often **best to assume zero drift**. 

The calculation for standard deviation measures the deviation from the average log return (drift), which must be estimated from the sample. This estimation can lead to misleading volatility calculations if the sample period includes unusually high or negative returns. For example, if a stock rises by 10% every day for ten days, the standard deviation would be zero because there is no deviation from the 10% average return. However, such trends are unrealistic over the long term, and using the sample log return as the expected future return can distort the volatility estimate.

By assuming a **zero mean** or drift, we prevent the volatility calculation from being influenced by extreme sample returns. In theory, over the long term, the expected return should be close to zero, as the forward price of an asset should reflect this assumption. This is why volatility calculations are typically more accurate when assuming **zero drift**.

After evaluating multiple GARCH(p, q) models, we select the **GARCH(2, 3)** model based on the **Akaike Information Criterion (AIC)**. The GARCH(2, 3) model has the lowest AIC value, indicating that it provides the best balance between goodness of fit and model complexity. Since the AIC penalizes models with more parameters but still favors those that explain the data well, selecting the model with the lowest AIC helps ensure we avoid overfitting while capturing the essential volatility dynamics in Bitcoin’s returns.

```python
# GARCH(2, 3) model
model = arch_model(df['log_returns'] * 1000, mean='Zero', vol='GARCH', p=2, q=3)
res = model.fit(last_obs=split_date, disp='off')
print(res.summary())
```
```bash
                       Zero Mean - GARCH Model Results                        
==============================================================================
Dep. Variable:            log_returns   R-squared:                       0.000
Mean Model:                 Zero Mean   Adj. R-squared:                  0.000
Vol Model:                      GARCH   Log-Likelihood:               -74953.0
Distribution:                  Normal   AIC:                           149918.
Method:            Maximum Likelihood   BIC:                           149977.
                                        No. Observations:               129239
Date:                Tue, Sep 24 2024   Df Residuals:                   129239
Time:                        14:58:27   Df Model:                            0
                               Volatility Model                              
=============================================================================
                 coef    std err          t      P>|t|       95.0% Conf. Int.
-----------------------------------------------------------------------------
omega      8.6867e-03  1.675e-03      5.186  2.148e-07  [5.404e-03,1.197e-02]
alpha[1]       0.1996  1.778e-02     11.229  2.925e-29      [  0.165,  0.234]
alpha[2]       0.0000  4.278e-02      0.000      1.000 [-8.384e-02,8.384e-02]
beta[1]        0.3958      0.256      1.549      0.121      [ -0.105,  0.897]
beta[2]        0.1163      0.192      0.607      0.544      [ -0.260,  0.492]
beta[3]        0.2756  6.526e-02      4.223  2.408e-05      [  0.148,  0.404]
=============================================================================

Covariance estimator: robust
```

Although the **GARCH(2, 3)** model was initially selected based on the lowest AIC and BIC values, upon inspecting the model's output, we observed that the **\( \alpha_2 \)** coefficient was not statistically significant (p-value = 1.000). This suggests that the second lag of the ARCH term does not meaningfully contribute to explaining the volatility dynamics. As a result, we opted to simplify the model by selecting a **GARCH(1, 3)** specification, which retains the significant coefficients while reducing unnecessary complexity. This simplification helps improve the interpretability of the model without sacrificing much in terms of fit.

```python
# GARCH(1, 3) model
model = arch_model(df['log_returns'] * scaling_factor, mean='Zero', vol='GARCH', p=1, q=3)
res = model.fit(last_obs=split_date, disp='off')
print(res.summary())
```
```bash
                       Zero Mean - GARCH Model Results                        
==============================================================================
Dep. Variable:            log_returns   R-squared:                       0.000
Mean Model:                 Zero Mean   Adj. R-squared:                  0.000
Vol Model:                      GARCH   Log-Likelihood:               -75368.1
Distribution:                  Normal   AIC:                           150746.
Method:            Maximum Likelihood   BIC:                           150795.
                                        No. Observations:               129239
Date:                Wed, Sep 25 2024   Df Residuals:                   129239
Time:                        10:49:40   Df Model:                            0
                              Volatility Model                              
============================================================================
                 coef    std err          t      P>|t|      95.0% Conf. Int.
----------------------------------------------------------------------------
omega      6.7208e-03  1.180e-03      5.694  1.239e-08 [4.408e-03,9.034e-03]
alpha[1]       0.2000  1.258e-02     15.903  6.074e-57     [  0.175,  0.225]
beta[1]        0.2600  4.694e-02      5.539  3.037e-08     [  0.168,  0.352]
beta[2]        0.2600  7.132e-02      3.645  2.671e-04     [  0.120,  0.400]
beta[3]        0.2600  3.244e-02      8.014  1.111e-15     [  0.196,  0.324]
============================================================================

Covariance estimator: robust
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
- **Bollerslev (1986)** demonstrated the success of GARCH models in explaining time-varying volatility in exchange rates.
- **Engle (1982)**, who pioneered the ARCH model (the predecessor to GARCH), showed how volatility clustering in financial markets can be modeled effectively with these frameworks.

Feel free to check out the GitHub repository for the complete code and try experimenting with different parameters to see how they affect volatility estimates.

Happy analyzing, and as always, may your trading strategies be well-informed!

