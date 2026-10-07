# A/B Testing & ANCOVA Analysis: Fast-Food Marketing Campaign Lift

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Statsmodels](https://img.shields.io/badge/Statsmodels-ANCOVA%20%26%20OLS-orange.svg)
![Seaborn](https://img.shields.io/badge/Seaborn-Data%20Viz-green.svg)
![Data Source](https://img.shields.io/badge/Data%20Source-Kaggle-yellow.svg)

An end-to-end statistical evaluation of a 4-week promotional campaign across 137 retail locations. 
This project leverages **ANCOVA modeling, sampling bias analysis, multi-variable variance reduction, and post-hoc power calculations** to evaluate campaign effectiveness and optimize marketing ROI.

---

## Executive Summary

A fast-food chain tested three distinct promotional strategies (`Promotion 1`, `Promotion 2`, and `Promotion 3`) across randomly assigned store locations over a 4-week trial. 

* **Key Finding**: **Promotion 1** and **Promotion 3** tied for the highest sales performance, significantly outperforming **Promotion 2**.
* **Quantified Lift**: The Promotion 1 generates an estimated +11$K additional revenue per store compared to Promotion 2 (95%CI:[6.57$K;14.97$K]), representing an average lift of 22% (95%CI:[14%;25%])
* **Business Decision**: Since P1 and P3 show statistically indistinguishable revenue lifts, the final choice between P1 and P3 should be based on execution cost and operational complexity.

---

## Key Results & Visualizations

### 1. Overall Promotion Lift (Controlled for Market Differences)
Promotions 1 and 3 drive substantially higher sales than Promotion 2 across all market sizes (Small, Medium, Large).

### 2. Temporal Stability & Novelty Bias
Analyzing weekly sales across the 4-week campaign confirmed that promotional performance remained stable over time, ruling out novelty bias or promotional fatigue.

---

## Statistical Methodology & Pipeline

### 1. Data Source & Aggregation
* **Data Source**: Kaggle - Marketing Campaign Dataset.
* **Aggregation**: Reduced 4 weekly measurements per store into a single store-level mean (`average_store_sales`). This keeps `LocationID` as the true unit of randomisation and prevents artificial inflation of the sample size.
* **Sample Size**: N = 137 store-level observations (n ~ 43--47 stores per promotion group).

### 2. Randomization Check (Bias Detection)
* **Market Size Balance**: Chi-square test of independence (p>0.05) confirmed no sampling bias across market sizes.
* **Store Age Balance**: One-Way ANOVA (p> 0.05) confirmed store age was evenly distributed across treatment groups.

### 3. Covariate Selection & Variance Reduction
* **Model Selection via Adjusted R²**
- `MarketID` was identified as nested within `MarketSize`. While `MarketID` explained over 90% of sales variance (R²_adj = 0.974), it severely overfitted the data by memorizing hyper-local market characteristics and consuming 9 degrees of freedom.
- To preserve statistical power and generalization, I specified the primary ANCOVA model using `MarketSize` (Small, Medium, Large) instead. This way, it combines two advantages :
  - reducing dummy variables from 9 to 2, keeping 132 residual degrees of freedom (N=137), and thus keeping the critical F-threshold low to allow the model to detect subtle sales uplifts with high sensitivity.
  - capturing macro-level market variance without absorbing the treatment effect of the promotions.
* **Evaluation of multiple OLS model specifications**
Adding `AgeOfStore` did not improve the Adjusted R² compared to the previous model, confirming that store age provides no additional explanatory power.
  
### 4. ANCOVA Model: `average_store_sales ~ C(Promotion) + C(MarketSize)`
* **Model Fit & Significance**:  
  - The model reached an **Adjusted R² = 0.974**, drastically lowering residual MSE and maximizing statistical power.  
  - The p-value associated with Promotion is **p < 0.001**, confirming that the differences in sales across promotions are not due to random chance.
* **Robustness Check (Outlier Sensitivity)**: Removing high-performing outlier stores (`MarketID == 3`) yielded consistent treatment coefficients for Promotion 2, confirming that the overall conclusions are robust against regional outliers.

### 5. Post-Hoc Analysis & Power Diagnostics
* **Post-Hoc Tests**: Pairwise t-tests with **Holm-Bonferroni correction** confirmed:
  * P1 vs P2: p < 0.001 (Statistically significant)
  * P3 vs P2: p < 0.001 (Statistically significant)
  * P1 vs P3: p > 0.05 (No significant difference)
* **Minimum Detectable Effect (MDE)**:
  * At alpha = 0.05$ and Power = 0.80, the experiment design had high enough sensitivity to detect a difference between the best and the worst promotion of 1.7K$ per store (ANOVA Power analysis).
  * At alpha = 0.05$ and Power = 0.80, the minimum difference detectable between any two Promotions is 1.6K$ per store (T-test Power analysis).

---

## Assumptions Verification for ANCOVA test

| Assumption | Diagnostic Tool | Result | Status |
| :--- | :--- | :--- | :--- |
| **Independence** | Experimental Design | Single store per unit | ✅ Passed |
| **Normality of Residuals** | Q-Q Plot & CLT (n > 30) | Residuals follow linear trend | ✅ Passed |
| **Homogeneity of Variance** | Levene's Test | p > 0.05 across groups | ✅ Passed |

---

## Notes on Method & Power Analysis

* **Covariate Selection & Model Fit**: Each added covariate reduces the residual degrees of freedom by 1. While losing a degree of freedom slightly lowers statistical power, controlling for `MarketID` significantly reduces the Sum of Squared Errors (SSE). Because the reduction in residual variance MSE_resid outweighs the loss in degrees of freedom, the overall effect size (f) and statistical power increase dramatically.
* **Pooled Analysis vs. Subgroup Analysis**: Analyzing all stores in a single ANCOVA model was critical rather than splitting the dataset by `MarketSize`. Subsetting would drop the sample size to n=43--45 per segment, severely reducing statistical power. Controlling for `MarketID` as a covariate in the full model (N=137) lowers residual variance while preserving sample size.
* **Data Limitations & Future Improvements**: Having pre-promotion baseline sales per store would have allowed a paired analysis to measure incremental growth relative to each store's baseline, eliminating between-store fixed effects.

---

## 📂 Repository Structure

```text
├── data/
│   └── WA_Marketing-Campaign.csv     # Raw dataset from Kaggle
├── notebooks/
│   └── ab_testing_promotions.ipynb   # Complete Analysis Notebook
├── README.md                         # Project documentation
└── requirements.txt                  # Python dependencies
