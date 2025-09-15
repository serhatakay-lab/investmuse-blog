---
title: "A Hidden Markov Model Analysis for Gold Historical Data"
date: 2025-09-15T10:00:00Z
draft: false
description: "Deep dive into gold price patterns using Hidden Markov Models to identify market regimes and trading opportunities through mathematical analysis"
tags: ["gold", "hidden-markov-models", "quantitative-analysis", "trading", "technical-analysis", "R", "time-series"]
categories: ["Quantitative Analysis", "Market Research"]
---

<style>
img {
  max-height: 60vh !important;
  width: auto !important;
  max-width: 100% !important;
  object-fit: contain !important;
  display: block !important;
  margin: 0 auto !important;
}
figcaption {
  text-align: center !important;
  font-style: italic !important;
  color: #666 !important;
  margin: 0.5rem auto 1.5rem auto !important;
  font-size: 0.9rem !important;
}
</style>

What is "Hidden States" concept? Why are the states "Hidden" and not observable? How can we foresee the states in Candlestick Chart data with the power of mathematics?

<figure>
<img src="/images/gold-market-regimes-timeline.png" alt="Gold Market Regimes Timeline">
<figcaption>Market Regimes by Years (Steady State, Walking on Ice, Crisis, Inflation)</figcaption>
</figure>

## Understanding Hidden Market Regimes

In a **Hidden Markov Model (HMM)**, the model identifies underlying "hidden" regimes or states in the data that may not be immediately obvious from any individual data point. When you look at an individual row for a specific date, you see just the observable data (like price or volume), but it's not always clear which market regime that data point belongs to. The HMM, on the other hand, learns from the entire dataset, finding patterns in the data sequence to infer the most probable hidden state (like bull, bear, strong bull, etc.) over time.

So, the model doesn't just look at a single data point but instead interprets patterns across a sequence of data. By understanding the transitions and probabilities of moving from one regime to another, the HMM can recognize when the market is likely in a "bullish" or "bearish" phase, even if a single day's data might not clearly indicate this.

## The Historical Foundation of Markov Models

The Markov Chain concept was introduced by the Russian mathematician **Andrey Markov** in the early 20th century, but his original motivation was actually not about financial time series or even time series analysis in general. Instead, Markov chains were initially devised to explore sequences of events with probabilistic dependencies — especially in systems where each event's probability depends on the previous event. His main goal was to challenge the prevailing view at the time that only independent events, like coin tosses, could be analyzed statistically.

### Markov's Motivation

Markov's initial experiments in 1906 used sequences of letters in poems to analyze patterns in a series where each letter's probability could depend on the previous letter. His purpose was to show that mathematical analysis could be applied to dependent events as well, rather than just independent events. Markov introduced Markov chains to describe sequences of events where the probability of each event depends only on the state of the previous event. This property is now known as the **Markov property** (or "memorylessness").

Markov famously demonstrated his theory using a sequence of letters in the Russian poem "Eugene Onegin" by Alexander Pushkin. By analyzing the occurrence of vowels and consonants, he showed that the likelihood of one type of letter following another was not independent. This analysis proved that statistical tools could handle sequential dependencies.

### Evolution to Time Series and the "Big Picture"

Over time, the Markov model was adapted and extended, leading to tools like the Hidden Markov Model (HMM), which allowed statisticians and scientists to infer unobserved states (like "bullish" or "bearish" market regimes) from observable data (such as daily prices).

## Why Time Series Applications are Suitable for Markov Models

**1. Sequential Dependency:** Many time series, including stock prices, weather patterns, and economic indicators, exhibit dependency where the next value or state is influenced by the current or previous states — perfect for the Markov property.

**2. Hidden Structure:** In financial markets, there are often underlying "hidden" forces (e.g., economic cycles, sentiment shifts) that influence observable data. The Hidden Markov Model allows for identifying these hidden regimes.

**3. Pattern Detection:** By observing sequences over time, HMMs can help uncover long-term trends or regime shifts that aren't obvious from individual data points. This "big picture" is what financial analysts are often interested in.

Following Markov's discrete-time work, researchers extended the model to continuous-time Markov chains. These models describe systems that change state continuously over time, which is valuable in physics, biology, and queueing theory.

By the mid-20th century, Markov chains were applied to weather prediction. Early meteorologists used Markov models to estimate the likelihood of weather conditions, like sunny or rainy days, based on the previous day's weather. This application showed how Markov chains could forecast future states, making them practical for predicting time series trends.

## Gold Data Analysis

Below R code creates a Hidden Markov Model (HMM) for identifying and visualizing market regimes in daily gold data, classified into four market states. Here's a step-by-step guide to understanding each part of the code and experimenting with various parameters to fine-tune the model for different outcomes:

### Step 1: Setting Up Libraries and Data

