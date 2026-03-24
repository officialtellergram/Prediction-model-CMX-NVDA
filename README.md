# NVDA-CMX Predictive Dashboard

An interactive, notebook-based analytics dashboard for comparing stakeholder equity performance, generating short-horizon next-session forecasts, and validating logged predictions over time.

## What it does

The dashboard combines three functions in one workflow:

- comparative six-month stakeholder performance visualization
- per-ticker next-session direction modeling
- internal prediction logging and audit validation

It is built for exploratory research, not live trading execution.

## Core features

### Multi-ticker performance visualization
The dashboard pulls six months of historical data from Yahoo Finance and normalizes each selected ticker to a common baseline. This makes it easy to compare relative performance across names with very different raw price levels.

Main chart features:
- multi-select ticker comparison
- normalized net change view
- interactive Plotly chart
- dark-mode dashboard styling

### Prediction summary and per-ticker model views
For each selected ticker, the notebook trains a lightweight machine learning model and produces a forecast for the next trading session.

Outputs include:
- test accuracy
- latest up probability
- next-session forecast (`UP` / `DOWN`)
- per-ticker prediction charts with signal markers

### Audit panel
The dashboard includes a read-only audit panel that tracks logged prediction performance over time.

The audit panel shows:
- total logged predictions
- validated predictions
- realized hit rate
- per-ticker accuracy leaderboard
- last 10 logged predictions

Normal widget changes do **not** write to the audit log. Logging happens only when the user clicks **Log Current Predictions**.

## Data source

Historical market data is pulled from **Yahoo Finance** using `yfinance`.

The notebook uses:
- adjusted daily close data for charts and model features
- daily volume data for model features
- a six-month rolling lookback for modeling
- a one-year lookback for validating logged predictions

## How the model works

Each ticker is modeled independently.

For a selected ticker, the dashboard downloads roughly six months of daily adjusted market data and engineers a small set of short-horizon technical features.

### Features
The current model uses:

- **1-day return** — most recent daily momentum
- **5-day return** — short-term trend over about one trading week
- **5-day / 10-day moving average ratio** — whether price is above or below recent trend
- **5-day rolling volatility** — how unstable recent returns have been
- **Volume change** — whether trading activity is expanding or contracting

### Target
The model is a binary classifier trained to answer:

> Will the next trading session close higher than the current session?

Target encoding:
- `1` = next session closes higher
- `0` = next session does not close higher

This means the model predicts **direction**, not move size.

### Train/test split
The six-month dataset is split into:
- **80% training**
- **20% testing**

### Model type
The current implementation uses a **Random Forest Classifier** from scikit-learn.

This was chosen because it is:
- simple to implement
- effective on small tabular datasets
- flexible enough to capture nonlinear relationships
- easy to use for exploratory modeling

## How predictions are generated

After training, the model takes the latest available feature row for each selected ticker and estimates the probability that the next trading session will close higher.

The dashboard then converts that probability into a directional forecast:

- probability **>= 50%** → `UP`
- probability **< 50%** → `DOWN`

Interpretation:
- **Latest Up Probability** = model-estimated chance that the next session closes higher
- **Next Session Forecast Prediction** = the final bullish or bearish label

Example:
- `65.4%` → `UP`
- `29.9%` → `DOWN`

## How performance is evaluated

The dashboard surfaces two different performance concepts.

### 1. Test Accuracy
This is the model’s holdout accuracy on the final 20% of the six-month historical sample.

It answers:

> How often did the model correctly classify next-session direction on unseen recent historical data?

### 2. Realized Audit Accuracy
This is based on logged predictions that were saved and later validated against actual next-session market outcomes.

It answers:

> How often were real logged forecasts correct after they were generated?

This is the more operationally meaningful metric over time.

## Logging and audit workflow

### Normal dashboard behavior
Changing selected tickers updates:
- the comparison chart
- the prediction summary
- the per-ticker prediction charts
- the audit panel display

But it does **not** append anything to the prediction log.

### Logging behavior
When the user clicks **Log Current Predictions**, the notebook:

1. generates a prediction snapshot for the currently selected tickers
2. records the timestamp
3. stores the signal date
4. stores predicted direction
5. stores predicted up probability
6. stores model test accuracy at prediction time
7. appends the snapshot to a CSV log

The audit validator later compares each logged prediction against the next available market session to determine whether the forecast was correct.

## Why normalization is used

The main stakeholder chart uses normalized percentage performance rather than raw prices.

That matters because raw prices are not directly comparable across names with different price levels. Normalization answers the more useful question:

> Which names outperformed or underperformed over the same six-month window?

## Design intent

This project is designed as a compact internal research dashboard for:
- stakeholder comparison
- short-horizon directional modeling
- visual forecast inspection
- prediction audit tracking

It is **not** intended to be a production trading system.

## Current limitations

- only about six months of daily data are used per ticker
- the model predicts direction only, not move magnitude
- features are limited to price and volume derivatives
- each ticker is modeled independently
- realized audit history only grows when snapshots are manually logged

## Possible future upgrades

- feature importance views
- confidence thresholds for filtering weak forecasts
- rolling realized accuracy charts
- benchmark comparison versus SPY / QQQ
- exportable audit reports
- scheduled daily snapshot logging
- richer macro and factor-aware inputs
- migration to Streamlit or Dash for broader deployment

## Tech stack

- **Python**
- **Jupyter Notebook / VS Code notebook environment**
- **pandas**
- **yfinance**
- **plotly**
- **ipywidgets**
- **scikit-learn**

## Setup

Install dependencies:

```bash
pip install yfinance pandas plotly ipywidgets scikit-learn
```

Open the notebook and run the main dashboard cell.

## Usage

1. Select one or more tickers from the multi-select widget.
2. Review the normalized stakeholder performance chart.
3. Review the prediction summary and per-ticker forecast charts.
4. Use **Log Current Predictions** to intentionally append a forecast snapshot.
5. Review the audit panel over time to measure realized prediction performance.

## Disclaimer

This dashboard is for exploratory analysis and internal validation only. It should not be treated as investment advice or as a production-grade forecasting system.
