# Methods

## What this is about

We wanted to know whether intraoperative and diagnostic ECG/PPG waveforms actually add useful information on top of regular clinical data when predicting outcomes after surgery or during a hospital stay. Two datasets, two different settings: VitalDB (intraoperative monitoring at Seoul National University Hospital) and MIMIC-IV (a US academic hospital dataset with a linked diagnostic ECG companion set). Models were built up step by step, from waveform alone to clinical alone to combinations, all with a proper holdout that never touched any of the model fitting.

---

## VitalDB

### Who's in it and what we're predicting

VitalDB has 6,388 noncardiac surgical patients with preop clinical data, intraop vitals, and the raw waveforms recorded during surgery (ECG Lead II, PPG, arterial line). Two outcomes:

1. **ICU admission** (yes/no), defined as any ICU stay greater than zero days.
2. **ICU stay length** (no ICU / 1-3 days / 4+ days). We picked this cutoff after noticing 68% of ICU stays (815 out of 1,204) were exactly one day, which reads more like standard post-op observation than someone actually crashing. The 4+ day group was meant to isolate patients who were genuinely sick.

We did **not** model in-hospital mortality in VitalDB. There are only 57 deaths out of 6,388 cases, just under 1%. An 80/20 holdout would leave around 11 deaths in the test set, which isn't enough to get a stable confidence interval out of, let alone trust the result. This is a dataset limitation, not us avoiding the outcome. ICU admission and stay length both have enough events to support a real holdout evaluation, so those are what we used.

### What went into the models

- **Clinical (19 features):** age, sex, BMI, ASA class, emergency flag, preop hypertension and diabetes, an ECG abnormality flag built off the `preop_ecg` text field (anything other than "Normal Sinus Rhythm" gets flagged), and nine preop labs (hemoglobin, platelets, PT, aPTT, sodium, potassium, glucose, albumin, creatinine, BUN, AST, ALT). Missingness here was small, under 10% across the board, filled with the training-set median.

- **Vitals (6 features):** intraop heart rate (mean, min, max, std) and SpO2 (mean, min), pulled from minute-level Solar8000 monitor data.

- **PPG (11 features):** mean, median, std, IQR, range, skewness, kurtosis, energy, mean absolute difference, min, max, computed on the raw PPG waveform after clipping to the 1st-99th percentile to knock out extreme artifacts.

- **ECG (11 features):** the same 11 stats, computed the same way, on raw ECG Lead II.

### Why we ended up with simple stats instead of real ECG intervals

We tried to pull actual clinical interval measurements out of the VitalDB ECG (RR, PR, QRS, QT) using neurokit2's automated peak detection and wave delineation. It didn't work well enough to use. Full delineation succeeded on only 1,601 of 6,355 cases, about 25%. A stripped-down version using just R-peak detection did even worse, 1,377 cases, 22%. Both yields are too low to trust, since whatever subset succeeds is probably not a random sample of the cohort, it's biased toward cleaner signals. Intraoperative ECG is just noisy: cautery interference, motion, lead artifact, all way more present than in a diagnostic-quality ECG taken in a clinic or on a ward. So we fell back to the same simple stats approach used for PPG, which worked on 6,346 of 6,355 cases (99.8%). This choice matters a lot for interpreting the cross-dataset comparison later, since MIMIC-IV's ECG features are the real clinical intervals, not statistical summaries.

### Model setup

XGBoost throughout, same hyperparameters everywhere: 300 trees, max depth 3, learning rate 0.03, subsample 0.8, column subsample 0.8.

For every outcome and feature combination, we split 80/20 first, before touching cross-validation or fitting anything, so the holdout never sees any part of model development. Within the 80% training set, 5-fold stratified CV handles evaluation and threshold picking. SMOTE gets applied strictly inside each training fold, never before the split and never on the holdout, so synthetic samples can't leak across the boundary. The classification threshold comes from maximizing F1 on the pooled out-of-fold CV predictions, then gets applied unchanged to the holdout.

The final model for holdout evaluation gets retrained on the full 80% training set after SMOTE, same hyperparameters. We report AUC-ROC and AUC-PR on the untouched 20% holdout, both with bootstrap confidence intervals (2,000 resamples). For the ordinal stay-length outcome, both metrics are macro-averaged one-vs-rest across the three classes, so the rare prolonged-stay group (about 3% of cases) doesn't just get drowned out by the no-ICU majority.

### How the six models break down

