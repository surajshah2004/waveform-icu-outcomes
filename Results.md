# Results

## VitalDB, ICU admission: ECG vs PPG

ECG cohort N=6,346, PPG cohort N=6,151. Both 80/20 holdout, 5-fold CV, SMOTE inside folds only.

| Model | ECG AUC-ROC (95% CI) | ECG AUC-PR (95% CI) | PPG AUC-ROC (95% CI) | PPG AUC-PR (95% CI) |
|---|---|---|---|---|
| Signal only | 0.6437 (0.6045-0.6840) | 0.3143 (0.2589-0.3700) | 0.7161 (0.6770-0.7537) | 0.3914 (0.3238-0.4585) |
| Clinical only | 0.7611 (0.7285-0.7927) | 0.4719 (0.4106-0.5338) | 0.7777 (0.7443-0.8094) | 0.4762 (0.4077-0.5410) |
| Vitals only | 0.7438 (0.7085-0.7787) | 0.4446 (0.3792-0.5124) | 0.6914 (0.6511-0.7306) | 0.3465 (0.2834-0.4084) |
| Signal + Clinical | 0.7927 (0.7604-0.8239) | 0.5152 (0.4509-0.5790) | 0.8405 (0.8129-0.8658) | 0.5444 (0.4779-0.6098) |
| Clinical + Vitals | 0.8243 (0.7968-0.8522) | 0.5561 (0.4947-0.6219) | 0.8213 (0.7912-0.8506) | 0.5550 (0.4895-0.6207) |
| Full (signal + clinical + vitals) | 0.8417 (0.8145-0.8677) | 0.5874 (0.5219-0.6521) | 0.8528 (0.8256-0.8771) | 0.5853 (0.5201-0.6484) |

What it adds, bootstrap delta AUC-ROC:

| Comparison | ECG | PPG |
|---|---|---|
| Signal+clinical vs clinical alone | +0.0315 (sig) | +0.0639 (sig) |
| Signal+clinical vs signal alone | +0.1482 (sig) | +0.1245 (sig) |
| Signal+clinical vs vitals+clinical | -0.0321 (sig, ECG worse) | +0.0200 (not sig, roughly even) |
| Full vs signal+clinical (what vitals add) | +0.0493 (sig) | +0.0122 (not sig) |
| Full vs clinical+vitals (what signal adds) | +0.0173 (sig) | +0.0327 (sig) |

---

## VitalDB, ICU stay length: ECG vs PPG

Same cohorts as above. Outcome: 0 = no ICU, 1 = short stay (1-3 days), 2 = prolonged (4+ days). AUC-ROC/PR are macro-averaged one-vs-rest across the three classes.

| Model | ECG macro AUC-ROC (95% CI) | ECG macro AUC-PR (95% CI) | PPG macro AUC-ROC (95% CI) | PPG macro AUC-PR (95% CI) |
|---|---|---|---|---|
| Signal only | 0.6330 (0.5893-0.6722) | 0.4007 (0.3790-0.4238) | 0.7113 (0.6734-0.7479) | 0.4436 (0.4184-0.4710) |
| Clinical only | 0.7753 (0.7438-0.8046) | 0.5876 (0.5359-0.6388) | 0.7747 (0.7348-0.8103) | 0.5714 (0.5035-0.6335) |
| Vitals only | 0.7372 (0.6959-0.7760) | 0.4794 (0.4384-0.5261) | 0.7421 (0.6973-0.7831) | 0.4871 (0.4307-0.5482) |
| Signal + Clinical | 0.7933 (0.7609-0.8214) | 0.6059 (0.5498-0.6585) | 0.8183 (0.7778-0.8549) | 0.5939 (0.5286-0.6606) |
| Clinical + Vitals | 0.8370 (0.8070-0.8625) | 0.6446 (0.5889-0.6947) | 0.8213 (0.7822-0.8534) | 0.6035 (0.5310-0.6691) |
| Full | 0.8453 (0.8157-0.8723) | 0.6512 (0.5940-0.7011) | 0.8457 (0.8073-0.8783) | 0.6177 (0.5494-0.6888) |

Deltas:

| Comparison | ECG | PPG |
|---|---|---|
| Signal+clinical vs clinical alone | +0.0186 (sig) | +0.0441 (sig) |
| Signal+clinical vs signal alone | +0.1611 (sig) | +0.1073 (sig) |
| Signal+clinical vs vitals+clinical | -0.0442 (sig, ECG worse) | -0.0032 (not sig, roughly even) |
| Full vs signal+clinical | +0.0519 (sig) | +0.0268 (sig) |
| Full vs clinical+vitals | +0.0079 (not sig, basically a coin flip) | +0.0240 (sig) |

---

## VitalDB, ECG + PPG combined (intersection cohort, ICU admission)

N=6,346 cases where both ECG and PPG extraction succeeded.

