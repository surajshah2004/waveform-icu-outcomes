# Methods

## Overview

This study evaluates the incremental predictive value of intraoperative and diagnostic physiologic waveform signals (photoplethysmography [PPG] and electrocardiography [ECG]) over structured clinical and laboratory features for postoperative and in-hospital outcome prediction. Two independent datasets were used: VitalDB, an intraoperative database from Seoul National University Hospital, and MIMIC-IV, a US academic ICU/inpatient database with a linked diagnostic ECG companion dataset. Across both datasets, models were built incrementally from waveform-only, to clinical-only, to combined feature sets, with rigorous holdout-based evaluation throughout.

---

## Dataset 1: VitalDB

### Cohort and Outcomes

VitalDB contains intraoperative case data from 6,388 noncardiac surgical patients, including preoperative clinical variables, intraoperative vital sign trends, and high-fidelity physiologic waveforms (ECG Lead II, PPG, arterial line) sampled during surgery. Two outcomes were modeled:

1. **Postoperative ICU admission (binary):** defined as any ICU stay duration greater than zero days following surgery.
2. **ICU stay duration (ordinal, 3-class):** defined as no ICU admission (0 days), short stay (1–3 days), or prolonged stay (4+ days). This ordinal cutoff was chosen because exploratory analysis of the stay-duration distribution showed that 68% of ICU admissions (815 of 1,204) lasted exactly one day, consistent with routine postoperative monitoring rather than significant clinical deterioration. The 4+ day cutoff was chosen to isolate cases of genuine prolonged critical illness from routine overnight observation.

**In-hospital mortality was not modeled in VitalDB.** The `death_inhosp` field contains only 57 events out of 6,388 cases (0.89% prevalence). An 80/20 holdout split would yield approximately 11 deaths in the holdout set, which is too few to support stable bootstrap confidence intervals or reliable model evaluation. This is a limitation of the dataset's case mix (a largely elective noncardiac surgical population with low baseline mortality) rather than a methodological choice to avoid the outcome. ICU admission and ICU stay duration were selected instead because their event rates (18.8% and the 1/2/3-class split) provide adequate statistical power for holdout-based evaluation.

### Feature Sets

- **Clinical features (19 variables):** age, sex, BMI, ASA physical status classification, emergency surgery flag, preoperative hypertension, preoperative diabetes, preoperative ECG abnormality flag (derived from the categorical `preop_ecg` field, coded 1 if any rhythm other than "Normal Sinus Rhythm" was documented), and nine preoperative laboratory values (hemoglobin, platelet count, prothrombin time, activated partial thromboplastin time, sodium, potassium, glucose, albumin, creatinine, BUN, AST, ALT). Missingness in clinical features ranged from 0% to 9.8% and was handled via median imputation, computed on the training set only and applied to the holdout set to avoid leakage.

- **Vitals features (6 variables):** intraoperative heart rate (mean, minimum, maximum, standard deviation) and SpO2 (mean, minimum), extracted from minute-resolution Solar8000 monitor data (`Solar8000/HR`, `Solar8000/PLETH_SPO2`) via the VitalDB API.

- **PPG features (11 variables):** statistical morphology features (mean, median, standard deviation, interquartile range, range, skewness, kurtosis, energy, mean absolute difference, minimum, maximum) computed on raw PPG waveform amplitude (`SNUADC/PLETH`) after clipping to the 1st–99th percentile to remove extreme artifacts.

- **ECG features (11 variables):** the same 11 statistical morphology features computed on raw ECG Lead II waveform amplitude (`SNUADC/ECG_II`), using identical clipping and extraction methodology to PPG for direct comparability.

### Why Simple Morphology Statistics Were Used Instead of Clinical Interval Measurements

An attempt was made to extract standard clinical ECG interval measurements (RR interval, PR interval, QRS duration, QT interval) from the raw VitalDB ECG waveforms using automated peak detection and wave delineation (neurokit2 library). This approach failed to produce a usable feature set: full wave delineation (P/QRS/T wave boundaries) succeeded on only 1,601 of 6,355 cases (25% yield), and a simplified R-peak-only detection approach succeeded on only 1,377 cases (22% yield). Both yields are too low to support modeling without introducing substantial selection bias, as the subset of cases with successfully extracted intervals would not be representative of the full surgical cohort. This failure is attributed to the noise characteristics of intraoperative ECG recordings, which are subject to electrocautery interference, patient motion, and lead artifact to a much greater degree than diagnostic-quality ECG recordings obtained in inpatient or outpatient settings. Given this constraint, simple statistical morphology features (extractable on 6,346 of 6,355 cases, 99.8% yield) were used instead for both ECG and PPG. This methodological choice is revisited in the Discussion section in light of the cross-dataset comparison with MIMIC-IV, where true clinical intervals were available and yielded a different pattern of results (see below).

