---
layout: post
title: A theorical review of the Hurst Exponent estimators 
categories: [Personal project, Theorical paper]
excerpt: In this post we will make a theorical review of several Hurst exponent estimators from the litterature.
---

WIP Article 

# I A reminder on the Hurst exponent 

The Hurst exponent lies between 0 and 1 and is usually denoted H. Its use and formal definition vary according to context. It can be used as a measure of long-term memory for time series. In the fBm context, for example, it is linked to the autocorrelations of the increment series and the speed at which these evolve as the lag increases. It thus quantifies the relative tendency of a time series to regress strongly towards the mean or to cluster in one direction. In this context, we distinguish three cases depending on its value, indicating the different properties of the process:  
	
- H = ½, the process is a Standard Brownian motion.
- H > ½, the process increments are anti-persistent. They indicate positive correlations and a long memory process. Values of H that deviate further from ½ seem favorable to the application of Momentum Strategy.
- H < ½, process increments are then persistent. They indicate negative correlations between points and mean-reverting behavior.

Consequently, if we estimate a Hurst exponent $H = \frac{1}{2}$, then the evolution of the series is partly predictable. If estimated correctly, it can therefore add value to a mechanism such as a Stop Loss Take Profit.  

Fractional Brownian motion (fBm) provides a suitable modeling framework for self-similar non-stationary stochastic processes. It has been widely used to model random phenomena in various fields of research. When working with rates, for example, these can be modeled as fractional Brownian motions. Their increments are then fractional Gaussian noise.  
Finally, and to complete this introduction to the Hurst coefficient, it's worth recalling that it was introduced in 1955 by Harold Hurst when he discovered that the R/S statistic (and which we describe below) he had constructed did not converge to 0.5 (the theoretical value for the models then in use) but 0.7 for many series.

Sources : 
- https://en.wikipedia.org/wiki/Hurst_exponent
- https://arxiv.org/pdf/1406.6018.pdf

# II Autocorrelation-based estimator 

Here we take advantage of the following relationship, which has a theoretical basis in the fBm framework:  

$$
\mathbf{Cov}(\mathbf{X}_{t+\mathbf{lag}},\ \mathbf{X}_t) = \mathbf{C} \times \mathbf{lag}^{2\mathbf{H}}
$$

It is then possible to estimate the Hurst coefficient from the covariance for different lags and a linear regression.  
An explanation of the relationship in a less formalized framework is given in the appendix.   

Sources :  

- https://towardsdatascience.com/introduction-to-the-hurst-exponent-with-code-in-python-4da0414ca52e
- https://mahowald.github.io/hurst/

# III RS-based estimator 

The exponent based on the R/S method is the historical estimator, introduced by Hurst. This estimator is based on the R/S statistic, also introduced by Hurst. Using this method, a Hurst coefficient can be calculated for any time series. However, in the absence of a model, the coefficient can only be used (with difficulty) to refute the stationary nature of a process. In the case of fBm, the R/S estimator is relevant, though not optimal. It should be noted, however, that the series considered is then the series of increments (which follows an fGm), and not the fBm.  
Here again, calculation of the Hurst exponent is based on an asymptotic relationship:  

$$
\mathbf{E}\left[ \left( \frac{\mathbf{R}}{\mathbf{S}} \right)_t \right] = C \times t^H
$$

This relationship can be explained within the formal framework of fBm. It was, however, simply inferred by Hurst in his early work.  
To calculate Hurst's coefficient using the R/S analysis method, we need to :  

At first calculate the Rescaled Range Series (R/S) :  

$$
\left( \frac{\mathbf{R}}{\mathbf{S}} \right)_t = \frac{\mathbf{R}_t}{\mathbf{S}_t} \quad \mathbf{t} = 2, \ldots, \mathbf{n}
$$

With $R_t$ et $S_t$ defined below.  
The expectation of 
$$\left( \frac{\mathbf{R}}{\mathbf{S}} \right)_t$$
is calculated by dividing the data set into intervals of equal size t: the regions 
$$\left[ \mathbf{X}_1, \mathbf{X}_t \right], \quad \left[ \mathbf{X}_{t+1}, \mathbf{X}_{2t} \right]$$ 
up to 
$$\left[ \mathbf{X}_{(\mathbf{m}-1)\mathbf{t}+1}, \mathbf{X}_{\mathbf{mt}} \right]$$ 
with 
$$\mathbf{m} = \lfloor \frac{\mathbf{n}}{\mathbf{t}} \rfloor$$.  
 
