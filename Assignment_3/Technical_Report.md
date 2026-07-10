# Technical Report — Predicting the Fair Market Price of a Used Car

**EECE 6544, Assignment #03 — WheelsBazaar**

## 1. What problem are we solving, and how did we frame it?

WheelsBazaar needs an automated, trustworthy price estimate for every used-car
listing. Today sellers set their own prices and analysts can hand-review fewer
than 25% of the 5,000 monthly listings, which lets overpriced cars clog inventory
and underpriced cars burn seller trust. The same estimate also underpins the
*WheelsBazaar Direct* instant-buy program, where a wrong number costs the company
real money on every purchase.

We framed this as a **supervised regression** problem: predict the continuous
target `Price` (₹) from 2,059 historical listings described by make, age, usage,
powertrain, and body dimensions.

## 2. How did we prepare the data?

Three fields hid numbers inside text, and one was a raw calendar year, so we
engineered clean numeric features:

| Raw column | Example | Engineered feature |
|---|---|---|
| `Engine` | `1198 cc` | `engine_cc` = 1198 |
| `Max Power` | `87 bhp @ 6000 rpm` | `power_bhp` = 87 |
| `Max Torque` | `109 Nm @ 4500 rpm` | `torque_nm` = 109 |
| `Year` | `2017` | `age` = 2022 − 2017 = 5 |

We collapsed the long tail of rare fuel types (Electric, LPG, Hybrid, dual-fuel)
into a single `Other` bucket, dropped high-cardinality columns with little price
signal (`Model`, `Location`, `Color`), and handled the ~3–7% missing values with
median imputation (numeric) and most-frequent imputation (categorical). All
preprocessing lives inside a scikit-learn `Pipeline` and is fit on the training
split only, so no information leaks from the test set.

## 3. Why do we model `log(Price)` instead of `Price`?

Price is extremely right-skewed — it ranges from ₹49,000 to ₹35,000,000, with a
few luxury cars dominating the tail. Trained on raw price, a model chases those
outliers and does poorly on the ordinary cars that make up most of the inventory.
We train on `log1p(Price)` and invert (`expm1`) before scoring, so every reported
error is on the real rupee scale.

## 4. Which model did we choose, and how well does it work?

We compared three models on the same 80/20 split, scored on the real ₹ scale:

| Model | R² | MAE | RMSE | MAPE |
|---|---|---|---|---|
| Naive (predict mean price) | −0.00 | ₹1,454,531 | — | — |
| Linear regression | 0.689 | ₹330,411 | ₹1,473,917 | 16.1% |
| **Gradient boosting** | **0.898** | **₹280,971** | **₹843,413** | **15.1%** |

Gradient boosting wins decisively. It explains ~90% of price variance versus 69%
for the linear model, because used-car depreciation is non-linear — a 10-year-old
luxury sedan and a 10-year-old hatchback lose value on completely different
curves, and tree ensembles capture those interactions. A 5-fold cross-validation
gives a stable log-scale R² of **0.941 ± 0.016**, confirming the result is not an
artifact of one lucky split.

## 5. What drives a used car's price?

Gradient-boosting feature importances (top drivers):

| Feature | Importance |
|---|---|
| Engine power (`power_bhp`) | 0.54 |
| Age | 0.13 |
| Width | 0.09 |
| Torque | 0.06 |
| Transmission (manual/automatic) | 0.09 (combined) |
| Length, kilometers, fuel-tank size, make | remainder |

Engine power is by far the strongest signal — it proxies the car's segment and
trim. Age is second (depreciation), followed by physical size, torque, and
transmission type. This matches real-world used-car intuition and gives the
result face validity.

## 6. How does this translate into business action?

**Instant fair-price quotes (WheelsBazaar Direct).** For any car the model returns
a price in milliseconds. Example — a 2018 Hyundai petrol hatchback, 45,000 km,
first owner, manual → **₹664,862 fair estimate**, ready to underwrite an
instant-buy offer.

**Over-/under-priced triage.** We compare each seller's asking price to the model
estimate and apply the business's own ±15% band:

- Asking **> 15% above** estimate → **OVERPRICED** (the listings that sit unsold).
- Asking **> 15% below** estimate → **UNDERPRICED** (sellers lose money, then complain).
- Within ±15% → **fair**, auto-cleared.

On the held-out test set this auto-clears **64%** of listings as fair and flags
the rest (≈20% overpriced, ≈16% underpriced) for analyst attention. Instead of
hand-reviewing under 25% of inventory, analysts now review only the ~36% that
actually look off — the entire 5,000/month catalog gets screened automatically.

## 7. Limitations and next steps

- MAPE is ~15%, so individual estimates carry a meaningful error band; for
  *WheelsBazaar Direct* we should quote a range and add a margin, not a single number.
- `Model`-level effects are only captured indirectly through `Make` + specs;
  target-encoding the model name (done leakage-safely) is the clearest next lift.
- The reference year (2022) is baked into `age`; the pipeline should re-derive age
  from the current date at inference time and be retrained periodically as the
  market moves.