### Model Architecture and Validation

All VitalDB models used gradient-boosted decision trees (XGBoost), with the following hyperparameters held constant across all models: 300 estimators, maximum tree depth of 3, learning rate of 0.03, subsample ratio of 0.8, and column subsample ratio of 0.8.

For each outcome and feature combination, an 80/20 stratified holdout split was performed first, before any cross-validation or model fitting, to ensure the holdout set was never used in any stage of model development or hyperparameter selection. Within the 80% training set, 5-fold stratified cross-validation was performed for model evaluation and decision threshold selection. The Synthetic Minority Oversampling Technique (SMOTE) was applied strictly within each training fold to correct class imbalance; SMOTE was never applied prior to the train/validation split within a fold, and never applied to the holdout set, to prevent synthetic oversampling from leaking information across the train/test boundary. The classification decision threshold was selected by maximizing F1 score on the pooled out-of-fold cross-validation predictions, then applied unchanged to the holdout set for final evaluation.

Final models for holdout evaluation were retrained on the full 80% training set (post-SMOTE) using the same hyperparameters. Performance was evaluated on the untouched 20% holdout set using area under the receiver operating characteristic curve (AUC-ROC) and area under the precision-recall curve (AUC-PR), both computed via bootstrap resampling (n=2,000 iterations) to obtain 95% confidence intervals. For the ordinal stay-duration outcome, AUC-ROC and AUC-PR were computed as one-vs-rest macro averages across the three outcome classes, giving equal weight to the rare prolonged-stay class (only 3% prevalence) rather than allowing it to be dominated by the common no-ICU class.

### Model Comparison Structure

To isolate the incremental contribution of each signal type, six models were fit for each outcome: (1) waveform signal only (PPG or ECG), (2) clinical features only, (3) vitals only, (4) signal + clinical, (5) clinical + vitals, (6) the full combination of signal + clinical + vitals. This structure allows direct comparison of: each signal's standalone discriminative power; each signal's added value when combined with clinical features alone; a head-to-head comparison of signal+clinical against vitals+clinical, holding clinical features constant; and the incremental value of adding vitals on top of signal+clinical, or adding the signal on top of clinical+vitals. Statistical significance for all incremental comparisons (deltas) was assessed via bootstrap resampling of the holdout predictions (n=2,000), with significance defined as a 95% confidence interval that excludes zero.

This consistent ordering (signal-only → clinical-only → signal+clinical → full) was adopted to mirror the structure used for the MIMIC-IV analysis (ECG-only → clinical-only → ECG+clinical → ECG+clinical+labs), where vitals plays the analogous structural role to laboratory values as the final incremental feature block. An earlier iteration of this analysis added vitals before the waveform signal in the model sequence, which obscured the waveform signal's standalone and combined contribution; this was identified as an inconsistency and corrected before finalizing the reported results.

### ECG vs. PPG Comparison

Because ECG and PPG feature extraction succeeded on slightly different subsets of cases (6,346 vs. 6,151 respectively, due to independent extraction failures for each signal type), the ECG-based and PPG-based six-model series were run on their respective full available cohorts rather than the intersection of cases with both signals. This maximizes statistical power for each individual analysis but means that the "clinical only" and "vitals only" rows are not numerically identical between the ECG and PPG tables, despite using the same feature definitions — the underlying patient cohort differs slightly between the two cohorts. A secondary, separate analysis was also performed on the intersection cohort (cases with both ECG and PPG successfully extracted, N=6,346) to directly test whether combining both signals provides additional value beyond either alone.

---

## Dataset 2: MIMIC-IV

### Cohort and Outcome

MIMIC-IV is a US academic hospital database from Beth Israel Deaconess Medical Center, linked here to its companion diagnostic ECG dataset (machine-measured 12-lead ECGs with automated interval annotations). The outcome modeled was in-hospital mortality (`hospital_expire_flag`).

The analytic cohort was constructed as follows: all diagnostic ECGs (n=800,035 records) were linked to hospital admissions by subject ID, retaining only ECGs that fell within the admission's documented start and end times (n=292,435 ECG-admission pairs). For admissions with multiple qualifying ECGs, the first chronological ECG was retained, yielding one ECG per admission (n=143,647 admissions). Mortality prevalence in this linked cohort was 3.6% (5,180 deaths).

