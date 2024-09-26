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

The summary output from fitting the **GARCH(1, 3)** model provides us with the estimated values of key parameters that offer insights into Bitcoin’s volatility dynamics. Let’s break them down:

1. **$\omega = 0.0180$**:
   - **Interpretation**: The parameter $\omega$ represents the long-term or baseline variance. A positive and significant value for $\omega$ (with a **p-value** much less than 0.05) indicates that there is a constant underlying level of volatility present in the returns. This forms the base level of volatility around which the time-varying volatility fluctuates. The relatively small value here suggests that although Bitcoin is volatile, its baseline variance is not excessively high, implying that spikes in volatility are largely driven by recent shocks and historical persistence.

2. **$\alpha_1 = 0.1996$**:
   - **Interpretation**: $\alpha_1$ captures the influence of recent squared returns ($r_{t-1}^2$) on current volatility. A value of **0.1996** shows that about 20% of the most recent market shock or price movement contributes to current volatility. This is quite significant, as it means that sudden, large price changes in Bitcoin will cause a noticeable increase in volatility. The highly significant **t-statistic** of **11.497** confirms that recent price movements strongly influence future volatility.

3. **$\beta_1 = 0.3958$**:
   - **Interpretation**: $\beta_1$ reflects the persistence of volatility. A value of **0.3958** indicates that about 40% of the previous period's volatility persists into the current period. This parameter highlights the **volatility clustering** effect: periods of high volatility are likely to be followed by more high-volatility periods. The significant **t-statistic** suggests that the impact of past volatility is substantial and long-lasting in Bitcoin's price series.

4. **$\beta_2 = 0.1163$**:
   - **Interpretation**: Although $\beta_2$ represents the influence of the second lag of volatility, its **p-value** of **0.358** shows that this term is not statistically significant. Therefore, this term contributes minimally to the model, and the second lagged volatility has little influence on current volatility in this dataset.

5. **$\beta_3 = 0.2756$**:
   - **Interpretation**: The parameter $\beta_3$ is also statistically significant, with a **t-statistic** of **7.455** and a **p-value** close to zero. This indicates that even the third lag of volatility plays a role in determining current volatility. The impact of this third lag shows that Bitcoin volatility exhibits persistence beyond just the immediate past, reinforcing the long memory often observed in financial time series.

## Goodness-of-fit Check 

After fitting our selected GARCH model, it is highly recommended to perform a **goodness-of-fit check** on the residuals to ensure that the model adequately captures the dynamics of your time series. The main goal of such checks is to assess whether the model residuals behave as expected—ideally, they should resemble white noise, meaning they have no autocorrelation and constant variance (homoscedasticity). We will work on the standardized residuals 

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

At first glance, the situation looks concerning: we observe clear volatility clustering, with high volatility in the initial months, and numerous extreme values, suggesting the residuals may deviate from normality. Let’s confirm this with rigorous statistical testing.

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

The **p-value** bellow 0.05 indicates that the residuals do significantly deviate from normality. 

After running the goodness-of-fit checks, the results indicate that the model’s residuals may still have issues. However, it’s possible to justify the continued use of the GARCH model even if all assumptions are not perfectly met. Here's how you can structure the follow-up:

### What to do then ?

Although the Ljung-Box tests for residuals and squared residuals show statistically significant autocorrelation (\(p < 0.05\)), and the Jarque-Bera test indicates that the residuals deviate from normality, these results do not necessarily invalidate the use of the GARCH model.

The significant autocorrelation in the residuals suggests that the GARCH model has not fully captured all the volatility patterns in the data. However, this does not imply that the model is entirely ineffective. GARCH models are specifically designed to capture **conditional heteroskedasticity**—volatility that changes over time based on past returns and variances. While the remaining autocorrelation suggests there are still some dynamics unexplained by the model, the GARCH framework remains one of the most robust tools for capturing volatility clustering, a key feature of financial markets like Bitcoin.

Additionally, it's possible that using an extended model, such as **EGARCH** (which captures asymmetric effects) or a **GARCH(1, 2)** or **GARCH(2, 1)**, may help address the autocorrelation issues. However, even with some remaining autocorrelation, GARCH still provides a more accurate representation of volatility compared to simpler models that assume constant variance.

The Jarque-Bera test shows that the residuals deviate from normality, which is quite common with financial applications. Financial returns, especially in highly volatile markets like Bitcoin, often exhibit **fat tails** (excess kurtosis) and **skewness**, which deviate from the assumptions of normality. This behavior is well-documented and consistent with the observed **non-Gaussian** nature of financial returns.

