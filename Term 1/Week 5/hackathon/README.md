# Mortgage denial risk prediction for housing counselling

This project builds and evaluates three machine learning classifiers to predict mortgage application denials using public records from the Home Mortgage Disclosure Act (FFIEC HMDA). The model is designed for a non-profit housing counselling organisation to help allocate pre-application advisory sessions to prospective buyers at high risk of loan rejection.

---

## Problem definition

Applying for a home purchase mortgage requires non-refundable expenditures on application fees, credit checks, and property appraisals. When a loan is denied, applicants lose these funds, forfeit earnest money deposits, and lose purchase contracts on homes. 

Under the Home Mortgage Disclosure Act, lenders must report the disposition of mortgage applications alongside financial and property characteristics. In Delaware in 2023, 15.4% of completed home purchase applications were denied. Historical denial rates show wide gaps across demographics: Black applicants faced a 23.6% denial rate compared to 11.6% for White applicants, and Hispanic applicants faced a 22.7% denial rate compared to 13.5% for non-Hispanic applicants.

This project addresses United Nations Sustainable Development Goal 8 (Decent Work and Economic Growth), specifically Target 8.10: strengthening the capacity of domestic financial institutions to encourage and expand access to banking, insurance, and financial services for all. Predicting denial risk before an applicant submits a formal application enables early financial coaching, budgeting adjustments, and credit remediation.

---

## User group and operational decision

The direct user of the model is a housing counselling agency. Counsellors use the prediction during intake consultations to identify which clients need structured pre-application interventions.

The affected population consists of low- and moderate-income prospective homebuyers seeking guidance before approaching a commercial lender. 

The primary error asymmetry is clear:
* A false negative occurs when an applicant at risk of denial is classified as safe. The applicant receives no extra coaching, applies to a lender, and suffers a loan rejection, losing time and non-refundable fees.
* A false positive occurs when an applicant who would have been approved is flagged as at risk. The applicant receives unnecessary counselling. This expends staff capacity, but does not harm the applicant.

Because false negatives carry the greater cost to the applicant, the primary evaluation metric is recall on the denied class. To prevent the model from gaming recall by flagging every applicant, candidate configurations must also maintain a precision equal to or higher than a simple debt-to-income heuristic.

---

## Dataset card

* Data source: Consumer Financial Protection Bureau (CFPB) / Federal Financial Institutions Examination Council (FFIEC) HMDA Data Browser API.
* URL: `https://ffiec.cfpb.gov/v2/data-browser-api/view/csv?states=DE&years=2023&actions_taken=1,3&loan_purposes=1`
* Data collector: Federal regulatory reporting collected annually from commercial lending institutions under federal statute.
* Temporal and geographic scope: State of Delaware, calendar year 2023.
* Population: Home purchase mortgage applications resulting in loan origination (action code 1) or loan denial (action code 3). Applications withdrawn or closed for incompleteness were excluded at retrieval.
* License: US Federal Government Public Domain data (open access).
* Volume: 15,762 unique applications after removing 13 duplicate entries.
* Target variable: `denied` (1 for denied applications, 0 for originated loans). Class distribution: 13,334 approved (84.6%), 2,428 denied (15.4%).
* Features used (14 pre-application variables):
  * Continuous: `loan_amount`, `property_value`, `income`, `loan_term`, `tract_to_msa_income_percentage`.
  * Categorical: `loan_type`, `lien_status`, `occupancy_type`, `construction_method`, `total_units`, `preapproval`, `submission_of_application`, `conforming_loan_limit`, `debt_to_income_ratio`.
* Variables set aside for fairness auditing: `derived_sex`, `derived_race`, `derived_ethnicity`, `applicant_age`.
* Known limitations: Public HMDA files redact applicant credit scores and precise credit histories for privacy reasons. Liquid asset reserves and appraisal details are also absent.

---

## Pipeline design and leakage prevention

To prevent post-decision data leakage, columns were evaluated for missingness differentials and code distributions between approved and denied loans. Fields such as interest rates, loan costs, HOEPA status, denial reason codes, and purchaser types only exist after underwriting decisions are made; all such fields were removed.

