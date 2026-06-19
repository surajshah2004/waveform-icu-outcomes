# Results

## VitalDB — ICU Admission: Waveform Signal Contribution (ECG vs PPG)

**ECG cohort:** N=6,346. **PPG cohort:** N=6,151. Both 80/20 stratified holdout, 5-fold cross-validation, SMOTE applied within training folds only.

| Model | ECG — AUC-ROC (95% CI) | ECG — AUC-PR (95% CI) | PPG — AUC-ROC (95% CI) | PPG — AUC-PR (95% CI) |
|---|---|---|---|---|
| Signal only | 0.6437 (0.6045–0.6840) | 0.3143 (0.2589–0.3700) | 0.7161 (0.6770–0.7537) | 0.3914 (0.3238–0.4585) |
| Clinical only | 0.7611 (0.7285–0.7927) | 0.4719 (0.4106–0.5338) | 0.7777 (0.7443–0.8094) | 0.4762 (0.4077–0.5410) |
| Vitals only | 0.7438 (0.7085–0.7787) | 0.4446 (0.3792–0.5124) | 0.6914 (0.6511–0.7306) | 0.3465 (0.2834–0.4084) |
| Signal + Clinical | 0.7927 (0.7604–0.8239) | 0.5152 (0.4509–0.5790) | 0.8405 (0.8129–0.8658) | 0.5444 (0.4779–0.6098) |
| Clinical + Vitals | 0.8243 (0.7968–0.8522) | 0.5561 (0.4947–0.6219) | 0.8213 (0.7912–0.8506) | 0.5550 (0.4895–0.6207) |
| Full (Signal + Clinical + Vitals) | 0.8417 (0.8145–0.8677) | 0.5874 (0.5219–0.6521) | 0.8528 (0.8256–0.8771) | 0.5853 (0.5201–0.6484) |

**Key incremental comparisons (bootstrap delta AUC-ROC, 95% CI):**