| Model | AUC-ROC (95% CI) | AUC-PR | F1 | Precision | Recall |
|---|---|---|---|---|---|
| Clinical only | 0.7304 (0.6938-0.7679) | 0.4152 | 0.3886 | 0.2845 | 0.6132 |
| Clinical + Vitals | 0.7949 (0.7618-0.8267) | 0.4910 | 0.4627 | 0.3827 | 0.5849 |
| Clinical + Vitals + ECG | 0.8128 (0.7809-0.8428) | 0.4991 | 0.4909 | 0.4281 | 0.5755 |
| Clinical + Vitals + PPG | 0.8427 (0.8172-0.8678) | 0.5526 | 0.5257 | 0.4524 | 0.6274 |
| Clinical + Vitals + PPG + ECG | 0.8458 (0.8213-0.8702) | 0.5481 | 0.5403 | 0.4486 | 0.6792 |

PPG beats ECG by a decent margin here (delta +0.0296, 95% CI 0.0118-0.0473, significant), and stacking both together over PPG alone gets you basically nothing (+0.0032, CI crosses zero). Once PPG is in the model, ECG isn't adding anything new.

---

## MIMIC-IV, XGBoost on ECG intervals + clinical + labs

Balanced 1:1 cohort, N=10,360 (5,180 deaths, 5,180 survivors).

| Model | CV AUC-ROC | Holdout AUC-ROC (95% CI) | Holdout AUC-PR (95% CI) | F1 | Precision | Recall |
|---|---|---|---|---|---|---|
| ECG only | 0.7423 | 0.7357 (0.7141-0.7578) | 0.7136 (0.6822-0.7441) | 0.7082 | 0.6092 | 0.8456 |
| Clinical only | 0.7434 | 0.7406 (0.7191-0.7608) | 0.7017 (0.6694-0.7332) | 0.7386 | 0.6073 | 0.9421 |
| ECG + Clinical | 0.8066 | 0.8014 (0.7826-0.8200) | 0.7738 (0.7460-0.8003) | 0.7601 | 0.6481 | 0.9189 |
| ECG + Clinical + Labs | 0.8473 | 0.8463 (0.8296-0.8616) | 0.8280 (0.8037-0.8507) | 0.7826 | 0.7008 | 0.8861 |

ECG+clinical over ECG alone: +0.0660 (sig). Over clinical alone: +0.0609 (sig). Labs on top of that: +0.0448 (sig).

---

## MIMIC-IV, CNN on raw 12-lead waveform

Same cohort, same holdout split as above.

| Model | CV AUC-ROC | Holdout AUC-ROC (95% CI) | Holdout AUC-PR (95% CI) |
|---|---|---|---|
| Lead II only | 0.7702 ± 0.0127 | 0.7780 (0.7573-0.7969) | 0.7419 (0.7105-0.7724) |
| 12-lead | 0.7838 ± 0.0165 | 0.7844 (0.7647-0.8032) | 0.7675 (0.7375-0.7954) |
| 12-lead + clinical | 0.7901 ± 0.0147 | 0.7968 (0.7773-0.8152) | 0.7703 (0.7393-0.7997) |
| 12-lead + clinical + labs | 0.8447 ± 0.0124 | 0.8403 (0.8232-0.8555) | 0.8185 (0.7917-0.8440) |

The fully fused CNN (0.8403) and the full XGBoost model (0.8463) land in basically the same place once you look at the CIs (0.8232-0.8555 vs 0.8296-0.8616, plenty of overlap). The CNN is learning straight from raw waveform shape, no hand-built intervals, and still gets to about the same place.

---

## ECG, across both datasets

| Where | Feature type | Standalone AUC-ROC |
|---|---|---|
| VitalDB, ICU admission | morphology stats, intraop | 0.6437 |
| VitalDB, ICU stay length | morphology stats, intraop | 0.6330 |
| MIMIC-IV, mortality | real clinical intervals, diagnostic | 0.7357 |

The big jump in MIMIC-IV isn't really about a different clinical problem, it's that the feature type is different. MIMIC-IV gets actual measured RR/PR/QRS/QT intervals, VitalDB only gets shape statistics because interval extraction failed on the noisier intraop signal (see Methods).

---

## What it all adds up to

1. PPG beats ECG, standalone and combined, for both VitalDB outcomes. PPG+clinical matches clinical+vitals (no real difference), ECG+clinical falls short of it both times.
2. Both signals add something real over clinical+vitals for ICU admission. For the harder ordinal stay-length outcome, PPG's contribution holds up but ECG's edges toward not-quite-significant.
3. Stacking ECG and PPG together doesn't beat PPG alone, the signals are basically redundant once PPG is in the model.
4. In MIMIC-IV, ECG and clinical features carry roughly equal weight alone, but combining them is a real step up, and labs add more on top of that.
5. A CNN trained on raw 12-lead waveform, once fused with clinical and lab data, performs about as well as an XGBoost model built on actual measured ECG intervals. Deep learning on the raw signal can apparently get you to a similar place without needing the hand-engineered features.