To address this class imbalance, a balanced 1:1 cohort was constructed via random undersampling of the majority class (survivors), matching the 5,180 deaths with an equal number of randomly sampled survivors (random seed fixed for reproducibility), yielding a final balanced cohort of N=10,360. This is a standard approach for imbalanced binary classification problems and is reported transparently as a limitation: the balanced cohort's prevalence (50%) does not reflect the true population mortality rate (3.6%), meaning that threshold-dependent metrics (F1, precision, recall) reported for this cohort should not be interpreted as representative of real-world deployment performance. AUC-ROC and AUC-PR, which are computed across all classification thresholds, are not affected by this rebalancing in the same way and remain the primary reported metrics.

### Feature Sets

- **ECG interval features (13 variables):** RR interval, PR interval, QRS duration, QT interval, heart rate (derived as 60000/RR interval), P axis, QRS axis, T axis, and the four underlying wave boundary timestamps (P onset, P end, QRS onset, QRS end, T end) from which the derived intervals were calculated. These are automated machine measurements provided directly in the MIMIC-IV-ECG companion dataset (`machine_measurements.csv`), not features extracted by the authors. All 13 features had 0% missingness in the linked cohort.

- **Clinical features (5 variables):** age (`anchor_age`), gender, admission type, insurance type, and race, all from the MIMIC-IV admissions and patients tables.

- **Laboratory features (8 variables):** the first recorded value after admission for BUN, creatinine, glucose, hematocrit, hemoglobin, platelet count, potassium, and sodium, extracted from the MIMIC-IV `labevents` table and linked by admission ID. Albumin was excluded from the final feature set due to 47% missingness in the balanced cohort, which was judged too high for reliable imputation; the remaining eight laboratory features had missingness ranging from 8.1% to 9.2% and were median-imputed.

### Why ECG Interval Features Outperformed VitalDB's Morphology Features

Because MIMIC-IV's ECG features are clinically validated machine measurements of cardiac timing (e.g., QT interval, QRS duration) rather than raw waveform statistical summaries, the standalone ECG-only model achieved a substantially higher holdout AUC-ROC (0.736) than VitalDB's ECG-only model using morphology statistics (0.644) on a comparable binary classification task. This supports the interpretation, discussed above, that the type of feature extraction (clinically meaningful interval timing vs. general waveform shape statistics) materially affects a signal's standalone predictive value, and that VitalDB's lower ECG performance reflects a feature engineering limitation imposed by intraoperative signal noise rather than an inherent lack of information in the ECG signal itself.

### Model Architecture and Validation

The same XGBoost hyperparameters, holdout/cross-validation structure, SMOTE-within-folds procedure, and bootstrap confidence interval methodology described for VitalDB were used for MIMIC-IV, with one structural difference: MIMIC-IV models followed the sequence ECG-only → clinical-only → ECG+clinical → ECG+clinical+labs, as VitalDB's "vitals" category has no direct analogue in MIMIC-IV (which lacks continuous intraoperative monitoring data); laboratory values served as the final incremental feature block instead.

### Convolutional Neural Network Models

In addition to the XGBoost analysis on machine-measured ECG intervals, a series of convolutional neural network (CNN) models were trained directly on raw 12-lead ECG waveform data (10-second recordings sampled at 500 Hz, 5,000 samples per lead) for the same balanced mortality cohort and the same 80/20 holdout split (held identical across all CNN and XGBoost models to allow direct comparison). Four CNN variants were evaluated in order of increasing input complexity: (1) Lead II only (single-channel input), (2) all 12 leads (12-channel input), (3) 12-lead waveform fused with the 5 clinical features via a late-fusion architecture, and (4) 12-lead waveform fused with both clinical and laboratory features (13 total tabular features).

All CNN models used a ResNet-style 1D convolutional architecture: an initial stem convolution (kernel size 15, stride 2) followed by three residual blocks with channel widths 32→64→128→256 (stride 2 per block), global average pooling, dropout (rate 0.5), and a final linear classification layer. For the multimodal variants, tabular features (clinical, or clinical+labs) were processed through a separate small fully-connected branch (two linear layers with ReLU activation) and concatenated with the pooled convolutional features before the final classification layer. All models were trained using binary cross-entropy with logits loss, the Adam optimizer (learning rate 1e-3), a learning rate scheduler that reduced the rate on validation plateau, and early stopping (patience of 4 epochs without improvement in validation AUC-ROC, maximum 20 epochs). The same 5-fold cross-validation and holdout evaluation procedure used for the XGBoost models was applied to each CNN variant, with one model retrained from scratch per fold.

### MIMIC-III Waveform CNN: Abandoned Analysis