The workflow follows strict separation rules:
1. Exact duplicates were dropped before splitting.
2. An 80/20 stratified split (`test_size=0.2`, `random_state=42`) separated 12,609 training cases from 3,153 test cases. The test partition was kept untouched until final evaluation.
3. Imputation, log transforms, and scaling were contained within a scikit-learn `Pipeline` and `ColumnTransformer`. Monetary columns were imputed with medians, log-transformed (`log1p`), and standardized. Categorical columns were imputed with modal values and encoded via `OneHotEncoder(handle_unknown="ignore")`.
4. Hyperparameter tuning ran across identical 5-fold stratified splits (`StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`).
5. The optimization objective selected the highest mean cross-validation recall among settings that achieved or exceeded the precision of the baseline debt-to-income heuristic (0.479).

---

## Model comparison

All models were evaluated on the held-out test partition of 3,153 applications (including 485 denials).

| Model | Best parameters | Train recall | CV recall | CV precision | CV F1 | Test recall | Test precision | Test F1 |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Dummy baseline | Most frequent class | 0.000 | 0.000 ± 0.000 | 0.000 ± 0.000 | 0.000 ± 0.000 | 0.000 | 0.000 | 0.000 |
| Simple rule | DTI $\ge 50\%$ | 0.466 | - | - | - | 0.468 | 0.477 | 0.472 |
| K-nearest neighbours | $k=1$, uniform weights | 0.998 | 0.563 ± 0.019 | 0.587 ± 0.015 | 0.575 ± 0.017 | 0.555 | 0.562 | 0.558 |
| Logistic regression | $C=10$, balanced weights | 0.703 | 0.696 ± 0.046 | 0.494 ± 0.018 | 0.578 ± 0.028 | 0.699 | 0.472 | 0.564 |
| Decision tree | Depth 10, leaf size 50, balanced | 0.726 | 0.688 ± 0.049 | 0.511 ± 0.030 | 0.585 ± 0.026 | 0.693 | 0.492 | 0.575 |

Test set performance matched cross-validation estimates within 0.01 across models, confirming that pre-processing remained isolated. K-nearest neighbours exhibited severe overfitting, collapsing from 0.998 training recall to 0.555 test recall.

Logistic regression and the decision tree achieved comparable test recall (0.699 and 0.693), each detecting approximately 110 more at-risk applicants than the debt-to-income rule alone.

---

## Recommended model

The decision tree (`max_depth=10`, `min_samples_leaf=50`, `class_weight='balanced'`) is recommended for operational use.

The decision tree matches logistic regression in identifying vulnerable applicants (catching 336 of 485 denials in the test set) while providing practical advantages:
* It generates 32 fewer false alarms on the test set (347 compared to 379 for logistic regression).
* It produces a lower disparity in false alarms across racial subgroups. Logistic regression flags approved Black applicants at 1.5 times the rate of approved White applicants (0.194 vs 0.127). The decision tree reduces this gap to 1.2 times (0.150 vs 0.127).
* Its splits function as explicit conditional rules, allowing counsellors to explain to an applicant why a flag was raised.

Feature importance in the decision tree is led by debt-to-income ratios, tract-to-MSA income percentages, loan amounts, and property values.

---

## Ethical evaluation and conditions of use

### Demographic representation and unobserved variables
The dataset represents mortgage applicants in Delaware in 2023. Certain groups, such as Native American applicants and applicants under 25 years old, represent fewer than 100 entries in the data. Subgroup evaluation is therefore statistically limited for these groups.

Because protected demographic attributes were kept out of model inputs, disparities enter the model through correlated financial variables, including neighbourhood income ratios and loan sizes.

### Error analysis
Denied applicants missed by the model have a median income of 100,000 dollars, larger loan amounts, and low debt-to-income ratios. These applicants were rejected by lenders for reasons unrecorded in HMDA data, such as credit scores, poor credit histories, or failed home inspections. Missed applicants were also four times more likely to lack debt-to-income values in their records (15% vs 4%). 

### Operational rules and guardrails
The model is governed by the following deployment requirements:
* It must be used exclusively to offer voluntary assistance. It must never be used to deny services, discourage applications, or gate access to housing programs.
* Counsellors must never tell an applicant they are guaranteed loan approval. Because credit scores are absent, approximately 30% of actual denials are missed by the system.
* The model must not be applied outside Delaware or to non-purchase loan categories without full re-training and re-auditing on local data.

---

## Environment and reproduction

### Dependencies
The analysis requires Python 3.10 or later and the following packages:
* `pandas` >= 2.0.0
* `numpy` >= 1.24.0
* `scikit-learn` >= 1.3.0
* `matplotlib` >= 3.7.0

The script automatically fetches the required raw HMDA data from the CFPB API, saves it to a local `data/` directory, runs all cross-validation sweeps, prints comparison tables, and displays demographic fairness checks.