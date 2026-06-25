# Portfolio Optimization & Monte Carlo Simulation

A quantitative portfolio analysis toolkit built in Python that combines **mean-variance optimization** with **Monte Carlo simulation** to model risk and return for Indian equity portfolios.

---

## Features

- **Sharpe-maximizing portfolio optimization** via constrained SLSQP (scipy)
- **Monte Carlo simulation** — 1000 correlated paths over a user-defined time horizon using multivariate normal sampling from the empirical covariance matrix
- **Benchmark comparison** — rolling alpha/beta, tracking error, information ratio, up/down capture vs. any index (default: Nifty 50)
- **Individual asset metrics** — CAGR, annualized volatility, Sharpe, Sortino, max drawdown, VaR (95%)
- **Automatic ticker pruning** — zero-weight assets from optimization are dropped before analysis and Monte Carlo
- **Data quality checks** — flags tickers with suspicious daily returns (>25%) that would poison the covariance matrix

---

## Project Structure

```
Historical Market Data
        │
        ▼
Portfolio Optimization
        │
        ▼
Optimal Asset Allocation
        │
        ├──────────────► Performance Analytics
        │                     │
        │                     ▼
        │             Benchmark Comparison
        │
        ▼
Monte Carlo Simulation
        │
        ▼
Future Portfolio Projections
        │
        ▼
Risk Assessment & Insights

 Classes
    ├── DataLoader                 # yfinance wrapper with date-range fetching
    ├── PerformanceMetrics         # Static methods: CAGR, vol, Sharpe, Sortino, drawdown, VaR
    ├── PortfolioAnalysis          # Weighted portfolio-level return, vol, Sharpe
    ├── PortfolioVisualization     # Cumulative return plots (individual + weighted)
    ├── BenchmarkComparison        # Alpha, beta, R², tracking error, info ratio, capture ratios
    └── MonteCarloSimulation       # Correlated path simulation with percentile fan chart
```

---

## Pipelines

Two end-to-end functions wrap the full workflow:

### `run_portfolio_pipeline()`

Fetches data → validates quality → optimizes weights → runs Monte Carlo.

```python
result, mc_sim = run_portfolio_pipeline(
    portfolio_tokens,
    start_date='2021-06-01',
    end_date='2026-01-01',
    num_simulations=1000,
    time_horizon=252,          # 1 trading year forward
    initial_investment=100000,
    risk_free_rate=0.0685,     # RBI repo rate
    min_weight=0.05,           # Minimum 5% per asset
    max_weight=0.30            # Maximum 30% per asset
)
```

### `run_analysis_pipeline()`

Takes optimized weights → drops zero-weight tickers → runs full diagnostics and benchmark comparison.

```python
active_data, active_weights, active_tickers = run_analysis_pipeline(
    data,
    weights=result["weights"],
    tickers=result["tickers"],
    risk_free_rate=0.0685,
    benchmark_ticker='^NSEI'
)
```

---

## Optimization

Portfolio weights are found by **maximizing the Sharpe ratio** subject to:
- Weights sum to 1
- Each weight bounded between `min_weight` and `max_weight`

$$\max_w \frac{w^\top \mu - r_f}{\sqrt{w^\top \Sigma w}}$$

`mu` and `Sigma` are estimated from historical daily returns, annualized by ×252.


---

## Monte Carlo

Each simulation path:
1. Draws `T` daily return vectors from `N(mu, Sigma)` — preserving cross-asset correlations
2. Computes weighted portfolio daily return
3. Clips to ±20% per day to prevent numerical blowup from outlier draws
4. Compounds into a cumulative value path

Output: percentile fan chart (5th / 25th / 50th / 75th / 95th) + probability of loss.

---

## Sample Output

