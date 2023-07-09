---
layout: post
title:  A closed-form filter for binary time series
categories: [Academic project]
excerpt: The aim of this project is to study and benchmark the performance of the "Optimal Particle Filter" introduced by Fanaso and al. in 2021, with the well-known Bootstrap Filter algorithm.
---

This project is a common work with Raphael Thabut and Eric Vong. It's a pratical implementation of [A closed-form filter for binary time series](https://www.researchgate.net/publication/352384574_A_closed-form_filter_for_binary_time_series) by Fanaso and al.

## Article presentation

Denoising data is always relevant to perform, as in reality, a lot of observations can be noised due to external effects. Most filtering methods to predict and smooth distributions relies on the assumption of the Gaussian nature of the state-space models. This is because we have closed form filters available. Even for a binary state-space model, we require Monte-Carlo strategies for prediction.  

This article claims that they have found a closed form filter for the dynamic probit models with a gaussian state variable. To do so, they showed that they belong to a class of distribution, the Skew Unified Normal (SUN), whose parameters can be updated with closed expression.  

They then show how the algorithm performs in practice, and how using, the properties of the SUN distribution, allows for online inference for high dimensions.  

## Zoom on the financial application


The articles gave an application example, using financial time series. Indeed, it focus on the analysis of two times series, the Nikkei 225 (Japanase Stock Index) and the CAC 40 (French Stock Index).   

As the latter opens after the former, one can try to use the cointegration effect between the two time series to obtain relevant predictions. Our data will be composed of :  

- $x_t$ where $x_t = 1$ if the Nikkei 225 opening has increased between the close value and opening value and $x_t = 0$ otherwise  
- $y_t$ where $y_t = 1$ if the CAC 40 opening has increased between the close value and opening value and $y_t = 0$ otherwise  

One important objective for a financial application would be to evaluate $Pr(y_t = 1 \mid x_t)$.   

This can be done using a probit model. We try to infer a binary value given two parameters, the trend of the CAC 40 when the Nikkei 225 opening is negative and the shift in the trend if the Nikkei 225 opening is positive. Mathematically, this means :   

$$ P(y_t = 1 \mid x_t) = \Phi[F_t \theta, 1] $$

with the following notations :   

- $F_t = (1, x_t)$
- $\theta = (\theta_1, \theta_2)$, $\theta_1$ being the trend and $\theta_2$ the shift

In the case of such an application, the purpose of the methods we describe in this report is to estimate the $\theta$ parameter effectively.  

## The SUN distribution  
  
Gaussian models are omnipresent in mathematical models. They have very strong properties and make models simpler. Although, one important drawback is that real life distribution tends to have some sort of asymetry.  

A first extension of the gaussian models, Skew Normals, has been proposed by Arellano-Valle and Azzalini (1996). Due to the fertility of this formulation, a unifying representation, the Skewed Unifyed Normals, has been proposed by them since 2006.  

Given $\theta \in R^q$, we say that $\theta$ has a unified skew-normal distribution, noted $\theta \sim {SUN}_{q, h}{(\xi, \Omega, \Delta, \gamma, \Gamma)}$, if its density function $\rho{(\theta)}$ is of form:  

$$ \phi_q{(\theta - \xi ; \Omega)} \frac{\Phi_{h}{(\gamma + \Delta^T \overline{\Omega}^{-1} \omega^{-1} ({\theta - \xi}); \Gamma - \Delta^T \overline{\Omega}^{-1} \Delta})}{\Phi_h{(\gamma ; \Gamma)}} $$

With:  

- $\phi_q{(\theta ; \Omega)}$ where $\phi_h$ is the centered gaussian density with covariance $\Omega$ matrix  
- $\Phi_{h}{(\gamma ; \Gamma)}$ where $\Phi_h$ is the centered gaussian cumulative distribution function with covariance $\Gamma$  
- $\Omega = \omega \overline{\Omega} \omega$ and $\overline{\Omega}$ is the correlation matrix  
- $\xi$ controls the location  
- $\omega$ controls the scale  
- $\Delta$ defines the dependance between the covariance matrixes $\Gamma, \overline{\Omega}$ 
- $\gamma$ controls the truncature  

The term:  

$$\frac{\Phi_{h}(\gamma + \Delta^T \overline{\Omega}^{-1} \omega^{-1} (\theta - \xi) ; \Gamma - \Delta^T \overline{\Omega}^{-1} \Delta)}{\Phi_{h}(\gamma ; \Gamma)}$$

induces the skewness as it is rescaled by the term $\Phi_{h}(\gamma ; \Gamma)$  

We have also the following distribution equality $\theta \overset{(d)}{=} \xi + \omega (U_0 + \Delta \Gamma^{-1} U_1)$ with $U_0 \indep U_1$ and $U_0 \sim \func{N_q}{0, \overline{\Omega} - \Delta \Gamma^{-1} \Delta^T}, U_1 \sim \func{TN_h}{0, \Gamma, -\gamma}$ truncated normal law below $-\gamma$. This representation allows for efficient computing, but we can also apply it to a probit model.  

