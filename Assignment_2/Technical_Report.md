# Technical Report — Predicting Telco Monthly Charges from Add-On Services

**EECE 6544, Assignment #02**

## 1. What does the base plan cost before any add-ons?

The model's intercept is $24.97/month. A customer with every indicator at 0 is expected to pay about $25.

## 2. How much does each individual add-on contribute to the monthly bill?

The learned coefficients, sorted by size:

| Add-on | Coefficient (monthly $) |
|---|---|
| InternetService_Fiber optic | +24.95 |
| PhoneService | +20.04 |
| StreamingTV | +9.98 |
| StreamingMovies | +9.95 |
| OnlineSecurity | +5.05 |
| TechSupport | +5.03 |
| MultipleLines | +5.01 |
| DeviceProtection | +5.01 |
| OnlineBackup | +4.98 |
| InternetService_No | −25.05 |

Each weight reads directly as the monthly price of that service.

## 3. Which add-on is the most expensive, and which is the cheapest?

The most expensive is the fiber-optic internet upgrade at +$24.95/month. The cheapest
is online backup at +$4.98/month, though the five $5-tier services (multiple lines,
online security, online backup, device protection, tech support) are essentially tied.

## 4. If a customer adds fiber internet plus both streaming services, how much should their bill rise?

$24.95 (fiber) + $9.98 (streaming TV) + $9.95 (streaming movies) ≈ +$44.88/month.

## 5. Given a specific bundle (phone service, multiple lines, and tech support), what monthly charge should we expect?

$24.97 (base) + $20.04 (phone) + $5.01 (multiple lines) + $5.03 (tech support) ≈
$55.05/month, which matches the model's prediction for that customer in the notebook.

## 6. Is knowing *which* services meaningfully better than knowing *how many*, for predicting the bill?

Yes, by a wide margin. On the identical train/test split, the single-variable baseline
(count of add-ons) was compared with the multivariable model:

| Model | R² | MAE | RMSE |
|---|---|---|---|
| Baseline — count of add-ons | 0.701 | $14.24 | $16.46 |
| Multivariable — which add-ons | **0.999** | **$0.79** | **$1.05** |

The count-only model fails because it treats a $24.95 fiber upgrade and a $4.98 backup
add-on as the same "one service." Knowing which services cuts the average error from about
$14 to under a dollar.

## 7. How accurate are our predictions on customers the model hasn't seen?

Within a few dollars. On the 1,409 held-out customers the model achieves
R² = 0.9988, a mean absolute error of $0.79, and an RMSE of $1.05, across bills
ranging from $18.25 to $118.75. 

## 8. Do the model's learned prices match Telco's actual price list, and are any coefficients surprising?

The coefficients land almost exactly on round price points: $25, $20, $10, and $5, which
strongly suggests the model recovered Telco's real tariff structure rather than noisy
statistical associations. One coefficient is negative: InternetService_No at −$25.05.
This is not a pricing anomaly. Because DSL is the reference internet category, its ~$25
internet component is absorbed into the intercept; customers with no internet service get
that amount backed out of their bill. No other coefficient is surprising in sign or
magnitude.
