# Portfolio Optimization & Risk Analytics

In this project, I originally built a portfolio optimization system to compare allocation methods. But before trusting the results, I audited the backtest and found that several implementation choices were materially affecting performance. I corrected the engine and quantified those effects through ablation analysis. After that, I compared six portfolio methods under the same walk-forward framework, focusing not on a single winner but on the trade-offs among return, downside risk, turnover, diversification, and transaction costs. Finally, I used the validated framework for illustrative risk-profile allocations and scenario analysis, while explicitly recognizing selection bias and the limits of historical evidence.


> **Report:** [https://github.com/rosenguyen2107/E-commerce-User-Churn-Retention-Modeling/blob/main/customer_retention_analysis_report.pdf](https://github.com/rosenguyen2107/Portfolio-Optimization-Risk-Analytics/blob/main/Report-Portfolio%20Optimization%20%26%20Risk%20Analytics.pdf)
> 
> **Notebook:** https://github.com/rosenguyen2107/Portfolio-Optimization-Risk-Analytics/blob/main/Portfolio%20Optimization%20%26%20Risk%20Analytics.ipynb
---

## What the system does

| | |
|---|---|
| **Universe** | 24 securities - 22 equities across 8 sectors, 1 Treasury ETF (IEF), 1 gold ETF (GLD) |
| **Out-of-sample window** | 1 Feb 2016 - 24 Jul 2026 · 2,635 trading days · 10.5 years |
| **Rebalances** | 126 monthly, on the last trading day of each completed month |
| **Estimation** | 252-day trailing window ending on the rebalance date |
| **Costs** | 10 bps on traded notional, charged at every rebalance including initialisation |
| **Risk-free rate** | FRED DGS3MO, point-in-time - never a full-sample average |
| **Data** | Frozen snapshot, vintage `2026-07-24`, SHA `8ed57bec6f01fa67` |
| **Reproducibility** | Result fingerprint `bf42baf6357816fa`, stable across kernel restarts |
| **Quality assurance** | 39 / 39 automated checks passed · 0 solver failures across 756 solves |

Methods compared: Maximum Sharpe (historical means), Minimum Variance, Risk Parity,
Hierarchical Risk Parity, walk-forward Black-Litterman, and Equal Weight - against SPY,
60/40 and a diversified 60/30/10 benchmark run through the same engine, window and cost
assumptions.

---

## Results

All figures realised, net of costs, over the identical out-of-sample window.

| Strategy | Net CAGR | Volatility | Sharpe | Max Drawdown | Ann. Turnover |
|---|---|---|---|---|---|
| Max Sharpe (Hist) | 28.22% | 15.83% | **1.51** | −20.75% | 1.53 |
| Min Variance | 15.53% | 10.02% | 1.26 | **−18.01%** | 0.50 |
| Equal Weight | 22.33% | 15.70% | 1.22 | −28.53% | **0.32** |
| Black-Litterman | 21.55% | 15.13% | 1.21 | −28.58% | 1.21 |
| Risk Parity | 18.32% | 12.85% | 1.19 | −26.46% | 0.56 |
| HRP | 15.58% | 11.09% | 1.15 | −20.61% | 0.80 |
| SPY *(benchmark)* | 15.52% | 17.76% | 0.77 | −33.72% | 0.05 |
| 60/40 *(benchmark)* | 9.68% | 10.50% | 0.71 | −21.30% | 0.15 |

The highest Sharpe belongs to a strategy selected on the same sample it is measured against, in a universe chosen in 2026 with
hindsight.

### Ranking by Sharpe alone picks the wrong winner

A nine-dimension scorecard (Sharpe, drawdown, turnover, cost drag, concentration, effective
holdings, weight stability, sub-period consistency and crisis-window return) puts
Min Variance first (mean rank 2.12), not Max Sharpe (4.00). Min Variance also produced the
best return through the deepest crisis in the sample: −17.88% against SPY's −33.72% over
19 Feb - 23 Mar 2020.

### One cost, three framings

Max Sharpe paid `$0.1395` of transaction cost per `$1` of starting capital:

- **13.95%** of initial capital 
- **1.00%** of gross terminal wealth
- **0.39 pp** of annual CAGR

All three are the same money. The first framing systematically penalises whichever strategy
grew the most, because the denominator is frozen at the start while costs are paid on a
growing portfolio. This repository reports gross-vs-net CAGR and terminal-wealth drag as the
primary measures.

---

## What the audit found

Five defects, corrected and then measured. The ablation runs one strategy through five nested
implementations, holding universe, estimation window, risk-free series, cost rate and
comparison window constant — so each row differs from the one above it by exactly one fix.

| Variant | CAGR | Sharpe | Δ CAGR |
|---|---|---|---|
| V1 Original-like approximation | 28.19% | 1.48 | — |
| V2 + corrected month-end dates | 29.02% | 1.54 | **+0.84%** |
| V3 + natural weight drift | 28.62% | 1.53 | −0.40% |
| V4 + pre-trade turnover | 28.60% | 1.53 | −0.03% |
| V5 Fully corrected | 28.60% | 1.53 | 0.00% |

Two things this measurement overturned, both of which an earlier draft had asserted without
evidence:

- Correcting the engine raised net CAGR by 0.41 pp and Sharpe by 0.05. The original
  implementation was understating results, not inflating them.
- The largest single step was the rebalance-date fix, not the removal of hidden daily
  rebalancing, which actually reduced CAGR by 0.40%.

The original implementation did not inflate results but distorted them, and the direction
of distortion differed by metric.

---

## Conclusions

Four statements this analysis supports, at the strength the evidence allows.

**1. Diversification beat selection on the measure that matters to a client.** Min Variance
produced the shallowest drawdown among tested active strategies - 18.01% against 33.72% for
SPY, while trading roughly a third as much as the highest-turnover method, and it ranked
first on the balanced scorecard.

**2. The highest Sharpe carries the most estimation risk.** Max Sharpe recorded 1.51, but it
was selected on the same sample it is evaluated on. That number is a description of one
historical path, not a property of the strategy.

**3. Costs are a first-order effect, not a footnote.** Transaction costs reduced performance
most for strategies with high traded notional: 0.39 pp of annual CAGR for the most active
method against 0.08 pp for the least - a five-fold difference driven by turnover, portfolio
value and the assumed rate together.

**4. Backtest construction is itself a source of investment risk.** Correcting the engine
changed net CAGR by 0.41 pp, which is larger than the gap between several competing strategies
in the results table. The direction of the error was not predictable in advance, which is the
argument for measuring rather than assuming.

---

## What this work does not support

| Claim | Why not |
|---|---|
| "The strategy will outperform" | Nothing here is predictive, every figure describes one historical sample |
| "The model guarantees doubling" | Simulated probabilities describe the resampled sample only - 3,980 of 4,000 paths is not all paths |
| "The portfolio is safe" | Lower historical volatility is not safety. Min Variance still lost 18% peak-to-trough |
| "The alpha proves skill" | The intercept is return unexplained by the included factors, measured on the selection sample |
| "Beat the market by X%" | The comparison is not risk-equivalent and the universe is survivorship-biased |
| Any headline Sharpe or CAGR quoted alone | Without the caveat below, the figure is misleading |

### The limitation that dominates everything above

**Survivorship and selection bias:** The 24 tickers were chosen in 2026
and include names that are prominent because they won (NVDA, META, LLY). Removing this
requires point-in-time index membership, which this dataset does not contain. Every return
figure and every projection percentile is optimistic by an amount I cannot quantify here.
