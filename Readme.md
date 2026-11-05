# MIS-C length of stay: prediction and risk factors

Multisystem Inflammatory Syndrome in Children (MIS-C) hospitalisations in the US. This project uses count regression models (Poisson and Negative Binomial), with Random Forest and Gradient Boosting for comparison, to identify factors associated with longer hospital stays and to test how well length of stay (LOS) can be predicted.

**Full write-up:** [Report.md](Report.md)

## Key findings

- **LOS is strongly overdispersed** (variance/mean = 26). Negative Binomial fits far better than Poisson, so it is the primary model.
- **Longest stays:** infants under 1 year (+68%), intubation (+58%), ECMO (+40%), dialysis (+39%), immune-modulator treatment (+27%) and severe respiratory involvement (+23%).
- **Delta period:** stays were 10% shorter than before Delta; Omicron did not differ.
- **Race and sex:** no independent association with LOS after adjusting for severity.
- **Prediction is modest:** the NB model has a test MAE of 3.09 days vs 3.41 days for always predicting the median (5 days). A model using only admission information does worse than that baseline.

## Data

- **Source:** CDC, *Multisystem Inflammatory Syndrome in Children (MIS-C) National Surveillance, February 2020 – December 2022* (downloaded 2026-04-07).
- **File:** `misc_c_surveillance.csv` — 9,332 records, 56 variables.
- **Analysis cohort:** 8,795 patients after cleaning (94.2% retained).

## Repository contents

| File | Description |
|---|---|
| `MISC_length_of_stay.ipynb` | Full analysis: exploration, cleaning, modelling, evaluation |
| `misc_c_surveillance.csv` | Raw surveillance data |
| `Report.md` | Methods, results and limitations |
| `Readme.md` | Project overview |

All tables and figures are produced inside the notebook.

## Methods in brief

- Cleaning: removed still-admitted patients (stay unfinished), records with missing LOS, severe same-day discharges (likely transfers) and 3 patients with unknown sex.
- 80/20 train/test split stratified by ICU admission; all encoding and imputation fitted on the training set only.
- Models compared with 5-fold cross-validation on the training set, then evaluated once on the held-out test set.

## Author

Marina Gorikova
