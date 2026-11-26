# Delhi Daily Mean Temperature Forecasting

Forecast Delhi's daily average temperature for the next 365 days using a SARIMAX model with Fourier terms.

- **Notebook:** `DailyDelhiClimateTrainTime_Series_Forecasting_Project.ipynb`
- **Data:** `DailyDelhiClimateTrain.csv` (daily weather in Delhi, 2013–2017)
- **Model:** ARIMA(3,1,3) on log10 temperature, plus 3 pairs of Fourier (sine/cosine) terms for the yearly cycle
- **Result:** 6.83% MAPE and 2.34 °C RMSE on held-out data, versus 9.88% and 3.53 °C for a "same day last year" baseline

---

## Table of Contents

1. [Project Goal](#project-goal)
2. [Dataset](#dataset)
3. [Method](#method)
4. [Results](#results)
5. [Output Files](#output-files)
6. [How to Run](#how-to-run)
7. [Project Structure](#project-structure)
8. [Known Limitations](#known-limitations)
9. [Possible Improvements](#possible-improvements)

---

## Project Goal

Build a model that predicts Delhi's daily mean temperature for the next 365 days, with a 95% confidence range for each day.

Specific steps covered in the notebook:

- Load and clean the daily data
- Split the series into trend, yearly pattern and noise
- Test if the data is stationary
- Model the yearly cycle with Fourier terms
- Pick ARIMA settings automatically with `auto_arima`
- Check the model errors (residuals)
- Test on unseen data and compare with a simple baseline
- Retrain on all data, forecast 365 days, export results

---

## Dataset

| Item | Detail |
|---|---|
| File | `DailyDelhiClimateTrain.csv` |
| Period | 01-01-2013 to 01-01-2017 |
| Rows | 1,462 (one per day, no gaps) |
| Date format | `dd-mm-yyyy` |
| Source | Not stated in the notebook. Add the link here. |

### Columns

| Column | Meaning | Used in model? |
|---|---|---|
| `date` | Calendar day (set as index) | Index |
| `meantemp` | Average daily temperature, °C | Yes (target) |
| `humidity` | Average daily humidity | No |
| `wind_speed` | Average daily wind speed | No |
| `meanpressure` | Average daily air pressure | No |

### Columns created in the notebook

| Column | Meaning |
|---|---|
| `meantemp_log` | log10 of `meantemp`. The model trains on this. |
| `sin1`–`sin3`, `cos1`–`cos3` | Fourier terms for the yearly cycle (period 365.25 days). Extra model inputs. |

---

## Method

1. **Load data.** Parse dates, set the date as index, sort, set daily frequency.
2. **Log transform.** Apply log10 to temperature.
3. **Decompose.** Additive decomposition with a 365-day period (trend, seasonal, residual).
4. **Stationarity test.** Augmented Dickey-Fuller (ADF) test on the log series.
   - Test statistic: -3.59, p-value: 0.006
   - The test calls the series stationary at the 5% level.
5. **Train/test split.** Time-based, no shuffling.
   - Train: first 80% = 1,169 days (2013-01-01 to 2016-03-14)
   - Test: last 20% = 293 days (2016-03-15 to 2017-01-01)
6. **Fourier features.** K = 3 sine/cosine pairs, one shared time origin for train, test and future dates.
7. **Model selection.** `auto_arima` with stepwise search, `max_p = 3`, `max_q = 3`, `d` chosen automatically, Fourier terms as external inputs.
   - Best model: ARIMA(3,1,3) with Fourier terms (AIC = -4712.55)
8. **Residual checks.** Fit SARIMAX on training data and plot diagnostics.
9. **Evaluation.** Predict the full test period in one go, convert back from log with `10 ** prediction`, score with MAPE and RMSE.
10. **Baseline.** Temperature on the same day one year earlier (365-day shift).
11. **Final model.** Retrain on all 1,462 days.
12. **Forecast.** 365 days from 2017-01-02 to 2018-01-01, with 95% confidence bounds converted back to °C.
13. **Export.** Save forecast as CSV and charts as images.

---

## Results

### Test set performance (293 days)

| Model | MAPE | RMSE (°C) |
|---|---|---|
| SARIMAX + Fourier (ARIMA 3,1,3) | **6.83%** | **2.34** |
| Baseline: same day last year | 9.88% | 3.53 |

The model beats the baseline by about 3 percentage points of MAPE and 1.2 °C of RMSE.

### Forecast (2017-01-02 to 2018-01-01)

- The forecast follows a smooth yearly curve: cool winter, peak around June (about 36–37 °C), small dip and plateau in the monsoon months, then cooling.
- The 95% range gets wider with time. By mid-2017 it spans roughly 29 °C to 45 °C.
- First rows of the forecast:

| Date | Forecast (°C) | Lower 95% | Upper 95% |
|---|---|---|---|
| 2017-01-02 | 11.26 | 9.80 | 12.93 |
| 2017-01-03 | 11.68 | 9.86 | 13.85 |
| 2017-01-04 | 12.43 | 10.30 | 15.01 |
| 2017-01-05 | 12.87 | 10.57 | 15.67 |
| 2017-01-06 | 13.09 | 10.70 | 16.02 |

---

## Output Files

Created by the notebook in the working folder:

| File | Content |
|---|---|
| `delhi_decomposition.pdf` | Trend, seasonal and residual plot |
| `delhi_model_evaluation.jpg` | Train, actual test and predicted values |
| `delhi_final_forecast.png` | History plus 365-day forecast with 95% band |
| `delhi_temperature_forecast.csv` | Columns: `Forecast`, `Lower_CI`, `Upper_CI`, indexed by date |

---

## How to Run

### Requirements

The notebook was run on Python 3.13 with these versions:

| Package | Version |
|---|---|
| numpy | 2.1.3 |
| pandas | 2.2.3 |
| scikit-learn | 1.6.1 |
| scipy | 1.15.3 |
| statsmodels | 0.15.0 |
| pmdarima | 2.1.1 |
| matplotlib | needed (version not shown in notebook) |

Install:

```bash
pip install numpy pandas matplotlib scikit-learn statsmodels pmdarima
```

### Steps

1. Put `DailyDelhiClimateTrain.csv` in the project folder.
2. **Fix the file path.** The notebook has a hardcoded Windows path in the "Load Data" cell:
   ```python
   os.chdir(r"C:\Users\HP\Desktop\Python for Data Science\Machine Learning\Time Series Fourcasting")
   data = pd.read_csv(r"C:\Users\HP\...\DailyDelhiClimateTrain.csv")
   ```
   Replace both with a relative path, for example `pd.read_csv("DailyDelhiClimateTrain.csv")`, and remove the `os.chdir` line.
3. Open the notebook and run all cells in order:
   ```bash
   jupyter notebook DailyDelhiClimateTrainTime_Series_Forecasting_Project.ipynb
   ```
4. `auto_arima` takes about 35 seconds on the original machine.

---

## Project Structure

```
.
├── DailyDelhiClimateTrainTime_Series_Forecasting_Project.ipynb
├── DailyDelhiClimateTrain.csv          # input (not generated)
├── delhi_decomposition.pdf             # output
├── delhi_model_evaluation.jpg          # output
├── delhi_final_forecast.png            # output
└── delhi_temperature_forecast.csv      # output
```

---

## Known Limitations

Read these before trusting the numbers.

**Data**
- Only 4 yearly cycles. The model has seen each season just 3–4 times.
- The test period is not a full year. It covers 15 Mar 2016 to 1 Jan 2017, so mid-January to mid-March is never tested.
- Humidity, wind and pressure are ignored.

**Model**
- The forecast is almost entirely the Fourier curve. The test plot shows a smooth line that misses day-to-day swings and under-predicts the summer peak. The ARIMA part adds little at long horizons.
- No comparison against a Fourier-only model (no ARIMA). So the value of ARIMA(3,1,3) is unproven.
- The baseline is weak. "Same day last year" copies one noisy day. A multi-year average for each calendar day would be a fairer test.
- The ADF test says the data is stationary (p = 0.006), yet `auto_arima` chose one round of differencing (`d = 1`). These two do not agree, and the notebook does not explain it.
- `auto_arima` picked a model with an intercept, but the refit SARIMAX uses no trend term. The refit model is therefore not exactly the one that was selected. The intercept was not significant (p = 0.66), so the effect is likely small.

**Residuals (from the diagnostics)**
- Errors are not normal: kurtosis 7.24, Jarque-Bera p ≈ 0. Heavy tails show up on the Q-Q plot, with large misses in a few winter periods.
- Error spread changes over time (heteroskedasticity test p ≈ 0.00).
- Autocorrelation of residuals looks fine in the correlogram.
- Because of the first two points, the 95% confidence bands are probably not exact.

**Evaluation**
- One train/test split only. No rolling-origin cross-validation.
- MAPE on °C is unstable at low temperatures, and one error of 2 °C counts more in winter than in summer.
- Converting back with `10 ** prediction` gives a median-type forecast, slightly below the mean. The bias is small here.
- The log transform is not justified in the notebook. No check shows that variance grew with the level of temperature.

**Reproducibility**
- Hardcoded absolute path (see How to Run).
- `pip install` runs inside the notebook, and package versions are not pinned in a file.

---

## Possible Improvements

- Add a `requirements.txt` and use relative paths.
- Compare against a Fourier-only regression and a day-of-year average baseline.
- Test different Fourier counts (K) and drop the log transform to see if it matters.
- Use rolling-origin cross-validation instead of one split.
- Add humidity, wind and pressure as external inputs (these would need their own forecasts for the future).
- Use more years of data if available.
- Add a short written interpretation of the residual plots to the notebook.

---

## Author

**Rajdip Sanyal**
