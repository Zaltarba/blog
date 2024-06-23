---
layout: post
title: Linking Cointegration and the Multi Factor Model 
categories: [Personnal project, Practical paper]
excerpt: This post is the second of our serie about pair trading and market neutral strategies ...
---

## Introduction

Just so you know, this article will be a direct follow-up to my [first article](https://zaltarba.github.io/blog/PairsTrading-1/) about pairs trading. After providing some global context, motivations, and information on pairs trading, I will go more in-depth about something I teased: the link between the beta(s) of a pair of stocks and cointegration.

## About Multi Factor Model

When we speak about the multi-factor model, we generally refer to a generalization of the CAPM model. The main idea behind it is: the market return isn't the only factor with a shared influence on the stocks. Fama and French developed a famous three-factor model. We will be working with it for the following, but keep in mind other models exist using, for instance, ETFs as factors. You can refer to [Analysis of Financial Time Series](https://cpb-us-w2.wpmucdn.com/blog.nus.edu.sg/dist/0/6796/files/2017/03/analysis-of-financial-time-series-copy-2ffgm3v.pdf) for more information on multi-factor models.

### The Fama French Model

The Fama-French model expands on the CAPM by adding two more factors to better capture the returns of a portfolio. Instead of just looking at the market return, it also considers the size of firms and the book-to-market values. The model is typically represented by the following equation:

$$
R_i - R_f = \beta_i (R_m - R_f) + s_i \cdot SMB + h_i \cdot HML + \alpha_i + \epsilon_i
$$

Here's a quick breakdown:
- $R_i$: Expected return of the portfolio
- $R_f$: Risk-free rate
- $\beta_i$: Sensitivity to market return
- $R_m$: Market return
- $SMB$: Size premium (Small Minus Big)
- $HML$: Value premium (High Minus Low)
- $s_i$ and $h_i$: Sensitivities to SMB and HML respectively
- $\alpha_i$: Alpha, representing the stock-specific return
- $\epsilon_i$: Error term, idiosyncratic

**Terminoly Alert** : For the folowing I will refer to the regression coefficients as the betas and the intercept as the alpha

So, instead of just betting on the market, the Fama-French model gives us a more nuanced view by adding these size and value factors. It's like getting a more detailed map for navigating the stock market! Good news about this model, French makes available to us those regressors (not live frequently updated). If you are a fellow quant doing some research you can look it [here](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html#Research).

## Link with the Cointegration 

Let's now explain the link between the beta of two stocks and their cointegration. Let's say we have two stocks, A and B, with proportional betas. What can we say about them?

In mathematical terms

$$
(\beta_A, s_A, h_A) = k \times (\beta_B, s_B, h_B)
$$

for some constant $k$. 

Then let's denote C the linear combination of A and minus $k \times B$.

$$
R_C = R_A - k \times R_B
$$

When substituting the expressions for $R_A$ and $R_B$, we get:

$$
R_C = \alpha_A - k \times \alpha_B + \epsilon_A - k \times \epsilon_B
$$

$\epsilon_A$ and $\epsilon_B$ are both two idiosyncratic risk. We thus get a stationnary serie. DOesn't it means that A and B are cointegrated ? Let's see.

To prove they are cointegrated, we need to consider their stock prices and how these prices relate to their returns. Let $P_A(t)$ and $P_B(t)$ be the prices of stocks A and B at time $t$. The returns $R_A(t)$ and $R_B(t)$ are given by:

$$
R_A(t) = \frac{P_A(t) - P_A(t-1)}{P_A(t-1)}
$$

$$
R_B(t) = \frac{P_B(t) - P_B(t-1)}{P_B(t-1)}
$$

Assuming the returns are driven by the market model:

$$
R_A(t) = \alpha_A + \beta_A \cdot R_m(t) + \epsilon_A(t)
$$

$$
R_B(t) = \alpha_B + \beta_B \cdot R_m(t) + \epsilon_B(t)
$$

Given $\beta_A = k \cdot \beta_B$, we can say:

$$
R_A(t) = \alpha_A + k \cdot \beta_B \cdot R_m(t) + \epsilon_A(t)
$$

$$
R_B(t) = \alpha_B + \beta_B \cdot R_m(t) + \epsilon_B(t)
$$

Now, let's examine the difference between the log prices of the two stocks:

$$
\log(P_A(t)) - k \cdot \log(P_B(t))
$$

The log return of stock A and stock B can be expressed as:

$$
\log(P_A(t)) - \log(P_A(t-1)) = R_A(t)
$$

$$
\log(P_B(t)) - \log(P_B(t-1)) = R_B(t)
$$

Using the market model for returns:

$$
\log(P_A(t)) - \log(P_A(t-1)) = \alpha_A + k \cdot \beta_B \cdot R_m(t) + \epsilon_A(t)
$$

$$
\log(P_B(t)) - \log(P_B(t-1)) = \alpha_B + \beta_B \cdot R_m(t) + \epsilon_B(t)
$$

Substitute these into our difference equation:

$$
\log(P_A(t)) - k \cdot \log(P_B(t)) - \left( \log(P_A(t-1)) - k \cdot \log(P_B(t-1)) \right) = \left( \alpha_A + k \cdot \beta_B \cdot R_m(t) + \epsilon_A(t) \right) - k \cdot \left( \alpha_B + \beta_B \cdot R_m(t) + \epsilon_B(t) \right)
$$

Simplify to:

$$
\log(P_A(t)) - k \cdot \log(P_B(t)) = \log(P_A(t-1)) - k \cdot \log(P_B(t-1)) + \alpha_A - k \cdot \alpha_B + \epsilon_A(t) - k \cdot \epsilon_B(t)
$$

Assuming $\alpha_A = k \cdot \alpha_B$, this reduces to:

$$
\log(P_A(t)) - k \cdot \log(P_B(t)) = \log(P_A(t-1)) - k \cdot \log(P_B(t-1)) + \epsilon_A(t) - k \cdot \epsilon_B(t)
$$

This shows that $\log(P_A(t)) - k \cdot \log(P_B(t))$ is a stationary series if $\epsilon_A(t) - k \cdot \epsilon_B(t)$ is stationary. Therefore, stocks A and B are cointegrated.

So, when stocks A and B have proportional betas, they are cointegrated. This means that despite short-term deviations, their prices will move together in the long run. The opposite is also true: if two stocks are cointegrated, their betas will be proportional.

This relationship is crucial for pairs trading strategies, as it allows traders to predict that any divergence in the prices of cointegrated stocks will eventually correct itself, providing opportunities for profit. It's like having a reliable compass that always points you back to equilibrium, no matter how far you stray!


## Conclusion 