In practice, to use all the data for the calculation, we choose a value of t that is proportional to n.  
The choice of t values to be used in the implementation is a parameter of the optimizable algorithm.  
Some authors recommend using m>50 to reduce the variance of the estimator.  

The Hurst exponent is then given by the equation below  

$$
\ln\left( \mathbf{E}\left[ \left(\frac{\mathbf{R}}{\mathbf{S}} \right)_t \right] \right) = \ln(C \times t^H) = \ln(C) + H \times \ln(t)
$$

The Hurst exponent can thus be calculated using linear regression.  
The calculation method used is an optimizable parameter of the algorithm.  

Rescaled range series are calculated as follows:  

1. Calculate Average Process Value $\mathbf{X}$ :  
$$
\mathbf{m} = \frac{1}{\mathbf{N}}\sum_{\mathbf{i}=1}^{\mathbf{t}}\mathbf{X}_{\mathbf{i}}
$$
2. Calculate the adjusted mean of the series $\mathbf{Y}$ :  
$$
\mathbf{Y}_\mathbf{i} = \mathbf{X}_\mathbf{i} - \mathbf{m}, \quad \mathbf{i}=1, 2, \ldots, \mathbf{t}
$$
4. Calculate the deviation series $\mathbf{Z}$ :  
$$
\mathbf{Z}_\mathbf{i} = \sum_{\mathbf{k}=1}^{\mathbf{t}} \mathbf{Y}_\mathbf{k}, \quad \mathbf{i} = 1, 2, \ldots, \mathbf{t}
$$
5. Calculate the range series $\mathbf{R}$ :  
$$
\mathbf{R} = \max\left(\mathbf{Z}_1, \ldots, \mathbf{Z}_\mathbf{t}\right) - \min\left(\mathbf{Z}_1, \ldots, \mathbf{Z}_\mathbf{t}\right)
$$
6. Calculate the standard deviation of the series $\mathbf{S}$ :  
$$
\mathbf{S}_\mathbf{t} = \sqrt{\frac{1}{\mathbf{t}-1}\sum_{\mathbf{i}=1}^{\mathbf{t}} \left(\mathbf{X}_\mathbf{i}-\mathbf{u}\right)^2}, \quad \mathbf{t}=1, 2, \ldots, \mathbf{n}
$$  
It should be noted that a slight bias in this method has been identified since 1955. A corrected estimator, still based on the R/S method, therefore exists. For the sake of brevity, its implementation is not detailed. However, it is available in the documentation (source 5).

Sources : 

- https://github.com/Mottl/hurst/
- https://stats.stackexchange.com/questions/398701/why-does-the-hurst-package-apply-a-finite-differencing-step-before-doing-rescale?noredirect=1&lq=1
- https://www.researchgate.net/publication/311520485_FORECASTING_THE_BEHAVIOR_OF_FRACTAL_TIME_SERIES_HURST_EXPONENT_AS_A_MEASURE_OF_PREDICTABILITY
- https://www.ijert.org/research/stock-market-data-analysis-using-rescaled-range-rs-analysis-technique-IJERTV3IS20736.pdf
- https://neuropsychology.github.io/NeuroKit/_modules/neurokit2/complexity/fractal_hurst.html#fractal_hurst

# IV Estimateur basé sur la méthode des moments 

The latter estimator has been constructed in the fBm framework. Without exhaustively describing the properties of fractional Brownian motion, this estimator is based on the following expectation :
  
From this, it is then possible to construct a convergent estimator of the Hurst coefficient: 

Sources : 

- https://www.researchgate.net/publication/335758275_Fractal_analysis_of_the_multifractality_of_foreign_exchange_rates
- https://www.researchgate.net/publication/333852181_Estimation_d'exposants_de_Hurst_dans_un_cadre_stationnaire
- https://inria.hal.science/inria-00074045

# V Annexe 

Explication informelle
 



Source : https://towardsdatascience.com/introduction-to-the-hurst-exponent-with-code-in-python-4da0414ca52e