For each outcome, six models: signal alone (PPG or ECG), clinical alone, vitals alone, signal+clinical, clinical+vitals, and everything together. This lets us see each signal's standalone power, what it adds on top of clinical alone, how it stacks up head-to-head against vitals (with clinical held constant), and what's left to gain once you've already got the full clinical+vitals baseline. Significance for any of these comparisons comes from bootstrapping the delta in holdout predictions (again, 2,000 resamples), counted as significant if the 95% CI doesn't cross zero.

We kept this exact order (signal, clinical, vitals, signal+clinical, clinical+vitals, full) to match the structure used for MIMIC-IV, where labs play roughly the same role vitals plays here, the last thing added on top. Worth noting: an earlier pass at this analysis added vitals before the waveform signal in the sequence, which made it impossible to actually see the signal's standalone contribution clearly. We caught that and reran everything with the corrected order before finalizing anything.

### ECG vs PPG, slightly different cohorts

ECG and PPG extraction didn't fail on exactly the same cases (6,346 vs 6,151 succeeded respectively), so the ECG and PPG six-model series each run on their own full cohort rather than the intersection. This keeps statistical power up for each one individually, but it does mean the "clinical only" and "vitals only" numbers aren't identical between the two tables, since the underlying patients differ slightly. We also ran a separate analysis on just the intersection cohort (both signals successfully extracted, N=6,346) to test directly whether stacking ECG and PPG together beats either alone.

---

## MIMIC-IV

### Cohort and outcome

MIMIC-IV is from Beth Israel Deaconess, linked here to its diagnostic ECG companion set (12-lead ECGs with automated machine-measured intervals already included). Outcome is in-hospital mortality.

Cohort construction: all 800,035 diagnostic ECGs got linked to admissions by subject ID, keeping only ECGs that actually fell within the admission's documented window (292,435 ECG-admission pairs survive that filter). Where an admission had more than one qualifying ECG, we kept the first one chronologically, landing at 143,647 admissions with exactly one linked ECG each. Mortality in that linked cohort sits at 3.6% (5,180 deaths).

That's too skewed to model directly, so we built a balanced 1:1 cohort by randomly undersampling survivors down to match the death count (fixed seed for reproducibility), giving a final cohort of 10,360. This is a fairly standard move for imbalanced classification, but it's worth flagging clearly: the balanced cohort's 50% prevalence doesn't reflect the real-world 3.6% rate, so anything threshold-dependent (F1, precision, recall) from this cohort shouldn't be read as what you'd actually see in deployment. AUC-ROC and AUC-PR aren't affected by the rebalancing in the same way, which is why those stay the primary numbers we report.

### Features

- **ECG intervals (13 features):** RR, PR, QRS duration, QT, heart rate (derived from RR), P/QRS/T axis, and the four wave-boundary timestamps the derived intervals come from. These are the machine measurements that ship with the MIMIC-IV-ECG dataset, not anything we extracted ourselves. Zero missingness across all 13.

- **Clinical (5 features):** age, gender, admission type, insurance, race.

- **Labs (8 features):** first value after admission for BUN, creatinine, glucose, hematocrit, hemoglobin, platelets, potassium, sodium, pulled from `labevents` and linked by admission ID. Albumin got dropped, 47% missing in the balanced cohort, too high to impute responsibly. The remaining 8 ranged from about 8-9% missing and got median-filled.

### Why ECG performs so much better here than in VitalDB

Since MIMIC-IV's ECG features are real, clinically validated interval measurements rather than statistical waveform summaries, the standalone ECG model here lands at 0.736 holdout AUC, well above VitalDB's ECG-only result of 0.644 on a similar binary task. That gap lines up with what we already suspected: the *kind* of feature extraction matters a lot, and VitalDB's weaker ECG result is more about what we could extract than about ECG being inherently less useful as a signal.

### Model setup

Same XGBoost hyperparameters, same holdout/CV/SMOTE structure as VitalDB. One difference: MIMIC-IV's model sequence goes ECG alone, clinical alone, ECG+clinical, ECG+clinical+labs, since there's no continuous intraoperative monitoring data here for a "vitals" category, labs take that final incremental slot instead.

### The CNN side

Alongside the XGBoost models on the machine-measured intervals, we trained a series of CNNs directly on the raw 12-lead waveform (10 seconds at 500 Hz, 5,000 samples per lead), using the same balanced cohort and the exact same 80/20 holdout split (identical seed and stratification, so this is a fair comparison against the XGBoost results). Four variants, increasing in complexity: Lead II alone, all 12 leads, 12-lead fused with the 5 clinical features, and 12-lead fused with clinical plus labs.

