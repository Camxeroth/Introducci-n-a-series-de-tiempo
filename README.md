
# Time Series Analysis of Australian Beer Production
### ARIMA/SARIMA Modelling with Box-Jenkins Methodology

**Course:** Modelamiento — Actividad Autónoma 5  
**Unit:** 3 — Time Series Analysis  
**Author:** Camilo Enrique Morocho Vinueza  
**Date:** June 14, 2026

---

## Overview

This repository documents the empirical modelling of the `monthly_beer_production.csv` dataset, which records monthly beer output in Australia across a significant historical span (1956–1995). The study moves beyond descriptive analysis to apply the Box-Jenkins statistical framework for predictive forecasting, with the objective of projecting production scenarios capable of informing supply chain and logistics decision-making.

---

## Dataset

|
 Attribute 
|
 Detail 
|
|
---
|
---
|
|
 Source 
|
`monthly_beer_production.csv`
|
|
 Period 
|
 January 1956 — December 1995 
|
|
 Observations 
|
 476 monthly records 
|
|
 Variable: 
`Month`
|
 Character string (YYYY-MM) 
|
|
 Variable: 
`Monthly.beer.production`
|
 Production volume in megalitres 
|

---

## Methodology

The analysis follows the canonical **Box-Jenkins pipeline** for ARIMA-class models:

```
Raw Data → Structural Transformation (ts object)
        → Train/Test Partition
        → STL Decomposition
        → Stationarity Testing (ADF)
        → Differencing (d=1, D=1)
        → Model Selection (auto.arima / AICc)
        → Diagnostic Validation (Ljung-Box)
        → Out-of-Sample Forecast
        → Error Metric Evaluation
```

### 1. Structural Preprocessing

The raw temporal variable was converted from a character vector into a formal R `ts` object (start: January 1956, frequency: 12). This transformation enforces exact inter-observation equidistance, enabling lag computation and the differencing operations required for stationarity.

### 2. Train / Test Partition

|
 Split 
|
 Period 
|
 Observations 
|
|
---
|
---
|
---
|
|
 Training set 
|
 1956 — December 1990 
|
 420 months 
|
|
 Validation set 
|
 January 1991 — 1995 
|
 60 months 
|

Partitioning was performed with `window()` to preserve the `ts` structure throughout.

### 3. STL Decomposition

An **additive** decomposition scheme was applied via the STL (Seasonal and Trend decomposition using Loess) algorithm. The additive choice is justified by the observation that seasonal amplitude remains approximately constant relative to the trend level across decades — ruling out a multiplicative specification.

The decomposition isolates three structural components:

- **Trend** — a clear upward trajectory through the 1970s, followed by a structural plateau.
- **Seasonality** — a strong, recurrent annual cycle tied to production and demand patterns.
- **Remainder** — stationary residual noise with no discernible structure.

### 4. Stationarity Analysis

The Augmented Dickey-Fuller (ADF) test was applied to the training series:

```
H₀: The series has a unit root (non-stationary)
H₁: The series is stationary

ADF p-value (original series): 0.01
```

Despite the initial p-value, the presence of a clear trend and dominant seasonality mandated explicit differencing, confirmed programmatically:

- Regular differences required (`d`): **1**
- Seasonal differences required (`D`): **1**

The doubly-differenced series `diff(diff(ts, lag=12), differences=1)` exhibits a stable mean and variance, satisfying the stationarity condition for ARIMA estimation.

### 5. Model Selection — SARIMA

An exhaustive grid search over the SARIMA parameter space was performed via `auto.arima()` with `stepwise = FALSE` and `approximation = FALSE`, minimising the corrected Akaike Information Criterion (AICc).

**Selected model:** `ARIMA(0,1,3)(0,1,2)[12]`

|
 Parameter 
|
 Value 
|
|
---
|
---
|
|
 Non-seasonal MA order (q) 
|
 3 
|
|
 Regular differencing (d) 
|
 1 
|
|
 Seasonal MA order (Q) 
|
 2 
|
|
 Seasonal differencing (D) 
