# Poloko-project-3
# 🦠 COVID-19 Global Data Analysis & Forecasting

A comprehensive end-to-end data analysis and time series forecasting project built on the Johns Hopkins University CSSE COVID-19 dataset. This project covers data ingestion, exploratory analysis, visualisation, stationarity testing, and ARIMA-based forecasting — producing actionable insights on the global trajectory of the pandemic.


## 📦 Dependencies

```bash
pip install pandas numpy matplotlib seaborn statsmodels pmdarima
```

---

## 🗂️ Dataset

Data is sourced directly from the **Johns Hopkins University CSSE GitHub repository** via raw CSV URLs, covering three global time series:

| File | Description |
|------|-------------|
| `time_series_covid19_confirmed_global.csv` | Cumulative confirmed cases by country |
| `time_series_covid19_deaths_global.csv` | Cumulative deaths by country |
| `time_series_covid19_recovered_global.csv` | Cumulative recoveries by country |

The `Province/State` column is dropped from all three datasets to consolidate records at the country level, and a preliminary EDA is performed — inspecting shape, column structure, data types, and missing values — before any transformations are applied.

---

## 📊 Step 1 — Data Ingestion & Cleaning

The pipeline begins by defining a base URL pointing to the JHU CSSE GitHub repository and loading all three datasets using `pd.read_csv()`. The `Province/State` column is removed using `axis=1` to simplify the data to country-level granularity. A basic EDA is then conducted using `.shape`, `.info`, `.columns`, and `.isnull().sum()` to assess data quality, confirm column structure, and identify any missing values before further processing.

---

## 📈 Step 2 — Data Visualisation

This section progressively builds from global trends down to granular country-level insights using `seaborn` and `matplotlib`.

### Global Trend
The dataset is aggregated by date using `groupby()` and a cumulative line chart is rendered, providing a high-level view of the pandemic's overall trajectory.

### Top 10 Country Comparisons
The dataset is filtered to the most recent date and the top 10 countries by confirmed cases, deaths, and recoveries are extracted using `sort_values()` and `head(10)`, visualised as horizontal bar charts for clean cross-country comparison.

### Country Progression Over Time
The top 5 countries by confirmed cases and deaths are plotted as time series line charts with `hue='Country'`, revealing how differently the pandemic evolved across regions.

### Growth Rate Analysis
A custom `calc_slope()` function computes the average daily growth rate of new confirmed cases per country using `np.polyfit()`. The top 15 countries by this slope are visualised in a bar chart, identifying nations with the steepest accelerations in case growth.

### Peak 30-Day Fatality Window
A rolling 30-day window analysis is performed across the top 10 countries by total confirmed cases, summing new deaths within each window to isolate each country's single deadliest 30-day period — plotted in firebrick red for stark visual impact.

---

## 📉 Step 3 — Rolling Mean & Standard Deviation

A 14-day rolling mean and standard deviation are computed on the daily new confirmed cases for a selected country (default: `US`). The raw series is first cleaned using `.clip(lower=0)` and `.fillna(0)` to remove negative corrections and fill gaps. All three series — raw daily cases, the 14-day mean, and the 14-day standard deviation — are plotted on a single chart, cutting through short-term noise to reveal the broader wave patterns of the pandemic.

> The 14-day window aligns with the widely used epidemiological reporting standard adopted globally during the pandemic.

---

## 🧪 Step 4 — Stationarity Testing (Augmented Dickey-Fuller)

Before fitting any forecasting model, the time series is formally tested for **stationarity** using the **Augmented Dickey-Fuller (ADF) test** — a critical prerequisite for ARIMA modelling.

A reusable `adf_test()` function is defined to wrap `statsmodels`' `adfuller()` method, reporting the ADF statistic, p-value, and a plain-English interpretation. The test is applied twice:

- **Raw series** — likely non-stationary due to visible trends and waves.
- **First-differenced series** — computed using `.diff().dropna()` to remove trends and stabilise the mean, typically yielding a stationary result suitable for forecasting.

> Forecasting models trained on non-stationary data tend to produce unreliable predictions — making this step an essential part of rigorous time series analysis.

---

## 🔍 Step 5 — ARIMA Order Selection (ACF & PACF)

The **Autocorrelation Function (ACF)** and **Partial Autocorrelation Function (PACF)** plots are generated on the differenced series across 40 lags to inform the selection of ARIMA order parameters `(p, d, q)`.

| Plot | Purpose |
|------|---------|
| **ACF** | Identifies the moving average order `q` — the lag at which autocorrelation cuts off |
| **PACF** | Identifies the autoregressive order `p` — the lag at which partial autocorrelation cuts off |

Both plots are applied to the stationary differenced series, as ACF and PACF diagnostics are only meaningful on stationary data.

---

## ✂️ Step 6 — Train/Test Split

The smoothed series (14-day rolling mean) is split chronologically at the **80th percentile** of its length — 80% for training and 20% for testing. The split is performed sequentially to preserve the temporal order of observations and prevent data leakage from future values into the training period. Both sets are visualised on a single chart to confirm the integrity of the split before model fitting.

---

## 🤖 Step 7 — ARIMA Model Fitting & Forecasting

The ARIMA model is fitted with order `(p=2, d=1, q=2)` — derived from the ACF and PACF diagnostics — on the smoothed training set. A full statistical summary is printed via `model_fit.summary()`. Forecasts are generated for the length of the test set using `get_forecast()`, with predicted means and confidence intervals extracted and plotted alongside the actual test values.

### Evaluation Metrics

| Metric | Description |
|--------|-------------|
| **RMSE** | Root Mean Squared Error — average forecast error in original units |
| **MAPE** | Mean Absolute Percentage Error — scale-independent accuracy measure |

The average confidence interval bounds are also reported, quantifying the range of uncertainty around each forecast.

> The `(p=2, d=1, q=2)` order is a principled starting point. For more rigorous optimisation, consider using `auto_arima` from the `pmdarima` library.

---

## 💡 Step 8 — Key Insights Summary

A final summary block consolidates the most significant findings from the entire analysis:

- **Global Totals** — Cumulative confirmed cases, deaths, and recoveries as of the latest date in the dataset.
- **Peak Single Day** — The date and count of the highest global new case total ever recorded.
- **Highest Burden Countries** — The nations with the most confirmed cases and the most deaths respectively.
- **Fastest Growing Region** — The country with the steepest average daily case growth slope.
- **ARIMA Model Performance** — MAPE and RMSE for the selected country's forecast.

> This summary block is modular — swapping the `COUNTRY` variable or updating the dataset will automatically refresh all printed insights without any code changes.

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/your-username/covid19-analysis.git
cd covid19-analysis

# Install dependencies
pip install -r requirements.txt

# Launch the notebook
jupyter notebook notebooks/covid19_analysis.ipynb
```

---

## ⚠️ Notes

- The Johns Hopkins CSSE dataset was officially discontinued in March 2023. All analysis reflects data up to that point.
- Negative values in `New_Confirmed` and `New_Deaths` columns result from retrospective data corrections by reporting authorities and are clipped to zero before modelling.
- ARIMA order parameters may need tuning depending on the selected country's data characteristics.

---
## 🙏 Acknowledgements

- **Johns Hopkins University CSSE** — for maintaining and publishing the global COVID-19 dataset throughout the pandemic.
- **statsmodels** — for the ARIMA and ADF test implementations.
- **seaborn & matplotlib** — for the visualisation layer.
