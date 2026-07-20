# Regime-Aware Dynamic Portfolio Allocation using Hidden Markov Models

## Overview

This project implements a **regime-aware portfolio allocation framework** that dynamically adjusts asset weights based on inferred market regimes using a **Gaussian Hidden Markov Model (HMM)**. Unlike traditional static portfolios, the strategy identifies latent market states (Bull, Bear, and Crisis) and optimizes portfolio allocations for each regime.

Each regime routes to a different convex optimization objective:

- **Bull** → Maximum return subject to a target volatility cap (risk-capped growth-seeking)
- **Bear** → Minimum volatility (defensive)
- **Crisis** → Minimum volatility (capital preservation)

The project is evaluated using a walk-forward backtesting methodology with realistic transaction costs, turnover controls, and benchmarked against traditional static allocation strategies via a full performance tear sheet.

---

## Objectives

- Detect latent market regimes using a Hidden Markov Model.
- Dynamically allocate portfolio weights according to the prevailing regime, with regime-specific risk budgets.
- Evaluate robustness through walk-forward backtesting.
- Compare against passive benchmark strategies using a standardized performance tear sheet.

---

## Assets

The strategy allocates among:

- SPY (US Equities)
- TLT (Long-Term Treasury Bonds)
- IEF (Intermediate-Term Treasury Bonds)
- GLD (Gold)
- HYG (High Yield Corporate Bonds)

Market stress information is incorporated through:

- VIX Index

---

# Project Architecture

```
Yahoo Finance
      │
      ▼
Data Collection
      │
      ▼
Feature Engineering
  • SPY Returns
  • TLT Returns
  • GLD Returns
  • HYG Returns
  • VIX
      │
      ▼
Hidden Markov Model
      │
      ▼
Market Regime Detection
(Bull / Bear / Crisis)
      │
      ▼
Regime-Specific Expected Returns
      │
      ▼
   ┌──────────────┴──────────────┐
   ▼                              ▼
Bull:                        Bear / Crisis:
Max Return                   Minimum
subject to                   Volatility
Target Volatility Cap
   └──────────────┬──────────────┘
                   ▼
         Turnover-Aware Weights
         (L1 penalty + no-trade band)
                   │
                   ▼
         Walk-Forward Backtest
                   │
                   ▼
       Performance Tear Sheet
   (Sharpe, Sortino, Max DD, Calmar,
      Turnover, vs. Static Benchmarks)
```

---

# Architecture Decisions

## 1. Hidden Markov Model

A Gaussian Hidden Markov Model was chosen because market regimes are **latent** and cannot be directly observed.

Advantages:

- Learns hidden market states automatically.
- Captures regime persistence.
- Suitable for financial time series.
## issue your own FRED API key for macro proxies data
---

## 2. Feature Selection

The HMM uses daily features:

- SPY Returns
- TLT Returns
- GLD Returns
- HYG Returns
- VIX

These features jointly capture:

- Equity momentum
- Flight-to-quality behavior
- Credit market conditions
- Market stress

---

## 3. Regime-Specific Portfolio Optimization

For every rebalance date:

1. Train HMM on historical data.
2. Predict current market regime.
3. Estimate expected returns using historical observations from the detected regime.
4. Route to the regime-appropriate convex optimization objective:
   - **Bull** — `max_return_with_risk_cap`: maximizes expected return subject to a hard portfolio-variance ceiling (`target_vol`, default 15% annualized). This replaces a plain max-Sharpe objective, so Bull allocations chase return within an explicit risk budget rather than an unconstrained Sharpe-optimal point that could carry more volatility than intended.
   - **Bear** — `min_volatility`, targeting capital preservation with moderate confirmation lag.
   - **Crisis** — `min_volatility`, with a wider regime-confirmation window for stability.
5. Apply turnover controls before finalizing weights (see Section 5).

This prevents a single allocation approach from being used across all market conditions, and lets Bull's risk exposure be tuned directly rather than being an implicit side effect of a Sharpe calculation.

---

## 4. Walk-Forward Validation

Instead of training once on the full dataset, the model is retrained periodically.

Advantages:

- Prevents look-ahead bias.
- Simulates real-world deployment.
- Produces realistic out-of-sample performance.

---

## 5. Transaction Costs & Turnover Controls

A proportional transaction cost model is incorporated during each rebalance, alongside two independent turnover-reduction mechanisms:

