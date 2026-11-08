# Stage 8 — Independent Test Evaluation

## Research objective

Evaluate confidence calibration and discrimination
of a frozen EfficientNet-B0 breast histopathology
classifier under BreaKHis magnification shift.

## Data partition

- Development: 49 patients
- Calibration: 16 patients
- Independent Test: 16 patients
- Test benign patients: 5
- Test malignant patients: 11
- Test images: 1,863
- Test patches: 29,671

Test image counts:

- 40x: 446 images
- 100x: 501 images
- 200x: 480 images
- 400x: 436 images

The 40x Test data represent clean internal evaluation.
The other magnifications represent magnification shift.

## Locked model and preprocessing

- Architecture: EfficientNet-B0
- Final checkpoint: Epoch 5
- Model checkpoint SHA-256: 84ea0871a309f3d68b8958b68b6da4f01adb31d3109fa4e0fc57f6670b63c9bb
- Image aggregation: mean patch logit
- Patch size: 224 x 224
- Candidate grid: 4 x 4
- Maximum patches per image: 16
- Minimum tissue fraction: 0.20

The original Development preprocessing operator,
including the fixed 0.98 resize factor, was
applied at every magnification.

This is NOT physical-resolution harmonization.

## Locked probability calibration

Temperature scaling:

- T = 4.069193139210192

Platt scaling:

- slope = 0.29141657172422586
- intercept = -0.7096590060187311

Both calibrators were fitted on Calibration
patients before independent Test evaluation.

Neither calibrator was refitted on Test data.

## Test metrics

| Magnification | Method | AUROC | ECE10 | Brier | NLL |
|---|---|---:|---:|---:|---:|
| 40x | Uncalibrated | 0.8479 | 0.1876 | 0.1873 | 1.6349 |
| 40x | Temperature scaling | 0.8479 | 0.1401 | 0.1583 | 0.5398 |
| 40x | Platt scaling | 0.8479 | 0.0973 | 0.1466 | 0.5005 |
| 100x | Uncalibrated | 0.7152 | 0.2103 | 0.2134 | 1.9422 |
| 100x | Temperature scaling | 0.7152 | 0.2031 | 0.1999 | 0.6549 |
| 100x | Platt scaling | 0.7152 | 0.1979 | 0.2017 | 0.6320 |
| 200x | Uncalibrated | 0.7454 | 0.2014 | 0.1977 | 1.4096 |
| 200x | Temperature scaling | 0.7454 | 0.1469 | 0.1820 | 0.5668 |
| 200x | Platt scaling | 0.7454 | 0.1293 | 0.1730 | 0.5331 |
| 400x | Uncalibrated | 0.6327 | 0.2906 | 0.3011 | 2.0165 |
| 400x | Temperature scaling | 0.6327 | 0.1754 | 0.2472 | 0.7414 |
| 400x | Platt scaling | 0.6327 | 0.1372 | 0.2351 | 0.6982 |

Average precision is the implementation used
for the AUPRC summary.

ECE uses ten equal-width malignant-probability
bins, with the final bin including 1.0.

Classification metrics use probability >= 0.5.

## Statistical uncertainty

- 1,000 bootstrap resamples
- Random seed: 631
- Sampling unit: patient
- Sampling with replacement
- Stratified by patient diagnosis
- Five benign and eleven malignant patient draws
  per resample
- Same draws across magnifications and methods
- Repeated sampled patients retain all their images
- 95% percentile confidence intervals

These intervals are exploratory because there
are only 16 independent Test patients.

## Main observations

AUROC declined under all three shifted
magnifications relative to the 40x Test set.

The calibrated probabilities showed lower
ECE, Brier score, and NLL than uncalibrated
probabilities in the reported point estimates.

The paired patient-bootstrap ECE intervals
supported reductions at 400x, but the ECE
difference intervals at 40x, 100x, and 200x
included zero.

Calibration did not change AUROC because
the fitted transformations preserved ranking.

The results do not establish clinical safety,
clinical effectiveness, or external
generalization.

## Reproducibility

This directory contains the inference
predictions, evaluation metrics, patient-level
bootstrap results, reliability-bin data,
figure, and integrity audits.

Source images and final CNN weights are not
included in this results package.

The Stage 8 notebook must be committed
separately under Notebooks/.

No Test-based model selection, CNN retraining,
or calibration refitting was performed.
