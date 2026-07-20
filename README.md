# Regime-Aware Dynamic Portfolio Allocation using Hidden Markov Models

## Overview

This project implements a **regime-aware portfolio allocation framework** that dynamically adjusts asset weights based on inferred market regimes using a **Gaussian Hidden Markov Model (HMM)**. Unlike traditional static portfolios, the strategy identifies latent market states (Bull, Bear, and Crisis) and optimizes portfolio allocations for each regime using a maximum Sharpe ratio framework.

The project is evaluated using a walk-forward backtesting methodology with realistic transaction costs and benchmarked against traditional allocation strategies.

---

## Objectives

- Detect latent market regimes using a Hidden Markov Model.
- Dynamically allocate portfolio weights according to the prevailing regime.
- Evaluate robustness through walk-forward backtesting.
- Compare against passive benchmark strategies.

---

## Assets

The strategy allocates among:

- SPY (US Equities)
- TLT (Long-Term Treasury Bonds)
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
Maximum Sharpe Optimization
      │
      ▼
Portfolio Weights
      │
      ▼
Walk-Forward Backtest
      │
      ▼
Performance Evaluation
```

---

# Architecture Decisions

## 1. Hidden Markov Model

A Gaussian Hidden Markov Model was chosen because market regimes are **latent** and cannot be directly observed.

Advantages:

- Learns hidden market states automatically.
- Captures regime persistence.
- Suitable for financial time series.

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
4. Compute optimal portfolio using Maximum Sharpe optimization.

This prevents a single allocation from being used across all market conditions.

---

## 4. Walk-Forward Validation

Instead of training once on the full dataset, the model is retrained periodically.

Advantages:

- Prevents look-ahead bias.
- Simulates real-world deployment.
- Produces realistic out-of-sample performance.

---

## 5. Transaction Costs

A proportional transaction cost model is incorporated during each rebalance.

This discourages excessive portfolio turnover and better reflects practical implementation.

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
- Optimize allocations
- Run walk-forward backtest
- Generate performance metrics

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

using

- Annual Return
- Annual Volatility
- Sharpe Ratio
- Sortino Ratio
- Calmar Ratio
- Maximum Drawdown

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

5. Execute the notebook or `main.py` without modifying model parameters.

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
- Dynamic covariance estimation
- Black-Litterman optimization
- Alternative regime detection models
- Bayesian optimization of hyperparameters
- Macroeconomic feature integration

---

# Results

The proposed regime-aware strategy demonstrates:

- Dynamic adaptation across market regimes.
- Improved downside performance during Bear markets.
- Competitive risk-adjusted returns compared with static benchmark portfolios.
- Low portfolio turnover due to transaction cost-aware optimization.

---

# License

This project is intended for academic and research purposes.

---

# Author

**Shreyansh Jaiswal**

Bachelor's in Mathematics and Computing

Indian Institute of Technology (IIT) Guwahati