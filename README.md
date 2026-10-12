# Forecasting UK "Flower" search interest: Prophet, SARIMA and Holt-Winters, re-evaluated like a demand planner

Two notebooks on the same question: how well can monthly UK search interest in flowers be forecast, and which model should a planning team trust?

| Notebook | What it does |
|---|---|
| `Predicting the Popularity of “Flowers” Searches on Google.ipynb` | The original exercise, adapted from Marco Peixeiro's book [Time Series Forecasting in Python](https://amzn.eu/d/eU0GuYo). Prophet, SARIMA and Holt-Winters compared on a single 12-month test (June 2023 to May 2024). Kept as the record of the June 2024 run, with corrections listed at the top. |
| `02_flower_searches_backtest_and_improved_forecast.ipynb` | My extension. The models re-evaluated on data refreshed to September 2026: monthly re-forecasts judged over three years, against a seasonal naive benchmark, measuring bias as well as accuracy, with the 2020–21 lockdown months cleansed from the history. Ends with a properly refitted forecast for October 2026 to September 2027. |

## Key findings (notebook 2)

The test period covers forecasts made every month from May 2023 to September 2025, each for the next 12 months, scored on June 2023 to September 2026 (348 forecasts per model). Every choice was made on an earlier validation period (forecasts made August 2021 to April 2022).

- **One test window picked the wrong model.** The SARIMA(2,1,2)(3,1,1)12 that won notebook 1's single 12-month test was the only model worse than the benchmark in validation, and it ranked ninth of ten in the test (WAPE 10.5% against the benchmark's 7.0%).
- **The improved forecast**, an equal-weight combination of each model family's validation winner (with the lockdown months cleansed and the SARIMA orders chosen consistently), scored 6.8% in validation and 6.7% in the test. It was the only approach in the top two in both periods, with 36% less error than notebook 1's choice.
- **Bias tells the story.** Prophet with notebook 1's settings over-forecast in 89% of test months (bias +8.0%), because its trend kept extrapolating the 2020–21 surge. Cleansing the lockdown months cut its bias to +5.3% and its WAPE from 10.5% to 8.7%.
- **"Same month last year" is hard to beat in a steady market.** The best model beat it by half a point in the test, over half of that in the first three months of the horizon. In validation, straight after lockdown, the models beat it by up to six points.

| Model | Validation WAPE | Test WAPE | Test bias | Test months over-forecast | Forecast value added (test) |
|---|---|---|---|---|---|
| Holt-Winters (notebook 1 baseline) | 8.6% | **6.5%** | −0.4% | 55% | +0.5 |
| **Combination of family winners** | **6.8%** | 6.7% | +1.7% | 71% | +0.4 |
| Holt-Winters, damped trend, cleansed | 9.1% | 6.7% | −0.3% | 57% | +0.3 |
| Holt-Winters, cleansed | 9.3% | 6.7% | +0.3% | 59% | +0.3 |
| SARIMA(0,1,1)(0,1,1)12, log scale | 12.0% | 6.8% | −0.5% | 55% | +0.2 |
| SARIMA(0,1,1)(0,1,1)12, log scale, cleansed | 9.5% | 7.0% | 0.0% | 59% | +0.1 |
| Seasonal naive (benchmark) | 12.9% | 7.0% | −1.8% | 43% | 0.0 |
| Prophet, cleansed | 6.8% | 8.7% | +5.3% | 86% | −1.7 |
| SARIMA(2,1,2)(3,1,1)12 (notebook 1's choice) | 14.5% | 10.5% | +0.2% | 53% | −3.4 |
| Prophet (notebook 1 settings) | 8.5% | 10.5% | +8.0% | 89% | −3.5 |

WAPE = Σ|forecast − actual| ÷ Σactual. Bias = Σ(forecast − actual) ÷ Σactual, so positive means over-forecasting. Forecast value added = benchmark WAPE − model WAPE, in percentage points. Full scores: `forecast_results/nb2_test_scores.csv`.

## Data

- **Source:** [Google Trends](https://trends.google.com/trends/explore?date=all&geo=GB&q=%2Fm%2F0c9ph5), topic "Flower" (`/m/0c9ph5`), United Kingdom, monthly, January 2004 to September 2026. Downloaded on 12 October 2026 and saved as `data/flower_uk_google_trends.csv`. October 2026 was a partial month and is flagged in the file and excluded.
- **Units:** an index of search interest, not a count: 100 is the month of peak interest in the period downloaded (May 2020).
- **Downloads differ.** Google Trends samples its data, so values move by a few points between downloads. Notebook 1 used a June 2024 download, which is not in this repository; for its test months the two downloads differ by up to eight points (notebook 2, section 1).

## Method (notebook 2)

- **Monthly re-forecasts:** at the end of each month, every model is refitted on the data known then and forecasts 12 months ahead.
- **Validation then test:** all choices (SARIMA orders, the model kept from each family) are made on forecasts made August 2021 to April 2022; the test (forecasts made May 2023 to September 2025) is only reported.
- **History cleansing:** the lockdown months, April 2020 to July 2021, are replaced by the normal seasonal shape on a level interpolated across the window. Every cleansed model is also run on the raw history.
- **SARIMA orders:** chosen by AIC with consistent differencing (d = 1, D = 1, log scale); among models within 2 AIC points of the best, the one with the fewest parameters.
- **Combination:** the equal-weight average of the validation winner in each family (exponential smoothing, ARIMA, Prophet).
- **Forward forecast:** the combination's members refitted on all data to September 2026; the 80% range comes from the combination's own test errors at each horizon. Saved to `forecast_results/nb2_forecast_2026-10_to_2027-09.csv`.

## Corrections to notebook 1

Listed at the top of the notebook. In brief: the data are a Google Trends index for the topic "Flower", not a proportion of keyword searches; the selected `holidays_prior_scale` was 0.01, not 10.0; the closing Prophet errors now match the table; Mothering Sunday is in March; the SARIMA search used D = 0 but the fitted model used D = 1; and the 12-month "future" forecast was the test-period forecast relabelled, because the model was never refitted on all the data.

## Repository structure

- `data/`: the Google Trends series used by notebook 2
- `figures/`: charts saved by notebook 2
- `forecast_results/`: test scores and the forward forecast from notebook 2

## Reproduce

```sh
git clone https://github.com/jduquegonzalez/Forecasting-Google-Search-Popularity-Prophet-SARIMA-and-Holt-Winters-Comparison.git
cd Forecasting-Google-Search-Popularity-Prophet-SARIMA-and-Holt-Winters-Comparison
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute 02_flower_searches_backtest_and_improved_forecast.ipynb
```

Notebook 2 runs in about five minutes. It was last run with Python 3.11, pandas 3.0, NumPy 2.4, statsmodels 0.15 and Prophet 1.5. Notebook 1 reads its data from a local path and is kept as run in June 2024.

## Credits and AI assistance

Notebook 1 is adapted from Marco Peixeiro's *Time Series Forecasting in Python*. Notebook 2 was built with AI assistance (Claude) and checked: every number in its text is computed in the notebook, and the headline test scores were recomputed independently.
