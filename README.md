# Chronic Disease Risk Profiling Using Data Mining
### CS5831 - Final Project | Group GR-1

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
- Applied **group-wise KNN imputation (k=5)** per `(yr_start, yr_end)` time window — only applied to indicators with less than 30% missing values to avoid unreliable imputation; higher-missingness indicators were left as-is
- Applied Z-score normalization via `StandardScaler`

### 2. Aggregation Strategy
Two parallel analytical views were constructed after imputation:

| View | Description |
|------|-------------|
| **Year-specific** | Analysis performed separately per `(yr_start, yr_end)` group, preserving temporal structure |
| **Aggregated (all-time)** | State-level averages across all time windows for a global view; light mean imputation applied post-aggregation for any remaining missingness |

### 3. Target Variable Engineering
- Constructed a **composite chronic disease burden score** by averaging normalized indicator values per observation
- Applied a **75th percentile threshold** to assign binary labels: `1 = High-Risk`, `0 = Low-Risk`
- Result: 119 high-risk and 357 low-risk observations

### 4. Supervised Classification
Four models were trained and evaluated using stratified 5-fold cross-validation and an 80/20 stratified train-test split:

| Model | Notes |
|-------|-------|
| Logistic Regression | L2 regularization, interpretable baseline |
| Random Forest | Bagging ensemble, tuned via RandomizedSearchCV |
| AdaBoost | Sequential boosting of weak learners |
| XGBoost | Gradient boosting with regularization, tuned via GridSearchCV |

**Evaluation metrics:** Accuracy, Precision, Recall, F1-Score, ROC-AUC

### 5. Unsupervised Clustering
K-Means and DBSCAN were applied under both analytical views:

**Year-specific clustering** — run independently per time window:
- PCA dimensionality reduction per group (after dropping remaining NaNs)
- K-Means with elbow method and silhouette-based k selection
- DBSCAN with grid search over epsilon and min_samples

**Aggregated clustering** — run on the all-time state-level dataset:
- PCA on the aggregated feature matrix
- K-Means with option for manual k override (auto-selection sometimes favored large k due to marginal silhouette differences)
- DBSCAN with parameter sweep

**Internal validation metrics:** Silhouette Score, Davies-Bouldin Index  
**Visualization:** PCA 2D projection with ground truth risk label comparison panel  
**Profiling:** Heatmap of top variable indicators across K-Means clusters

---

## Repository Structure
```
├── CS5831_FinalProject .ipynb          # Main analysis notebook
├── column_abbreviation_guide.csv       # Mapping of abbreviated to original column names
├── pivoted_df2.csv                     # Pivoted Crude Prevalence dataset
├── clean_df.csv                        # KNN-imputed cleaned dataset
├── clean_df_alltime_processed.csv      # Aggregated all-time processed dataset
├── U.S._Chronic_Disease_Indicators.csv # Raw dataset (not tracked — download separately)
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
3. Launch JupyterLab and open `CS5831_FinalProject .ipynb`
4. Run all cells top to bottom

---

## Key Findings

- **West Virginia (2019)** had the highest composite burden score (0.864), followed by Kentucky and Mississippi
- **Top predictors** of high-risk classification: lack of broadband access, COPD smoking rate, fair/poor self-rated health, physical inactivity, and poor diet indicators
- **Random Forest and XGBoost** were the best-performing classifiers based on ROC-AUC
- **KNN imputation grouped by time window** produced a more stable and statistically reliable dataset compared to global median imputation
- **K-Means** produced more interpretable clusters than DBSCAN across both temporal and aggregated views; DBSCAN flagged a large proportion of observations as noise, consistent with the diffuse high-dimensional structure of the data

---

## References

1. Watson, K.B., et al. (2024). Chronic Disease Indicators: 2022–2024 refresh and web tool enhancements. *Preventing Chronic Disease*, 21.
2. Benavidez, G.A., Kershaw, K.N., et al. (2024). Sociodemographic and geographic disparities in chronic disease prevalence in the United States. *Preventing Chronic Disease*, 21.

---

*Michigan Technological University — CS5831 Data Mining — Spring 2026* 
