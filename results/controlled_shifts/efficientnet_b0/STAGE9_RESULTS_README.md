# Stage 9 — Controlled Distribution Shift Evaluation

## Research objective

Evaluate how controlled color, blur, and noise
perturbations affect discrimination, classification,
and confidence calibration of the final frozen
EfficientNet-B0 breast histopathology classifier.

## Evaluation cohort

- Dataset: BreaKHis
- Partition: Original independent Test
- Magnification: 40x
- Independent patients: 16
- Benign patients: 5
- Malignant patients: 11
- Images per condition: 446
- Patches per condition: 7,087
- Shifted conditions: 9
- Original clean reference conditions: 1
- Total shifted patch evaluations: 63,783
- Total shifted image predictions: 4,014

The clean reference was reused from Stage 8.

## Frozen model

- Architecture: EfficientNet-B0
- Checkpoint: Final Epoch 5
- Checkpoint SHA-256: 84ea0871a309f3d68b8958b68b6da4f01adb31d3109fa4e0fc57f6670b63c9bb
- Patch size: 224 x 224 pixels
- Aggregation: arithmetic mean of patch logits
- Classification threshold: 0.5

No CNN weights were changed during Stage 9.

## Controlled perturbations

### Stain-like color shift

Fixed RGB optical-density channel scaling.

Severity parameters:

- Low: 0.05
- Medium: 0.10
- High: 0.20

This is a controlled stain-like color perturbation.
It is NOT a validated physical H&E stain
transformation or stain-separation method.

### Gaussian blur

Spatial Gaussian blur, measured in patch pixels.

- Low: sigma 0.5
- Medium: sigma 1.0
- High: sigma 2.0

### Gaussian noise

Additive RGB Gaussian noise on [0, 1] pixels.

- Low: sigma 0.02
- Medium: sigma 0.05
- High: sigma 0.10

The noise seed was fixed to 631.
Noise was deterministic for each original patch.

The same underlying noise realization was scaled
for all three severity levels.

## Preprocessing controls

Original Stage 8 tissue-selected patches were reused.

Original source image resizing, patch coordinates,
and patch selection were preserved.

Perturbations were applied after original patch
extraction and before ImageNet normalization.

This isolates changes in pixel appearance from
changes in patch sampling.

## Calibration

Temperature scaling:

T = 4.069193139210192

Platt scaling:

slope = 0.29141657172422586
intercept = -0.7096590060187311

Both methods were fitted before Test evaluation.
They were not refitted on shifted images.

## Evaluation metrics

- AUROC
- Average precision
- Balanced accuracy
- Sensitivity and specificity
- Expected Calibration Error, 10 equal-width bins
- Brier score
- Binary negative log-likelihood

All evaluations use image-level predictions.

## Descriptive results

| Condition | AUROC | Balanced accuracy | Raw ECE | Temperature ECE | Platt ECE |
|---|---:|---:|---:|---:|---:|
| clean | 0.8479 | 0.7459 | 0.1876 | 0.1401 | 0.0973 |
| stain_like_od_low | 0.8397 | 0.7516 | 0.1887 | 0.1458 | 0.1044 |
| stain_like_od_medium | 0.8311 | 0.7459 | 0.1895 | 0.1533 | 0.1152 |
| stain_like_od_high | 0.8132 | 0.7459 | 0.1988 | 0.1768 | 0.1410 |
| gaussian_blur_low | 0.8466 | 0.7363 | 0.2006 | 0.1676 | 0.1249 |
| gaussian_blur_medium | 0.8432 | 0.7382 | 0.1997 | 0.1616 | 0.1221 |
| gaussian_blur_high | 0.8252 | 0.7716 | 0.1674 | 0.0926 | 0.1124 |
| gaussian_noise_low | 0.8599 | 0.7516 | 0.1730 | 0.1127 | 0.0758 |
| gaussian_noise_medium | 0.8859 | 0.7706 | 0.1654 | 0.1245 | 0.1305 |
| gaussian_noise_high | 0.8836 | 0.6514 | 0.2762 | 0.1928 | 0.1273 |

## Patient-level statistical analysis

- Bootstrap resamples: 1,000
- Random seed: 631
- Sampling unit: patient
- Sampling with replacement
- Stratified by patient diagnosis
- Five benign and eleven malignant patient draws
  per bootstrap replicate
- Same draws for all methods and conditions
- Repeated patient draws preserve all their images
- Intervals: 95% percentile bootstrap

Bootstrap comparisons include:

1. Shifted condition minus clean reference
2. Calibrated method minus comparison method

## Key findings

High stain-like color shift produced an observed
AUROC change of
-0.0347,
95% CI
[-0.0765,
0.0064].

Medium Gaussian noise produced an observed
AUROC change of
+0.0380,
95% CI
[+0.0046,
+0.0849].

At high noise, Platt scaling minus uncalibrated
ECE was
-0.1488,
95% CI
[-0.1743,
-0.0490].

Calibration can improve probability quality
without changing discrimination.

Different shift types affect ranking,
classification thresholds, and probability
calibration differently.

## Statistical limitations

This is a secondary exploratory sensitivity
analysis on previously evaluated Test patients.

There are only 16 independent Test patients.

Confidence intervals are patient-stratified
percentile bootstrap intervals.

The many related comparisons were not adjusted
for multiple hypothesis testing.

Intervals excluding zero should not be treated
as definitive confirmatory evidence.

The selected perturbations are synthetic.
They do not establish robustness to all real
laboratory or acquisition shifts.

The model is not validated for clinical use.

## Reproducibility

The Stage 9 outputs contain:

- Locked experimental protocol
- Perturbation validation and image previews
- Per-condition patch and image predictions
- Inference integrity audits
- Performance and calibration metrics
- Patient-level bootstrap results
- Shift-versus-clean comparisons
- Calibration-method comparisons
- Robustness figures
- SHA-256 inventory

The clean reference is stored in the
Stage 8 results directory.

The Stage 9 notebook should be committed
separately under Notebooks/.

Original dataset images and CNN weights are
not included in this results package.

No CNN retraining, calibrator refitting,
or Test-based severity tuning was performed.
