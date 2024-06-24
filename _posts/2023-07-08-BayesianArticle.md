---
layout: post
title:  How to cluster time series within a bayesian framework
categories: [Academic project]
excerpt: The purpose of this project is to group a set of time series into different clusters and, in doing so, estimate the statistical model describing the time series of each cluster. To conduct our estimations, we adopt a Bayesian framework and utilize Monte Carlo simulation methods and hidden Markov chains.
image: /blog/images/BNP.png
---

# A model-based clustering of multiple time-series algorithm

This project is a common work with Gabriel Guaquiere. It is a pratical implementation of the methods from [Model-Based Clustering of Multiple Time Series](https://www.researchgate.net/publication/4756297_Model-Based_Clustering_of_Multiple_Time_Series) by Frühwirth-Schnatter, and S. Kaufmann.  
You can check the python implementation on my [Github](https://github.com/Zaltarba/Bayesian_statistics_project.git).

## Introduction

The purpose of this project is to group a set of time series into different clusters and, in doing so, estimate the statistical model describing the time series of each cluster. To conduct our estimations, we adopt a Bayesian framework and utilize Monte Carlo simulation methods and hidden Markov chains.

Let $(y_{i,t})$, with $t=1,...,T$, be a multiple time series among N other time series $i=1,...,N$. We assume that these series belong to K different clusters, and that all time series belonging to cluster $k$ are described by the same statistical model with a group-specific parameter, $\theta_k$. The membership to group $k$ for each time series $i$ is initially unknown and is estimated simultaneously with the parameters $\theta_k$. Additionally, we assume that the parameters $\theta_k$ are specific to each cluster.

We introduce :  
- the vector $S = (S_1,S_2,...,S_N)$ where $\forall i \in 1,..,N$ $S_i$ $\in 1,...,K$ indicates the group to which time series $i$ belongs
- the vector $\phi = (\eta_1, ..., \eta_K)$ where $\eta_k$ indicates the proportion of time series belonging to cluster $k$.

Using an MCMC algorithm, we will iteratively estimate the vector $S$, followed by the vectors $\theta$ and $\phi$.

## The Model

### Theoretical Perspective

For $i = 1,..,N$, the conditional density of $y_i$ given $\theta_{S_i}$ is written as:  

$$p(y_i \mid \theta_{S_i}) = \prod_{t=1,...,T} p(y_{i,t} \mid y_{i,t-1},..,y_{i,0},\theta_{S_i})~~~~~~~~~~~~~~~~~~~~~~(1)$$ 

Where $p(y_i \mid y_{i,t-1},..,y_{i,0},\theta_{S_i})$ is a known density that depends on the chosen model.  

Therefore,  
 
$$p(y_i \mid S_i, \theta_1,...,\theta_K) = p(y_i \mid \theta_{S_i})~~~~~~~~~~~~~~~~~~~~~~(2)$$

Next, we establish a probabilistic model for the variable $S = (S_1,..,S_N)$. We assume that $S_1, S_2,..,S_N$ are pairwise independent a priori, and for all $i = 1,..,N$, we define the prior probability $Pr(S_i = k)$, the probability that time series $i$ belongs to cluster $k$. We assume that for each series $i$, we have no prior knowledge of which cluster it belongs to. Hence,  

$$Pr(S_i = k \mid \eta_1,..,\eta_K) = \eta_k~~~~~~~~~~~~~~~~~~~~~~(3)$$

The sizes of the groups $(\eta_1,..,\eta_K)$ are initially unknown and are estimated using the data.  

### The MCMC Algorithm

The estimation of the parameter vector $\psi = (\theta_1,..,\theta_k,\phi,S)$ using MCMC is done in two steps:

**Step 1**  

We fix the parameters $(\theta_1,..,\theta_K,\phi)$ and estimate $S$.

In this step, we will assign a group $k$ to each time series $i$ using the posterior $p(S_i \mid y,\theta_1,..,\theta_K,\phi)$.

Based on Bayes' theorem and the above, we know that:

$$p(S_i = k \mid y,\theta_1,..,\theta_K,\phi) \propto p(y_i \mid \theta_k)Pr(S_i = k \mid \phi)~~~~~~~~~~~~~~~~~~~~~~(4)$$

Using equations (1) and (3), we will calculate this posterior for $k = 1,..,K$, and using Python, we will simulate a draw of $S_i$ and assign it to a group $k$.

**Step 2**  

We fix the classification $S$ and estimate the parameter vector $(\theta_1,..,\theta_K,\phi)$.

Conditioned on $S$, the variables $\theta$ and $\phi$ are independent. Since the parameter $\theta_k$ is specific to cluster $k$, we group all time series belonging to group $k$.  

Thus, $\theta_k$ is estimated using the posterior (5) and a Metropolis-Hastings algorithm:  

$$p(\theta_k \mid y,S_1,..,S_N) = \prod_{i : S_i = k} p(\theta_k \mid y_i) = \prod_{i : S_i = k} p(y_i \mid \theta_k)p(\theta_k)~~~~~~~~~~~~~~~~~~~~~~(5)$$  

Where the prior $p(\theta_k)$ depends on the chosen model.  

Finally, we estimate $\phi = (\eta_1,..,\eta_k)$ using the posterior (6) and a Metropolis-Hastings algorithm:  

$p(\phi \mid S,y) = p(y \mid S,\phi,\theta_1,..,\theta_K)$  
$= p(y \mid S,\phi,\theta_1,..,\theta_K) \times p(S \mid \phi) \times p(\phi)~~~~~~~~~~~~~~~~~~~~~~(6)$  
$= \prod_{k=1,...,K} \prod_{i : S_i = k} p(y_i \mid \theta_k) \prod_{j = 1,...,N}Pr(S_j \mid \phi)p(\phi)$  

Where the prior distribution of $\phi$ is a Dirichlet distribution (4,..,4).  

We will estimate $\psi = (\theta_1,..,\theta_k,\phi,S)$ by repeating these two steps P times, after initializing $\psi^{0} = (\theta_1^{0},..,\theta_k^{0},\phi^{0},S^{0})$.

## Implementation  

From a practical point of view, the likelihood of the estimated ARIMAX(p, 0, d) models was calculated using the statsmodels library.  

To avoid a final classification with zero time series in a cluster, we decided to randomly select about ten series in such cases to update the cluster parameters. Otherwise, if the size of a cluster is reduced to zero in the early iterations, the coefficients associated with the model would not be updated.  

The two steps described in the previous section were used for a Gibbs sampling algorithm. For each step, a Metropolis-Hastings algorithm was implemented. A random walk was performed to find the model coefficients, with ten successive iterations for each step. We selected this number based on the results we obtained and taking into account the complexity of the final algorithm.  

## Results  

### Model 1  

The first model is an ARMA(1,1) model of the form:  

$y_{i,t} = \alpha_{S_i}y_{i,t-1} + \beta_{S_i}\epsilon_{t-1} + \epsilon_t$  

Here, $\theta_k$ = $(\alpha_k,\beta_k)$, where $\alpha_k$ and $\beta_k$ are the AR(1) and MA(1) parameters, respectively, of the time series belonging to cluster $k$.  

We set $K = 2$ and $N = 100$, and use the following priors:  

$\forall k \in 1,2 : \alpha,\beta \sim \mathcal{N}(0,,\frac{1}{3})\phi \sim \mathcal{D}(4,4)\Pr(S_i = k \mid \eta_1,..,\eta_K) = \eta_k$  

Since $\epsilon_t \sim \mathcal{N}(0,\sigma^2)$, we have:  
  
$y_{i,t} \mid y_{i,t-1},..,y_{i,0},\theta_{S_i} \sim \mathcal{N}(\alpha_{S_i}y_{i,t-1} + \beta_{S_i}\epsilon_{t-1}, \sigma^2)$  

We are able to calculate the posteriors (4), (5), and (6) and estimate the parameters using the method described in Section 2.  

The results obtained after 5000 iterations are as follows:  

| Coefficients   | $\alpha$   | $\beta$   | $\sigma^2$   | Cluster Sizes|
|----------------|------------|-----------|--------------|---------------|
| True Value     | 1 and -0.8     | 0.1 and 0.1  | 0.1 and 0.1      | 80 and 20  |
| Estimated Value| 1.00 and 0.81  | 0.08 and 0.11 | 0.09 and 0.10    | 80 and 20 |
| Error (%)      | 0.03% and 0.91%| 17.3% and 12.6%| 0.19% and 0.95%  | 0% and 0%|

Our model successfully assigned each series to its cluster and accurately estimated the $\alpha$ parameter for each cluster. However, it performed less well in estimating the $\beta$ (MA(1)) parameter. Perhaps with a more appropriate prior, the error could have been reduced. The variance of the residuals was estimated with high precision.

The graphs below illustrate the convergence of the $\alpha$, $\beta$, $\phi$, $\sigma^2$, $S$, and $\phi$ parameters over the iterations for each cluster.

![Results for the first model](/blog/images/model1_graph-1.png)

It can be observed that the parameters converge to their true values around the 1000th iteration.

## Model 2

The second model is an ARMAX(1,1) model of the form:

$y_{i,t} = \alpha_{S_i}y_{i,t-1} + \beta_{S_i}\epsilon_{t-1} + \gamma_{S_i}x_t + \epsilon_t$

Here, $\theta_k$ = $(\alpha_k,\beta_k,\gamma_k)$, where $\alpha_k$, $\beta_k$, and $\gamma_k$ are the AR(1), MA(1), and the coefficient for the exogenous variable, respectively, of the time series belonging to cluster $k$. The process $x_t$ is a random walk and is observed.

We set K = 3 and N = 100, and use the same priors as in Model 1.

Since $\epsilon_t \sim \mathcal{N}(0,\sigma^2)$, with $\sigma^2$ known, we have:

$y_{i,t} \mid y_{i,t-1},..,y_{i,0},\theta_{S_i} \sim \mathcal{N}(\alpha_{S_i}y_{i,t-1} + \beta_{S_i}\epsilon_{t-1} + \gamma_{S_i}x_t, \sigma^2)$

We are able to calculate the posteriors (4), (5), and (6) and estimate the parameters using the method described in Section 2.

The results obtained after 5000 iterations are as follows:

| Coefficients   | $\alpha$          | $\beta$         | $\gamma$            | $\sigma^2$        | Cluster Sizes |
|----------------|-------------------|-----------------|---------------------|-------------------|---------------|
| True Value     | 1, -0.8, 0.5      | 0.1, 0.1, 0.5   | 0.1, 0.5, 0.9       | 0.1 0.1 0.1       | 60, 20, 20    |
| Estimated Value| 0.99 -0.75 -0.50  | 0.11, 0.03 0.55 | 0.091, 0.497, 0.886 | 0.099 0.097 0.098 | 60, 20, 20    |
| Error (%)      | 0.07% 6.88% 0.56% | 14.29% 70.58% 9.35% | 9.4%, 0.57%, 1.54% | 0.46% 2.03% 1.32% | 0%, 0%, 0%    |

Our model successfully assigned each series to its cluster, and it accurately estimated the $\alpha$ and $\gamma$ parameters within each cluster. However, it performed less well in estimating the $\beta$ (MA(1)) parameter. Perhaps with a more appropriate prior, the error could have been reduced.

The graphs below illustrate the convergence of the $\alpha$, $\beta$, $\gamma$, $\phi$, $\sigma^2$, $S$, and $\phi$ parameters over the iterations for each cluster.

![Results for the second model](/blog/images/model2_graph-1.png)

It can be observed that the parameters converge to their true values around the 1000th iteration.

## Conclusion

By using a Bayesian approach and Monte Carlo methods, we successfully grouped time series into different clusters. We observed that the estimations of the AR parameters were very close to the true values, and our algorithms made no classification errors. However, the estimations of the MA parameters were often less accurate. This difficulty arises from the challenges involved in estimating the parameters specific to ARIMA models. Additionally, we assumed that all parameters follow the same prior distribution

## Bibliography 

14. Frühwirth-Schnatter, and S. Kaufmann, (2008). Model-based clustering of multiple time-series, Journal of Business and Economic Statistics, 26, 78 – 89