| Comparison | ECG | PPG |
|---|---|---|
| Signal+Clinical over Clinical alone | +0.0315 (significant) | +0.0639 (significant) |
| Signal+Clinical over Signal alone | +0.1482 (significant) | +0.1245 (significant) |
| Signal+Clinical vs Vitals+Clinical (head-to-head) | −0.0321 (significant, ECG worse) | +0.0200 (not significant, equivalent) |
| Full over Signal+Clinical (vitals' added value) | +0.0493 (significant) | +0.0122 (not significant) |
| Full over Clinical+Vitals (signal's added value) | +0.0173 (significant) | +0.0327 (significant) |

---

## VitalDB — ICU Stay Duration (Ordinal): Waveform Signal Contribution (ECG vs PPG)

**ECG cohort:** N=6,346. **PPG cohort:** N=6,151. Outcome: 0 = No ICU, 1 = Short Stay (1–3 days), 2 = Prolonged Stay (4+ days). Macro-averaged, one-vs-rest AUC-ROC and AUC-PR across all three classes.

| Model | ECG — Macro AUC-ROC (95% CI) | ECG — Macro AUC-PR (95% CI) | PPG — Macro AUC-ROC (95% CI) | PPG — Macro AUC-PR (95% CI) |
|---|---|---|---|---|
| Signal only | 0.6330 (0.5893–0.6722) | 0.4007 (0.3790–0.4238) | 0.7113 (0.6734–0.7479) | 0.4436 (0.4184–0.4710) |
| Clinical only | 0.7753 (0.7438–0.8046) | 0.5876 (0.5359–0.6388) | 0.7747 (0.7348–0.8103) | 0.5714 (0.5035–0.6335) |
| Vitals only | 0.7372 (0.6959–0.7760) | 0.4794 (0.4384–0.5261) | 0.7421 (0.6973–0.7831) | 0.4871 (0.4307–0.5482) |
| Signal + Clinical | 0.7933 (0.7609–0.8214) | 0.6059 (0.5498–0.6585) | 0.8183 (0.7778–0.8549) | 0.5939 (0.5286–0.6606) |
| Clinical + Vitals | 0.8370 (0.8070–0.8625) | 0.6446 (0.5889–0.6947) | 0.8213 (0.7822–0.8534) | 0.6035 (0.5310–0.6691) |
| Full (Signal + Clinical + Vitals) | 0.8453 (0.8157–0.8723) | 0.6512 (0.5940–0.7011) | 0.8457 (0.8073–0.8783) | 0.6177 (0.5494–0.6888) |

**Key incremental comparisons (bootstrap delta macro AUC-ROC, 95% CI):**

| Comparison | ECG | PPG |
|---|---|---|
| Signal+Clinical over Clinical alone | +0.0186 (significant) | +0.0441 (significant) |
| Signal+Clinical over Signal alone | +0.1611 (significant) | +0.1073 (significant) |
| Signal+Clinical vs Vitals+Clinical (head-to-head) | −0.0442 (significant, ECG worse) | −0.0032 (not significant, equivalent) |
| Full over Signal+Clinical (vitals' added value) | +0.0519 (significant) | +0.0268 (significant) |
| Full over Clinical+Vitals (signal's added value) | +0.0079 (not significant, borderline) | +0.0240 (significant) |

---

## VitalDB — ECG vs PPG vs Combined Signal (ICU Admission, Intersection Cohort)

**Cohort:** N=6,346 cases with both ECG and PPG successfully extracted. 80/20 stratified holdout, 5-fold cross-validation, SMOTE within folds.

| Model | AUC-ROC (95% CI) | AUC-PR (95% CI) | F1 | Precision | Recall |
|---|---|---|---|---|---|
| Clinical only | 0.7304 (0.6938–0.7679) | 0.4152 | 0.3886 | 0.2845 | 0.6132 |
| Clinical + Vitals | 0.7949 (0.7618–0.8267) | 0.4910 | 0.4627 | 0.3827 | 0.5849 |
| Clinical + Vitals + ECG | 0.8128 (0.7809–0.8428) | 0.4991 | 0.4909 | 0.4281 | 0.5755 |
| Clinical + Vitals + PPG | 0.8427 (0.8172–0.8678) | 0.5526 | 0.5257 | 0.4524 | 0.6274 |
| Clinical + Vitals + PPG + ECG | 0.8458 (0.8213–0.8702) | 0.5481 | 0.5403 | 0.4486 | 0.6792 |

**Key finding:** PPG morphology features are a significantly stronger predictor of ICU admission than ECG morphology features (delta AUC-ROC, PPG over ECG: +0.0296, 95% CI +0.0118 to +0.0473, significant). Combining both signals provides no additional benefit over PPG alone (delta AUC-ROC, combined over PPG: +0.0032, 95% CI −0.0057 to +0.0124, not significant), indicating redundancy between the two waveform signals once PPG is included.

---

## MIMIC-IV — In-Hospital Mortality: XGBoost (ECG Intervals + Clinical + Labs)

**Cohort:** Balanced 1:1 cohort, N=10,360 (5,180 deaths, 5,180 survivors). 80/20 stratified holdout, 5-fold cross-validation, SMOTE within folds.

| Model | CV AUC-ROC | Holdout AUC-ROC (95% CI) | Holdout AUC-PR (95% CI) | F1 | Precision | Recall |
|---|---|---|---|---|---|---|
| ECG only | 0.7423 | 0.7357 (0.7141–0.7578) | 0.7136 (0.6822–0.7441) | 0.7082 | 0.6092 | 0.8456 |
| Clinical only | 0.7434 | 0.7406 (0.7191–0.7608) | 0.7017 (0.6694–0.7332) | 0.7386 | 0.6073 | 0.9421 |
| ECG + Clinical | 0.8066 | 0.8014 (0.7826–0.8200) | 0.7738 (0.7460–0.8003) | 0.7601 | 0.6481 | 0.9189 |
| ECG + Clinical + Labs | 0.8473 | 0.8463 (0.8296–0.8616) | 0.8280 (0.8037–0.8507) | 0.7826 | 0.7008 | 0.8861 |

**Key incremental comparisons (bootstrap delta AUC-ROC, 95% CI):**
- ECG+Clinical over ECG alone: +0.0660 (0.0516 to 0.0797), significant
- ECG+Clinical over Clinical alone: +0.0609 (0.0428 to 0.0785), significant
- Labs over ECG+Clinical: +0.0448 (0.0322 to 0.0577), significant

---

## MIMIC-IV — In-Hospital Mortality: Convolutional Neural Network

**Cohort:** Same balanced 1:1 cohort, N=10,360, linked to raw 10-second 12-lead ECG waveforms (500 Hz, 5,000 samples/lead). 80/20 stratified holdout (identical split to the XGBoost analysis), 5-fold cross-validation. ResNet-style 1D CNN architecture (see Methods).

| Model | CV AUC-ROC | Holdout AUC-ROC (95% CI) | Holdout AUC-PR (95% CI) |
|---|---|---|---|
| Lead II only | 0.7702 ± 0.0127 | 0.7780 (0.7573–0.7969) | 0.7419 (0.7105–0.7724) |
| 12-lead | 0.7838 ± 0.0165 | 0.7844 (0.7647–0.8032) | 0.7675 (0.7375–0.7954) |
| 12-lead + Clinical (multimodal) | 0.7901 ± 0.0147 | 0.7968 (0.7773–0.8152) | 0.7703 (0.7393–0.7997) |
| 12-lead + Clinical + Labs (full multimodal) | 0.8447 ± 0.0124 | 0.8403 (0.8232–0.8555) | 0.8185 (0.7917–0.8440) |

**Key finding:** The full multimodal CNN (raw 12-lead waveform + clinical + laboratory features, holdout AUC-ROC 0.8403) achieved performance statistically comparable to the structured-feature XGBoost model using the same feature categories (ECG intervals + clinical + labs, holdout AUC-ROC 0.8463) — the 95% confidence intervals for the two models overlap substantially (0.8232–0.8555 vs. 0.8296–0.8616). This indicates the CNN, learning directly from raw waveform morphology with no hand-engineered interval measurements, matches the performance of a model built on clinically validated interval features.

---

## Cross-Dataset Comparison: ECG Signal Performance

| Analysis | Dataset | Feature Type | ECG-Only Holdout AUC-ROC |
|---|---|---|---|
| ICU admission | VitalDB | Statistical morphology (intraoperative) | 0.6437 |
| ICU stay duration | VitalDB | Statistical morphology (intraoperative) | 0.6330 |
| In-hospital mortality | MIMIC-IV | Clinical interval measurements (diagnostic) | 0.7357 |

The substantially higher standalone ECG performance in MIMIC-IV is attributed to the difference in feature extraction methodology rather than a difference in the underlying clinical task: MIMIC-IV's ECG features are clinically validated machine measurements of cardiac timing (RR, PR, QRS, QT intervals), while VitalDB's ECG features are general statistical summaries of waveform shape, necessitated by the failure of automated interval extraction on noisy intraoperative recordings (see Methods, "Why Simple Morphology Statistics Were Used").

---

## Summary of Primary Findings

1. **PPG outperforms ECG as a standalone and combined intraoperative signal** for both ICU admission and ICU stay duration prediction in VitalDB. PPG+Clinical matches the performance of Clinical+Vitals (no significant difference), while ECG+Clinical performs significantly worse than Clinical+Vitals for both outcomes.
2. **Both ECG and PPG provide significant incremental value over a full clinical+vitals baseline** for ICU admission. For the more difficult ordinal ICU stay duration outcome, PPG's incremental value remains significant while ECG's becomes borderline non-significant.
3. **Combining ECG and PPG provides no additional benefit over PPG alone**, indicating redundancy between the two signals once the stronger signal (PPG) is included.
4. **In MIMIC-IV, ECG and clinical features are complementary and roughly equally predictive in isolation** (AUC-ROC 0.7357 vs 0.7406), but their combination yields a significant improvement over either alone, and laboratory values add further significant incremental value.
5. **A CNN trained directly on raw 12-lead ECG waveforms, fused with clinical and laboratory features, achieves performance statistically comparable to a structured XGBoost model using clinically validated ECG interval measurements**, demonstrating that deep learning on raw waveforms can substitute for hand-engineered interval features without a meaningful loss in discriminative performance.
