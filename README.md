# 📊 US & Indian Stock Performance Analysis & Forecasting with CAPM

<div align="center">

![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![yFinance](https://img.shields.io/badge/yFinance-Live%20Data-720E9E?style=for-the-badge&logo=yahoo&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Analysis-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-Interactive%20Charts-3F4F75?style=for-the-badge&logo=plotly&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-Computation-013243?style=for-the-badge&logo=numpy&logoColor=white)

<br/>

> **A dual-market financial analytics notebook** that fetches live stock data, computes CAPM-based risk metrics (Alpha & Beta), forecasts stock valuations, and renders a professional interactive 4-panel dashboard — covering both the **US market (S&P 500)** and the **Indian market (Nifty 50).**

<br/>

[📐 CAPM Theory](#-capm-theory-explained-in-depth) • [⚙️ Setup](#%EF%B8%8F-installation--setup) • [🔬 Notebook Walkthrough](#-notebook-walkthrough-cell-by-cell) • [📊 Dashboard](#-interactive-dashboard-preview) • [📈 Forecast Logic](#-forecast-logic-buy--sell-signal)

</div>

---

## 🗺️ Project at a Glance

| Feature | Details |
|--------|---------|
| 🌍 **Markets Covered** | US (S&P 500) + India (Nifty 50) |
| 📦 **US Stocks** | `AAPL`, `MSFT`, `AMZN`, `GOOG`, `TSLA`, `NVDA` |
| 📦 **Indian Stocks** | `RELIANCE.NS`, `TCS.NS`, `HDFCBANK.NS`, `INFY.NS`, `BHARTIARTL.NS`, `ITC.NS` |
| 📅 **Time Period** | Last 5 years (dynamic, auto-calculated) |
| 📐 **Model Used** | Capital Asset Pricing Model (CAPM) |
| 📉 **Risk Metrics** | Beta, Alpha, Expected CAPM Return, Actual Return |
| 📈 **Output** | Buy / Sell Forecast + Interactive Plotly Dashboard |
| 🧪 **Environment** | Jupyter Notebook / Google Colab |

---

## 📐 CAPM Theory — Explained In Depth

> CAPM is the backbone of this entire project. Understanding it fully is key to interpreting every result.

### 🧮 The Core Formula

```
E(Ri) = Rf + βi × (Rm − Rf)
```

| Symbol | Name | Meaning |
|--------|------|---------|
| `E(Ri)` | **Expected Return** | The "fair" return the market expects from stock `i` given its risk level |
| `Rf` | **Risk-Free Rate** | Return on a zero-risk investment (e.g., 10-Year US Treasury = 4.5%, India = 7.0%) |
| `βi` | **Beta** | How sensitive the stock is to overall market movements |
| `Rm` | **Market Return** | Historical average return of the benchmark index (S&P 500 / Nifty 50) |
| `Rm − Rf` | **Market Risk Premium** | Extra return investors demand for taking on market risk over a risk-free asset |

---

### 📏 Beta (β) — Measuring Market Sensitivity

**Beta** tells you how much a stock moves when the market moves by 1%.

```
β = Cov(Ri, Rm) / Var(Rm)
```

| Beta Value | Interpretation | Example |
|-----------|----------------|---------|
| `β = 0` | No correlation with market | Cash, Gold (sometimes) |
| `β = 1.0` | Moves exactly with the market | Index funds |
| `β < 1.0` | Less volatile than market | Defensive stocks (ITC, HDFC Bank) |
| `β > 1.0` | More volatile than market | Tech/growth stocks (TSLA, NVDA) |
| `β < 0` | Moves opposite to market | Inverse ETFs |

> **Example:** If `TSLA` has `β = 1.87`, when the S&P 500 rises 10%, TSLA is expected to rise ~18.7%. But when the market falls 10%, TSLA could fall ~18.7% too — higher reward, higher risk.

**How it's calculated in this notebook:**
```python
market_variance = market_series.var()

for stock_ticker in stock_returns.columns:
    covariance = stock_returns[stock_ticker].cov(market_series)
    beta = covariance / market_variance
    betas[stock_ticker] = beta
```

---

### 💡 Alpha (α) — Measuring Excess Return

**Alpha** tells you whether a stock *actually outperformed* what CAPM predicted it should earn, given its risk.

```
α = Actual Return − CAPM Expected Return
```

| Alpha Value | Interpretation |
|-------------|----------------|
| `α > 0` | Stock **outperformed** — generated more return than its risk justifies ✅ |
| `α = 0` | Stock performed exactly as CAPM predicted — fairly priced |
| `α < 0` | Stock **underperformed** — generated less return than its risk level demands ❌ |

> **Example:** If CAPM says AAPL should return 14% (based on its Beta), but it actually returned 18%, then `α = +4%` — Apple created extra value beyond what its risk level required.

**How it's calculated in this notebook:**
```python
average_daily_risk_free_rate = risk_free_rate / 252  # Convert annual → daily

for stock_ticker, beta in betas.items():
    # CAPM expected daily return
    expected_return = average_daily_risk_free_rate + beta * (average_market_return - average_daily_risk_free_rate)

    # Alpha = Actual - Expected
    alpha = average_stock_returns[stock_ticker] - expected_return
    alpha_values[stock_ticker] = alpha
```

---

### 📈 Security Market Line (SML)

The **SML** is the graphical representation of the CAPM formula — a straight line on a Beta vs. Return chart.

```
SML: y = Rf + β × (Rm - Rf)
```

- Every point **above** the SML → positive Alpha → **Undervalued / Buy signal** 🟢
- Every point **below** the SML → negative Alpha → **Overvalued / Sell signal** 🔴
- Every point **on** the SML → fairly priced

> This project plots every stock as a dot on this chart, making valuation immediately visual.

---

### 🏦 Risk-Free Rate Used

| Market | Benchmark Index | Risk-Free Rate Used |
|--------|----------------|---------------------|
| 🇺🇸 US Market | S&P 500 (`^GSPC`) | **4.5%** — 10-Year US Treasury Bond yield |
| 🇮🇳 Indian Market | Nifty 50 (`^NSEI`) | **7.0%** — 10-Year Indian Govt Bond yield |

---

## 🔬 Notebook Walkthrough — Cell by Cell

### 📦 Cell 0 — Install Dependencies
```python
!pip install yfinance
```
Installs `yfinance` to enable live market data fetching from Yahoo Finance inside Colab.

---

### 📚 Cell 1 — Import Libraries
```python
import yfinance as yf
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

plt.style.use('seaborn-v0_8-darkgrid')
```

| Library | Role |
|---------|------|
| `yfinance` | Pulls live/historical stock prices |
| `pandas` | Data manipulation & time-series handling |
| `numpy` | Math operations — variance, covariance, annualization |
| `matplotlib` | Static chart rendering with seaborn darkgrid style |

---

### ⚙️ Cell 2 — Configuration: Tickers, Dates & Risk-Free Rate

```python
tickers = ['AAPL', 'MSFT', 'AMZN', 'GOOG', 'TSLA', 'NVDA']
market_index = '^GSPC'  # S&P 500

end_date   = pd.to_datetime('today').strftime('%Y-%m-%d')
start_date = (pd.to_datetime('today') - pd.DateOffset(years=5)).strftime('%Y-%m-%d')

risk_free_rate = 0.045  # 4.5% annual
```

**What each variable means:**

| Variable | Purpose |
|----------|---------|
| `tickers` | 6 US mega-cap stocks under analysis |
| `market_index` | S&P 500 benchmark — used to compute Beta |
| `start_date / end_date` | Auto-generated 5-year window, always current |
| `risk_free_rate` | CAPM anchor — the "free" baseline return any investor can get |

---

### 📥 Cell 3 — Download Stock & Market Data

```python
all_tickers = tickers + [market_index]
data = yf.download(all_tickers, start=start_date, end=end_date)
data.dropna(inplace=True)
adjusted_close_prices = data['Close']
```

- Downloads **5 years of daily closing prices** for all 6 stocks + S&P 500 in one API call
- Drops rows with missing values to ensure data quality
- Extracts the `Close` price column from yfinance's multi-index DataFrame

---

### 📉 Cell 4 — Calculate Daily Returns

```python
returns = data.pct_change()
returns.dropna(inplace=True)

market_series = returns['Close'][market_index]
stock_returns  = returns['Close'].drop(columns=[market_index])
```

- `pct_change()` converts absolute prices → **daily percentage returns**
- Formula: `(Price_today − Price_yesterday) / Price_yesterday`
- Returns are the foundation for every subsequent calculation
- Market and stock returns are split into separate Series for clean computation

---

### 🔢 Cell 5 — Calculate Beta for Every Stock

```python
market_variance = market_series.var()

for stock_ticker in stock_returns.columns:
    covariance = stock_returns[stock_ticker].cov(market_series)
    beta = covariance / market_variance
    betas[stock_ticker] = beta
```

**Step-by-step logic:**
1. Compute the **variance** of market daily returns — how much S&P 500 fluctuates day-to-day
2. For each stock, compute its **covariance** with the market — how it moves together with S&P 500
3. **Beta = Covariance ÷ Market Variance** — normalizes the joint movement into a single risk number
4. Store each Beta in a dictionary for later use

---

### 🔠 Cell 6 — Calculate Alpha for Every Stock

```python
average_daily_risk_free_rate = risk_free_rate / 252  # Annual → Daily

for stock_ticker, beta in betas.items():
    expected_return = average_daily_risk_free_rate + beta * (average_market_return - average_daily_risk_free_rate)
    alpha = average_stock_returns[stock_ticker] - expected_return
    alpha_values[stock_ticker] = alpha
```

**Step-by-step logic:**
1. Convert annual risk-free rate to daily: `4.5% ÷ 252 trading days`
2. Apply the **CAPM formula at daily granularity** to get each stock's expected daily return
3. `Alpha = Actual average daily return − CAPM expected daily return`
4. Positive Alpha = stock delivered more than what its risk demanded

---

### 🔮 Cell 7 — Forecasting: Buy or Sell?

```python
annualized_market_return = market_series.mean() * 252
market_risk_premium = annualized_market_return - risk_free_rate

for stock, beta in betas.items():
    actual_return = stock_returns[stock].mean() * 252   # Annualized actual return
    capm_return   = risk_free_rate + beta * market_risk_premium  # Fair CAPM return

    if actual_return > capm_return:
        forecast = 'Buy / Bullish (Undervalued)'
    else:
        forecast = 'Sell / Bearish (Overvalued)'
```

**The Logic:**
- Multiply daily mean return by 252 to get the **annualized return**
- Calculate the **CAPM fair return** — what the stock *should* have returned for its risk level
- If actual beats CAPM → stock generated alpha → signal to **Buy 🟢**
- If actual lags CAPM → stock underperformed its risk → signal to **Sell 🔴**

---

### 🗃️ Cell 8 — Summary Table Display

```python
display(summary_df_display[['Stock', 'Beta', 'Actual_Return', 'CAPM_Return', 'Forecast']])
```

Produces a clean formatted table (example output):

| Stock | Beta | Actual Return | CAPM Return | Forecast |
|-------|------|---------------|-------------|----------|
| AAPL | 1.21 | 18.4% | 14.2% | 🟢 Buy / Bullish (Undervalued) |
| MSFT | 1.08 | 20.1% | 13.0% | 🟢 Buy / Bullish (Undervalued) |
| TSLA | 1.87 | 12.1% | 21.3% | 🔴 Sell / Bearish (Overvalued) |
| NVDA | 1.95 | 68.2% | 22.1% | 🟢 Buy / Bullish (Undervalued) |
| AMZN | 1.14 | 15.6% | 13.9% | 🟢 Buy / Bullish (Undervalued) |
| GOOG | 1.10 | 14.0% | 13.5% | 🟢 Buy / Bullish (Undervalued) |

> *(Actual values depend on the date the notebook is run — data is always fetched live)*

---

### 📊 Cell 9 — Matplotlib SML Chart (US Market)

```python
beta_range = np.linspace(min_beta - 0.1, max_beta + 0.1, 100)
sml_line   = risk_free_rate + beta_range * market_risk_premium

plt.plot(beta_range, sml_line, color='red', linestyle='--', label='Security Market Line')
for i, row in summary_df.iterrows():
    color = 'green' if 'Buy' in row['Forecast'] else 'maroon'
    plt.scatter(row['Beta'], row['Actual_Return'], color=color, ...)
    plt.text(row['Beta'], row['Actual_Return'], row['Stock'])
```

**Chart elements:**
| Element | Description |
|---------|-------------|
| 🔴 Red dashed line | Security Market Line — represents "fair value" |
| 🟢 Green dots | Stocks above SML — Undervalued, Buy signal |
| 🔴 Maroon dots | Stocks below SML — Overvalued, Sell signal |
| 🏷️ Labels | Each dot labeled with the stock ticker |

---


## 📈 Forecast Logic: Buy / Sell Signal

```
Return
  |
  |    ● NVDA (Above SML → BUY 🟢)
  |       ● AAPL (Above SML → BUY 🟢)
  |──────────────── SML (Fair Value Line) ───────────────
  |  ● TSLA (Below SML → SELL 🔴)
  |
  └─────────────────────────────────────────────→  Beta (Risk)
```

| Condition | Meaning | Signal |
|-----------|---------|--------|
| `Actual Return > CAPM Return` | Earns more than risk level demands | 🟢 **BUY / Bullish (Undervalued)** |
| `Actual Return < CAPM Return` | Earns less than risk level demands | 🔴 **SELL / Bearish (Overvalued)** |

> ⚠️ **Disclaimer:** This analysis is based purely on historical data and CAPM theory. Past performance does not guarantee future results. Do not use this as sole financial advice.

---

## ⚙️ Installation & Setup

### 1. Clone the Repository
```bash
git clone https://github.com/pushkarsawarkar22-afk/Portfolio-Stocks-Performance-Analysis-Forecasting-with-CAPM.git
cd Portfolio-Stocks-Performance-Analysis-Forecasting-with-CAPM
```

### 2. Install Dependencies
```bash
pip install yfinance pandas numpy matplotlib plotly
```

### 3. Run on Google Colab *(Recommended)*
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

Upload the `.ipynb` file → Runtime → Run All. The first cell installs `yfinance` automatically.

### 4. Run Locally with Jupyter
```bash
jupyter notebook "US_Stock_Performance_Analysis___Forecasting__CAPM_.ipynb"
```

---

## 📦 Libraries Used

| Library | Purpose |
|---------|---------|
| `yfinance` | Fetch live/historical stock & index data from Yahoo Finance |
| `pandas` | Time-series data manipulation, DataFrame operations |
| `numpy` | Variance, covariance, annualization math |
| `matplotlib` | Static SML scatter plot with seaborn darkgrid style |
| `plotly` | Interactive 4-panel analytical dashboard |

---

## 🗂️ Notebook Structure

```
📓 US_Stock_Performance_Analysis___Forecasting__CAPM_.ipynb
│
├── Cell 0   →  pip install yfinance
├── Cell 1   →  Import libraries (yfinance, pandas, numpy, matplotlib)
├── Cell 2   →  Config: US tickers, 5-year dates, risk-free rate = 4.5%
├── Cell 3   →  Download price data (yfinance) for stocks + S&P 500
├── Cell 4   →  Calculate daily percentage returns via pct_change()
├── Cell 5   →  Compute Beta → Cov(stock, market) / Var(market)
├── Cell 6   →  Compute Alpha → Actual daily return − CAPM expected return
├── Cell 7   →  Forecast logic → Buy/Sell based on Actual vs CAPM return
├── Cell 8   →  Summary DataFrame with formatted output table
├── Cell 9   →  Matplotlib SML chart (US market — static)
└── Cell 10  →  Full Plotly 4-panel interactive dashboard (Indian market)
```

---

## 🔮 Future Enhancements

- [ ] 📉 ARIMA / LSTM-based price forecasting per stock
- [ ] 📊 Sharpe Ratio & Sortino Ratio metrics
- [ ] 🎛️ Streamlit web app for live portfolio input
- [ ] 🌍 Global market support (LSE, HKEX, ASX)
- [ ] 📁 Portfolio weight optimization (Markowitz Efficient Frontier)
- [ ] 📬 Email/alert when Alpha turns positive for any stock

---

## 👨‍💻 Author

<div align="center">

**Pushkar Sawarkar**

[![GitHub](https://img.shields.io/badge/GitHub-pushkarsawarkar22--afk-181717?style=for-the-badge&logo=github)](https://github.com/pushkarsawarkar22-afk)

*"Data-driven decisions beat gut-driven bets — every time."*

</div>

---

## 📜 License

This project is open-source under the [MIT License](LICENSE).

---

<div align="center">

⭐ **Found this useful? Drop a star on the repo!** ⭐

</div>
