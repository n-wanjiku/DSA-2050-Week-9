# DSA 2050A — Week 9: Model Selection and Defensible Evaluation

**Group name/number:** Group 7
**Members and student IDs:** Mark Mayana - 669482, Lesala Monaheng - , Nicholas Wanjiku - , 
**Submission:** One shared GitHub repository link per group  
**Course:** DSA 2050A Data Science Methodology  
**Week:** 9

## Group Responsibilities

The lab was completed collaboratively using one shared notebook and repository. Record each member's concrete contribution below.

| Group member | Student ID | Work completed | GitHub username or commit reference |
|---|---|---|---|
| Name | Student ID | Contribution | Evidence |
|---|---|---|---|
| Mark Mayana | 669482 | Data inspection, leakage exclusions, leakage-safe preprocessing pipeline, stratified 5-fold CV setup (Steps 1–3) | `maryanarh` — commits: "first comit", "2nd commit", "3rd commit", "4th commit" |
| Lesala Monaheng | 669[___] | Hyperparameter tuning for logistic regression and decision tree, experiment log, out-of-fold confusion matrix and error analysis (Steps 4–7) | [GitHub username] — commit: "Add model tuning, experiment log and OOF error analysis" |
| Nicholas Wanjiku | 676114 | Subgroup analysis, permutation importance, selection rationale, frozen selection and one-time final test, README and repo final checks (Steps 8–10 + reflection) | n-wanjiku — commits: "Added subgroup analysis and permutation importance", "Freeze selection, run final test and write README" ||

## Study Objective

The objective was to develop and compare classification models for predicting customer churn. The lab focused on selecting a defensible model using development data, evaluating its errors, examining subgroup performance, and assessing its ability to generalize to unseen data.

Model selection was performed using cross-validation on development data only. The final test set was reserved for evaluating the selected model and was not used for further tuning.

## Data Statement

The dataset contains synthetic, fictional customer records created for teaching purposes. It does not contain real customer information.

The prediction task is to classify whether a customer will churn based on available customer and service characteristics.

## Setup and Reproduction

1. Open this repository folder in VS Code.
2. Create or select a Python environment (Python 3.10–3.13 recommended).
3. Open **Terminal → New Terminal**.
4. Install the required dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

5. Open `notebooks/Week9_Model_Selection_Lab.ipynb`.
6. Select the appropriate Python kernel.
7. Run the notebook cells in order. Complete the model-selection rationale before executing the final-test evaluation.

## Prediction-Time Features and Excluded Fields

- **Target:** `churn`
  - `1` = Customer churned.
  - `0` = Customer stayed.

- **Excluded fields:**
  - `customer_id`: An identifier rather than a meaningful behavioural predictor. Including it could encourage the model to learn irrelevant patterns.
  - `cancellation_confirmed`: Excluded because it contains information that becomes available after the churn outcome and would introduce target leakage.

- **Allowed prediction features:**
  - `tenure_months`
  - `monthly_charges_kes`
  - `support_tickets_90d`
  - `service_calls_90d`
  - `plan_type`
  - `region`
  - `autopay`
  - `customer_segment`

These features describe customer tenure, service usage, support interactions, payment arrangements, and customer characteristics that can be available when making a prediction.

## Experiments and Model Selection

- **Primary evaluation metric:** F1-score.
- **Cross-validation:** 5-fold stratified cross-validation.
- **Shuffle:** `True`.
- **Random seed:** `42`.

### Candidate Model Comparison

The following results were obtained from the experiment log. F1 values and standard deviations are shown to three decimal places.

| Model | Hyperparameter | Mean F1 | SD F1 |
|---|---|---:|---:|
| Logistic Regression | C = 1 | 0.470 | 0.053 |
| Logistic Regression | C = 10 | 0.470 | 0.053 |
| Logistic Regression | C = 0.1 | 0.465 | 0.054 |
| Decision Tree | Depth = 3 | 0.398 | 0.049 |
| Decision Tree | Depth = 5 | 0.382 | 0.023 |
| Decision Tree | Depth = 8 | 0.372 | 0.051 |

### Selected Model

- **Chosen model:** Logistic Regression.
- **Selected hyperparameter:** `C = 1` (confirm against `reports/frozen_selection.json`).
- **Mean cross-validation F1:** `0.4701`.
- **Standard deviation:** Approximately `0.053`.

### Model Selection Rationale

Logistic Regression with C = 1 was selected because it achieved a mean five-fold cross-validation F1-score of approximately 0.4701, outperforming the best Decision Tree, which achieved a mean F1-score of 0.398. This indicates that Logistic Regression provided a better balance between precision and recall when identifying customers likely to churn. Its standard deviation of approximately 0.053 indicated some variation across the validation folds. Although the Decision Tree with a maximum depth of 3 had slightly lower variation, its average predictive performance was weaker. Logistic Regression was also preferred because its coefficients provide a relatively interpretable way of examining relationships between customer characteristics and churn predictions. C = 1 and C = 10 produced identical results at the displayed precision; C = 1 is recorded as the selected configuration and should agree with the frozen selection file. The decision was based on development cross-validation results rather than final-test performance.

## Error and Subgroup Analysis

### Out-of-Fold Error Analysis

