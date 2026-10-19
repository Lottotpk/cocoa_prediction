# Cocoa Price Forecasting: Time-Series Analysis

Forecasting ICCO cocoa prices through the 2024 price shock, when cocoa roughly doubled in a few months. The project compares exponential smoothing, ARIMA, structural-break-aware models, and a spectral/Fourier model with Ghana rainfall as an exogenous driver.

> Course project for **STA457 Time Series Analysis**, University of Toronto (exchange, Winter 2025). Written in R.

## Data

| Source | Coverage | Notes |
|---|---|---|
| ICCO daily cocoa price (US$/tonne), `Daily Prices_ICCO.csv` | Oct 1994 – Feb 2025, ~7,800 trading days | Duplicates and one corrupt row removed |
| NOAA daily weather, 10 Ghanaian stations (Kumasi, Sunyani, Takoradi, …), `Ghana_data.csv` | 1990 – Nov 2024, ~53,000 rows | Monthly mean precipitation and temperature; missing months imputed with that calendar month's average |

- **Train/test split:** Nov 2024 – Feb 2025 (4 months) is held out. Everything before is training data.
- **Frequency:** models are fit on monthly average prices unless stated otherwise.

## Methods

1. **Exploration and stationarity:** log transform and first differencing. ACF/PACF analysis. The ADF test rejects a unit root for differenced log prices (p < 0.01).
2. **ETS(A,A,A):** additive error, trend and seasonality. Fit on post-January-2024 data.
3. **ARIMA:** order selected by `auto.arima` (exhaustive, no stepwise approximation) on monthly log prices, giving ARIMA(2,1,2). Residual diagnostics: Ljung-Box p = 0.87 (no remaining autocorrelation). Histogram and Q-Q plots show heavy tails.
4. **Structural break:** shock months are flagged where the monthly log return exceeds 0.2 (Feb–Apr 2024). A break test at Feb 2024 rejects parameter stability (p ≈ 9×10⁻⁶). Separate pre- and post-shock models are then fit. The post-shock model is refit on daily data, giving ARIMA(3,1,2), because only ~9 post-shock monthly points exist.
5. **Spectral analysis + exogenous rainfall:** a smoothed periodogram shows a dominant cycle of about 15.6 months. Two Fourier harmonic pairs at that period are added as regressors. Lagged Ghana precipitation is added, with the lag chosen by cross-correlation. Model: regression with ARIMA(2,1,2) errors.

## Results (4-month hold-out, monthly average price)

| Model | Test RMSE (US$/tonne) |
|---|---|
| **ETS(A,A,A), post-shock data** | **1,972** |
| ARIMA(2,1,2) on log prices, full history | 3,464 |
| ARIMA(2,1,2) + Fourier terms + Ghana precipitation | 3,527 |

**Takeaways**
- Models fit on the full history under-forecast badly. The 2024 regime shift makes pre-2024 dynamics a poor guide, which is why the simple post-shock ETS model wins.
- The precipitation coefficient was not statistically significant (estimate −0.005, s.e. 0.009). At monthly frequency, rainfall added little beyond the price's own dynamics.
- Residuals are fat-tailed after the shock. This motivates volatility modelling (e.g. GARCH) as a next step.

## Run

Open `STA457_project.ipynb` in Jupyter with an R kernel, or run the cells in RStudio. Required packages:

```r
install.packages(c("ggplot2", "astsa", "forecast", "dplyr", "lubridate", "tidyr",
                   "tseries", "strucchange", "tsoutliers"))
```

The notebook writes `train.csv` and `test.csv` on first run.

## Tech stack
R · forecast · astsa · tseries · strucchange · ggplot2 · dplyr
