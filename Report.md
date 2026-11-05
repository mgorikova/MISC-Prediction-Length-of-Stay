# Length of hospital stay in MIS-C: risk factors and prediction

## 1. Aim

To identify patient and clinical factors associated with longer hospital stays in children with Multisystem Inflammatory Syndrome in Children (MIS-C), and to assess how accurately length of stay (LOS) can be predicted.

## 2. Data

CDC national MIS-C surveillance data, February 2020 – December 2022: **9,332 records** and **56 variables**, covering demographics, census region, onset month, organ involvement, treatments, ICU care and outcome.

### Data cleaning (9,332 → 8,795 patients, 94.2% retained)

| Step | Patients removed |
|---|---:|
| Still admitted (LOS incomplete, right-censored) | 366 |
| Unknown outcome with missing LOS | 99 |
| Remaining missing LOS | 50 |
| Severe same-day discharge (ICU or shock, LOS = 0; likely inter-facility transfer) | 19 |
| Sex unknown (too few to estimate) | 3 |
| Negative LOS | 0 |

- **Columns removed:** three vaccination variables (>80% missing) and five identifier or near-constant columns.
- **Race:** four indicator columns combined into one variable.
- **Census region:** 248 records (2.7%) were suppressed by CDC for privacy.
- **Variant period** derived from onset month: Pre-Delta (before July 2021), Delta (July–November 2021), Omicron (from December 2021).
- **Unknown outcome:** the 100 patients with a recorded LOS were kept; their LOS matched discharged patients (median 5 days, Mann–Whitney p = 0.28).

## 3. Patient characteristics

| | No ICU | ICU | Overall |
|---|---:|---:|---:|
| Patients | 3,815 | 4,980 | 8,795 |
| LOS, median [IQR], days | 4 [3–5] | 6 [4–9] | 5 [4–7] |
| Age 5–11 years | 44.2% | 47.6% | 46.1% |
| Male | 59.7% | 60.6% | 60.2% |
| Black race | 22.9% | 33.7% | 29.0% |
| Obesity | 21.1% | 26.3% | 24.0% |
| Shock | 3.4% | 69.8% | 41.0% |
| Severe cardiac involvement | 55.4% | 91.6% | 75.9% |
| Died | 0.0% | 1.5% | 0.9% |

The full Table 1, with p-values, is in the notebook.

## 4. Methods

- **Outcome:** hospital LOS in days (a count).
- **Model choice:** LOS variance is 26 times its mean, so a Poisson model (which assumes variance = mean) is inappropriate. **Negative Binomial (NB)** regression was the primary model; results are reported as incidence rate ratios (IRR), where IRR > 1 means a longer stay.
- **Predictors (40):** age group, sex, race, ethnicity, census region, variant period, comorbidities, organ involvement, ICU care, shock, respiratory support and treatments. Demographic variables were kept regardless of significance as standard confounders.
- **Validation:** 80/20 train/test split (train n = 7,036, test n = 1,759), stratified by ICU admission. Reference categories, encoding and imputation were fitted on the training set only.
- **Comparison models:** Poisson, NB with ICU × shock and ICU × cardiac interactions, Random Forest and Gradient Boosting, compared by 5-fold cross-validated MAE on the training set.

## 5. Results

### 5.1 Factors associated with LOS (Negative Binomial)

| Factor | IRR (95% CI) | Interpretation |
|---|---|---|
| Age < 1 year (vs 5–11) | 1.68 (1.55–1.83) | 68% longer stay |
| Intubation / mechanical ventilation | 1.58 (1.48–1.69) | 58% longer |
| ECMO | 1.40 (1.23–1.58) | 40% longer |
| Dialysis | 1.39 (1.19–1.61) | 39% longer |
| Treated with immune modulators | 1.27 (1.22–1.32) | 27% longer |
| Severe respiratory involvement | 1.23 (1.18–1.28) | 23% longer |
| Severe neurologic involvement | 1.19 (1.11–1.28) | 19% longer |
| Other comorbidities | 1.18 (1.13–1.24) | 18% longer |
| Severe gastrointestinal involvement | 1.17 (1.13–1.21) | 17% longer |
| ICU level of care | 1.08 (1.03–1.13) | 8% longer |
| Obesity | 1.04 (1.01–1.08) | 4% longer |
| Delta period (vs Pre-Delta) | 0.90 (0.87–0.94) | 10% shorter |
| Antiplatelet treatment | 0.87 (0.84–0.91) | 13% shorter |

- **No independent association:** sex, race, Hispanic ethnicity, Omicron period, shock and chronic lung disease (all p > 0.05).
- **Shock sensitivity check:** removing shock changed AIC by only −1.3; shock overlaps with ICU care and intubation, and the main results do not depend on it.
- The full IRR table for all 40 predictors is in the notebook.

### 5.2 Model comparison (test set, n = 1,759)

| Model | CV MAE (± SD) | Test MAE | Within ±5 days |
|---|---|---:|---:|
| **Negative Binomial** | **3.049 ± 0.329** | **3.090** | **90.3%** |
| NB + interactions | 3.055 ± 0.321 | 3.092 | 90.2% |
| Poisson | 3.067 ± 0.326 | 3.098 | 90.2% |
| Random Forest | 3.103 ± 0.326 | 3.128 | 89.1% |
| Gradient Boosting | 3.211 ± 0.342 | 3.397 | 87.7% |

- Negative Binomial had the lowest error on both cross-validation and the test set, and is the most interpretable.
- All models generalised well (train–test MAE gap 0.08–0.47 days).
- Test RMSE (12.5 days) is much larger than MAE because of a small number of very long stays (up to 378 days).

### 5.3 How predictable is LOS?

| Model | Test MAE (days) | Spearman correlation |
|---|---:|---:|
| Baseline: always predict median (5 days) | 3.41 | — |
| NB, admission information only | 3.75 | 0.22 |
| NB, full model (in-hospital variables) | 3.09 | 0.55 |

- The full model improves on the baseline by only about 9%.
- Using only information available at admission (demographics, region, variant period, comorbidities), the model performs **worse** than the baseline. Most of the predictive information comes from what happens during the stay.

## 6. Conclusions

1. Negative Binomial regression is the appropriate model for LOS in MIS-C because of strong overdispersion.
2. Infants under 1 year and children needing organ support (ventilation, ECMO, dialysis) or immune-modulator therapy had the longest stays.
3. Stays were shorter during the Delta period, possibly reflecting improved recognition and management of MIS-C.
4. After adjusting for clinical severity, race and sex were not associated with LOS.
5. The model is useful for **explaining** which factors are linked to longer stays, but not for **predicting** an individual child's stay, especially at admission.

## 7. Limitations

- **In-hospital predictors:** ICU care, ventilation and treatments occur during the stay; their associations partly reflect that longer stays allow more interventions (reverse causation), and they cannot be used for prediction at admission.
- **Deaths kept in the outcome:** for the 75 children who died, LOS is time to death rather than to recovery (a competing event).
- **Census region:** 248 suppressed values were imputed with the most common region (South), which may bias regional estimates.
- **Race categories:** the dataset records only White, Black, Other and Unknown, with one category per patient, so finer groups (for example Asian or multiracial) cannot be examined.
- **Surveillance data:** case reporting and completeness vary by jurisdiction and over time.
- **Transfers:** LOS reflects the reporting hospital only; children transferred between hospitals may have incomplete stays.

## 8. Reproducibility

All analysis is in `MISC_length_of_stay.ipynb`. Random seeds are fixed (`random_state = 42`), so results are reproducible with **Run All**.