The out-of-fold evaluation produced the following results:

| Prediction outcome | Count |
|---|---:|
| Correct predictions | 1,949 |
| False negatives | 566 |
| False positives | 205 |
| Total observations | 2,720 |

A false negative occurs when a customer actually churns but is predicted to stay. A false positive occurs when a customer stays but is incorrectly predicted to churn.

False negatives are particularly important because they represent missed opportunities for the provider to intervene and retain customers. False negatives accounted for approximately 73.4% of the 771 incorrect predictions.

### Subgroup Recall Analysis

| Customer subgroup | Sample size | Actual churners | Recall | Precision | False negatives |
|---|---:|---:|---:|---:|---:|
| Established customers | 1,143 | 212 | 0.2028 | 0.6143 | 169 |
| New customers | 443 | 299 | 0.8763 | 0.7120 | 37 |

Recall was substantially lower for established customers than for new customers. This suggests that the model was considerably more effective at identifying churn among new customers. However, the groups have different sample sizes and churn rates. This is a diagnostic finding that warrants further investigation rather than a causal explanation.

## Permutation Importance

The permutation importance analysis identified the following features:

| Feature | Mean importance | Standard deviation |
|---|---:|---:|
| `tenure_months` | 0.079 | 0.024 |
| `support_tickets_90d` | 0.035 | 0.016 |
| `customer_segment` | 0.018 | 0.006 |
| `monthly_charges_kes` | 0.016 | 0.023 |
| `plan_type` | 0.015 | 0.015 |
| `region` | 0.006 | 0.013 |
| `service_calls_90d` | 0.003 | 0.007 |
| `autopay` | 0.002 | 0.017 |

The two most important features were `tenure_months` (mean importance 0.079) and `support_tickets_90d` (mean importance 0.035). These results indicate that the selected model relied most heavily on customer tenure and recent support-ticket activity when making predictions.

### Limitations of Permutation Importance

- **Correlated features:** When predictors contain similar information, their importance may be distributed across correlated variables, making individual features appear less influential.
- **No causal interpretation:** Permutation importance measures predictive reliance, not causation. A high importance score does not prove that changing a feature would directly change customer churn.

## Final Test Evaluation

The selected Logistic Regression model was evaluated on the held-out test set containing 680 observations.

| Metric | Final test result |
|---|---:|
| Test observations | 680 |
| Accuracy | 0.7000 |
| F1-score | 0.4486 |
| Precision | 0.5845 |
| Recall | 0.3640 |

### Final-Test Confusion Matrix

Rows represent actual classes and columns represent predicted classes.

| | Predicted stay (0) | Predicted churn (1) |
|---|---:|---:|
| Actual stay (0) | 393 | 59 |
| Actual churn (1) | 145 | 83 |

The model correctly identified 393 customers who stayed and 83 customers who churned. It incorrectly classified 145 actual churners as customers who would stay (false negatives) and generated 59 false-positive predictions.

### Cross-Validation vs Final-Test Performance

| Evaluation | F1-score |
|---|---:|
| Mean cross-validation F1 | 0.4701 |
| Final-test F1 | 0.4486 |
| Difference (test − CV) | -0.0215 |

The final-test F1-score was approximately 0.0215 lower than the cross-validation mean. This relatively small decrease suggests that the model generalized reasonably well to unseen observations, although its performance was slightly weaker on the test set. The final-test recall of 0.3640 also indicates that the model identified only a proportion of actual churners. Improving churn detection should therefore remain a priority in future development.

The final-test results were used for evaluation and were not used to retune the model.

## Limitations and Ethics

1. **Synthetic dataset:** The records may not accurately represent real customer behaviour, churn patterns, or relationships between customer characteristics in an actual telecommunications environment.
2. **Limited representation of customer dynamics:** The dataset may not adequately capture changing customer preferences, seasonal behaviour, market conditions, and other factors that influence churn over time.
3. **Unequal subgroup performance:** The considerable difference in recall between new and established customers raises concerns about whether the model performs equally well across customer groups.

Before deploying a real churn prediction system, the model should be validated using actual customer data and a time-based holdout. Subgroup performance and probability calibration should also be examined. Human review should remain part of the retention process, and the intervention threshold should be determined using the financial costs of missed churners and unnecessary retention campaigns. Model performance should also be monitored regularly to detect changes in customer behaviour and potential model drift.

## Evidence Checklist

- [✔️] Executed notebook with explanations.
- [✔️] `reports/experiment_log.csv`
- [✔️] `reports/subgroup_metrics.csv`
- [✔️] `reports/permutation_importance.csv`
- [✔️] `reports/frozen_selection.json` created before test evaluation.
- [✔️] `reports/final_metrics.json` generated once.
- [✔️] Two generated PNG charts.
- [✔️] Meaningful commits or co-authorship evidence for all group members.
- [✔️] Completed group contribution table with names and student IDs.

## Final Group Review

- [✔️] One team member verified that the notebook runs from start to finish.
- [✔️] The group discussed and approved the written model-selection rationale.
- [✔️] No one used final-test scores to revise the model.
- [✔️] One representative will submit the shared repository URL to the course LMS.

## GitHub Repository URL

https://github.com/maryanarh/DSA-2050-Week-9
