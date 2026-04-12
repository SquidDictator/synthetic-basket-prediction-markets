# Exploring Synthetic Basket Relationships in Prediction Markets

## Overview

This project explores whether relative mispricing exists between related contracts in prediction markets. A synthetic basket is constructed from multiple teams, and deviations from this basket are analysed using a simple mean-reversion framework.

The aim is not to build a production-ready strategy, but to test whether inconsistencies in relative probabilities can be observed under simplified assumptions.

---

## Motivation

In traditional markets, synthetic baskets are used in relative value strategies to compare assets against a constructed benchmark.

In prediction markets, contracts are inherently interdependent. For example, teams competing for a limited number of positions create a constrained probability structure, where changes in one contract should influence others.

This project investigates whether short-term inconsistencies arise in this structure, and whether they resemble mean-reverting behaviour or reflect differences in update timing and liquidity.

---

## Methodology

A synthetic basket is constructed as the average probability of two teams. The spread between a target team and the basket is calculated and standardised using a rolling z-score.

Trading signals are generated when the spread deviates beyond a threshold and closed when it reverts. A simplified cost model is included, consisting of:

* flat commission per trade
* volatility-scaled slippage

The backtest uses hourly resampled data with forward filling to handle missing observations.

---

## Results

Results show that performance is not uniform across teams. While some configurations remain profitable after costs, others weaken significantly.

This suggests that the observed behaviour may not reflect a consistent mean-reversion effect, but instead may be driven by asymmetric factors such as differences in liquidity, update frequency, or market attention across contracts.

---

## Key Insight

Interdependence between contracts does not necessarily imply mean reversion.

While probabilities are linked through competition for limited outcomes, deviations may reflect new information or asynchronous updates rather than temporary mispricing.

This highlights the importance of distinguishing between structural relationships and tradable inefficiencies.

---

## Limitations

The dataset contains missing and irregular observations, and forward filling may introduce artificial divergence between contracts.

The synthetic basket is constructed using a simple average, which is a simplification of how baskets would be built in practice. Parameter choices have not been fully stress-tested, and results may be sensitive to these assumptions.

As such, findings should be interpreted as exploratory rather than definitive.

---

## Next Steps

Future work could focus on:

* testing lead-lag relationships between contracts
* exploring alternative basket constructions (e.g. weighted or regression-based)
* incorporating more realistic execution constraints

---

## Repository Structure

* `notebook.ipynb` — main analysis and backtest
* `results/` — generated plots
* `AllTop4Data.csv` — input data

---

## Summary

This project demonstrates how synthetic baskets can be used to explore relative relationships in prediction markets, and highlights the challenges of distinguishing true inefficiencies from data and market structure effects.
