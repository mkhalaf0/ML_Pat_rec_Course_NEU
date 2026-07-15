# HealthGuard Insurance MiniProject #02

This repository contains the complete EECE 6544 MiniProject #02 workflow for medical-cost exploration, regression, and expensive-customer classification.

## Main findings

- The cleaned dataset has **1,337 unique records** after removing one duplicate.
- Smokers average **3.80x** the charges of non-smokers.
- The best regression model is **Decision Tree Regression** with test RMSE **$4,345.88** and R² **0.897**.
- The best classifier is **Support Vector Classifier (RBF)** with F1 **0.939** and ROC-AUC **0.961**.

## Download the dataset from Kaggle

Dataset page: `https://www.kaggle.com/datasets/mirichoi0218/insurance`


### Kaggle API method

```bash
pip install kaggle
kaggle datasets download -d mirichoi0218/insurance -p . --unzip
```

## Run the project

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Open `HealthGuard_Insurance_MiniProject02.ipynb` and choose **Run All**.

To execute from the command line:

```bash
jupyter nbconvert --to notebook --execute --inplace HealthGuard_Insurance_MiniProject02.ipynb
```

## Reproducibility notes

- Random state: 42.
- Train/test split: 80/20.
- Preprocessing occurs inside model pipelines.
- Ridge, Lasso, and tree hyperparameters are selected using cross-validation on training data.
- Raw and cleaned dataframes are kept separate.
