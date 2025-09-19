---
layout: post
title: "Multi-Factor Models: Could Less Be Better?"
categories: [Statistics, Quantitative Finance,]
excerpt: "A literature-based exploration of the current debate in asset pricing: should we embrace high-dimensional stochastic discount factor learning, or stick with parsimonious classic approaches? Both schools of thought coexist in today’s research."
image: /thumbnails/HighDimensionMultiFactorModels.jpeg
hidden: True
tags: [asset pricing, factor models, stochastic discount factor, machine learning, finance, sdf, IPAT]
---

# Multi-Factor Models: Could Less Be Better?

## Table of Contents

1. [Factor Models and the Stochastic Discount Factor in Asset Pricing](#introduction)  
2. [The Case for Less: Low-Dimensional Factor Models and Interpretability](#smallmodels)  
3. [The Case for More: High-Dimensional SDF Learning with Machine Learning](#bigmodels)  
4. [Could Less Still Be Better in Asset Pricing?](#conclusion)



## Factor Models and the Stochastic Discount Factor in Asset Pricing  {#introduction}


Most readers in finance are already familiar with factor models. They are the workhorses of empirical asset pricing: we regress returns on a small set of systematic risks, extract betas, and see whether they explain the cross-section. For hedge fund and asset manager, this is more than academic: knowing which factors drive returns allows to construct portfolios with targeted exposures, hedge unwanted risks, and forecast performance across different market regimes. Rumors even says factor models are used to do alpha. But at their core, factor models are not just a regression trick, they are deeply connected to the stochastic discount factor (SDF), the central object of modern asset pricing (Cochrane, 2005).

Historically, factor models were designed to be parsimonious. The CAPM is the cleanest example: a single factor, the market portfolio, is enough to generate testable implications. Why the focus on simplicity? Because in the 1970s–1990s, working with too many factors was computationally and statistically intractable. With only a few regressors, the covariance matrix is easy to invert, and betas admit a neat closed-form solution.

Yet, as empirical work expanded, the factor zoo appeared: dozens, then hundreds of characteristics proposed as “priced risks.” Each paper brought new candidates — value, momentum, profitability, investment, and so on. And more recently, the field has pushed even further. The Instrumented Principal Components Approach (IPCA) of Kelly et al. (2019) and related machine learning methods embrace high-dimensional SDF learning, where the discount factor is estimated flexibly from a vast space of predictors.

This evolution leaves us with a natural question: Should asset pricing rely on fewer factors, or embrace more? In this post, we will take a historical perspective, from the early parsimonious models to today’s machine-learning approaches, and compare their strengths and weaknesses. Let’s try together to trace this journey.

<!--
Schematic timeline (CAPM → Factor Zoo → IPCA / ML)
Paste this directly into a Jekyll post (inside HTML) or save as .svg
Accessible: includes <title> and <desc>
-->
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
  <text x="150" y="178" text-anchor="middle" class="node-sub">Parsimonious — single market factor</text>
  <text x="150" y="196" text-anchor="middle" class="year">(Sharpe, Lintner — canonical 1960s)</text>

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
  <text x="430" y="160" text-anchor="middle" class="node-label">Factor Zoo</text>
  <text x="430" y="178" text-anchor="middle" class="node-sub">Many proposed characteristics / factors</text>
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
  <text x="710" y="160" text-anchor="middle" class="node-label">IPCA / ML</text>
  <text x="710" y="178" text-anchor="middle" class="node-sub">Instrumented PC & high-dimensional SDF learning</text>
  <text x="710" y="196" text-anchor="middle" class="year">(Kelly et al., 2019 → present)</text>

  <!-- Optional connecting dots (timeline progression) -->
  <g fill="#2b2b2b" opacity="0.12">
    <circle cx="280" cy="110" r="4"></circle>
    <circle cx="340" cy="110" r="4"></circle>
    <circle cx="520" cy="110" r="4"></circle>
    <circle cx="610" cy="110" r="4"></circle>
  </g>

  <!-- Small caption (non-essential for SEO but informative) -->
  <text x="30" y="30" class="node-sub" style="font-size:13px">Schematic: evolution from parsimonious factor models toward high-dimensional SDF estimation.</text>
</svg>
</div>


## The Case for Less: Low-Dimensional Factor Models and Interpretability  {#smallmodel}


## The Case for More: High-Dimensional SDF Learning with Machine Learning  {#bigmodel}  


## Conclusion  {#conclusion}  