Even though the residuals do not follow a perfect normal distribution, the GARCH model can still provide valuable insights by capturing time-varying volatility. If we expect fat-tailed distributions, we could enhance the model by specifying a **non-normal distribution** (e.g., Student's t-distribution) for the residuals, which might better reflect the empirical properties of the data.

While the assumption checks indicate some limitations, GARCH models still offer several advantages:
- **Volatility Clustering**: GARCH is uniquely equipped to handle volatility clustering, which simpler models like moving averages or constant volatility models cannot capture.
- **Risk Management**: Even if the residuals are not perfectly normal, GARCH provides a much better framework for measuring and managing risk, particularly in volatile markets like cryptocurrencies.
- **Forecasting Accuracy**: Despite some residual issues, GARCH models often outperform simpler models in terms of volatility forecasting accuracy. The ability to dynamically adjust volatility estimates based on recent data makes GARCH models highly useful for real-world applications, including portfolio management, options pricing, and risk assessment.

While the goodness-of-fit checks reveal some areas where the model could be improved, these results do not disqualify the use of the GARCH model. Financial time series are notoriously difficult to model perfectly, and violations of normality or remaining autocorrelation are expected in complex markets like cryptocurrencies. GARCH remains a valuable tool in capturing the essential dynamics of volatility, and with potential enhancements—such as using a different distribution for the residuals or exploring higher-order GARCH models—we can further refine the model for better performance.

This approach justifies continuing with the GARCH model by acknowledging its limitations while emphasizing its practical usefulness in capturing important volatility dynamics.

You're absolutely correct to question why a **GARCH(1, 2)** model might solve the issue, given that the current model is already **GARCH(1, 3)**. The suggestion of trying **GARCH(1, 2)** or **GARCH(2, 1)** typically comes from a process of **model refinement**, but in this case, simply reducing the number of lags in volatility would not directly address the autocorrelation issue if a higher-lag model like **GARCH(1, 3)** has already been fitted.

### Why **GARCH(1, 2)** or **GARCH(2, 1)** might not solve the issue

The issue you're encountering—remaining autocorrelation in the residuals and squared residuals—suggests that the current **GARCH(1, 3)** model is not fully capturing the volatility dynamics, but **reducing** the lag from 3 to 2 in **GARCH(1, 2)** may actually worsen the situation by omitting relevant lagged information that the model needs to capture.

In fact, given that the model already has three lags for the volatility terms (\(\beta_j\)), decreasing the number of lags is unlikely to improve the model’s fit. If the issue lies in the residuals or ARCH effects, then simply switching to a lower-lag specification like **GARCH(1, 2)** may not resolve the problem, and it could even reduce the model's ability to capture longer-term volatility dynamics.

### What might actually improve the model?

To address the remaining autocorrelation and improve the model fit, there are more appropriate adjustments you could consider:

**Consider a GARCH(2, 3) or higher-order GARCH**: Increasing the number of lags in the GARCH terms (i.e., moving to **GARCH(2, 3)** or **GARCH(2, 2)**) could better capture the autocorrelation if volatility has a longer memory. This would allow for more flexibility in modeling both short-term and longer-term volatility effects.

**Use an EGARCH or GJR-GARCH model**: If the market exhibits **asymmetry** in volatility (i.e., large negative returns cause higher volatility than large positive returns), a **GARCH** model may not fully capture these dynamics. An **EGARCH** (Exponential GARCH) or **GJR-GARCH** model can account for this asymmetry, and they often perform better in capturing volatility clustering and shocks in markets like Bitcoin.

**Change the residual distribution**: If the **Jarque-Bera** test shows that the residuals deviate significantly from normality, consider using a **Student’s t-distribution** or **Generalized Error Distribution (GED)** for the residuals. This change can help capture the fat tails observed in financial time series more effectively than the standard normal distribution.

   ```python
   model = arch_model(df['log_returns'], vol='Garch', p=1, q=3, dist='t')
   res = model.fit(disp='off')
   print(res.summary())
   ```

**Re-evaluate data transformations or pre-processing**: Sometimes, residual autocorrelation can be a sign that the data itself is not fully stationary or that further transformations (such as using logarithmic differences or returns) are needed. Double-checking that the series is adequately pre-processed (stationary) is critical.

In summary, reducing the number of lags in the volatility equation from **GARCH(1, 3)** to **GARCH(1, 2)** may not necessarily resolve the autocorrelation issue. Instead, increasing the order of the GARCH terms, using an asymmetric model like **EGARCH**, or adopting a different residual distribution (e.g., Student’s t-distribution) are more promising solutions. These alternatives will allow the model to better handle complex volatility dynamics and deviations from normality, particularly in volatile assets like Bitcoin.

## Forecasting Volatility Using GARCH

Once we have select and fit one model, we can use it to forecast future volatility. The **GARCH(1,3)** model allows us to generate conditional forecasts for the next few periods.

```python
# Forecast future volatility
forecasts = garch_fit.forecast(horizon=60)
print(forecasts.variance[-1:])
```
```bash
Jarque-Bera Test Statistic: 5551677.823833454
P-value: 0.0
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