An earlier exploratory attempt was made to train a CNN on continuous ICU bedside monitor waveforms from MIMIC-III for in-hospital mortality prediction (3,237 patients, 277 deaths, 8.6% event rate). This analysis was abandoned after multiple architecture and training configuration attempts consistently showed early overfitting (validation AUC peaking at epoch 1–2 around 0.62–0.69, then declining as training loss approached zero), attributed to a mismatch between the number of available unique patients and the number of training windows generated per patient, which allowed the model to memorize patient-specific noise patterns rather than learn generalizable features. The final reported holdout AUC for this abandoned line of analysis was 0.620 (95% CI 0.534–0.711), which is not included in the final results as it does not meet the bar for a reportable finding and was superseded by the MIMIC-IV diagnostic ECG CNN analysis described above, which used a substantially larger and cleaner cohort.

---

## Dataset 3: MOVER (Reported Separately)

A third dataset, MOVER (a perioperative database from UC Irvine), was used for a related but separate analysis (Paper 1, targeting JAMIA Open) comparing an XGBoost model on preoperative clinical features (AUC 0.825) against the Revised Cardiac Risk Index (RCRI, AUC 0.671) for predicting perioperative cardiac complications. This analysis is methodologically independent of the VitalDB/MIMIC-IV waveform comparison described above and is not included in the present results.

An exploratory extension of the MOVER analysis incorporating intraoperative ECG and arterial line waveform morphology features was also attempted, using OR-duration-based matching to link de-identified waveform files to surgical cases (mean matching error approximately 8 minutes). This analysis was not included in the final results: the cohort with successfully linked and matched waveform features was small (885 unique patients, 114 with the cardiac outcome), and the addition of waveform features did not improve performance over clinical and vital sign features alone in a properly leakage-free, patient-level holdout evaluation (waveform-augmented model holdout AUC 0.556 vs. 0.721 for clinical+vitals alone). This negative result is attributed to the limited sample size relative to the rarity and clinical heterogeneity of the cardiac complication outcome, rather than a genuine absence of signal in the waveform data, and is noted here for completeness but excluded from the primary results given its inconclusive nature.

---

## General Methodological Principles Applied Across All Analyses

1. **Holdout sets were always constructed before any cross-validation, SMOTE application, or hyperparameter selection**, and were never used for any purpose other than final, single-pass performance evaluation.
2. **SMOTE was always applied within training folds only**, never prior to a train/validation split, to prevent synthetic minority samples from leaking information between training and evaluation data.
3. **All confidence intervals were computed via bootstrap resampling (n=2,000) directly on holdout predictions**, not on cross-validation predictions, per explicit guidance to ensure the reported uncertainty reflects genuine generalization performance.
4. **Decision thresholds for F1/precision/recall were selected using cross-validation out-of-fold predictions only**, then applied unchanged to the holdout set, to avoid threshold selection bias from the holdout data.
5. **Median imputation for missing features was always computed on the training set only** and applied identically to the holdout set, to prevent information leakage through imputation statistics.
6. **Where multiple surgeries or admissions existed for the same patient (MOVER only)**, train/test and cross-validation splits were performed at the patient level (via GroupKFold/grouped splitting) rather than the case level, to prevent the same patient from appearing in both training and evaluation data.

---

## Limitations

- VitalDB's ECG morphology features (statistical summaries of raw waveform shape) are not directly comparable to MIMIC-IV's ECG interval features (clinically validated machine measurements of cardiac timing); the two analyses should be interpreted as testing related but distinct hypotheses about waveform utility, not as a fully harmonized cross-dataset replication.
- VitalDB in-hospital mortality was not modeled due to insufficient event count (57 deaths, 0.89% prevalence) for reliable holdout-based evaluation.
- MIMIC-IV's balanced 1:1 cohort does not reflect true population mortality prevalence (3.6%); precision, recall, and F1 metrics reported for this cohort are not directly generalizable to real-world deployment without recalibration.
- Race was included as a clinical/demographic feature in the MIMIC-IV models and appeared among the top SHAP-ranked predictors in the full multimodal model. This is interpreted as the model capturing socioeconomic and healthcare-access-related signal rather than a biological effect of race itself, and raises fairness considerations that would need to be addressed (e.g., via stratified fairness analysis) prior to any clinical application of this model.
- VitalDB's intraoperative ECG and PPG morphology feature extraction did not yield viable clinical interval measurements (RR, PR, QRS, QT) despite attempted automated wave delineation, due to signal noise characteristics of intraoperative recordings; this limited the ECG feature set to general waveform statistics rather than clinically interpretable interval measurements.
- The MOVER waveform-augmented analysis, while not included in the primary results, suggests that intraoperative waveform morphology features may require substantially larger cohorts than were available in that dataset to demonstrate reliable incremental predictive value for rare, heterogeneous outcomes such as perioperative cardiac complications.