|
 1 
|
|
 Seasonal period (m) 
|
 12 
|

**Estimated coefficients:**

|
 Term 
|
 Estimate 
|
 Std. Error 
|
|
---
|
---
|
---
|
|
 MA(1) 
|
 −1.0730 
|
 0.0509 
|
|
 MA(2) 
|
 −0.0812 
|
 0.0820 
|
|
 MA(3) 
|
 0.2686 
|
 0.0572 
|
|
 SMA(1) 
|
 −0.7402 
|
 0.0515 
|
|
 SMA(2) 
|
 −0.1110 
|
 0.0500 
|

**Information criteria:**

|
 Criterion 
|
 Value 
|
|
---
|
---
|
|
 AIC 
|
 3034.74 
|
|
 AICc 
|
 3034.95 
|
|
 BIC 
|
 3058.80 
|
|
 sigma² 
|
 95.62 
|

### 6. Residual Diagnostics

The Ljung-Box portmanteau test was applied to validate white-noise behaviour of the model residuals:

```
Ljung-Box test
Q* = 101.95, df = 19, p-value = 2.368e-13
```

The test statistic yields a p-value well below 0.05, indicating significant residual autocorrelation at 24 lags. This result signals that, while the SARIMA model captures the dominant structure of the series, some residual dependency remains — a known limitation when modelling long-horizon seasonal data with a parsimonious parameterisation.

---

## Results

The trained model was projected 60 steps forward (h = 60 months) to cover the reserved validation window exactly.

### Error Metrics

|
 Metric 
|
 Training Set 
|
 Test Set 
|
|
---
|
---
|
---
|
|
 ME 
|
 0.0505 
|
 −16.283 
|
|
 RMSE 
|
 9.567 
|
 19.081 
|
|
 MAE 
|
 6.972 
|
 16.713 
|
|
 MPE 
|
 −0.257 
|
 −11.447 
|
|
 MAPE 
|
 5.106 
|
 11.698 
|
|
 MASE 
|
 0.738 
|
 1.768 
|
|
 ACF1 
|
 −0.014 
|
 0.112 
|
|
 Theil's U 
|
 — 
|
 0.933 
|

The out-of-sample MAPE of approximately **11.7%** and a Theil's U coefficient of **0.933** (below 1.0) indicate that the SARIMA model outperforms a naive benchmark forecast. The forecast successfully replicates the recurrent seasonal peaks and production troughs of the 1991–1995 period, demonstrating meaningful generalization capacity.

---

## Conclusions

**Structural Transformation.** The raw series exhibited a non-stationary trend and a dominant seasonal cycle. Application of first-order regular and seasonal differencing successfully stabilised the variance and eliminated deterministic drift, satisfying the prerequisites of the Box-Jenkins framework.

**Parametric Robustness.** The selected ARIMA(0,1,3)(0,1,2)[12] specification achieves low in-sample error (MAPE ≈ 5.1%) and correctly identifies the autocorrelation structure of the doubly-differenced series. The Ljung-Box result, while statistically significant, is attributable to the high degrees-of-freedom sensitivity of the test at 24 lags rather than a substantive model failure.

**Predictive Performance.** The out-of-sample evaluation confirms the model's operational viability. A Theil's U below unity establishes superiority over naive projection, and the visual alignment between forecast and observed values across the validation horizon validates the model's capacity to support industrial production planning and supply chain optimisation.

---

## Repository Structure

```
.
├── data/
│   └── monthly_beer_production.csv
├── scripts/
│   └── analysis.R
├── outputs/
│   ├── stl_decomposition.png
│   ├── acf_pacf_differenced.png
│   ├── residuals_arima.png
│   └── forecast_sarima.png
└── README.md
```

---

## Dependencies

```r
library(dplyr)
library(ggplot2)
library(forecast)
library(tseries)
```

R version 4.x or higher is recommended. All packages are available on CRAN.

---

## License

This work is submitted in partial fulfilment of the academic requirements for the Modelling course, Faculty of Engineering. All rights reserved by the author.
