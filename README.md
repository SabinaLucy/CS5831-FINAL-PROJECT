# Chronic Disease Risk Profiling Using Data Mining
### CS5831 - Final Project | Group GR-1

**Members:** Sabina Bimbi · Ali Mansouri · Terence Mpofu · Lydia Tembreull  
**Dataset:** [CDC U.S. Chronic Disease Indicators (CDI)](https://catalog.data.gov/dataset/u-s-chronic-disease-indicators)

---

## Overview

This project applies data mining techniques to the CDC U.S. Chronic Disease Indicators (CDI) dataset to identify and predict high-risk states based on chronic disease prevalence patterns. The raw dataset contains over 300,000 observations across standardized public health indicators reported at the U.S. state level.

The project is framed as a mixed data mining problem combining **supervised classification** and **unsupervised clustering** to uncover geographic disparities in chronic disease burden across the United States.

---

## Problem Statement

Chronic diseases are a leading cause of morbidity and mortality in the United States, with significant variation between states. Rather than simply reporting statistics, this project builds predictive models to identify which states fall into different risk tiers and which indicators drive that risk — supporting better-informed public health interventions and resource allocation.

---

## Dataset

- **Source:** CDC U.S. Chronic Disease Indicators (updated February 3, 2025)
- **Raw Size:** 309,215 observations × 34 columns (long format)
- **Coverage:** U.S. states and territories
- **Selected subset:** `Crude Prevalence` data type. After heavy filtering for sparsity, pivoting, and dropping columns with 100% missing values, the final analytical dataset consists of **55 unique state/territory observations × 24 predictors**.

---

## Methodology

### 1. Data Preprocessing
- Filtered to `Crude Prevalence` indicators for cross-state comparability.
- Reshaped from long to wide format via pivot table.
- Applied **KNN Imputation** to preserve the variation and correlation structures between similar states.
- Applied Z-score normalization via `StandardScaler` so features are equally weighted for distance-based algorithms.

### 2. Target Variable Engineering
- Constructed a **composite chronic disease burden score** by summing the normalized indicator values per state.
- Applied percentile thresholds to assign 3 ordinal risk classes:
  - **Low Risk (0):** Score < 25th percentile
  - **Moderate Risk (1):** 25th percentile ≤ Score < 75th percentile
  - **High Risk (2):** Score ≥ 75th percentile

### 3. Supervised Classification
Models were trained to predict the 3-tier risk categories. Due to the small sample size (n=55) and class imbalance, a **stratified 5-fold cross-validation** was utilized.

| Model | Notes |
|-------|-------|
| Logistic Regression | L2 regularization, interpretable baseline |
| Random Forest | Bagging ensemble (200 estimators), tuned via RandomizedSearchCV |
| AdaBoost | Sequential boosting of weak learners |
| XGBoost | Gradient boosting with regularization, tuned via GridSearchCV |

**Evaluation metrics:** Accuracy, Precision, Recall, F1-Score, ROC-AUC

### 4. Unsupervised Clustering
K-Means and DBSCAN were applied to identify natural geographic health groupings independently of the assigned risk labels:

- **PCA dimensionality reduction** for 2D spatial visualization.
- **K-Means:** Optimal number of clusters determined via the Elbow Method and Silhouette Analysis (optimal k=3).
- **DBSCAN:** Grid search over epsilon and min_samples to account for varying densities.

**Internal validation metrics:** Silhouette Score, Davies-Bouldin Index  
**Visualization:** PCA 2D projection with ground truth risk label comparison panel  
**Profiling:** Heatmap of top variable indicators across K-Means clusters

---

## Repository Structure
```text
├── CS5831_FinalProject.ipynb           # Main analysis notebook
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
