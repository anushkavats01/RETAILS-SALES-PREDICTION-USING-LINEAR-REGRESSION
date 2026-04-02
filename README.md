# 📦 RetailForecast — Inventory Demand Prediction

> Machine learning–powered demand forecasting to minimize stockouts and reduce overstock across retail supply chains.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.4-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat-square)]()

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Model Details](#model-details)
- [Project Structure](#project-structure)
- [Results](#results)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

**RetailForecast** predicts product-level demand across retail locations using historical sales data, seasonal trends, promotional calendars, and external signals (e.g., weather, holidays). The system generates weekly or daily reorder recommendations to help procurement teams act ahead of supply disruptions.

**Business impact targets:**
- Reduce stockout events by **~30%**
- Decrease excess inventory holding costs by **~20%**
- Cut manual forecasting effort from days to minutes

---

## Key Features

- 📊 **Multi-model ensemble** — combines gradient boosting, SARIMA, and exponential smoothing
- 🏪 **Store-SKU level granularity** — per-location predictions, not just aggregate demand
- 📅 **Calendar-aware** — holiday effects, promotional lifts, and seasonal decomposition built-in
- ⚡ **Fast batch inference** — forecasts thousands of SKUs in under 60 seconds
- 📈 **Interactive dashboard** — Streamlit app for exploring predictions and confidence intervals
- 🔄 **Retraining pipeline** — automated weekly model refresh with drift detection

---

## Architecture

```
Raw Sales Data
      │
      ▼
┌─────────────────────┐
│   Data Ingestion    │  ← CSV / SQL / S3
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Feature Engineering│  ← Lag features, rolling stats,
│                     │    calendar flags, price elasticity
└────────┬────────────┘
         │
         ▼
┌────────────────────────────────────────┐
│           Model Layer                  │
│  ┌──────────┐  ┌────────┐  ┌───────┐  │
│  │  XGBoost │  │ SARIMA │  │  ETS  │  │
│  └────┬─────┘  └───┬────┘  └───┬───┘  │
│       └────────────┴───────────┘       │
│              Ensemble Blend            │
└────────────────────┬───────────────────┘
                     │
                     ▼
          ┌──────────────────┐
          │  Forecast Output │  ← 7/14/28-day horizon
          │  + Reorder Signal│
          └──────────────────┘
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- pip or conda

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/retail-forecast.git
cd retail-forecast

# Create a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Environment Setup

Copy the example environment file and configure your data sources:

```bash
cp .env.example .env
```

| Variable | Description | Example |
|---|---|---|
| `DATA_SOURCE` | Input data backend | `csv`, `postgres`, `s3` |
| `DB_CONNECTION_STRING` | Database URI (if applicable) | `postgresql://user:pass@host/db` |
| `S3_BUCKET` | S3 bucket name (if applicable) | `my-retail-data` |
| `FORECAST_HORIZON` | Days to forecast ahead | `28` |
| `RETRAIN_SCHEDULE` | Cron expression for retraining | `0 2 * * 1` |

---

## Usage

### 1. Prepare Your Data

Your input data should follow this schema:

```
data/raw/sales.csv
```

| Column | Type | Description |
|---|---|---|
| `date` | `YYYY-MM-DD` | Transaction date |
| `store_id` | string | Unique store identifier |
| `sku_id` | string | Product SKU |
| `units_sold` | int | Daily units sold |
| `price` | float | Sale price |
| `on_promotion` | bool | Promotional flag |

### 2. Run the Full Pipeline

```bash
python pipeline/run.py --config configs/default.yaml
```

### 3. Generate Forecasts Only

```bash
python forecast.py \
  --input data/processed/features.parquet \
  --horizon 28 \
  --output outputs/forecasts.csv
```

### 4. Launch the Dashboard

```bash
streamlit run app/dashboard.py
```

Then open [http://localhost:8501](http://localhost:8501) in your browser.

### 5. Evaluate Model Performance

```bash
python evaluate.py --forecasts outputs/forecasts.csv --actuals data/actuals.csv
```

---

## Model Details

### Feature Engineering

| Feature Group | Examples |
|---|---|
| Lag features | `units_sold_lag_7`, `units_sold_lag_28` |
| Rolling statistics | `rolling_mean_14`, `rolling_std_7` |
| Calendar | `day_of_week`, `is_holiday`, `days_to_holiday` |
| Promotional | `promo_flag`, `promo_depth`, `promo_duration` |
| Price | `price`, `price_change_pct`, `elasticity_estimate` |

### Ensemble Strategy

The final forecast blends three base models using a learned ridge regression meta-model:

- **XGBoost** — captures non-linear interactions and promotional effects
- **SARIMA** — strong seasonal + trend decomposition for stable SKUs
- **ETS (Exponential Smoothing)** — robust baseline for low-volume, noisy products

Blend weights are retrained weekly and stored in `models/meta/`.

### Evaluation Metrics

| Metric | Description |
|---|---|
| **MAPE** | Mean Absolute Percentage Error |
| **WMAPE** | Weighted MAPE (volume-weighted) |
| **Bias** | Systematic over/under-forecasting |
| **Fill Rate** | % of demand met without stockout |

---

## Project Structure

```
retail-forecast/
├── app/
│   └── dashboard.py          # Streamlit forecast explorer
├── configs/
│   └── default.yaml          # Pipeline configuration
├── data/
│   ├── raw/                  # Source data (gitignored)
│   └── processed/            # Feature-engineered datasets
├── models/
│   ├── base/                 # Serialized base models
│   └── meta/                 # Ensemble blend weights
├── notebooks/
│   ├── 01_eda.ipynb          # Exploratory data analysis
│   ├── 02_feature_eng.ipynb  # Feature development
│   └── 03_model_eval.ipynb   # Performance benchmarking
├── pipeline/
│   ├── ingest.py             # Data ingestion
│   ├── features.py           # Feature engineering
│   ├── train.py              # Model training
│   └── run.py                # End-to-end orchestration
├── tests/
│   ├── test_features.py
│   └── test_models.py
├── evaluate.py               # Standalone evaluation script
├── forecast.py               # Standalone inference script
├── requirements.txt
├── .env.example
└── README.md
```

---

## Results

Results on held-out test set (last 8 weeks of data, 500 SKUs across 12 stores):

| Model | MAPE ↓ | WMAPE ↓ | Bias |
|---|---|---|---|
| Naive baseline | 31.4% | 28.7% | +4.2% |
| SARIMA | 22.1% | 19.8% | -1.1% |
| XGBoost | 17.6% | 15.3% | +0.4% |
| **Ensemble (ours)** | **13.8%** | **11.9%** | **+0.1%** |

*Tested on product categories: Apparel, Electronics, FMCG, Home Goods.*

---

## Roadmap

- [ ] Add deep learning baseline (N-BEATS / Temporal Fusion Transformer)
- [ ] Supplier lead-time integration for reorder point optimization
- [ ] Multi-echelon inventory support (DC → store)
- [ ] REST API endpoint for real-time forecast serving
- [ ] Automated anomaly detection on incoming sales data

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "feat: add your feature"`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

Please ensure all tests pass before submitting:

```bash
pytest tests/ -v
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

<p align="center">Built with ☕ and too much inventory data</p>
