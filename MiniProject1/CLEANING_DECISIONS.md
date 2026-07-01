# Cleaning Decisions Log

Judgment calls made while turning the raw export into the tidy `sales` table. The raw DataFrame
(`raw`) is loaded once and never mutated; every step below produces a new object.

| # | Decision | What we did | Why |
|---|---|---|---|
| 1 | **Encoding** | Read with `encoding="ISO-8859-1"`. | The file is not UTF-8; the `£` symbol breaks a UTF-8 read. |
| 2 | **Cancellations** | Removed rows whose `invoice_no` starts with `C`. | They are reversals, not completed sales. |
| 3 | **Non-positive quantity** | Removed `quantity <= 0`. | Negative/zero units are returns or adjustments, not sales. |
| 4 | **Non-positive price** | Removed `unit_price <= 0`. | Zero/negative prices are free items, debt write-offs, or bad data — not valid revenue. |
| 5 | **Non-product codes** | Removed stock codes containing **no digit** (e.g. `POST`, `DOT`, `M`, `BANK CHARGES`, `AMAZONFEE`, `CRUK`, `PADS`, gift cards). | Postage, fees, and manual adjustments are not products and would distort product/revenue analysis. |
| 6 | **Blank descriptions** | **Dropped** the ~1,454 rows with a missing description (demonstrated `fillna` first, then chose `dropna`). | Almost all coincide with non-product/adjustment lines; filling them with a placeholder would invent data. |
| 7 | **Missing `customer_id` (~25%)** | **Kept** these rows in `sales`; added a `customer_known` flag. Customer-level questions use the identified subset only. | They are still real completed sales and matter for revenue/product/market totals. Dropping a quarter of valid revenue would bias every headline number downward. |
| 8 | **Duplicate rows** | Removed exact duplicates with `drop_duplicates()` (~5k rows). | Repeated identical lines are export artefacts and would double-count revenue. |
| 9 | **Country labels** | `EIRE → Ireland`, `RSA → South Africa`, `Unspecified → missing`. | Standardise inconsistent labels; `Unspecified` is not a real market so it should be missing, not a country. |
| 10 | **Column naming** | Renamed all columns to `snake_case`. | Consistent, code-friendly schema. |
| 11 | **Dropped helper column** | Added `desc_len` to inspect description quality, then dropped it. | Diagnostic only; no role in the final analysis. |
| 12 | **Date parsing** | Parsed `invoice_date` to datetime (`%m/%d/%Y %H:%M`). | Required for time-series resampling and seasonality. |
| 13 | **Region merge** | **Left** join with the country→region lookup; unmatched countries labelled `Other`. | An inner join would silently delete sales for unmapped countries and understate revenue. |
| 14 | **Returns analysis** | Re-derived returns/cancellations from `raw` for Q6. | They are excluded from `sales` by design, but still needed to quantify the return rate. |

## Assumptions
- A leading-`C` invoice or non-positive quantity/price is **not** a completed sale.
- Stock codes with no digit are **non-products**.
- Missing customer IDs are **anonymous retail sales**, not data errors.
- The export ends **early December 2011**, so the final month is partial and not comparable.

## Impact
- Raw rows: **541,909** → clean sales rows: **522,685** (~**96.5%** kept).
- ~**3.5%** removed as invalid; ~**25%** of rows kept *without* a customer ID (flagged).
