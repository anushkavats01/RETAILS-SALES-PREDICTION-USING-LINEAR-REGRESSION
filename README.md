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

Overview
This project builds an end-to-end machine learning pipeline to forecast retail product demand (units sold) from store and product attributes. It covers data exploration, preprocessing, feature engineering, model training, evaluation, and a browser-based prediction UI powered by Gradio.

Features

Exploratory Data Analysis — distribution plots, regional/seasonal breakdowns, and a full correlation heatmap
Feature Engineering — three derived signals: Demand_Supply_Gap, Price_Discount_Ratio, Price_vs_Competitor
Controlled Noise Injection — Gaussian noise on Demand Forecast to produce a realistic R² range (0.60–0.80)
Train / Validation / Test Split — 70 / 15 / 15 with StandardScaler fitted only on training data (no leakage)
5-Fold Cross Validation — fold-by-fold R² scores to confirm generalisation
Gradio Frontend — interactive sliders and dropdowns; returns predicted units sold plus derived insight metrics
Persisted Artefacts — model, scaler, and feature list saved with joblib for reuse


Project Structure
.
├── retail_demand_forecast_v2.ipynb   # Main notebook (EDA → training → Gradio UI)
├── retail_store_inventory.csv        # Source dataset (~73,100 rows)
├── demand_forecast_model.pkl         # Saved Linear Regression model
├── scaler.pkl                        # Saved StandardScaler
├── features.pkl                      # Saved feature list
└── README.md

Requirements
python >= 3.8
pandas
numpy
matplotlib
seaborn
scikit-learn
joblib
gradio
Install all dependencies:
bashpip install pandas numpy matplotlib seaborn scikit-learn joblib gradio

Quick Start

Clone the repo and add the dataset

bash   git clone https://github.com/<your-username>/retail-demand-forecast.git
   cd retail-demand-forecast
   # Place retail_store_inventory.csv in the project root

Run the notebook
Open retail_demand_forecast_v2.ipynb in Jupyter and run all cells top-to-bottom. The final cell launches the Gradio app.
Use the Gradio UI
After the last cell executes, Gradio will print a local URL (e.g. http://127.0.0.1:7860). Open it in your browser, fill in the product/store details, and click Predict Demand.


Dataset
PropertyValueFileretail_store_inventory.csvRows~73,100TargetUnits SoldCategorical featuresCategory, Region, Weather Condition, SeasonalityNumeric featuresInventory Level, Units Ordered, Demand Forecast, Price, Discount, Competitor Pricing, Holiday/PromotionDate-derived featuresMonth, DayOfWeek, Quarter

Pipeline Steps
StepDescription1Import libraries2Load dataset3EDA — distributions, boxplots, correlation heatmap4Data cleaning & preprocessing (date parsing, null imputation, label encoding)5Controlled Gaussian noise on Demand Forecast6Feature engineering (3 derived features)7Train / validation / test split + StandardScaler8Linear Regression training, validation metrics9Test set evaluation + 5-fold cross validation10Result visualisations (actual vs predicted, residuals, coefficients, CV scores)11Save model artefacts with joblib12Gradio interactive frontend

Model Performance
MetricValueAlgorithmLinear RegressionEvaluationMAE, RMSE, R² on held-out test setCross Validation5-fold CV R² (mean ± std)Target R² range0.60 – 0.80

Gradio UI Inputs & Outputs
Inputs

Product Category, Store Region, Season
Inventory Level, Units Ordered, Demand Forecast
Selling Price, Discount (%), Competitor Price
Weather Condition, Holiday/Promotion flag
Month, Day of Week, Quarter

Outputs
OutputDescriptionPredicted Units SoldPrimary demand forecastDemand-Supply GapDemand Forecast − Inventory LevelPrice vs CompetitorWhether price is higher or lower, and by how muchPrice Discount RatioPrice / (Discount + 1)

Acknowledgements
Built as the major project for the Intel Artificial Intelligence programme (VUIP111).
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
