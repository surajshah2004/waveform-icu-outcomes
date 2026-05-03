# Perioperative Cardiac Risk Prediction Using Intraoperative Hemodynamic Data

Code and analysis pipeline for:

**Machine Learning Prediction of Perioperative Cardiac Complications Using 
Intraoperative Physiological Data: A Multi-Database Analysis of the MOVER 
and MIMIC-IV Datasets**

Suraj Shah, BA — University of California, Berkeley  
Suhina Chand, BS — University of California, Irvine

---

## What This Paper Does

Existing perioperative cardiac risk tools like the RCRI rely entirely on 
information collected before surgery. This project asks a simple question: 
does what actually happens during the operation tell us anything useful?

Using 55,301 noncardiac surgical encounters from UC Irvine Medical Center 
(MOVER) and a parallel analysis on 70,488 admissions from MIMIC-IV, we 
trained machine learning models that incorporate intraoperative heart rate 
and oxygen saturation alongside standard preoperative features. We then used 
SHAP to understand which features actually drove predictions and compared 
those patterns across two very different patient populations.

---

## Key Results

- XGBoost model with intraoperative features: **AUC 0.823** (95% CI 0.776–0.868)
- Preoperative-only logistic regression: AUC 0.753
- RCRI benchmark on the same cohort: AUC 0.671
- Temporal validation (trained 2017–2020, tested 2021–2023): AUC 0.682
- Mean intraoperative heart rate ranked 3rd in SHAP importance, above all individual comorbidity flags including prior MI, CHF, and CAD
- In MIMIC-IV (higher acuity, 13.22% event rate), prior MI dominated — suggesting the relative importance of intraoperative physiology vs. preoperative history depends heavily on patient acuity

---

## Data Access

Both datasets are publicly available and free to access.

**MOVER** (UC Irvine Medical Center, 2017–2023)  
https://mover.ics.uci.edu/download.html  
No institutional affiliation required.

**MIMIC-IV v3.1** (Beth Israel Deaconess Medical Center, 2008–2022)  
https://physionet.org/content/mimiciv/  
Requires PhysioNet credentialing and CITI training completion.

Raw data files are not included in this repository and should not be 
redistributed. File paths in the notebooks reflect local directory 
structure and will need to be updated to match your setup.

---

## Repository Structure

notebooks/
  periop_cardiac_analysis.ipynb   — main MOVER analysis, model development, SHAP
  mimic_analysis.ipynb            — parallel MIMIC-IV analysis
  supplementary_analyses.ipynb    — temporal validation, subgroup analysis, calibration

figures/
  figure1_roc_curves.png
  figure2_calibration.png
  figure3_shap_bar_mover.png
  figure4_shap_beeswarm_mover.png
  figure5_shap_comparison.png
  figure6_shap_beeswarm_mimic.png
  figure7_dca.png
  figure8_subgroup_forest.png
  figure_s1_pdp_hr.png

tables/
  table1_demographics.csv
  table2_model_performance.csv

---

## Requirements

pip install -r requirements.txt

Core dependencies: pandas, numpy, scikit-learn, xgboost, shap, matplotlib, 
scipy, dcurves. All random seeds set to 42 throughout.

---

## Reproducibility

All analyses were run in Python 3.11 on Windows using Anaconda. Random seeds 
are set to 42 at every applicable step. Given the size of MOVER's flowsheet 
file (~40 million rows), intraoperative feature extraction requires chunked 
processing and may take 15–30 minutes depending on hardware.

---

## License

MIT
