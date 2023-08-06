---
layout: post
title: A theorical review of the Hurst Exponent estimators 
categories: [Personal project, Theorical paper]
excerpt: In this post we will make a theorical review of several Hurst exponent estimators from the litterature.
---

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
\mathbit{E}[{(\mathbit{R}/\mathbit{S})}_\mathbit{t}]= C×tH
$$
This relationship can be explained within the formal framework of fBm. It was, however, simply inferred by Hurst in his early work.
To calculate Hurst's coefficient using the R/S analysis method, we need to :

1. Calculate the Rescaled Range Series (R/S)
${(\mathbit{R}/\mathbit{S})}_\mathbit{t}=\mathbit{R}_\mathbit{t}/\mathbit{S}_\mathbit{t}\ \ \ \ \ \ \mathbit{t}=\ \mathbf{2},\ \cdots,\ \mathbit{n}}$
Avec R_t\ et\ S_t définis plus bas
L’espérance de {(\mathbit{R}/\mathbit{S})}_\mathbit{t}}
est calculée en divisant l’ensemble des données en intervalles de même taille t : les régions 
\left[\mathbit{X}_\mathbf{1},\mathbit{X}_\mathbit{t}\right], \left[\mathbit{X}_{\mathbit{t}+\mathbf{1}},\mathbit{X}_{\mathbf{2}\mathbit{t}}\right] jusqu’à \left[\mathbit{X}_{(\mathbit{m}-\mathbf{1})\mathbit{t}+\mathbf{1}},\mathbit{X}_{\mathbit{mt}}\right] où \mathbit{m}=\mathbf{floor}(\frac{\mathbit{n}}{\mathbit{t}}).
En pratique, pour utiliser toutes les données pour le calcul, on choisit une valeur de t qui est proportionnelle à n.
Le choix des valeurs de t à utiliser dans l’implémentation est un paramètre de l’algorithme optimisable
Il est recommandé par certains auteurs afin de diminuer la variance de l’estimateur d’utiliser m>50
Ainsi l’exposant de Hurst est donné par l’équation ci-dessous 

\mathbf{ln}{(\mathbit{E}[(\mathbit{R}/\mathbit{S})}_\mathbit{t}]=ln( C×tH)=ln(C)+H×ln(t)

L’exposant de Hurst peut ainsi être calculé à l’aide d’une régression linéaire. 
La méthode de calcul employée est un paramètre optimisable de l’algorithme
Les séries des plages Rescaled sont-elles calculées de la manière suivante : 

2. Calculer la Valeur moyenne du processus \mathbf{X}.

\mathbit{m}=\ \frac{\mathbf{1}}{\mathbit{N}}\sum_{\mathbit{i}=\mathbf{1}}^{\mathbit{t}}\mathbit{X}_\mathbit{i}

3. Calculer la moyenne ajustée de la série \mathbf{Y}
\mathbit{Y}_\mathbit{i}=\ \mathbit{X}_\mathbit{i}-\mathbit{m},\ \ \ \ \ \ \ \mathbit{i}=\mathbf{1},\ \mathbf{2},\ \cdots,\ \mathbit{t}

4. Calculer la série d’écart \mathbf{Z}
\mathbit{Z}_\mathbit{i}=\sum_{\mathbit{k}=\mathbf{1}}^{\mathbit{t}}\mathbit{Y}_\mathbit{k}\ \ \ \ \ \ \mathbit{i}=\mathbf{1},\ \mathbf{2},\ \cdots,\ \mathbit{t}

5. Calculer la série de plages R
\mathbit{R}=\ \mathbit{max}\left(\mathbit{Z}_\mathbf{1},\ \cdots,\ \mathbit{Z}_\mathbit{t}\right)-\mathbit{min}\left(\mathbit{Z}_\mathbf{1},\ \cdots,\ \mathbit{Z}_\mathbit{t}\right)

6. Calculer l'écart-type de la série S
\mathbit{S}_\mathbit{t}=\sqrt{\frac{\mathbf{1}}{\mathbit{t}-\mathbf{1}}\sum_{\mathbit{i}=\mathbf{1}}^{\mathbit{t}}\left(\mathbit{X}_\mathbit{i}-\mathbit{u}\right)^\mathbf{2}}\ \ \ \ \ \ \mathbit{t}=\mathbf{1},\ \mathbf{2},\ \cdots,\ \mathbit{n}

Il est à noter qu’un léger biais de cette méthode a depuis 1955 été identifié. Un estimateur corrigé, toujours basé sur la méthode R/S existe ainsi. Dans un souci de concision, son implémentation n’est pas détaillée. Elle est néanmoins accessible dans la documentation (source 5).

Sources : 

- https://github.com/Mottl/hurst/
- https://stats.stackexchange.com/questions/398701/why-does-the-hurst-package-apply-a-finite-differencing-step-before-doing-rescale?noredirect=1&lq=1
- https://www.researchgate.net/publication/311520485_FORECASTING_THE_BEHAVIOR_OF_FRACTAL_TIME_SERIES_HURST_EXPONENT_AS_A_MEASURE_OF_PREDICTABILITY
- https://www.ijert.org/research/stock-market-data-analysis-using-rescaled-range-rs-analysis-technique-IJERTV3IS20736.pdf
- https://neuropsychology.github.io/NeuroKit/_modules/neurokit2/complexity/fractal_hurst.html#fractal_hurst

# IV Estimateur basé sur la méthode des moments 

Ce dernier estimateur a été construit dans le cadre des fBm. Sans décrire de manière exhaustive les propriétés du mouvement brownien fractionnaire, cet estimateur s’appuie sur l’espérance suivante :
  
Il est alors possible à partir de celle de construire un estimateur convergent du coefficient du Hurst : 
 
Sources : 

- https://www.researchgate.net/publication/335758275_Fractal_analysis_of_the_multifractality_of_foreign_exchange_rates
- https://www.researchgate.net/publication/333852181_Estimation_d'exposants_de_Hurst_dans_un_cadre_stationnaire
- https://inria.hal.science/inria-00074045

# V Annexe 

Explication informelle
 



Source : https://towardsdatascience.com/introduction-to-the-hurst-exponent-with-code-in-python-4da0414ca52e


