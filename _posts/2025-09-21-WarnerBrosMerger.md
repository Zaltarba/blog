---
layout: post
title: "Merger Arbitrage Explained: Warner Bros Discovery, Paramount & Market-Implied Probabilities"
categories: [Quantitative Finance, Python]
excerpt: "Learn how to analyze merger arbitrage opportunities using the Warner Bros Discovery and Paramount deal rumors. Break down market-implied probabilities, downside risk, and expected value like a quant."
image: /thumbnails/WarnerBros.jpeg
hidden: False
tags: [Merger Arbitrage, Warner Bros Discovery, Paramount, Skydance, Event-Driven, Quantitative Finance, Quant]
---

## Disclaimer 

This post is for informational and educational purposes only. It does not constitute financial, investment, or trading advice. The author currently holds a position in Warner Bros Discovery (WBD) at the time of writing. 

## Table of Contents

1. [What Is Merger Arbitrage?](#what-is-merger-arbitrage)
2. [Understanding Implied Probability from Market Prices](#understanding-implied-probability-from-market-prices)
3. [A Simple Merger Arbitrage Methodology](#merger-arbitrage-methodology)
4. [Case Study: Warner Bros Discovery & Paramount/Skydance](#case-study-warner-bros-discovery-paramountskydance)
    - 4.1 [Deal Overview](#deal-overview)
    - 4.2 [Price-Based Probability Estimation](#price-based-probability-estimation)
    - 4.3 [Sensitivity Analysis](#sensitivity-analysis)
5. [Risks and Considerations](#risks-and-considerations)
6. [Building an Arbitrage Framework](#building-an-arbitrage-framework)
7. [Conclusion](#conclusion)

## What Is Merger Arbitrage  {#what-is-merger-arbitrage}

I’ve held a large position in Warner Bros Discovery (WBD) since 2023, so when news broke of a potential takeover by Paramount/Skydance, it naturally caught my attention. This post is partly motivated by that position, and more broadly by the opportunity to walk through how merger arbitrage can be evaluated when a real-world event unfolds.

Let’s walk through it.

Merger arbitrage is an event-driven investment strategy built around one simple idea: when a company announces it’s acquiring another, there's a temporary pricing inefficiency and that create an opportunity. Here’s how it works.

When an acquisition is announced, the acquirer offers to buy the target company, **usually at a premium**. For example, they might offer 24\( \$ \) per share for a company that was trading at 15\( \$ \). The target's stock typically jumps on the news... but not all the way to 24\( \$ \). Why not? Because there’s still risk: the deal might fall through, get delayed, or be blocked by regulators. So the market applies a discount, pricing in the possibility of failure.

This gap between the current market price of the target and the deal price is called the **spread**. Merger arbitrageurs make bets on it to make an arbitrage.

In a pure **cash deal**, the arbitrage setup is straightforward: you buy the target company’s shares after the deal is announced and wait. If the deal closes, the acquirer pays cash, and you pocket the difference. If the deal breaks, the stock usually drops, sometimes sharply, and you lose money. So the whole game is about **probabilities** and **payoffs**. In this post I will only talk about this kind of deal.

The wider the spread, the more the market is signaling uncertainty. A narrow spread? The market sees a high chance of the deal going through. A wide spread? More risk or at least, more perceived risk.

In some cases, the spread can even go negative, the stock trades above the offer price. That usually means investors are betting on a bidding war or a sweetened deal.

Merger arbitrage isn’t about guessing whether a stock will go up or down. It’s about **evaluating a specific corporate event**, estimating the probability of success, and judging whether the potential return justifies the risk. It's closer to probabilistic thinking than traditional investing.

It’s not glamorous. It’s not momentum-driven. But when done systematically with proper sizing, scenario modeling, and discipline it can be a effective strategy (here a [first article](https://www.aima.org/asset/676DA5D6-8CE4-42D7-A2C6171EAC6382DC/) and a [second one](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=315639) documenting this assertion).

## Understanding Implied Probability from Market Prices  {#understanding-implied-probability-from-market-prices}

When a public company announces it’s being acquired, the offer is almost always above the last trading price, often significantly. That premium is what makes merger arbitrage possible.

Let’s work on a simple example. Imagine a small public company : let's call it Banana. It is trading at 10\( \$ \) per share. One morning, Apple announces it wants to acquire that company for 20\( \$ \) a share, in cash. That’s a 100% premium.

What happens to the stock? It jumps, but not to 20\( \$ \). Maybe it shoots up to 18\( \$ \) or 19\( \$ \). That difference between the offer price 20\( \$ \) and the new trading price (say, 18\( \$ \)) is called the **spread**. And it contains valuable information.

### The Spreak is the Market-Priced Deal Risk

If the market believed with 100% certainty that the deal would close exactly as proposed, the stock would immediately trade at 20\( \$ \). But that almost never happens. There’s always *some* risk : antitrust issues, financing problems, board pushback, political pressure, or just the chance that the acquirer changes its mind.

So the market prices in that uncertainty. In our example, the 2\( \$ \) gap between 18\( \$ \) and 20\( \$ \) is how the market *discounts* the possibility that the deal might not happenn or might happen later, or on worse terms.

From that spread, we can actually **estimate the implied probability** of the deal closing.

### A Simple Formula

We can model the current price as a weighted average of two outcomes:

- The deal closes, and the target gets 20\( \$ \)  
- The deal fails, and the target falls back to its pre-deal price (say, 10\( \$ \))

We can write this as:

$$
\text{Market Price} = p \times \text{Deal Price} + (1 - p) \times \text{Fallback Price}
$$

Where:
- $p$ is the **implied probability** of deal success
- Deal Price is the offer amount (e.g. 20\( \$ \))
- Fallback Price is the estimated value if the deal fails (e.g. 10\( \$ \))

Solving for $p$:

$$
p = \frac{\text{Market Price} - \text{Fallback Price}}{\text{Deal Price} - \text{Fallback Price}}
$$

Plug in the numbers from our example:

$$
p = \frac{18 - 10}{20 - 10} = \frac{8}{10} = 0.80
$$

So, in this case, the market is pricing in **an 80% chance** that the deal will close as announced.

## Merger Arbitrage Methodology  {#merger-arbitrage-methodology}

The implied probability isn’t just academic : it’s your decision-making compass.

If you’re looking at a potential merger arbitrage setup, here’s the process in plain terms:

- **Step 1: Understand what the market is pricing in.**  
  The current spread between the offer price and the trading price reflects the market’s best guess at the probability the deal will close. Not with certainty, but under prevailing assumptions: risk, timing, regulatory pressure, etc.

- **Step 2: Decide whether you agree.**  
  This is the core of the strategy. You’re not predicting prices — you’re evaluating scenarios. Is the market underestimating the chance of success? Are people missing a regulatory blocker? Has the risk already been overstated?

- **Step 3: Think in outcomes, not absolutes.**  
  There are usually two (or more) outcomes: the deal closes, or it doesn’t. Each comes with a probability, a timeline, and a payoff. Your goal is to model the expected value (EV) of the trade, given those inputs.

### The Expected Value Framework

Let’s break it into a simple two-outcome model:

- **Deal Price (D):** The price paid if the acquisition closes  
- **Current Price (C):** What the target trades at now  
- **Fallback Price (F):** Estimated value if the deal fails  
- **Probability of Success (p):** Your estimate (or market’s implied)  

Then your **expected value (EV)** per share is:

$$
\text{EV} = p \times D + (1 - p) \times F
$$

If EV is meaningfully above the current price (after adjusting for time and risk), it’s potentially a good trade. If not, you pass.

### A Quick Example

Suppose:

- Deal price = 24\( \$ \)   
- Current stock price = 19\( \$ \)  
- Fallback (if deal fails) = 12\( \$ \)  

Using the formula to back out the **implied probability**:

$$
p = \frac{19 - 12}{24 - 12} = \frac{7}{12} \approx 58\%
$$

Now suppose you think, because you are an expert in the industry for example, that the chance of success is more like 70%. Plug it in:

$$
\text{EV} = 0.70 \times 24 + 0.30 \times 12 = 16.8 + 3.6 = 20.4
$$

The expected value under your assumptions is 20.40\( \$ \), which is above the current price 19\( \$ \). That suggests a potential edge *if* your assumptions are more accurate than the market’s. When making a are more complex model, you should take into account also the time required for the deal to go trough, and think more into target and fallback distributions than fixed parameters.


## Case Study: Warner Bros Discovery & Paramount/Skydance  {#case-study-warner-bros-discovery-paramountskydance}

### Deal Overview  {#deal-overview}

According to a [New York Post report (Sep 19, 2025)](https://nypost.com/2025/09/19/media/paramount-skydance-eyes-takeover-bid-for-warner-bros-discovery-report/), Skydance Media, which recently acquired control of Paramount Global, is now exploring a bid to acquire Warner Bros Discovery at a price in the range of **22\( \$ \)–24\( \$ \) per share**.

This isn’t a formal offer (yet), but it’s enough to move the market. Want to see it ? Here the Warner Bros stock price over the last days (and bonus the Python code to do it at home) : 

```python
import yfinance as yf
import matplotlib.pyplot as plt
import datetime

# Fetch WBD price data
ticker = yf.Ticker("WBD")
start_date = "2025-06-01"
end_date = "2025-09-21"
data = ticker.history(start=start_date, end=end_date)

# Plot
plt.figure(figsize=(10, 5))
plt.plot(data.index, data["Close"], label="WBD Close Price", color="navy")
plt.axvline(datetime.datetime(2025, 9, 10), color='red', linestyle='--', label="Deal Rumor (Sep 19)")
plt.title("WBD Stock Price – June to September 2025")
plt.xlabel("Date")
plt.ylabel("Price ($)")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

<img src="/blog/images/WarnerBros_fig_1.png" alt="Warner Bros Stock Evolution">

Analyst reactions have been mixed:

- TD Cowen’s Doug Creutz downgraded WBD from Buy to Hold, arguing that without a deal, the stock could fall back to **11\( \$ \)–12\( \$ \)**, based on fundamentals. ([MarketWatch, Sep 20, 2025](https://www.marketwatch.com/story/buying-wbds-stock-ahead-of-a-possible-paramount-deal-this-analyst-is-cautious-0b4fa36c))

How to get info like that easily at home ? Here again python is your friend. Here a small code enabling you to get recent headlines on any ticker :

```python
import yfinance as yf
from datetime import datetime

# Fetch the lastest news article for Warner Bros Discovery (WBD)
ticker = yf.Ticker("WBD")
news_items = ticker.get_news(count=1)

# Format and print the news like an RSS reader
item = news_item['content']
title = item["title"]
published = item["displayTime"]
link = item["canonicalUrl"]['url']
summary = item['summary']
    
print(f"📌 {title}")
print(summary)
print(f"🔗 {link}\n")
```

```bash
📌 Wells Fargo Raises Warner Bros. Discovery (WBD) Price Target, Sees M&A Potential
Warner Bros. Discovery Inc. (NASDAQ:WBD) ranks among the best communication services stocks to buy now.
Wells Fargo boosted its price target for Warner Bros. Discovery Inc. (NASDAQ:WBD) to 14$ from 13$ on
September 11, retaining an Equal Weight rating on the company’s shares. The revision reflects
Wells Fargo’s belief that Warner Brothers Discovery’s Studios and […]
🔗 https://finance.yahoo.com/news/wells-fargo-raises-warner-bros-075936106.html
```

So what is the market currently pricing in? Let’s quantify it.

### Price-Based Probability Estimation  {#price-based-probability-estimation}

Let’s plug in some numbers based on the reporting and price action:

- **Deal price (X)**: We will use 24\( \$ \) 
- **Current price (Y)**: 19\( \$ \) (WBD has traded between 18\( \$ \)–20\( \$ \) since the rumor broke)  
- **Fallback price (B)**: 11\( \$ \) (where analysts estimate WBD would trade without a deal)

Using the implied probability formula:

$$
p = \frac{Y - B}{X - B} = \frac{19 - 11}{24 - 11} = \frac{8}{13} \approx 61.5\%
$$

So based on this spread, the market is implying roughly a **61–62% chance** that the deal goes through *as rumored*.

Again using *yfinance* and *Python* you can easily look at the evolution of the merger probability (under this parameters of deal price and fallback of course) : 

import yfinance as yf
import matplotlib.pyplot as plt

```python
# Fetch WBD price data
ticker = yf.Ticker("WBD")
start_date = "2025-06-01"
end_date = "2025-09-21"
data = ticker.history(start=start_date, end=end_date)

deal_price = 24  # Assumed acquisition price
fallback_price = 11  # Assumed fallback price

data["Implied_Probability"] = (data["Close"] - fallback_price) / (deal_price - fallback_price)

plt.figure(figsize=(10, 4))
plt.plot(data.index, data["Implied_Probability"], label="Implied Deal Probability", color="purple")
plt.axhline(0.5, color='gray', linestyle='--', alpha=0.7)
plt.title("Market-Implied Probability of WBD Acquisition")
plt.xlabel("Date")
plt.ylabel("Probability")
plt.ylim(0, 1)
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

<img src="/blog/images/WarnerBros_fig_2.png" alt="Market-Implied Probability of WBD Acquisition">

Let’s break down the potential payoff structure:

- **Upside if deal closes**: 5\( \$ \) per share → about **+26.3%** return from 19\( \$ \) to 24\( \$ \)  
- **Downside if deal fails**: Loss of 8\( \$ \) per share → about **–42.1%** from 19\( \$ \) to 11\( \$ \)  

If the deal closes in **6 months**, that’s a 26% return in half a year, or over **50% annualized** (not adjusting for taxes or compounding). But **this return is priced in**. Meaning the risk of dowside exist and is priced. The question you must ask yourself in this situation is more : is the market probability near the truth ? ie is the market efficient ? 

### Sensitivity Analysis  {#sensitivity-analysis}

The final step is asking: *how sensitive is this trade to changes in assumptions?*

Let’s run a few scenarios.

#### Scenario A: More optimistic fallback

Suppose the market is wrong about the downside, and the true fallback price is 14\( \$ \), not 11\( \$ \). Then:

$$
p = \frac{19 - 14}{24 - 14} = \frac{5}{10} = 50\%
$$

So in this more optimistic view, the **implied probability drops to 50%**, because the downside isn’t as severe.

#### Scenario B: Lower deal price

What if the deal gets negotiated down to 22\( \$ \) ? Then:

$$
p = \frac{19 - 11}{22 - 11} = \frac{8}{11} \approx 72.7\%
$$

Ironically, a **lower offer price** results in a *higher implied probability* — because the spread narrows.

This shows that the probability estimate is sensitive to both inputs: the upside and the fallback. What if we want to look at it in more scalable fashion ? The 24\( \$ \) dollars per share is maybe a good payoff guess, but what about the fallback price ? Warner Bros Stock has move from 8 to 14\( \$ \) regularly over the last 2 years. Again (and for the last time) we can use Python to study the sensitivity to this fallback price. What we want to see is the *implied probability*. 

```python 
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# Set deal price
deal_price = 24
# Define fallback and market price ranges
fallbacks = np.arange(8, 12.5, 0.5)  # $8 to $12
prices = np.arange(15, 23.5, 0.5)    # $11 to $24

# Create a matrix to store implied probabilities
spread_matrix = np.zeros((len(fallbacks), len(prices)))

# Compute implied probabilities
for i, b in enumerate(fallbacks):
    for j, y in enumerate(prices):    
        spread_matrix[i, j] = (y - b) / (deal_price - b)
    
# Clip to range [0, 1] for clarity
spread_matrix = np.clip(spread_matrix, 0, 1)
spread_matrix = spread_matrix[::-1, :]
# Plot heatmap
plt.figure(figsize=(12, 6))
sns.heatmap(spread_matrix, annot=True, fmt=".2f", cmap="YlGnBu",
            xticklabels=prices, yticklabels=fallbacks[::-1],
            cbar_kws={'label': 'Implied Probability'})

plt.xlabel("Market Price ($)")
plt.ylabel("Fallback Price ($)")
plt.title("Implied Probability Sensitivity — WBD Deal at 24$")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

<img src="/blog/images/WarnerBros_fig_3.png" alt="Implied Probability Sensitivity — WBD Deal at 24">

### What Makes This Deal Tricky?

Beyond price modeling, this deal carries **significant regulatory risk**.

According to a [Columbia Journal of Law & the Arts article](https://journals.library.columbia.edu/index.php/lawandarts/announcement/view/826), a Paramount–WBD merger would likely trigger scrutiny under the 2023 DOJ/FTC Merger Guidelines. The combined entity would have substantial market share in theatrical distribution, streaming, and television — possibly pushing it into “highly concentrated” territory by antitrust standards.

Key regulatory concerns:

- **Horizontal overlap** in film studios and content production  
- **Vertical integration** risks (content + distribution)  
- **Media plurality** concerns (CNN + CBS News under one roof)  
- **Public interest tests** from the FCC (because of news media and licenses)

So even if Skydance makes a formal offer, the path to closing could be long — and far from guaranteed.

### Where We Are Now

To sum up:

- The market is pricing in **~60% chance** of a 24\( \$ \) share deal based on current prices and downside estimates  
- Regulatory headwinds are real — and potentially deal-breaking  
- If you think the market is **too cautious**, there may be opportunity  
- If you think the market is **too optimistic**, it might be time to hedge or trim

The rest depends on how you model risk and whether you trust your estimates more than the market.

## Conclusion  {#conclusion}

The Warner Bros Discovery–Paramount/Skydance case offers a real-time example of how markets digest deal rumors, price in uncertainty, and reflect a collective estimate of risk, all through a single number: the stock price.

This case illustrates a few key points:

- **Markets imply probabilities** through price — not opinion. While analysts debate whether WBD is worth 11\( \$ \) or 14\( \$ \) without a deal, the stock price incorporates not just that debate, but the *likelihood* of various outcomes.
- **Implied probability is not truth** : it's just what the current prices suggest. If you have a differentiated view on the deal's odds or timing, that’s where alpha can live.
- **A simple framework goes a long way**. By mapping out:
  - the potential deal price (upside)
  - the fallback value (downside)
  - the timeline to closing
  - the regulatory and strategic context
  ...you can structure a probabilistic view of the trade, rather than just a gut feeling.

This isn’t forecasting. It’s scenario modeling and it forces clarity of thought.

What we’ve walked through here is a first-principles framework: pricing, spreads, probabilities, payoffs.

But more complex models are absolutely possible:

- **Multi-scenario trees**, with branching outcomes (e.g. competing bids, revised terms, partial deals)
- **Bayesian updates** as new information arrives (regulatory signals, earnings calls, leaks)
- **Monte Carlo simulations** to model a range of regulatory, political, or timing variables
- **Option-based models** to account for volatility in both upside and fallback prices

You don’t *need* these tools to start — but as the stakes grow, or as you allocate more capital to merger arbitrage, adding depth to your modeling can help improve risk management and return profile.