Given $y_t = (y_{1t}, \ldots, y_{mt})^T \in \set{0, 1}^m$ a binary vector and $\theta_t = (\theta_{1t}, \ldots, \theta_{pt})^T \in \R^p$ the vectors of parameters. The dynamic probit model is then defined as:  `

$$p{(y_t \vert \theta_t)} = \Phi_m{(B_t F_t \theta_t ; B_t V_t B_t)}$$
$$\theta_t = G_t \theta_{t-1} + \varepsilon_t, \varepsilon_t \sim \func{N_p}{(0, W_t)}$$

With $\theta_0 \sim \func{N_p}{a_0, P_0}$ with $B_t = \mathrm{diag}{(2y_{1t} - 1, \ldots, 2y_{mt} - 1)}$  

This representation is equivalent to another formulation (Albert and Chib 1993) as followed:  

$$p{(z_t \vert \theta_t)} = \phi_m{(z_t - F_t \theta_t; V_t)}$$
$$\theta_t & = & G_t \theta_{t-1} + \varepsilon_t, \varepsilon_t \sim \func{N_p}{0, W_t}$$  

With $y_{it} = 1_{z_{it} > 0}$   

Then, under such a dynamic, the first step filtering distribution is:  

$$ (\theta_1 \mid y_1) \sim \mathrm{SUN}_{p,m}{\xi_{1 \mid 1}, \Omega_{1 \mid 1}, \Delta_{1 \mid 1}, \gamma_{1 \mid 1}, \Gamma_{1 \mid 1}} $$

With the following parameters:

- $\xi_{1 \mid 1} = G_1 a_0$  
- $\Omega_{1 \mid 1} = G_1 P_0 G_1^T + W_1$  
- $\Delta_{1 \mid 1} = \overline{\Omega_{1 \mid 1}} \omega_{1 \mid 1} F_1^T B_1 s_1^{-1}$    
- $\gamma_{1 \mid 1} = s_1^{-1} B_1 F_1 \xi_{1 \mid 1}$   
- $\Gamma_{1 \mid 1} = s_1^{-1} B_1 \times {F_1 \Omega_{1 \mid 1} F_1^T + V_1}B_1 s_1^{-1}$
- $s_1 = (\mathrm{diag}{(F_1 \Omega_{1 \mid 1}F_1^T + V_1)})^{1/2}$  
  
More generally, if $(\theta_t \mid y_{1:t-1}) \sim SUN_{p,m(t-1)}$ t t ${(\xi_{t-1 \mid t-1}, \Omega_{t-1 \mid t-1}, \Delta_{t-1 \mid t-1}, \gamma_{t-1 \mid t-1}, \Gamma_{t-1 \mid t-1})$ is the filtering distribution at $t-1$, then the predictive distribution at $t$ is $(\theta_t \mid y_{1:t-1}) \sim \mathrm{SUN}_{p,m(t-1)}(\xi_{t \mid t-1}, \Omega_{t \mid t-1}, \Delta_{t \mid t-1}, \gamma_{t \mid t-1}, \Gamma_{t \mid t-1})$ with the following parameters:   

- $\xi_{t \mid t-1} = G_t \xi_{t-1 \mid t-1}$  
- $\Omega_{t \mid t-1} = G_t \Omega_{t-1 \mid t-1}G_t^T + W_t$  
- $\Delta_{t \mid t-1} = w_{t-1 \mid t-1}^{-1} G_t w_{t -1 \mid t-1} \Delta_{t-1 \mid t-1}$ 
- $\gamma_{t \mid t-1} = \gamma_{t-1 \mid t-1}$  
- $\Gamma_{t \mid t-1} = \gamma_{t-1 \mid t-1}$  

And at time $t$, the filtering distribution are defined as:  

- $\xi_{t \mid t} = \xi_{t \mid t-1}$  
- $\Omega_{t \mid t} = \Omega_{t \mid t-1}$    
- $\Delta_{t \mid t} = [\Delta_{t \mid t-1}, \overline{\Omega_{t \mid t}} \omega_{t \mid t} F_t^T B_t s_t^{-1}]$  
- $\gamma_{t \mid t} = [\gamma_{t \mid t-1}^T, \xi_{t \mid t}^T F_t^T B_t s_t^{-T}]$  
- $\Gamma_{t \mid t} = |A|B||B|C|$  
- $A = \Gamma_{t \mid t-1}$  
- $B = s_t^{-1} B_t F_t \omega_{t \mid t} \Delta_{t \mid t-1}$  
- $C = s_t^{-1} B_t(F_t \Omega_{t \mid t} F_t^T + V_t) B_t s_t^{-1}$  
- $s_t = (\mathrm{diag}{F_t \Omega_{t \mid t} F_t^T + V_t})^{1/2}$  

For the smoothing, it is available with similar forms of equations.  
