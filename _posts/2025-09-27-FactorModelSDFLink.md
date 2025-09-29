---
layout: post
title: "The SDF Explained: Why Factor Models Actually Work"
categories: [Quantitative Finance]
excerpt: "Learn why factor models work in finance. Beneath their growing complexity in the number of factors lies a deeper unifying idea: factor models are, at their core, a way to approximate the Stochastic Discount Factor (SDF)."
image: /thumbnails/LinkingFactorModelsAndSDF.jpeg
hidden: False
tags: [risk neutral measure in finance, stochastic discount factor tutorial, finance, factor model, fama french, capm]
---

Asset pricing has evolved dramatically over the past several decades, shifting from simple models based on a single market factor to an expansive [zoo](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4605976) of multifactor frameworks. But beneath the growing complexity lies a deeper unifying idea: factor models are, at their core, a way to approximate the Stochastic Discount Factor (SDF). This post is not about which factors “work” or how to pick them, it's about why factor models work at all. Drawing on insights from [Cochrane (2005)](https://www.johnhcochrane.com/asset-pricing), we'll explore how the empirical world of factor investing is fundamentally tied to the SDF.

## Table of Contents

1. [Factor Models Dominating Role in Modern Asset Pricing](#Context)  
2. [SDF: The True Concept Behind Asset Prices](#SDF)
3. [Factor Models as SDFs in Disguise](#FactorModelsAreSDF)
4. [Conclusion](#Conclusion)

## Factor Models in Finance: A Historical Perspective {#Context}

In the beginning, investors created the Market. The Market was one, with a clear form and simple measure. And the investors said: Let risk be priced. And risk was priced. And they called this the [Capital Asset Pricing Model](https://doi.org/10.2307/2977928), born in 1964, the first of its kind.

But investors said: Let there be more than one risk, for markets are many and varied. And so in 1992 arose the [Fama–French Three-Factor Model](https://en.wikipedia.org/wiki/Fama%E2%80%93French_three-factor_model). And they called it the Threefold Model, for it spanned size, value, and the market itself. Thus were born factors, and with them the first risk portfolios.

<div style="max-width: 600px; margin: 0 auto;">
<svg width="100%" height="auto" viewBox="0 0 900 220" preserveAspectRatio="xMidYMid meet" role="img" aria-labelledby="title desc">
  <title id="title">Timeline: CAPM → Factor Zoo → IPCA / ML</title>
  <desc id="desc">A simple horizontal timeline with three labeled nodes: CAPM (parsimonious), Factor Zoo (many candidate factors), and IPCA / ML (high-dimensional SDF learning).</desc>

  <defs>
    <!-- Arrow head for the timeline -->
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="#2b2b2b"/>
    </marker>

    <!-- Soft drop shadow for nodes -->
    <filter id="softShadow" x="-50%" y="-50%" width="200%" height="200%">
      <feGaussianBlur stdDeviation="3" result="blur"/>
      <feOffset in="blur" dx="0" dy="2" result="offsetBlur"/>
      <feMerge>
        <feMergeNode in="offsetBlur"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>

    <!-- Common styles -->
    <style type="text/css"><![CDATA[
      .timeline-line { stroke: #2b2b2b; stroke-width: 3; fill: none; marker-end: url(#arrow); }
      .node-circle { fill: #ffffff; stroke: #1f77b4; stroke-width: 2.5; }
      .node-label { font-family: Inter, "Helvetica Neue", Arial, sans-serif; font-size: 14px; fill: #111111; font-weight: 600; }
      .node-sub { font-family: Inter, "Helvetica Neue", Arial, sans-serif; font-size: 12px; fill: #333333; }
      .year { font-family: Inter, "Helvetica Neue", Arial, sans-serif; font-size: 11px; fill: #666666; }
    ]]></style>
  </defs>

  <!-- Background -->
  <rect width="100%" height="100%" fill="transparent"/>

  <!-- Timeline baseline -->
  <line x1="70" y1="110" x2="830" y2="110" class="timeline-line" />

  <!-- CAPM node -->
  <g transform="translate(150,110)" filter="url(#softShadow)">
    <circle r="28" class="node-circle" />
    <!-- Icon: simple market glyph -->
    <g transform="translate(-10,-10) scale(0.8)" fill="#1f77b4" aria-hidden="true">
      <rect x="0" y="6" width="6" height="18" rx="1"></rect>
      <rect x="8" y="0" width="6" height="24" rx="1"></rect>
      <rect x="16" y="3" width="6" height="21" rx="1"></rect>
    </g>
  </g>
  <text x="150" y="160" text-anchor="middle" class="node-label">CAPM</text>
  <text x="150" y="178" text-anchor="middle" class="node-sub"> Sharpe, Single factor</text>
  <text x="150" y="196" text-anchor="middle" class="year">(1960 — 1990s)</text>

  <!-- Factor Zoo node -->
  <g transform="translate(430,110)" filter="url(#softShadow)">
    <circle r="28" class="node-circle" />
    <!-- Icon: multiple small bars -->
    <g transform="translate(-18,-12)" aria-hidden="true">
      <rect x="0" y="10" width="4" height="14" rx="1" fill="#2ca02c"></rect>
      <rect x="6" y="4" width="4" height="20" rx="1" fill="#2ca02c"></rect>
      <rect x="12" y="0" width="4" height="24" rx="1" fill="#2ca02c"></rect>
      <rect x="18" y="7" width="4" height="17" rx="1" fill="#2ca02c"></rect>
    </g>
  </g>
  <text x="430" y="160" text-anchor="middle" class="node-label">Multi Factors Models</text>
  <text x="430" y="178" text-anchor="middle" class="node-sub"> Fama French, small number of factors </text>
  <text x="430" y="196" text-anchor="middle" class="year">(1990s — 2010s)</text>

  <!-- IPCA / ML node -->
  <g transform="translate(710,110)" filter="url(#softShadow)">
    <circle r="28" class="node-circle" />
    <!-- Icon: neural/pc glyph -->
    <g transform="translate(-10,-14)" aria-hidden="true">
      <circle cx="0" cy="8" r="3" fill="#d62728"></circle>
      <circle cx="12" cy="0" r="3" fill="#d62728"></circle>
      <circle cx="24" cy="8" r="3" fill="#d62728"></circle>
      <path d="M0 8 L12 0 L24 8" fill="none" stroke="#d62728" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
    </g>
  </g>
  <text x="710" y="160" text-anchor="middle" class="node-label">Factor Zoo</text>
  <text x="710" y="178" text-anchor="middle" class="node-sub">Many proposed factors</text>
  <text x="710" y="196" text-anchor="middle" class="year">(2010 → present)</text>

  <!-- Optional connecting dots (timeline progression) -->
  <g fill="#2b2b2b" opacity="0.12">
    <circle cx="280" cy="110" r="4"></circle>
    <circle cx="340" cy="110" r="4"></circle>
    <circle cx="520" cy="110" r="4"></circle>
    <circle cx="610" cy="110" r="4"></circle>
  </g>
</svg>
</div>

And again the investors spoke: Let us make factors in our own image, many and diverse, to capture the world as we see it. And so began the great multiplication. From momentum in the 1990s, to profitability and investment in the 2010s, factors spread and multiplied, until there was a vast [zoo](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4605976) of them, each claiming to price risk and explain return.

But despite the proliferation of models that price the market and explain the cross-section of returns, one central question remains: Where does all this actually come from? After all, divine metaphors aside, no one created the market from thin air with well structured properties and linear pricing properties. In this post, I want to explore one possible line of reasoning that helps explain why factor models work, not just how. My thinking here is largely influenced by the reading of [Cochrane (2005)](https://www.johnhcochrane.com/asset-pricing). 



## SDF: The True Concept Behind Asset Prices {#SDF}

When first encountering factors, OLS estimators, shrinkage, and $t$-tests, it can feel rather abstract—especially if the end goal is making an investment decision. Yet one principle is more intuitive and foundational: the **law of one price**, which underlies most modern asset pricing theory.

### The Law of One Price

The Law of One Price states that in an arbitrage-free market, two assets with identical cash flows must have the same price ([G. Cassel (1918)](https://doi.org/10.2307/2223329)). In practice, of course, there are frictions such as limited accessibility, bid-ask spreads, or information asymmetries. Nevertheless, it forms the backbone of asset pricing: two portfolios delivering the same payoffs should have the same price, and two bonds with identical cash flows and risk characteristics should offer the same yield.

### From the Law of One Price to the SDF

This idea can be formalized mathematically through the **stochastic discount factor (SDF)**, also called the pricing kernel [Cochrane (2005)](https://www.johnhcochrane.com/asset-pricing). The SDF is a random variable $m_{t+1}$ that links future payoffs $X_{t+1}$ to current prices $P_t$:

$$
P_t = \mathbb{E}_t \left[ m_{t+1} \, X_{t+1} \right]
$$

Equivalently, for excess returns $r_{t+1}$ (returns over the risk-free rate), the SDF satisfies:

$$
\mathbb{E}_t \left[ m_{t+1} \, r_{t+1} \right] = 0
$$

Intuitively, this states that **all assets are “fairly priced”** given the SDF. The SDF is closely related to risk-neutral probabilities and can be thought of as a weighting factor that adjusts future payoffs to their present value while accounting for risk preferences. Go check [this previous post](https://zaltarba.github.io/blog/RandomWalkvsMartingale/) if you want info on that. 

### The SDF in Practice

In theory, if we knew the distribution of $m_{t+1}$, pricing would be straightforward. This leads to the “everything is factored into the price,” which you can see on [Benjamin Channel](https://www.youtube.com/@benjjjaamiinn). But in reality, the SDF is unobservable, so we must approximate it. One approach could be Bayesian: treat the SDF as a latent random variable and update a prior distribution with market realizations over time.  

<div style="text-align: center;">
  <img 
    src="/blog/images/thats_my_quant.jpeg" 
    alt="That's my Quant"
    style="max-height: 350px; width: auto; max-width: 100%;">
</div>

In practice, however, finance has favored **simpler approximations**. Factor models, estimated via OLS regressions, can be interpreted as **linear projections of the SDF onto observed returns**. This is why factor models—CAPM, Fama–French, and beyond—capture so much pricing information: they approximate the true, unobservable stochastic discount factor.

## Factor Models as SDFs in Disguise {#FactorModelsAreSDF}

At this point, you might be thinking: *“What’s the link between factor models and the SDF?”* If you already know where this is going, congratulations mate.  

**Estimating a factor model**, whether it is CAPM, the [Fama–French Five-Factor Model](https://doi.org/10.1016/j.jfineco.2014.10.010
), or a custom model with hundreds of factors selected via LASSO, is, at its core, **an SDF estimation problem**. This was a revelation for me while reading John Cochrane’s *Asset Pricing* (Chapter 6).

### From Factor Models to the SDF

Now consider the reverse. Suppose we have a factor model:

$$
r_{i,t+1} = \sum_{k=1}^K \beta_{ik} F_{k,t+1} + \epsilon_{i,t+1}, \quad i = 1, \dots, N
$$

with factors $F_k$ and factor loadings $\beta_{ik}$. We want an **implied SDF** $m_{t+1}^{\text{impl}}$ that satisfies the SDF condition for all assets:

$$
\mathbb{E}_t \left[ m_{t+1}^{\text{impl}} \, r_{i,t+1} \right] = 0 \quad \forall i.
$$

Cochrane shows that we can construct it as a **linear combination of the factors**:

$$
m_{t+1}^{\text{impl}} = 1 - \sum_{k=1}^K \lambda_k F_{k,t+1},
$$

where the weights $\lambda_k$ solve the system:

$$
\mathbb{E}_t \Big[ (1 - \sum_{k=1}^K \lambda_k F_{k,t+1}) \, r_{i,t+1} \Big] = 0, \quad i = 1, \dots, N.
$$

In matrix notation, let $R$ be the $N \times 1$ vector of excess returns and $F$ the $N \times K$ factor matrix:

$$
\mathbb{E}_t[R] = F \lambda \quad \implies \quad \lambda = (F^\top F)^{-1} F^\top \mathbb{E}_t[R].
$$

This provides an **explicit SDF estimate** from any factor model. Every factor captures a dimension of risk, and the combination of factors approximates the true, unobservable SDF $M_t = 1-\lambda^\top F_t$. This is why factor models are powerful : they encode the pricing kernel in a tractable, estimable form.

### Why Use Factor Models Instead of Directly Approximating the Stochastic Discount Factor?

If the true power of a factor model lies in its deep connection to the stochastic discount factor (SDF), why not bypass the factors entirely and approximate the SDF directly to predict future prices?

The idea might be to approximate the SDF $ M $ and then use the fundamental pricing equation $ P_t = \mathbb{E}[M_{t+1} \times P_{t+1}] $ to compute fair prices and identify whether an asset is over or undervalued. However, this approach isn’t as straightforward or as informative as it might seem.

For example, suppose $ M = 1 $ almost surely. Then you’re left with $ P_t = \mathbb{E}[P_{t+1}] $, which offers no new insight. This illustrates that simply knowing the distribution of the SDF is not enough.

What really matters when working with $\mathbb{E}[M_{t+1} R_{t+1}] = 0$ is understanding the joint distribution of $ (M_t, R_t) $ the stochastic discount factor and the asset returns together. This joint distribution is difficult to characterize directly.

Factor models offer a practical solution by capturing the sensitivity of returns to underlying risk factors. This sensitivity gives us a tractable way to understand how returns co-move with the SDF, essentially providing a handle on their joint behavior.

In other words, the key mathematical value of factor models is that they let us approximate the joint dynamics between $ M_t $ and $ R_t $ through the factors $ F $. Let's see that with some equations : 

$$
\mathbb{E}_t[R_{t+1} \times M_{t+1}] = 0
$$

which implies

$$
\mathbb{E}_t\big[R_{t+1} \times (1 - \lambda^\top F_{t+1})\big] = 0
$$

and thus

$$
\mathbb{E}_t[R_{t+1}] = \mathbb{E}_t[R_{t+1} F_{t+1}^\top] \lambda
$$

When using the OLS estimator, the betas of the stock (an $ N \times K $ matrix) are defined as

$$
B_{t+1} = R_{t+1} F_{t+1}^\top (F_{t+1} F_{t+1}^\top)^{-1}
$$

Since

$$
R_{t+1} F_{t+1}^\top = R_{t+1} F_{t+1}^\top (F_{t+1} F_{t+1}^\top)^{-1} (F_{t+1} F_{t+1}^\top),
$$

we get

$$
R_{t+1} F_{t+1}^\top = B_{t+1} F_{t+1} F_{t+1}^\top
$$

Letting \\( \Sigma_{FF}^{t+1} = \mathbb{E}_t[F_{t+1} F_{t+1}^\top] \\), we have the final expression

$$
\mathbb{E}_t[R_{t+1}] = B_{t+1} \Sigma_{FF}^{t+1} \lambda
$$

By starting with basic properties of the stochastic discount factor, we ultimately arrive at a pricing formula that depends only on our factor model. This is powerful because it allows us to shift the focus from estimating relationships across all $ N $ assets to working with a much smaller set of $ K $ factors.

Plenty of work must be then done on estimating $B_{t+1}$, $\Sigma_{FF}^{t+1}$ and $\lambda_{t+1}$. When $ K \ll N $, and assuming the factors are not highly correlated, the factor covariance matrix $\Sigma_{FF}^{t+1}$ is easier to estimate, invert, and work with in general.

This dimensionality reduction offers several practical benefits, including more stable estimates, reduced computational complexity, and better interpretability of risk-return trade-offs.

## Conclusion  {#Conclusion}

At first glance, factor models may look like clever statistical tools thrown at return data to extract patterns. But a deeper look reveals something more fundamental: they are not just predictive regressions, they are structured approximations of the Stochastic Discount Factor, the very object at the heart of modern asset pricing. This tells us *why* factor models work.

So next time you're running a factor regression, remember: you're not just crunching numbers, you’re approximating the invisible hand that prices all possible states of the economy.

Got thoughts, questions, or counterpoints? Drop a comment below. I’d love to hear your take.










