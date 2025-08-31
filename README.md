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

The primary learners are tree ensembles tuned for high recall at an operational precision floor, with the logistic scorecard retained as a transparent benchmark. We train a class-weighted Random Forest under a stratified 5-fold scheme, using randomized search over depth, number of trees, min samples per split/leaf, and feature subsampling; out-of-bag estimates provide an additional cross-check for generalization while permutation importance is used to verify that economic drivers (affordability ratios, credit history, tenure) dominate spurious artifacts. In parallel, we fit an XGBoost classifier on the same engineered matrix, computing scale_pos_weight from class balance and tuning depth, learning rate, trees, min_child_weight, subsample, colsample_bytree, and L1/L2 regularization with early stopping on a validation fold using AUC-PR as the selection metric. Because ensemble probabilities are often miscalibrated, the selected model is passed through a post-hoc calibration step (isotonic or Platt) trained on the validation set, after which we choose an operating threshold from the precision-recall curve to maximize recall subject to a precision ≥ 0.60; this directly reflects business priorities to minimize missed defaulters while controlling false approvals. Final model choice between RF and XGB is based on calibrated recall at the precision floor, stability of feature importance across folds, and operational footprint; whichever champion is selected is packaged in a scikit-learn Pipeline that encapsulates preprocessing, imputation, and the estimator, enabling reproducible scoring, drift monitoring, and straightforward challenger swaps in production.

---

## Feature Engineering
We begin by hardening the raw application, bureau, and installment fields into model-ready signals that behave well statistically and are defensible in audit. Invalid labels (such as “XNA”) and columns with pervasive missingness are removed, while monetary and duration variables are winsorized at the 99.5th percentile to curb leverage from input errors; where distributions are strictly positive and extremely skewed, a log transform is applied after winsorization to stabilize variance without distorting rank order. High-cardinality categoricals (for example, occupation or organization types) are collapsed into business-meaningful buckets and one-hot encoded with a dropped reference level to avoid dummy traps; remaining gaps in continuous features are imputed using a five-nearest-neighbour scheme that preserves local multivariate structure. To express borrower capacity and behaviour in more economic terms, we derive ratios and deltas—payment-to-income, credit-to-income, utilization proxies, ageing/tenure bins, and simple recency indicators from repayment history—while guarding against leakage by restricting all engineered fields to information available at application time. Finally, we screen features with a mutual-information filter to remove near-noise predictors, then use model-based importance from a preliminary Random Forest run to confirm that a trimmed set (≈50 predictors) retains nearly all of the predictive power, improving stability and train time without sacrificing accuracy.

---

## Results & Discussion
The logistic model exhibits clear score separation between defaulters and non-defaulters and returns economically sensible coefficient signs under a decisive block-wise Wald test; at the F1-selected threshold, overall correct classification approaches two-thirds with balanced type-I and type-II errors. The tuned Random Forest improves validation AUC to roughly 0.742, maintains performance with a trimmed set of fifty features, and, when thresholded at 0.389 to satisfy the precision constraint, delivers recall of 0.867 and precision of 0.604 on the test set, a trade-off aligned with the lender’s priority to minimize missed defaulters.

---
## Conclusion & Future Scope
A carefully engineered pipeline—spanning aggressive cleaning, MI-filtered L2-regularized logistic regression, and a tuned Random Forest with recall-oriented thresholding—produces calibrated, interpretable, and operationally robust default risk estimates suitable for NBFC deployment. Next steps include profit-based cut-off optimization and scenario simulation, expanded stability testing across time and cohorts, and exploration of calibrated gradient-boosting variants to push AUC without sacrificing auditability or calibration quality.

---
