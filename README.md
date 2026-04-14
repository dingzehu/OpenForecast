# OpenForecast

**End-to-end PM2.5 air-quality forecasting for Baden-Württemberg** — IoT sensor ingestion → anomaly detection → Bidirectional LSTM prediction → interactive geospatial visualisation.

![Python](https://img.shields.io/badge/Python-3.7%2B-blue)
![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-orange)
![scikit-learn](https://img.shields.io/badge/scikit--learn-0.20%2B-f7931e)
![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)

---

## Demo

<div align="center">
  <img src="https://github.com/dingzehu/OpenForecast/blob/main/heatmap/interactive_map.gif" width="650"/>
  <p><em>Animated 24-hour temperature heatmap across Baden-Württemberg sensor stations</em></p>
</div>

---

## Overview

Air quality monitoring in Baden-Württemberg relies on a distributed IoT sensor network ([OpenSenseMap / luftdaten](https://opensensemap.org/)) reporting 14 environmental variables in real time. Raw sensor data is noisy, contains outliers, and arrives asynchronously — making it unsuitable for direct analysis.

This project builds a full data pipeline:

```
InfluxDB (IoT sensor network, 14 variables)
  └─► Spatial filter (Shapely + bundesland.shp)      — keep only BW sensors
        └─► Outlier detection (Z-score / IQR / Isolation Forest)
              └─► Feature scaling (RobustScaler / MinMaxScaler)
                    └─► Bidirectional LSTM + Attention  — multivariate time-series forecasting
                          └─► Geospatial visualisation  — Folium heatmap + Seaborn line charts
```

---

## Tech Stack

| Category | Libraries |
|---|---|
| Deep learning | TensorFlow / Keras — Bidirectional LSTM, Conv1D, Attention |
| Machine learning | scikit-learn — Isolation Forest, RobustScaler, MinMaxScaler |
| Data processing | pandas, NumPy, SciPy |
| Geospatial | Shapely, pyshp, Folium, ipyleaflet |
| Visualisation | Matplotlib, Seaborn |
| Data ingestion | InfluxDB (`DataFrameClient`), python-dotenv |

---

## Modules

### 1. `prediction/` — Multivariate LSTM Forecasting

Two notebooks demonstrating model iteration from a baseline BiLSTM to a Conv1D + Attention architecture:

| Notebook | Architecture | Training samples | Epochs |
|---|---|---|---|
| `LSTM_multivariate_P1_parallel_20200427.ipynb` | BiLSTM (128 units) + Dropout(0.2) | 633 | 50 |
| `LSTM_multivariate_P1_parallel_Attention_LSTM.ipynb` | Conv1D(64) → MaxPool → BiLSTM(64) → Attention | 75 | 50 |

**Results (real, from notebook outputs):**

| Model | Training MSE (start → end) | Validation MSE (start → end) |
|---|---|---|
| Baseline BiLSTM | 1.39 → 0.030 | 0.29 → 0.010 |
| Conv1D + BiLSTM + Attention | — | Test MSE: 1.28 (small-dataset POC) |

The attention notebook is a proof-of-concept on a reduced dataset (75 training samples) and serves as an architectural reference alongside the baseline.

---

### 2. `outlier_detection__correlation/` — Anomaly Detection & Correlation

Three notebooks comparing **Z-score**, **IQR**, and **Isolation Forest** on PM2.5 and meteorological variables:

| Notebook | Variables | Raw samples |
|---|---|---|
| `pm25_humidity_outlier_detection.ipynb` | PM2.5 vs Humidity | 24 |
| `pm25_temperature_outlier_detection.ipynb` | PM2.5 vs Temperature | 24 |
| `temperature_humidity_outlier_detection.ipynb` | Temperature vs Humidity | 36,323 |

**Key findings (Pearson r, after Isolation Forest cleaning):**

| Variable pair | Pearson r | Interpretation |
|---|---|---|
| PM2.5 vs Humidity | **+0.30** | Higher humidity associates with more particulate matter |
| PM2.5 vs Temperature | **−0.16** | Weak inverse relationship |
| Temperature vs Humidity | **−0.46** | Strong inverse — warmer air holds less relative humidity |

Isolation Forest (300 estimators, 10% contamination) detects more anomalous points than statistical methods, retaining cleaner data for downstream correlation analysis.

---

### 3. `heatmap/` — Animated Geospatial Heatmap

`folium_heatmap.ipynb` queries 24 hourly snapshots from InfluxDB, spatially filters to Baden-Württemberg using `bundesland.shp`, and renders an animated `HeatMapWithTime` (Folium). Output: `Heatmap.html` (interactive) and `interactive_map.gif` (preview).

---

### 4. `line_chart/` — Per-Sensor Time Series

`line_chart.ipynb` applies the same InfluxDB → spatial filter → hourly aggregation pipeline and plots individual Seaborn line charts for each sensor station, showing diurnal temperature variation across the sensor network (35 charts).

---

## How to Run

**1. Clone the repo and install dependencies:**

```bash
git clone https://github.com/dingzehu/OpenForecast.git
cd OpenForecast
pip install tensorflow scikit-learn pandas numpy scipy folium ipyleaflet \
            shapely pyshp influxdb python-dotenv branca matplotlib seaborn plotly six
```

**2. Configure InfluxDB credentials:**

```bash
cp prediction/.env.example .env
# Edit .env and fill in your InfluxDB connection details:
# INFLUXDB_HOST, INFLUXDB_PORT, INFLUXDB_USER, INFLUXDB_PASSWORD, INFLUXDB_DB
```

**3. Run notebooks in order:**

| Step | Folder | Purpose |
|---|---|---|
| 1 | `heatmap/` | Explore the sensor network spatially |
| 2 | `outlier_detection__correlation/` | Clean data and compute correlations |
| 3 | `prediction/` | Train and evaluate LSTM models |
| 4 | `line_chart/` | Visualise per-sensor time series |

> **Note:** The InfluxDB instance used in development (`metrics.gwdg.de`) is no longer active. Notebooks display pre-computed outputs and can be adapted to any InfluxDB endpoint.

---

## Dataset

| Field | Detail |
|---|---|
| Source | [OpenSenseMap](https://opensensemap.org/) / luftdaten IoT sensor network |
| Region | Baden-Württemberg, Germany |
| Variables | 14 — including temperature, humidity, PM2.5 (P1), PM10 (P2), pressure |
| Aggregation | Hourly mean per sensor station |
| Storage | InfluxDB time-series database (`sensor` measurement) |
| Training set (LSTM) | 792 samples (baseline) / 75 samples (attention POC) |

---

## License

[GNU General Public License v3.0](LICENSE)
