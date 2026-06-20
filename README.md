# Waveform signals for ICU outcome prediction

This is the code for a project looking at whether intraoperative and diagnostic ECG/PPG waveforms add anything useful on top of standard clinical data when predicting postoperative and inpatient outcomes. Two datasets: VitalDB (intraop, from Seoul National University Hospital) and MIMIC-IV (US academic hospital, with the linked diagnostic ECG companion set).

## What's here

| Notebook | Dataset | What it predicts |
|---|---|---|
| `01_VitalDB_ICU_Admission.ipynb` | VitalDB | Whether a patient ends up in the ICU after surgery |
| `02_VitalDB_ICU_Stay_Duration.ipynb` | VitalDB | How long they stay if they do (no ICU / short / prolonged) |
| `03_MIMIC-IV_Mortality.ipynb` | MIMIC-IV | In-hospital mortality, XGBoost vs a CNN on raw waveform |

`Methods.md` has the full writeup of how everything was set up and why, including the stuff that didn't work. `Results.md` has all the tables.

## How the modeling is set up

The holdout split always happens first, before any CV or oversampling. 5-fold CV runs on the training set only, mainly to pick a decision threshold and check the model isn't wildly unstable across folds. SMOTE gets applied inside each fold, never before the split, never on the holdout itself. Confidence intervals come from bootstrapping the holdout predictions, 2000 resamples.

For VitalDB the six models per signal go: signal alone, clinical alone, vitals alone, signal+clinical, clinical+vitals, then everything together. For MIMIC-IV it's ECG alone, clinical alone, ECG+clinical, then ECG+clinical+labs since there's no continuous vitals data there the way there is intraoperatively.

## Data you'll need

For the VitalDB notebooks: `clinical_data.csv` (grab it with `vitaldb.load_cases()` or pull straight from `https://api.vitaldb.net/cases`), plus the three feature CSVs (`vitaldb_ppg_features.csv`, `vitaldb_ecg_features.csv`, `vitaldb_hr_spo2.csv`). Extraction code is in notebook 1, commented out by default since it takes a while to run against the API.

For MIMIC-IV: you'll need credentialed PhysioNet access for `admissions.csv.gz`, `patients.csv.gz`, `labevents.csv.gz` from MIMIC-IV core, plus `machine_measurements.csv` and `record_list.csv` from the MIMIC-IV-ECG companion set, and the raw `.dat`/`.hea` waveform files for the CNN part.

## Dependencies

pandas, numpy, scikit-learn, xgboost, imbalanced-learn, matplotlib, torch, wfdb, vitaldb, tqdm, scipy

## TL;DR on what we found

PPG beats ECG by a decent margin for the intraoperative VitalDB outcomes. PPG+clinical performs about the same as clinical+vitals, ECG+clinical does noticeably worse. Some of that gap is probably because we could only get basic waveform stats out of the intraop ECG (tried pulling actual RR/QRS/QT intervals but the extraction yield was too low to trust), whereas MIMIC-IV's ECG comes with proper machine-measured intervals already and performs a lot better as a standalone signal there. The CNN trained directly on raw 12-lead waveforms, once you fuse in clinical and lab data, lands in the same ballpark as the XGBoost model built on hand-engineered ECG intervals.