```r
# Ensure required libraries are installed and loaded
packages <- c("depmixS4", "TTR", "quantmod", "tidyverse", "expm")
for (pkg in packages) {
  if (!require(pkg, character.only = TRUE)) {
    install.packages(pkg, dependencies = TRUE)
    library(pkg, character.only = TRUE)
  }
}

# Get Gold Price data from Yahoo
getSymbols("GC=F", src = "yahoo", from = "2000-01-01", to = Sys.Date())

data <- data %>%
  mutate(
    Returns = log(Close / lag(Close)),
    RSI = TTR::RSI(Close, n = 14),
    Volatility = TTR::volatility(Close, n = 14, calc = "close")
  ) %>%
  na.omit()
```

Here:
- **Returns:** Measures daily returns
- **RSI (Relative Strength Index):** Indicates overbought or oversold conditions
- **Volatility:** Shows price fluctuations over a rolling 14-day period

### Step 2: Defining the HMM Model

The model is created with 4 states, each representing a possible market regime (e.g., strong bear, weak bear, weak bull, strong bull). Each state is defined by Gaussian distributions of Returns, RSI, and Volatility:

```r
hmm_model <- depmix(
  response = list(Returns ~ 1, RSI ~ 1, Volatility ~ 1),
  data = data,
  nstates = 4,
  family = list(gaussian(), gaussian(), gaussian())
)
```

**Tuning Parameters:**
- **Number of states (nstates):** Increasing the number of states may capture more subtle nuances in market behavior but can lead to overfitting. Start with a lower number if uncertain.
- **Response variables:** You could add or remove variables (like adding Momentum) to test different models.

### Step 3: Setting Initial Transition Probabilities

The initial transition probabilities define the likelihood of transitioning between states. Here, each state has a high probability (0.85) of remaining in the same state:

```r
init_transitions <- c(0.85, 0.05, 0.05, 0.05, # Row 1
                      0.05, 0.85, 0.05, 0.05, # Row 2
                      0.05, 0.05, 0.85, 0.05, # Row 3
                      0.05, 0.05, 0.05, 0.85) # Row 4
```

This setup emphasizes stability within each state, so the system tends to remain in a given regime unless there is a significant shift.

**Experimentation:**
- **Increase/decrease self-transition probabilities (like 0.85):** Lowering this value increases transitions to other states, simulating a more dynamic market.
- **Change transition probabilities between states:** For example, increasing the transition from a weak bear to a weak bull could capture more frequent market reversals.

### Step 4: Fitting the Model

The fit function optimizes model parameters for the given data:

```r
hmm_fit <- fit(hmm_model)
```

### Step 5: Analyzing the Transition Matrix and Steady States

The transition matrix describes the probabilities of moving from one state to another after the model is fitted. Additionally, we compute steady states to see the long-term probability of each regime:

```r
transition_matrix <- matrix(getpars(hmm_fit)[seq_len(4^2)], nrow = 4, byrow = TRUE)
steady_states <- (transition_matrix %^% 100)[1, ]
```

### Step 6: Predicting States and Visualizing

Finally, we extract and plot the state (regime) assignments for each data point:

```r
states <- posterior(hmm_fit)$state
data$State <- factor(states, labels = c("Strong Bear", "Weak Bear", "Weak Bull", "Strong Bull"))
```

Each state is visualized with different colors to show the market's changing behavior over time:

```r
ggplot(data, aes(x = Date, y = Adjusted, color = State, group = 1)) +
  geom_line() +
  scale_color_manual(values = c("Strong Bear" = "red",
                                "Weak Bear" = "orange",
                                "Weak Bull" = "yellow",
                                "Strong Bull" = "green")) +
  labs(title = "Price with Hidden Market Regimes",
       x = "Date",
       y = "Price",
       color = "Market Regime") +
  theme_minimal()
```

<figure>
<img src="/images/gold-price-hidden-regimes.png" alt="Gold Price with Hidden Market Regimes">
<figcaption>Gold Price Data Market Regimes after 2000</figcaption>
</figure>

## Considerations and Experimentation Tips

- **Number of States:** Adding more states may identify specific patterns (e.g., volatile uptrends) but can increase complexity.

- **Response Variables:** Experiment with different variables (e.g., Momentum, Moving Average) for tailored insights.

- **Transition Probabilities:** Customize the transition matrix based on expected market behaviors (e.g., sudden drops in volatile markets).

This model provides a simplified method for identifying market regimes, allowing for deeper analyses and potential strategy testing based on state transitions. Experimenting with parameters will provide insights into the model's sensitivity and relevance to different market conditions.

---

*This analysis demonstrates how mathematical models originally developed for analyzing poetry can be applied to modern financial markets, revealing hidden patterns that guide investment decisions.*