All four use a ResNet-style 1D CNN: a stem convolution (kernel 15, stride 2), three residual blocks at 32/64/128/256 channels (stride 2 each), global average pooling, dropout at 0.5, and a linear output layer. For the fused variants, the tabular features go through a small separate branch (two linear layers, ReLU) and get concatenated with the pooled conv features before the final layer. Training used binary cross-entropy with logits, Adam at 1e-3, a plateau scheduler, and early stopping after 4 epochs without improvement (20 epoch cap). Same 5-fold CV and holdout structure as the XGBoost side, with a fresh model trained per fold.

### MIMIC-III waveform CNN, abandoned

Before settling on the MIMIC-IV CNN approach, we tried something similar on MIMIC-III's ICU bedside monitor waveforms (3,237 patients, 277 deaths, 8.6% event rate). It didn't work. Validation AUC consistently peaked early (epoch 1-2, around 0.62-0.69) and then dropped as training loss kept falling, which points to overfitting, probably because the ratio of unique patients to training windows let the model start memorizing patient-specific noise instead of learning anything generalizable. Final holdout AUC there was 0.620 (95% CI 0.534-0.711), not good enough to report as a finding, and it's not in the results below. The MIMIC-IV CNN work described above replaced this entirely, with a much larger and cleaner cohort to work with.

---

## MOVER (separate paper)

A third dataset, MOVER (UC Irvine perioperative data), is being used for a related but separate analysis targeting JAMIA Open, comparing an XGBoost model on preop clinical features (AUC 0.825) against RCRI (AUC 0.671) for perioperative cardiac complications. That work is methodologically independent of the waveform comparison described here and isn't part of these results.

We also tried extending MOVER with intraoperative ECG and arterial line waveform features, matching de-identified waveform files to surgical cases by OR duration (average matching error around 8 minutes). That didn't make it into the final results either. The cohort with successfully linked waveform features was small (885 unique patients, 114 with the cardiac outcome), and once we ran a proper patient-level, leakage-free holdout evaluation, adding waveform features actually made things worse, not better (0.556 holdout AUC with waveforms vs 0.721 for clinical+vitals alone). We think this is a sample size and outcome-rarity problem more than evidence that waveforms have nothing to offer here, but it's inconclusive enough that we're leaving it out and just noting it for completeness.

---

## General rules we followed everywhere

1. Holdout sets get built before any CV, SMOTE, or hyperparameter work, and never get touched again until the single final evaluation pass.
2. SMOTE only happens inside training folds, never before a train/val split and never on holdout data.
3. All confidence intervals are bootstrapped directly off holdout predictions (2,000 resamples), not off CV predictions, since that's what actually reflects generalization.
4. Decision thresholds for F1/precision/recall come from CV out-of-fold predictions only, then get applied to the holdout unchanged, so the threshold itself isn't picked using holdout data.
5. Median imputation always uses training-set medians, applied identically to the holdout, so no information leaks through the imputation step.
6. Where the same patient shows up more than once (MOVER), splits are grouped at the patient level so nobody appears in both train and test.

---

## Where this falls short

- VitalDB's ECG features are statistical summaries of waveform shape, MIMIC-IV's are real clinical interval measurements. These two analyses are related but shouldn't be read as a clean apples-to-apples replication across datasets, they're testing somewhat different things.
- VitalDB mortality wasn't modeled at all, only 57 events, not enough for a trustworthy holdout evaluation.
- MIMIC-IV's balanced cohort doesn't reflect real mortality prevalence (3.6%), so precision/recall/F1 from that cohort shouldn't be taken as deployment-realistic numbers without recalibration.
- Race shows up as a clinical feature in the MIMIC-IV models and lands among the top SHAP features in the full model. We read that as the model picking up on socioeconomic and access-to-care signal rather than anything biological, and it's a fairness issue that would need real attention before this gets anywhere near clinical use.
- VitalDB's intraop ECG and PPG extraction never produced usable clinical intervals despite trying automated delineation, purely a signal noise issue with intraoperative recordings, which limited us to general waveform stats instead of anything clinically interpretable on the ECG side.
- The MOVER waveform attempt suggests intraoperative waveform features might need a much bigger cohort than we had to show real incremental value for something as rare and varied as perioperative cardiac complications.
