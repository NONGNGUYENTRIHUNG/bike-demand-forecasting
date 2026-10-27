# Bike Rental Demand Forecasting

Can yesterday's and last week's rental counts help predict how many bikes will be rented tomorrow? This project uses historical daily bike-sharing data to compare a simple baseline with two forecasting models. The goal is to demonstrate a repeatable approach to planning demand, rather than to make a claim about a real organization's savings.

## Project at a glance

- **Data:** Capital Bikeshare daily counts from 2011–2012, distributed by the [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/275/bike+sharing+dataset).
- **Forecast:** One day ahead, with actual demand becoming available after each day.
- **Methods:** Same day last week baseline, ridge regression, and random forest.
- **Tools:** Python, pandas, scikit-learn, Matplotlib. The results CSV can be imported into Tableau.
- **Main result:** Ridge regression had the lowest validation MAE. On the final test period its MAE was **827 rentals per day**, compared with **1,165** for the baseline—about **29% lower**. This is an evaluation on historical data, not a live operational result.

![Actual and predicted demand](results/forecast.png)

## How the project works

Each row represents a day. The forecast uses calendar information, the previous day's count, the count seven days earlier, and the average count over the previous seven days. The target day's actual weather and rental counts are not used as inputs. The model is a rolling one-day-ahead forecast: after a day ends, its actual count is available for the following day's prediction. It does not produce a seven-day forecast all at once.

The data was ordered by date and split chronologically:

| Period | Purpose |
| --- | --- |
| January 2011–March 2012 (after seven days of lag history) | Train candidate models |
| April–July 2012 | Compare models by mean absolute error (MAE) and select one |
| August–December 2012 | Evaluate the selected model on unseen dates |

After selection, the chosen model was refit on the training and validation periods and evaluated once on the test period. The baseline uses the actual rental count from the same day one week earlier. The feature values for each test day use counts from earlier days only, including earlier days in the test period, reflecting the rolling daily forecast.

| Split | Model | MAE | RMSE |
| --- | --- | ---: | ---: |
| Validation | Same day last week | 1,103.77 | 1,470.52 |
| Validation | Ridge regression | 825.28 | 1,104.06 |
| Validation | Random forest | 1,147.08 | 1,365.91 |
| Test | Same day last week | 1,165.06 | 1,728.40 |
| Test | Ridge regression, selected | 826.68 | 1,151.79 |

**MAE** is the average absolute difference between the forecast and actual demand. **RMSE** also measures forecast error but gives larger errors more weight. Lower is better for both. The test period contains 153 days. Numbers are rounded.

## Run it yourself

1. Install Python 3.9 or later.
2. Open a terminal in this project folder and run:

```bash
python -m pip install -r requirements.txt
python src/forecast.py
```

The script creates `results/metrics.csv`, `results/test_predictions.csv`, and `results/forecast.png`. Run it from any working directory; it resolves paths relative to the project folder.

## Tableau idea

Connect Tableau to `results/test_predictions.csv`. Put `date` on Columns and `actual` and `prediction` on Rows as two lines. Add `baseline` for comparison. You can also create `ABS([actual] - [prediction])` to inspect days with larger errors. A Tableau workbook is not included.

## Limitations and next steps

- The data is from 2011–2012, so the numbers do not represent current demand. Results from bikes cannot be assumed to transfer directly to a pantry or another service.
- This is one historical train/validation/test split; evaluate across several rolling time windows before relying on a model.
- The model does not use weather forecasts, events, station capacity, or service disruptions. Adding information available before the forecast could improve it, but must be tested.
- A one-day forecast assumes yesterday's actual total is available. For multi-day forecasts, future lag values would need a different strategy.
- Demand counts reflect rentals observed, which can be limited by bike availability; they may not capture all unmet demand.

## Source and attribution

Fanaee-T, Hadi. *Bike Sharing Dataset*. UCI Machine Learning Repository, 2013. https://doi.org/10.24432/C5W894. The accompanying original data documentation is in `data/source_readme.txt`. Check the dataset page for its license and citation details before reuse.
