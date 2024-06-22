---
layout: post
title: An Introduction to Pair Trading and Market Neutral Strategies
categories: [Personnal project, Practical paper]
excerpt: This post is the first of a serie about pair trading and market neutral strategies ...
---

## Introduction

I have started working on pairs trading and market-neutral trading strategies in the US equity market over the last few months. Man, what a challenge! Here, I will try to give a clear and concise introduction to the topic and introduce the frameworks I will be delving into more deeply in the next posts.

## General Overview

Before speaking of pairs trading, let's first define market-neutral strategies. A market-neutral strategy aims to provide uncorrelated sources of returns from the market. The Capital Asset Pricing Model (CAPM), even if now outdated, provides a nice framework to explain the intuition:

The return of an asset can be expressed as $r_t - r_f = \alpha + (\beta \cdot (r_m - r_f)) + \epsilon_t$, with:
- $r_f$ the risk-free return
- $r_m$ the market return
- $\alpha$ the expected excess return
- $\beta$ the correlation with the market

With this framework, if you manage to construct a portfolio with a null beta (i.e., the linear combination of the assets within is such that the sum of their betas is null), you then get a portfolio whose returns are uncorrelated from those of the market! Bear and bull markets don't matter anymore! What's the point? Why not just OTM calls with one week until expiration and pray to get rich? Well, to that, I would say, based on my understanding of Harry Markowitz's works, that you surely should do both. I am sure an efficient frontier exists mixing those strategies. 

Anyway, when working on a market-neutral strategy, the quant researcher's role is thus to reduce its strategy's beta as much as possible and to maximize its alpha (i.e., its expected outperformance).

## About Pairs Trading 

Sorry had to go to lunch 
