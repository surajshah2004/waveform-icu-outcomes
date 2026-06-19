# Waveform Signals for Perioperative and ICU Outcome Prediction

Evaluates the incremental predictive value of intraoperative and diagnostic physiologic waveform signals (PPG, ECG) over structured clinical and laboratory features, across two independent datasets: VitalDB (intraoperative, Seoul National University Hospital) and MIMIC-IV (US academic hospital, with linked diagnostic ECG).

## Notebooks

| Notebook | Dataset | Outcome | Description |
|---|---|---|---|
| `01_VitalDB_ICU_Admission.ipynb` | VitalDB | Postoperative ICU admission (binary) | Six-model comparison (ECG and PPG) with full waveform extraction pipeline |
| `02_VitalDB_ICU_Stay_Duration.ipynb` | VitalDB | ICU stay duration (ordinal, 3-class) | Six-model comparison (ECG and PPG), macro-averaged AUC-ROC/AUC-PR |
| `03_MIMIC-IV_Mortality.ipynb` | MIMIC-IV | In-hospital mortality (binary) | XGBoost on ECG intervals + clinical + labs, and CNN on raw 12-lead waveforms |

## Methodology Summary

- **Holdout-first design:** all 80/20 stratified holdout splits are constructed before any cross-validation, SMOTE application, or hyperparameter selection.
- **5-fold stratified cross-validation** on the 80% training set, used for model evaluation and decision threshold selection only.
- **SMOTE applied within training folds only** — never prior to a train/validation split, never on holdout data.
- **Bootstrap confidence intervals (n=2,000)** computed directly on holdout predictions for AUC-ROC and AUC-PR.
- **Consistent model comparison structure:** for VitalDB, all six-model series follow Signal-only → Clinical-only → Vitals-only → Signal+Clinical → Clinical+Vitals → Full. For MIMIC-IV: ECG-only → Clinical-only → ECG+Clinical → ECG+Clinical+Labs.

See `Methods.md` and `Results.md` for the full write-up, including limitations and data exclusions (e.g., VitalDB in-hospital mortality was not modeled due to insufficient event count).

## Required Data Files

**VitalDB notebooks** require (place in working directory or update paths):
- `clinical_data.csv` — VitalDB clinical/preop data (downloadable via `vitaldb.load_cases()` or `https://api.vitaldb.net/cases`)
- `vitaldb_ppg_features.csv`, `vitaldb_ecg_features.csv`, `vitaldb_hr_spo2.csv` — extracted via the waveform extraction cells in Notebook 1 (commented out by default; uncomment to re-extract from the VitalDB API, or supply pre-extracted CSVs)

**MIMIC-IV notebook** requires (credentialed PhysioNet access):
- `mimic-iv-3.1/hosp/admissions.csv.gz`, `mimic-iv-3.1/hosp/patients.csv.gz`, `mimic-iv-3.1/hosp/labevents.csv.gz`
- `machine_measurements.csv` and `record_list.csv` (MIMIC-IV-ECG companion dataset)
- Raw waveform `.dat`/`.hea` files (MIMIC-IV-ECG, for the CNN section)

## Dependencies

```
pandas numpy scikit-learn xgboost imbalanced-learn shap matplotlib torch wfdb vitaldb tqdm scipy
```

## Key Findings

- PPG morphology features outperform ECG morphology features for intraoperative ICU outcome prediction in VitalDB; PPG+Clinical matches Clinical+Vitals performance, while ECG+Clinical performs significantly worse.
- ECG clinical interval measurements (MIMIC-IV) substantially outperform ECG morphology statistics (VitalDB) as a standalone signal, attributed to feature extraction methodology rather than the underlying clinical task — see Methods for discussion of why automated interval extraction failed on intraoperative VitalDB recordings.
- A CNN trained directly on raw 12-lead ECG waveforms, fused with clinical and laboratory features, achieves performance statistically comparable to a structured XGBoost model using clinically validated ECG intervals (overlapping 95% CIs).
