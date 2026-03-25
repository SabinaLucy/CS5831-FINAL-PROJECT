# Chronic Disease Risk Profiling Using Data Mining
### CS5831 — Final Project | Group GR-1

**Members:** Sabina Bimbi · Ali Mansouri · Terence Mpofu · Lydia Tembreull  
**Dataset:** [CDC U.S. Chronic Disease Indicators (CDI)](https://catalog.data.gov/dataset/u-s-chronic-disease-indicators)

---

## Overview

This project applies data mining techniques to the CDC U.S. Chronic Disease Indicators (CDI) dataset to identify and predict high-risk states based on chronic disease prevalence patterns. The dataset contains approximately 300,000 observations across 115 standardized public health indicators reported at the U.S. state level over multiple years.

The project is framed as a mixed data mining problem combining **supervised classification** and **unsupervised clustering** to uncover geographic and temporal disparities in chronic disease burden across the United States.

---

## Problem Statement

Chronic diseases are a leading cause of morbidity and mortality in the United States, with significant variation between states and demographic groups. Rather than simply reporting statistics, this project builds predictive models to identify which states are at high risk and which indicators drive that risk — supporting better-informed public health interventions and resource allocation.

---

## Dataset

- **Source:** CDC U.S. Chronic Disease Indicators (updated February 3, 2025)
- **Size:** ~300,000 observations × 34 columns (long format)
- **Coverage:** U.S. states and territories, multiple years, stratified by sex and race/ethnicity
- **Selected subset:** `Crude Prevalence` data type → 476 state-year observations × 81 indicators after pivoting and cleaning

---

## Methodology

### 1. Data Preprocessing
- Filtered to `Crude Prevalence` indicators for cross-state comparability
- Reshaped from long to wide format via pivot table
- Renamed columns with structured abbreviations (see `column_abbreviation_guide.csv`)
- Imputed missing values using column-wise median
- Applied Z-score normalization via `StandardScaler`

### 2. Target Variable Engineering
- Constructed a **composite chronic disease burden score** by averaging normalized indicator values per observation
- Applied a **75th percentile threshold** to assign binary labels: `1 = High-Risk`, `0 = Low-Risk`
- Result: 119 high-risk and 357 low-risk observations

### 3. Supervised Classification
Four models were trained and evaluated using stratified 5-fold cross-validation and an 80/20 train-test split:

| Model | Notes |
|-------|-------|
| Logistic Regression | L2 regularization, interpretable baseline |
| Random Forest | Bagging ensemble, tuned via RandomizedSearchCV |
| AdaBoost | Sequential boosting of weak learners |
| XGBoost | Gradient boosting with regularization, tuned via GridSearchCV |

**Evaluation metrics:** Accuracy, Precision, Recall, F1-Score, ROC-AUC

### 4. Unsupervised Clustering
Two clustering methods were applied to the full scaled feature matrix:

| Method | Result |
|--------|--------|
| K-Means | Optimal k=2 via silhouette score · Silhouette: 0.3940 · Davies-Bouldin: 2.4395 |
| DBSCAN | 3 clusters · 252 noise points · Silhouette: 0.3185 · Davies-Bouldin: 0.7540 |

Clusters were visualized via PCA 2D projection and profiled by their top variable indicators.

---

## Repository Structure
```
├── CS5831_FinalProject_.ipynb     # Main analysis notebook
├── column_abbreviation_guide.csv  # Mapping of abbreviated to original column names
├── U.S._Chronic_Disease_Indicators.csv  # Raw dataset (not tracked — download separately)
└── README.md
```

---

## How to Run

### Requirements
```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost
```

### Environment
The notebook was developed using Python 3.10 in a conda environment (`dmsp26`) and is compatible with both **JupyterLab** (local/server) and **Google Colab**.

### Steps
1. Download the dataset from the [CDC portal](https://catalog.data.gov/dataset/u-s-chronic-disease-indicators) and place it in the same directory as the notebook as `U.S._Chronic_Disease_Indicators.csv`
2. Clone this repository:
```bash
git clone https://github.com/SabinaLucy/CS5831-FINAL-PROJECT.git
cd CS5831-FINAL-PROJECT
```
3. Launch JupyterLab and open `CS5831_FinalProject_.ipynb`
4. Run all cells top to bottom

---

## Key Findings

- **West Virginia (2019)** had the highest composite burden score (0.864), followed by Kentucky and Mississippi
- **Top predictors** of high-risk classification: lack of broadband access, COPD smoking rate, fair/poor self-rated health, physical inactivity, and poor diet indicators
- **Random Forest and XGBoost** were the best-performing classifiers based on ROC-AUC
- **K-Means (k=2)** produced more interpretable clusters than DBSCAN, which flagged over 50% of observations as noise — consistent with the diffuse, high-dimensional nature of the data

---

## References

1. Watson, K.B., et al. (2024). Chronic Disease Indicators: 2022–2024 refresh and web tool enhancements. *Preventing Chronic Disease*, 21.
2. Benavidez, G.A., Kershaw, K.N., et al. (2024). Sociodemographic and geographic disparities in chronic disease prevalence in the United States. *Preventing Chronic Disease*, 21.

---

*Michigan Technological University — CS5831 Data Mining — Spring 2026*

