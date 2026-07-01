# Cobblestone Gifts — Sales Data Cleaning & Analysis

**EECE 6544 · Mini Project #01 · Summer 2026**

Cleaning and analysis of a raw UK online-retail export (~542k transaction lines,
Dec 2010 – Dec 2011) for *Cobblestone Gifts*. The project profiles a messy real-world dataset,
applies the full pandas wrangling toolkit (all 21 techniques), builds a tidy completed-sales
table, and answers seven business questions for the Head of Commercial.

## What this project does
1. **Profiles** the raw export (encoding, shape, missingness, impossible values).
2. **Cleans** it — removing cancellations, returns, non-product lines, bad prices/quantities, and
   duplicates; standardising country labels; and making a documented decision to *keep* anonymous
   (no-customer-ID) sales for revenue analysis.
3. **Engineers & aggregates** features (revenue, region, monthly trend) to answer real questions
   about seasonality, best sellers, markets, customer concentration, order value, and returns.

## Dataset
**E-Commerce Data (UK Online Retail)** — UCI ML Repository, hosted on Kaggle:
https://www.kaggle.com/datasets/carrie1/ecommerce-data

The raw file is **not UTF-8** — it is read with `encoding="ISO-8859-1"`.

### How to download
- **Option A — website:** open the Kaggle page, click *Download*, and unzip. You get `data.csv`
  (~45 MB, 541,909 rows).
- **Option B — Kaggle API:**
  ```bash
  pip install kaggle               # place kaggle.json in ~/.kaggle/
  kaggle datasets download -d carrie1/ecommerce-data
  unzip ecommerce-data.zip
  ```

Place `data.csv` in the project root (next to the notebook).

## How to run
```bash
python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook mini_proeject1.ipynb   # Run All
```
Running the notebook end-to-end re-creates `clean_online_retail.csv` and the charts in `figures/`.

## Repository contents
| File | Description |
|---|---|
| `mini_proeject1.ipynb` | Cleaning + analysis notebook; all 21 techniques labelled by number, plus the 7 business questions. |
| `clean_online_retail.csv` | The cleaned completed-sales dataset (output). |
| `FINDINGS.md` | One-page findings summary answering the seven business questions, with charts. |
| `CLEANING_DECISIONS.md` | Log of the judgment calls made (what was dropped / kept / changed) and why. |
| `figures/` | Charts used in the findings (monthly revenue, best sellers, top markets). |
| `requirements.txt` | Python libraries + versions. |
| `build_notebook.py` | Helper that programmatically generates the notebook (for reproducibility). |

## Summary of findings
- **Total revenue:** ≈ **£10.25M** for the year.
- **Seasonality:** strong autumn build-up; **Nov 2011** peaks ~**84%** above an average month.
- **Markets:** overwhelmingly UK; top non-UK markets are Netherlands, Ireland, Germany, France.
- **Customers:** highly concentrated — the top **1%** of identified customers ≈ **32%** of their
  revenue. This is a **wholesale-driven** business.
- **Order value:** non-UK average order (~£815) is far larger than UK (~£487).
- **Data quality:** ~**3.5%** of raw rows removed as invalid; anonymous-customer sales kept (flagged).
  Trustworthy for board-level revenue/market analysis, with two footnotes (customer concentration
  covers ~75% of revenue; the final month is partial).
