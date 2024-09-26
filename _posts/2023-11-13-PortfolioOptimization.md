---
layout: post
title: About Portfolio Optimization
categories: [Personnal Project]
excerpt: Let's talk about modern Portfolio Optimization !
hidden: true
---

# Portfolio Optimization: Balancing Risk and Return

## Introduction
Welcome to our comprehensive guide on portfolio optimization, a vital concept for anyone involved in the world of investing. Whether you're just starting as an investor or you're a seasoned financial professional, the ability to balance risk and return is crucial for financial success. This post aims to demystify the complex concepts of portfolio optimization, provide historical context, and delve into its mathematical foundations. Join us as we explore how these principles can guide smarter investment decisions.

## The History of Modern Portfolio Theory (MPT)
Modern Portfolio Theory (MPT), introduced by Harry Markowitz in his seminal 1952 paper "Portfolio Selection," revolutionized the approach to investment portfolio management. Markowitz's theory, which later garnered him a Nobel Prize in Economics, proposed a systematic and quantitative method to construct investment portfolios that could maximize returns for a given level of risk. This approach shifted the focus from individual asset selection to the interplay of assets within a portfolio, laying the groundwork for what we now understand as portfolio optimization.

## Basic Concepts of Investing
Before we delve into the depths of MPT, let's establish some foundational concepts in investing:

### Risk
In the realm of investing, risk refers to the possibility that the actual returns on an investment will be different from the expected outcomes, particularly when these returns are lower than expected. Risk is quantified as the standard deviation of the expected return, providing a measure of the volatility and uncertainty associated with an investment.

### Return
Return, on the other hand, represents the gain or loss on an investment over a specified period. It's typically expressed as a percentage increase or decrease over the initial investment cost and is the reward an investor seeks in compensation for taking on investment risk.

### Diversification
Diversification is a strategy employed to reduce risk in a portfolio. It involves spreading investments across a variety of assets to minimize the impact of any single asset's performance on the overall portfolio. The idea is that a decline in one asset can be offset by gains in another, thereby reducing the potential for significant losses.

## Calculating the Correlation Matrix

The correlation matrix is a key element in portfolio theory, measuring the extent to which two securities move in relation to each other. We explore several methods for calculating it, each with its own approach and nuances.

### Method 1: Historical Returns
#### Description
This method uses historical returns of assets to calculate the correlation coefficients.

#### Mathematical Details
- Calculate the returns for each asset over a set period.
- Use the formula: 
  $$
  \rho_{ij} = \frac{Cov(R_i, R_j)}{\sigma_i \sigma_j}
  $$
  where $Cov(R_i, R_j)$ is the covariance between the returns of assets i and j, and $\sigma_i$ and $\sigma_j$ are their standard deviations.

#### Pros
- Intuitive and straightforward.
- Based on actual historical data.

#### Cons
- Assumes past relationships between assets will continue.
- May not capture new market dynamics.

### Method 2: Factor Models
#### Description
Factor models like the CAPM or Fama-French model estimate correlations based on shared responses to economic factors.

#### Mathematical Details
- Identify relevant economic factors (e.g., market risk, size, value).
- Estimate asset sensitivities to these factors.
- Correlation is inferred from factor exposures.

#### Pros
- Considers broader market and economic influences.
- Can be more forward-looking than historical data.

#### Cons
- Requires identifying the correct factors.
- More complex to implement.

## Computing the Expected Return of a Stock

Several methods can be used to compute the expected return of a stock, each with its own mathematical approach and considerations:

### Method 1: Historical Average Return
#### Description
Calculates the average return of a stock based on historical data.

#### Mathematical Details
- Formula: 
  $$
  E(R_i) = \frac{1}{N} \sum_{t=1}^{N} R_{i,t}
  $$
  where $R_{i,t}$ is the return in period t, and N is the number of periods.

#### Pros
- Simple and easy to understand.
- Directly based on past performance.

#### Cons
- Assumes past performance indicates future results.
- Sensitive to the time frame selected.

### Method 2: Dividend Discount Model (DDM)
#### Description
The DDM estimates the stock's value based on its future dividend payments.

#### Mathematical Details
- Formula: 
  $$
  E(R_i) = \frac{D_{i,1}}{P_i} + g
  $$
  where $D_{i,1}$ is the expected dividend next year, $P_i$ is the current price, and g is the dividend growth rate.

#### Pros
- Incorporates future expectations.
- Useful for dividend-paying stocks.

#### Cons
- Not applicable to non-dividend stocks.
- Depends on growth rate accuracy.

### Method 3: Capital Asset Pricing Model (CAPM)
#### Description
CAPM relates the expected return of a stock to its risk relative to the overall market.

#### Mathematical Details
- Formula: 
  $$
  E(R_i) = R_f + \beta_i (E(R_m) - R_f)
  $$
  where $R_f$ is the risk-free rate, $\beta_i$ is the stock's beta, and $E(R_m)$ is the expected market return.

#### Pros
- Accounts for systematic risk.
- Widely used and accepted.

#### Cons
- Assumes a linear relationship between risk and return.
- Beta may not be stable over time.


## Modern Portfolio Theory (MPT)
Modern Portfolio Theory offers a quantitative framework for assembling a portfolio of assets that aims to maximize expected return for a given level of risk. It suggests that an investment's risk and performance should be assessed not in isolation but as part of the overall portfolio.

### Key Equations
In the heart of MPT are several key mathematical concepts:
- **Expected Return of a Portfolio**: The expected return of a portfolio, denoted as $E(R_p)$, is calculated as the weighted average of the expected returns of the individual assets in the portfolio. The formula is given by $E(R_p) = \sum w_i E(R_i)$, where $w_i$ represents the proportion of the portfolio's total value invested in each asset, and $E(R_i)$ is the expected return of asset i.
- **Portfolio Variance**: The variance of a portfolio, denoted as $\sigma_p^2$, measures the dispersion of the returns from the expected return. It's calculated using the formula $\sigma_p^2 = \sum w_i^2 \sigma_i^2 + \sum\sum w_i w_j \sigma_i \sigma_j \rho_{ij}$, where $\sigma_i$ and $\sigma_j$ are the standard deviations of the returns on assets i and j, respectively, and $\rho_{ij}$ is the correlation coefficient between the returns on these assets.

## The Efficient Frontier
The Efficient Frontier is a concept central to MPT. It represents a set of portfolios that provide the highest expected return for a given level of risk or, alternatively, the lowest risk for a given level of expected return.

### Mathematical Explanation
The Efficient Frontier is derived from the optimization problem of finding the set of portfolios that maximize expected return for a specific level of risk or minimize risk for a given level of expected return. This optimization problem involves:
- **Maximizing**: $E(R_p)$, the expected return of the portfolio.
- **Subject to**: The constraint that $\sigma_p^2$, the portfolio variance, remains within a specified range.
- **Constraint**: The sum of the weights of the assets in the portfolio $\sum w_i$ equals 1.

