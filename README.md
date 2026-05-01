# Perioperative Cardiac Risk Prediction Using Intraoperative Hemodynamic Data

## Overview
Complete analytical pipeline for:

> Machine Learning Prediction of Perioperative Cardiac Complications Using Intraoperative Hemodynamic Data: A Multi-Database Analysis of the MOVER and MIMIC-IV Datasets

## Key Findings
- Model 2 (XGBoost + intraoperative features) achieved AUC 0.823 (95% CI 0.776-0.868)
- Outperformed RCRI benchmark (AUC 0.671, 95% CI 0.615-0.726)
- Intraoperative heart rate trajectory ranked above all comorbidity flags in SHAP analysis
- Temporally validated AUC 0.682 exceeded RCRI benchmark
- Comparative MIMIC-IV analysis revealed context-dependent risk factor dominance

## Data Access
### MOVER
- URL: https://mover.ics.uci.edu/download.html
- Free, no institutional affiliation required

### MIMIC-IV
- URL: https://physionet.org/content/mimiciv/
- Free, requires PhysioNet credentialing

## Requirements
pip install -r requirements.txt

## Reproducibility
All random seeds set to 42.

## Authors
Suraj Shah
[Co-author name] — Cornell

## License
MIT
