```markdown
# Exploring Synthetic Basket Relationships in Prediction Markets

### A small experiment in relative-value trading and synthetic baskets using prediction-market data.

---

## Overview

The Premier League Top 4 race provides a natural example of interdependent prediction-market contracts. Because there are a limited number of qualifying positions, changes in one team's probability may affect the probabilities of its competitors.

This project tests a simple relative-value strategy:

> When one team's Top 4 probability becomes unusually expensive or cheap relative to a basket of peer teams, take a position that benefits if the difference returns toward its recent average.

For each target team, the average probability of the other two teams is used as a synthetic benchmark. The difference between the target and this benchmark is monitored over time.

The project investigates whether unusually large deviations contain systematic mean-reverting behaviour, or whether they instead reflect differences in liquidity, market attention and the speed at which information is incorporated into related contracts.

The goal is not to present a production-ready trading system, but to explore the structure and limitations of relative-value signals in prediction markets.

---

## Strategy at a Glance

The strategy trades mean reversion in the relative spread between one team and a basket of peer teams.

For each target team:

1. Calculate the average probability of the other two teams.
2. Use this average as a synthetic basket.
3. Subtract the basket probability from the target probability.
4. Measure whether the resulting spread is unusually high or low relative to its recent history.
5. Take a relative-value position that benefits if the spread returns toward its rolling average.
6. Close the position when the spread has sufficiently reverted.

The trading rules are:

- If the target is unusually cheap relative to the basket, go **long the target and short the basket**.
- If the target is unusually expensive relative to the basket, go **short the target and long the basket**.
- Exit when the relative spread moves back toward its recent average.
- Reverse the position if the signal crosses the opposite entry threshold.

The strategy is therefore not attempting to predict whether a team's probability will rise or fall in absolute terms. It is attempting to predict how the target will perform **relative to the peer basket**.

### Example

Suppose the market probabilities are:

- Aston Villa Top 4: 40%
- Manchester United Top 4: 50%
- Chelsea Top 4: 46%

The synthetic basket for Aston Villa is:

\[
\text{Basket} = \frac{50\% + 46\%}{2} = 48\%
\]

The relative spread is:

\[
\text{Spread} = 40\% - 48\% = -8\%
\]

A negative spread does not automatically mean Aston Villa is mispriced. Villa may normally trade below the other two teams because of differences in team quality, fixtures or market expectations.

The strategy instead asks whether the current spread is unusually negative compared with its own recent history.

If it is, the model takes a simulated position that is:

- Long Aston Villa
- Short the equal-weight Manchester United and Chelsea basket

The trade profits if Aston Villa subsequently outperforms the basket.

Aston Villa's probability does not necessarily need to rise. The trade can also profit if Villa falls by less than Manchester United and Chelsea.

---

## Motivation

Synthetic baskets are commonly used in traditional relative-value strategies to compare an asset with a constructed benchmark of related assets.

Prediction markets provide a useful setting for this type of analysis because contracts representing related outcomes may be economically linked.

Teams competing for the same limited number of Top 4 positions create a constrained probability structure. Positive information about one team may reduce the relative chances of its competitors, while market participants may incorporate the same information into different contracts at different speeds.

This raises two questions:

1. Do temporary inconsistencies emerge between related prediction-market contracts?
2. Are those inconsistencies large and persistent enough to remain after estimated execution costs?

---

## Methodology

### Basket Construction

For each target team, a synthetic basket is constructed as the equal-weight average probability of the remaining two teams.

For target team \(i\):

\[
B_{i,t} = \frac{P_{j,t} + P_{k,t}}{2}
\]

where:

- \(P_{i,t}\) is the target team's probability at time \(t\)
- \(P_{j,t}\) and \(P_{k,t}\) are the probabilities of the two peer teams
- \(B_{i,t}\) is the synthetic basket probability

The relative spread is defined as:

\[
S_{i,t} = P_{i,t} - B_{i,t}
\]

A positive spread means the target is priced above the basket.

A negative spread means the target is priced below the basket.

The strategy does not assume that the fair value of the spread is zero. Persistent differences between teams may be justified by team quality, form, fixtures or other information.

Instead, the strategy compares the spread with its own recent behaviour.

---

### Signal Generation

The spread is standardised using a rolling mean and rolling standard deviation:

\[
z_t = \frac{S_t - \mu_t}{\sigma_t}
\]

where:

- \(S_t\) is the current relative spread
- \(\mu_t\) is the rolling mean of the spread
- \(\sigma_t\) is the rolling standard deviation
- \(z_t\) measures how unusual the current spread is

The signal rules are:

- **Large negative z-score:** long the target and short the basket
- **Large positive z-score:** short the target and long the basket
- **Exit:** close the position when the z-score moves back toward zero
- **Reversal:** switch direction if the z-score reaches the opposite entry threshold

The position is designed to profit from changes in the relative spread rather than movements in the target contract alone.

---

### Avoiding Lookahead Bias

Signals are shifted forward by one period.

A signal calculated using information available at time \(t\) is therefore executed at time \(t+1\).

This prevents the model from using a price movement to both generate a signal and execute a trade at the same historical price.

---

### Execution Assumptions

To reduce unrealistic backtest performance, the model includes simplified trading frictions:

- One-period execution delay
- Volatility-adjusted slippage
- Flat commission on entry and exit
- Equal weighting across the two basket components
- Fixed position sizing
- No compounding

These assumptions make the simulation more conservative, but they do not fully reproduce real prediction-market execution.

In particular, the model does not completely capture:

- Contract-level order-book depth
- Bid-ask spreads
- Partial fills
- Available liquidity
- Market impact
- Platform-specific fees
- Restrictions on taking negative exposure

---

### Data Handling

The market data is processed using the following steps:

- Resampled to hourly intervals
- Missing values forward-filled
- Extreme probability regions near 0% and 100% excluded
- Signals calculated using rolling historical information
- Positions shifted forward before returns are calculated

Extreme probability zones are removed because contracts behave differently near their boundaries. As probabilities approach 0% or 100%, potential price movement becomes asymmetric and the assumptions behind a stable relative spread become less reliable.

---

## Results

After incorporating estimated transaction costs, the strategy remains profitable in several tested configurations.

Performance is not uniform across teams.

| Target | Basket | Net Profit (£) | Return (%) | Win Rate (%) | Profit Factor | Max Drawdown (%) |
|---|---|---:|---:|---:|---:|---:|
| Aston Villa | Man Utd + Chelsea | 894.85 | 89.5 | 55.4 | 1.70 | -13.8 |
| Man Utd | Aston Villa + Chelsea | 2195.83 | 219.6 | 64.5 | 2.87 | -11.6 |
| Chelsea | Aston Villa + Man Utd | 1290.70 | 129.1 | 57.7 | 1.89 | -15.7 |

### Example Equity Curve

![Equity Curve](results/Man_Utd_backtest.png)

The variation in performance suggests that the observed behaviour is not a simple universal relationship.

It may be influenced by asymmetric factors such as:

- Differences in contract liquidity
- Differences in market attention
- Stale or irregular price updates
- Team-specific news
- Differences in the speed of information incorporation
- The suitability of the chosen basket

The results should therefore be interpreted as evidence of possible relative structure rather than proof of a persistent market inefficiency.

---

## Interpretation

A profitable backtest does not necessarily mean that the prediction market contains an exploitable inefficiency.

An extreme relative spread may arise because:

- One contract reacts to information faster than another
- One team receives new fundamental information
- A contract has low liquidity or stale prices
- Missing data creates an artificial divergence
- The basket omits other relevant teams
- The relationship between the contracts has changed

The central challenge is distinguishing a temporary pricing inconsistency from a justified change in relative fundamentals.

---

## Limitations

### Heuristic Basket Construction

The equal-weight basket is simple and interpretable, but it is not theoretically exact.

It assumes that both peer teams are equally informative for the target. In reality, one team may be a much closer substitute or competitor than another.

The analysis also includes only three teams, despite the Top 4 race involving a wider group of competitors.

---

### Assumption of a Stable Relationship

The strategy assumes that the spread has a sufficiently stable recent distribution for rolling mean reversion to be meaningful.

This relationship can break following:

- Injuries
- Managerial changes
- Fixture changes
- Transfers
- Major changes in form
- New information about competing teams

A large deviation may therefore represent a new regime rather than a temporary inconsistency.

---

### End-of-Season Effects

As the season approaches its conclusion, probabilities move closer to 0% or 100%.

At this stage:

- The remaining fixture set becomes more important
- Each result can cause a larger probability update
- Contract behaviour becomes increasingly nonlinear
- Probability boundaries limit potential movement
- Historical spread relationships may become less reliable

---

### Liquidity Variation

Different contracts may have different levels of:

- Trading activity
- Order-book depth
- Bid-ask spread
- Price update frequency
- Market attention

These differences are not fully captured by the simplified execution model.

---

### Missing Data and Forward Filling

Forward filling assumes that the last observed probability remains valid until a new observation appears.

If one contract updates more frequently than another, forward filling can create an apparent divergence that reflects stale data rather than a tradeable pricing inconsistency.

---

### Simplified Execution

The backtest uses estimated slippage and commissions rather than reconstructing each historical order book.

As a result, realised trading performance could be substantially worse than the simulated results.

---

### Backtest and Selection Risk

The results are based on a limited historical sample and a small selection of teams.

Testing multiple parameters or team combinations may also introduce selection bias. A profitable configuration could partly reflect noise rather than a relationship that generalises out of sample.

---

## Next Steps

Potential extensions include:

- Testing lead-lag relationships between contracts
- Using regression-based basket weights
- Weighting contracts by liquidity
- Including more teams in the synthetic basket
- Testing alternative entry and exit thresholds
- Applying walk-forward validation
- Evaluating multiple seasons
- Testing other prediction-market categories
- Reconstructing bid-ask spreads and order-book depth
- Measuring sensitivity to execution delay
- Comparing mean-reversion signals with momentum signals
- Testing whether results remain after stricter out-of-sample validation

---

## Repository Structure

- `notebook.ipynb` — main analysis and backtest
- `results/` — generated figures and equity curves
- `AllTop4Data.csv` — input prediction-market data

---

## Summary

This project applies a simple relative-value framework to related prediction-market contracts.

A synthetic basket is created from peer teams, and the target team's probability is compared with that benchmark. When the relative spread becomes unusually large, the strategy takes a position that benefits if the spread returns toward its recent average.

The results suggest that some relative spreads display potentially mean-reverting behaviour after estimated transaction costs. However, performance varies substantially across teams and may be affected by liquidity, stale pricing, omitted variables and changes in market structure.

The project therefore highlights both:

- The potential for identifying temporary inconsistencies across related prediction-market contracts
- The difficulty of determining whether those inconsistencies represent genuine tradeable inefficiencies
```
