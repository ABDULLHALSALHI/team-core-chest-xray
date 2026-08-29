# ConvNeXt-Tiny 320×320 Experiment Summary

## Overview

This experiment evaluates a high-resolution ConvNeXt-Tiny pipeline for multi-label chest X-ray classification on the Team Core NIH ChestX-ray14 capstone subset.

The model predicts 14 pathology labels independently using sigmoid outputs:

- Atelectasis
- Consolidation
- Infiltration
- Pneumothorax
- Edema
- Emphysema
- Fibrosis
- Effusion
- Pneumonia
- Pleural_Thickening
- Cardiomegaly
- Nodule
- Mass
- Hernia

`No_Finding` is not treated as a model output.

This is an educational research prototype and is not clinically validated.

## Final Model

- Architecture: ConvNeXt-Tiny
- Pretraining: ImageNet
- Input size: 320×320
- Loss: BCEWithLogitsLoss
- Optimizer: AdamW
- Learning rate: 1e-4
- Weight decay: 1e-4
- Scheduler: ReduceLROnPlateau
- Mixed precision: enabled
- Maximum epochs: 15
- Early stopping patience: 4
- Training augmentation:
  - Resize to 320×320
  - Random rotation ±7°
  - Mild brightness/contrast jitter
- No horizontal flip

The best BCE checkpoint occurred at epoch 3.

## Validation Performance

- Validation Macro AUROC: **0.8121**
- Validation Macro F1 at fixed 0.50 threshold: **0.2955**
- Validation Macro F1 using validation-optimized thresholds: **0.4252**

Threshold optimization was performed on the validation set only.

## Final Test Performance

- Test Macro AUROC: **0.8158**
- Test Macro F1 at fixed 0.50 threshold: **0.3034**
- Test Macro F1 using validation-optimized thresholds: **0.4236**
- Test Macro ECE: **0.0258**

The validation and test results were close, which suggests stable generalization on the current patient-level split.

## Per-Pathology Test AUROC

| Pathology | AUROC |
|---|---:|
| Hernia | 0.9232 |
| Edema | 0.9207 |
| Emphysema | 0.9156 |
| Cardiomegaly | 0.9039 |
| Pneumothorax | 0.8460 |
| Effusion | 0.8423 |
| Mass | 0.8269 |
| Fibrosis | 0.8101 |
| Atelectasis | 0.7733 |
| Nodule | 0.7580 |
| Pleural_Thickening | 0.7565 |
| Consolidation | 0.7407 |
| Infiltration | 0.7081 |
| Pneumonia | 0.6961 |

The strongest ranking performance was observed for Hernia, Edema, Emphysema, and Cardiomegaly. Pneumonia and Infiltration remained the most difficult classes.

## BCE vs Focal Loss

A second ConvNeXt-Tiny experiment used focal loss with gamma = 2 and alpha = 0.25.

- ConvNeXt BCE best validation Macro AUROC: **0.8121**
- ConvNeXt Focal best validation Macro AUROC: **0.8145**

Although focal loss produced a slightly higher macro AUROC (+0.0024), BCE was retained as the final model because it performed better on more individual classes and provided a more balanced class-wise result.

The focal experiment therefore served as a useful loss-function comparison, but not as the selected final checkpoint.

## Calibration

The uncalibrated ConvNeXt probabilities were evaluated using Expected Calibration Error.

- Validation Macro ECE: **0.0245**
- Test Macro ECE: **0.0258**

Because the test ECE was already well below 0.05, additional temperature scaling was not applied.

## Robustness Analysis

### Sex

| Sex | Images | Macro AUROC |
|---|---:|---:|
| Female | 1816 | 0.8165 |
| Male | 2088 | 0.8138 |

The difference between female and male subgroup performance was small.

### Age

| Age Group | Images | Macro AUROC | Classes Evaluated |
|---|---:|---:|---:|
| 0–17 | 174 | 0.8110 | 13 |
| 18–39 | 916 | 0.8249 | 14 |
| 40–59 | 1772 | 0.8164 | 14 |
| 60–79 | 988 | 0.7945 | 14 |
| 80+ | 54 | 0.7235 | 13 |

Performance remained relatively consistent for patients under 60, then decreased in older groups. The 80+ result should be interpreted cautiously because it contains only 54 images and only 13 classes were evaluable.

### View Position

| View Position | Images | Macro AUROC |
|---|---:|---:|
| AP | 1641 | 0.7989 |
| PA | 2263 | 0.8084 |

Performance was relatively consistent between AP and PA views, with a small advantage for PA images.

## Final Decision

The ConvNeXt-Tiny 320×320 BCE model was selected as the final single-model candidate because it:

- exceeded 0.80 test Macro AUROC
- exceeded 0.40 threshold-tuned test Macro F1
- achieved low calibration error
- showed stable performance across sex and view-position subgroups
- improved several class-wise AUROCs compared with the earlier DenseNet baseline

The main remaining weakness is reduced performance for some difficult labels, especially Pneumonia and Infiltration, and lower robustness in older age groups.

## Repository Artifacts

Related files:

- `notebooks/convnext-tiny-320-training.ipynb`
- `results/convnext_thresholds.json`
- `results/convnext_final_test_metrics.csv`
- `results/convnext_robustness_summary.csv`

Model checkpoint files are intentionally not stored directly in the standard GitHub repository.