- **L1 turnover penalty** (`lambda_turnover`): added directly inside each optimization objective, so the optimizer itself trades off risk/return against transaction cost rather than jumping fully to a new target every period. Regime-specific values are tuned independently for Bull, Bear, and Crisis, including within the Bull risk-capped objective.
- **No-trade band** (`no_trade_band`): after solving, if the L1 distance between the newly optimal weights and the previous weights falls below a threshold, the trade is skipped entirely for that period (no cost incurred), while the underlying target is still recomputed at every rebalance date.

Together these discourage excessive portfolio turnover and better reflect practical implementation costs.

---

# Repository Structure

```
.
├── data/
│   └── downloaded market data
│
├── notebooks/
│   └── strategy development
│
├── src/
│   ├── data.py
│   ├── hmm.py
│   ├── optimizer.py
│   ├── backtest.py
│   └── metrics.py
│
├── figures/
│
├── results/
│
├── requirements.txt
│
└── README.md
```

---

# Installation

Clone the repository.

```bash
git clone https://github.com/your_username/regime-aware-portfolio.git

cd regime-aware-portfolio
```

Create a virtual environment.

```bash
python -m venv venv
```

Activate the environment.

Windows

```bash
venv\Scripts\activate
```

Linux/Mac

```bash
source venv/bin/activate
```

Install dependencies.

```bash
pip install -r requirements.txt
```

---

# Running the Project

Run the notebook

or

```bash
python main.py
```

The script will

- Download historical market data
- Compute features
- Detect regimes
- Optimize allocations (risk-capped max-return for Bull, min-volatility for Bear/Crisis)
- Apply turnover controls
- Run walk-forward backtest
- Generate the performance tear sheet

---

# Testing Instructions

The following validation checks were performed.

## Data Integrity

- Missing values removed
- Trading dates aligned
- Daily return calculations verified

---

## Regime Detection

Verify that

- Three market regimes are detected.
- Regimes are economically interpretable.
- State transitions are stable.

---

## Portfolio Constraints

Check

- Portfolio weights sum to one.
- No negative weights.
- Bull allocations respect the target volatility cap.
- Allocation constraints satisfied.

---

## Walk-Forward Validation

Confirm

- No future information is used.
- Model retrains only using historical observations.
- Performance is evaluated strictly out-of-sample.

---

## Benchmark Comparison

Compare against

- 60/40 Portfolio
- Equal Weight Portfolio

using a standardized performance tear sheet covering

- Annualized Return
- Annualized Volatility
- Sharpe Ratio
- Sortino Ratio
- Maximum Drawdown
- Calmar Ratio
- Turnover (average per rebalance and annualized)

---

# Reproducibility Guide

To reproduce the reported results:

1. Clone the repository.

2. Install all dependencies from `requirements.txt`.

3. Set the random seed.

```python
np.random.seed(42)
```

```python
random.seed(42)
```

4. Use the same historical period.

5. Execute the notebook or `main.py` without modifying model parameters, including the Bull `target_vol` cap and regime-specific `lambda_turnover` / `no_trade_band` settings.

6. Results should closely match the reported performance, subject to minor numerical differences from optimization solvers and data updates.

---

# Technologies Used

- Python
- pandas
- NumPy
- yfinance
- hmmlearn
- CVXPY
- SciPy
- Matplotlib
- Seaborn

---

# Future Improvements

Potential extensions include:

- Regime probability weighted allocation
- Dynamic, regime-conditional covariance estimation (rather than a single fitted covariance reused across regimes)
- Shrinkage of regime-conditional expected returns (e.g., James-Stein / Bayesian shrinkage)
- Recency-weighted (exponentially weighted) regime return estimates
- Black-Litterman optimization
- Alternative regime detection models
- Bayesian optimization of hyperparameters, including the Bull target volatility cap
- Macroeconomic feature integration

---

# Results

The proposed regime-aware strategy demonstrates:

- Dynamic adaptation across market regimes, with an explicit, tunable risk budget for Bull-regime growth-seeking.
- Improved downside performance during Bear markets.
- Competitive risk-adjusted returns compared with static benchmark portfolios, evaluated via Sharpe, Sortino, Max Drawdown, and Calmar ratio.
- Turnover-aware allocation that limits unnecessary trading costs relative to a naive rebalance-every-period approach.

---

# License

This project is intended for academic and research purposes.

---

# Author

**Shreyansh Jaiswal**

Bachelor's in Mathematics and Computing

Indian Institute of Technology (IIT) Guwahati