```
Effective data range: 2024-03-18 to 2025-12-31

============================================================
PORTFOLIO OPTIMIZATION RESULTS
============================================================

Optimal Weights
------------------------------------------------------------
EBBETF0430.NS     25.00%
GOLDBEES.NS       25.00%
BHARTIARTL.NS     16.73%
MON100.NS         12.47%
HDFCBANK.NS        9.10%
MARUTI.NS          7.00%
BAJFINANCE.NS      2.83%
SBIN.NS            1.74%
HINDUNILVR.NS      0.12%
ITC.NS             0.00%
SUNPHARMA.NS       0.00%
ASIANPAINT.NS      0.00%
AXISBANK.NS        0.00%
HCLTECH.NS         0.00%
ICICIBANK.NS       0.00%
INFY.NS            0.00%
JUNIORBEES.NS      0.00%
KOTAKBANK.NS       0.00%
LT.NS              0.00%
MOSMALL250.NS      0.00%
NIFTYBEES.NS       0.00%
RELIANCE.NS        0.00%
TCS.NS             0.00%
TITAN.NS           0.00%
ULTRACEMCO.NS      0.00%
WIPRO.NS           0.00%
^BSESN             0.00%
^NSEI              0.00%

Portfolio Metrics
------------------------------------------------------------
Expected Annual Return : 26.42%
Annual Volatility      : 8.22%
Sharpe Ratio           : 2.380
============================================================

Initializing Monte Carlo Simulation
------------------------------------------------------------
BAJFINANCE.NS  :   2.83%
BHARTIARTL.NS  :  16.73%
EBBETF0430.NS  :  25.00%
GOLDBEES.NS    :  25.00%
HDFCBANK.NS    :   9.10%
HINDUNILVR.NS  :   0.12%
MARUTI.NS      :   7.00%
MON100.NS      :  12.47%
SBIN.NS        :   1.74%
------------------------------------------------------------
Simulations  : 1,000
Time Horizon : 252 trading days
------------------------------------------------------------

==================================================
MONTE CARLO SIMULATION SUMMARY
==================================================
Initial Investment: 10,000
Time Horizon: 252 trading days
Number of Simulations: 1,000
--------------------------------------------------
Final Portfolio Value Statistics:
  Mean:     13,053
  Median:   12,974
  Std Dev:  1,084
  Min:      10,366
  Max:      16,688
--------------------------------------------------
Percentile Projections:
  5th percentile:  11,340
  95th percentile: 15,012
--------------------------------------------------
Probability of Loss: 0.0%
==================================================
```
```

Active tickers after optimization: ['BAJFINANCE.NS', 'BHARTIARTL.NS', 'EBBETF0430.NS', 'GOLDBEES.NS', 'HDFCBANK.NS', 'HINDUNILVR.NS', 'MARUTI.NS', 'MON100.NS', 'SBIN.NS']

********** Individual Ticker Stats **********
Annual Return (%):
 Ticker
BAJFINANCE.NS    45.52
BHARTIARTL.NS    33.72
EBBETF0430.NS     8.54
GOLDBEES.NS      71.81
HDFCBANK.NS      13.33
HINDUNILVR.NS     1.37
MARUTI.NS        55.43
MON100.NS         8.13
SBIN.NS          26.04
dtype: float64
Annual Volatility (%):
 Ticker
BAJFINANCE.NS    26.50
BHARTIARTL.NS    21.79
EBBETF0430.NS     3.10
GOLDBEES.NS      16.29
HDFCBANK.NS      18.15
HINDUNILVR.NS    19.47
MARUTI.NS        21.82
MON100.NS        22.08
SBIN.NS          24.01
dtype: float64
Sharpe Ratio:
 Ticker
BAJFINANCE.NS    1.46
BHARTIARTL.NS    1.23
EBBETF0430.NS    0.54
GOLDBEES.NS      3.99
HDFCBANK.NS      0.36
HINDUNILVR.NS   -0.28
MARUTI.NS        2.23
MON100.NS        0.06
SBIN.NS          0.80
dtype: float64
Max Drawdown (%):
 Ticker
BAJFINANCE.NS   -16.77
BHARTIARTL.NS   -13.89
EBBETF0430.NS    -2.08
GOLDBEES.NS     -10.42
HDFCBANK.NS     -12.93
HINDUNILVR.NS   -27.94
MARUTI.NS       -20.44
MON100.NS       -26.02
SBIN.NS         -23.94
dtype: float64
Sortino Ratio:
 Ticker
BAJFINANCE.NS    2.14
BHARTIARTL.NS    1.87
EBBETF0430.NS    0.71
GOLDBEES.NS      5.73
HDFCBANK.NS      0.50
HINDUNILVR.NS   -0.43
MARUTI.NS        3.90
MON100.NS        0.08
SBIN.NS          0.99
dtype: float64
VaR (95%):
 -0.0177

********** Portfolio Return Stats **********
Portfolio Return (%): 26.42
Portfolio Volatility (%): 8.22
Portfolio Sharpe Ratio: 2.38


============================================================
BENCHMARK COMPARISON REPORT
============================================================
Benchmark: ^NSEI (Nifty 50)
Period: 2024-03-19 to 2025-12-30
------------------------------------------------------------

Annualized Returns:
  Portfolio:  26.52%
  Benchmark:  10.44%
  Difference: +16.08%

Annualized Volatility:
  Portfolio:  8.23%
  Benchmark:  13.07%

Risk Metrics:
  Beta:              0.441
  Alpha (annual):    18.09%
  R-squared:         0.490
  Correlation:       0.700

Performance Metrics:
  Tracking Error:    9.38%
  Information Ratio: 1.714
  Up Capture:        59.5%
  Down Capture:      28.9%
============================================================
```

<img width="895" height="515" alt="image" src="https://github.com/user-attachments/assets/cd0b15a6-3ab7-42d2-b305-203c0a88363f" />
<img width="895" height="515" alt="image" src="https://github.com/user-attachments/assets/c3442192-4637-4692-948c-ae66c3dbb063" />
<img width="895" height="515" alt="image" src="https://github.com/user-attachments/assets/2d936ee1-c8bb-49ae-8858-ea902f76f49c" />
---

## Tech Stack

| Library | Usage |
|---|---|
| `yfinance` | Market data ingestion (NSE/BSE tickers) |
| `numpy` | Linear algebra, covariance, simulation |
| `pandas` | Time-series alignment, resampling |
| `scipy.optimize` | SLSQP constrained optimization |
| `matplotlib` | Visualizations |

---

## Known Limitations

- **Estimation error in `mu`**: Daily mean returns are noisy estimators of expected return. Even 3–5 years of data gives wide confidence intervals. Shrinkage partially corrects this but the optimizer is still sensitive to the backtest window.
- **Normal return assumption**: Multivariate normal sampling underestimates tail risk. Indian small caps and ETFs exhibit fat tails and occasional extreme daily moves.
- **No transaction costs or taxes**: Optimal weights assume frictionless rebalancing. STCG (15%) and brokerage costs are not modeled.

---

## Usage Notes

- Use `start_date = '2021-06-01'` or later if your universe includes `MON100.NS`, `MOSMALL250.NS`, or `EBBETF0430.NS` — these ETFs listed between 2019–2021 and return NaN-heavy columns before that date.
- The `data.dropna()` call after fetching aligns all tickers to a common trading calendar. Rows where any ticker is missing are dropped entirely.
- `^NSEI` requires no `.NS` suffix and is the correct yfinance symbol for the Nifty 50 index.
