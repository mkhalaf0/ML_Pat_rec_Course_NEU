# EECE 6544 Assignment 06 - Household Energy Forecasting

This submission uses PyTorch to forecast the next 24 hours of household global
active power from the previous seven days (168 hours) of electrical measurements.

## Deliverables

- `Assignment_06_LSTM.ipynb` - complete, self-contained PyTorch solution
- `Assignment#06.pdf` - assignment specification
- `individual+household+electric+power+consumption.zip` - supplied UCI dataset
- `requirements.txt` - Python dependencies

## Setup

Python 3.11 is recommended. Create and activate a virtual environment, then run:

```bash
python -m pip install -r requirements.txt
```

Open `Assignment_06_LSTM.ipynb` in Jupyter and run all cells from top to bottom.
Keep the dataset ZIP in the same directory as the notebook. The notebook reads
`household_power_consumption.txt` directly from the archive, so manual extraction
is unnecessary.

## Method

- Convert `?` sensor readings to missing values.
- Resample minute measurements to hourly arithmetic means.
- Impute gaps independently within the train, validation, and test partitions
  using time interpolation and training medians for remaining boundary gaps.
- Hold out the final six calendar months for testing and the preceding three
  months for validation.
- Fit MinMax scalers only on training data to prevent future leakage.
- Use 168-hour multivariate input windows and direct 24-hour output targets.
- Train a PyTorch `nn.LSTM` with a residual connection around the previous day's
  24-hour consumption profile.
- Trigger anomaly alerts after two consecutive hourly residuals exceed three
  validation-residual standard deviations.

## Recorded results

The executed notebook currently reports:

- Test MAE: **0.447 kW**
- Test RMSE: **0.644 kW**
- Conventional pointwise test MAPE: **62.29%**
- Mean inference latency: **3.34 ms** per 24-hour forecast
- Test alert-rate proxy: **0.20%**

The latency and below-5% alert-rate constraints are met. The requested MAPE of
10% or less is not met and is reported honestly. Percentage error is particularly
severe for near-zero household loads, while individual appliance usage remains
difficult to predict over an open-loop 24-hour horizon.

Because this dataset has no labeled anomalies, the alert-episode rate is a
conservative false-alarm proxy rather than a labeled precision or recall measure.

## Generated files

When all notebook cells are run, an `artifacts/` directory is created containing
the hourly cache, PyTorch checkpoint, training history, metrics JSON, and forecast
plot. These outputs are reproducible and do not need to exist before running the
notebook.
