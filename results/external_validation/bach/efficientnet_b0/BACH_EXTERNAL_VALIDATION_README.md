# BACH External Validation — EfficientNet-B0

## Research objective

Evaluate the frozen BreaKHis-trained EfficientNet-B0 and
its previously fitted probability calibration methods on
the external BACH breast histopathology microscopy dataset.

This directory contains the reproducible evaluation outputs
from `04_external_data_preparation.ipynb`.

## External dataset

- Dataset: BACH microscopy images
- Original images: 400
- Normal: 100
- Benign: 100
- InSitu: 100
- Invasive: 100
- Retained tissue patches: 6,255
- Original images are not included in this package

### Binary classification labels

| BACH class | Binary target |
|---|---|
| Normal | 0 — Nonmalignant |
| Benign | 0 — Nonmalignant |
| InSitu | 1 — Malignant |
| Invasive | 1 — Malignant |

Normal tissue and benign lesions are distinct categories.
The original four labels are retained for subgroup analysis.

## Frozen preprocessing

- Source resolution: 0.42 micrometers per pixel
- Target resolution: 0.50 micrometers per pixel
- Image resampling: LANCZOS
- Patch size: 224 x 224 pixels
- Tissue selection: Original BreaKHis hybrid tissue mask
- Candidate grid: 4 x 4
- Maximum patches: 16 per image
- Minimum tissue fraction: 0.20
- Normalization: ImageNet mean and standard deviation
- No random augmentation

All 400 images and 6,255 patch selections were verified
against the original BreaKHis preprocessing functions.

## Frozen model and calibration

- CNN: EfficientNet-B0
- Checkpoint: Final Development model, Epoch 5
- Aggregation: Mean raw patch logit per image
- Classification threshold: 0.5

Model SHA-256:

`84ea0871a309f3d68b8958b68b6da4f01adb31d3109fa4e0fc57f6670b63c9bb`

The frozen calibration formulas are:

- Uncalibrated: sigmoid(z)
- Temperature: sigmoid(z / 4.069193139210192)
- Platt: sigmoid(0.29141657172422586 * z - 0.7096590060187311)

Here, z is the image-level mean raw logit.

The model and calibrators were fixed before BACH inference.
No BACH images were used for training, calibration fitting,
threshold optimization, or model selection.

## External evaluation results

| Method | AUROC | Balanced accuracy | Sensitivity | Specificity | ECE-10 | Brier | NLL |
|---|---:|---:|---:|---:|---:|---:|---:|
| uncalibrated | 0.66692 | 0.55500 | 0.76000 | 0.35000 | 0.33510 | 0.34766 | 1.35137 |
| temperature | 0.66692 | 0.55500 | 0.76000 | 0.35000 | 0.13431 | 0.24675 | 0.68676 |
| platt | 0.66692 | 0.62250 | 0.59500 | 0.65000 | 0.10627 | 0.23265 | 0.66187 |

AUROC and average precision use monotonic raw-logit
scores to avoid numerical probability saturation.

ECE uses 10 equal-width positive-class probability bins.

Lower ECE, Brier score, and negative log-likelihood
indicate better probability quality.

## Main findings

- External BACH AUROC was approximately 0.667.
- Temperature scaling and Platt scaling reduced overall
  probability calibration errors.
- Platt scaling improved overall balanced accuracy at
  the frozen threshold.
- Platt scaling reduced malignant sensitivity while
  increasing nonmalignant specificity.
- All three monotonic transformations preserved AUROC.
- Diagnostic-class analysis revealed different tradeoffs
  across Normal, Benign, InSitu, and Invasive.

Cross-dataset differences are descriptive and do not
isolate the causal effect of distribution shift.

## Exploratory uncertainty analysis

- 1,000 bootstrap resamples
- Seed: 631
- Resampling unit: original image
- 100 images sampled with replacement per original class
- Four-class-stratified sampling
- Same sampled indices across all probability methods
- 95% unadjusted percentile intervals
- Paired calibration method differences

Verified BACH patient identifiers are unavailable.

Consequently, the intervals are exploratory image-level
estimates, not patient-level confidence intervals.
Dependence among images could make them too narrow.

Multiple comparisons were not adjusted.

## Research limitations

- External images have different domain characteristics.
- Normal tissue is combined with benign lesions in
  the binary evaluation.
- Patient grouping cannot be verified.
- ECE depends on binning and sample composition.
- Some reliability bins contain few observations.
- Calibration improvements do not imply clinical safety.
- Results are not evidence of clinical readiness.

## Reproducibility

Research repository commit at packaging:

`f0a018e30eee8890e3a8b290de6a4527481b0c4d`

Frozen external protocol SHA-256:

`2d45dd06bbe1c2aafa89a1918bac915e151611c1d39d97eda8ea1fbf3e0087f3`

All other packaged research files are listed in
`bach_package_file_inventory.csv`, with SHA-256 hashes.

The inventory does not contain its own hash.

The original source images and model checkpoint are
excluded from this package.

The initial metadata preflight records the runtime
before image recovery. The separate recovery audit
documents successful restoration and verification.

The notebook is committed separately under `Notebooks/`.

## Repository destination

`results/external_validation/bach/efficientnet_b0/`
