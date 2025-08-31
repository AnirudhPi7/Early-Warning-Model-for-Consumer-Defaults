# Early Warning Model for Consumer Defaults

## Introduction
This project develops an auditable early-warning scorecard that estimates default risk at loan application time, motivated by elevated delinquency in India’s small-ticket personal credit segment and the operational need to reduce loss provisions. The modeling draws on the Home-Credit Default Risk corpus whose demographics, income, bureau history, and installment attributes mirror what Indian digital lenders obtain through CKYC, bureau pulls, and bank-statement parsing, allowing transferrable insights for NBFC workflows.

---

## Background
A transparent credit model must balance statistical rigor with regulatory scrutiny: features require careful cleaning and imputation, class imbalance must be controlled, and diagnostics must demonstrate parsimony, calibration, and discrimination. The study therefore pairs a regularized logistic scorecard—favored for interpretability and auditability—with a stronger non-linear benchmark in Random Forests to test whether incremental complexity materially improves ranking and classification without sacrificing stability.

---

## Problem Statement
The aim is to estimate the probability of default at origination and to translate scores into accept or refer decisions with thresholds that align to business loss tolerance, while answering which borrower attributes drive risk and whether the final specification satisfies a complete diagnostic template for model governance and NBFC deployment.

---

## Dataset
After removing identifiers and imputing missing monetary fields within occupation buckets, the working sample retains 219,831 complete cases with a binary target indicating default within 720 days of origination; the raw corpus broadly reflects lender-available fields but requires extensive cleaning before modeling

---

## Data Preparation
Invalid categorical codes such as the spurious “XNA” label and blank IDs are dropped, columns with more than half their values missing are removed, and high-cardinality categoricals like occupation and organization types are collapsed into business-meaningful buckets; the default minority is handled by random undersampling to equalize classes, remaining categoricals are one-hot encoded with first-level drop, numerics are winsorized at the 99.5th percentile to curb outliers, and gaps are imputed using a five-nearest-neighbour estimator to preserve multivariate structure.

---

## Methodology
The baseline classifier is a binary logistic regression estimated by maximum likelihood after reducing dimensionality with a mutual-information filter that retains the twenty most informative predictors; coefficients are then shrunk with an ℓ² penalty via fit_regularized(L1_wt=0.0) to control variance without zeroing parameters. A non-linear benchmark is introduced through a Random Forest trained with class weighting and tuned by randomized search over depth, splits, leaf size, feature subsampling, and number of trees, followed by model-based feature selection that proves fifty predictors can match full-feature performance; the final classification threshold is chosen on validation probabilities to maximize recall under a precision ≥ 0.60 constraint.

---

## Diagnostics
Evaluation on a held-out test fold emphasizes parsimony, calibration, and discrimination: information criteria improve versus unpruned candidates, residual deviance falls, and Nagelkerke R² reaches about 0.21; the Hosmer–Lemeshow statistic with ten risk deciles yields χ² = 8.05 (p = 0.428), indicating excellent calibration across the probability spectrum, while ROC AUC for the logistic scorecard is 0.73, evidencing useful rank separation for frontline decisions.

---

## Results & Discussion
The logistic model exhibits clear score separation between defaulters and non-defaulters and returns economically sensible coefficient signs under a decisive block-wise Wald test; at the F1-selected threshold, overall correct classification approaches two-thirds with balanced type-I and type-II errors. The tuned Random Forest improves validation AUC to roughly 0.742, maintains performance with a trimmed set of fifty features, and, when thresholded at 0.389 to satisfy the precision constraint, delivers recall of 0.867 and precision of 0.604 on the test set, a trade-off aligned with the lender’s priority to minimize missed defaulters.

---
## Conclusion & Future Scope
A carefully engineered pipeline—spanning aggressive cleaning, MI-filtered L2-regularized logistic regression, and a tuned Random Forest with recall-oriented thresholding—produces calibrated, interpretable, and operationally robust default risk estimates suitable for NBFC deployment. Next steps include profit-based cut-off optimization and scenario simulation, expanded stability testing across time and cohorts, and exploration of calibrated gradient-boosting variants to push AUC without sacrificing auditability or calibration quality.

---
