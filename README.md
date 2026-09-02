# Explainable Multi-Label Chest X-Ray Diagnosis

Multi-label classification of 14 thoracic pathologies from chest radiographs, with Grad-CAM localization evaluated against radiologist-drawn bounding boxes.

Samsung Innovation Campus AI Capstone — Team Core.

Dataset: NIH ChestX-ray14, a stratified 31,077-image subset covering 11,907 patients.

---

## Data splits

Five patient-level splits. Splitting is defined over patients rather than images because one patient can hold many studies — mixing them across train and test lets the network identify the individual instead of the pathology.

| Split | Images | Patients | Purpose |
|---|---|---|---|
| `train` | 18,013 | 7,930 | Training |
| `val` | 3,978 | 1,699 | Hyperparameters and thresholds |
| `test` | 3,904 | 1,700 | Final classification results |
| `loc_tune` | 2,606 | 289 | Grad-CAM threshold selection |
| `loc_report` | 2,576 | 289 | Final localization results |

All ten pairwise combinations return zero shared patients.

Patients holding radiologist-annotated bounding boxes are withheld from the classification splits entirely and used only for Grad-CAM evaluation, split evenly so the localization threshold is never selected on the data it is reported against.

**The model has 14 outputs.** `No Finding` is the absence of the fourteen pathologies, not a class.

---

## Repository

    notebooks/
      01_data_preparation.ipynb          Patient-level splitting (Colab; run and frozen)
      02_densenet_baseline.ipynb         DenseNet-121 training, 224x224
      03_evaluation.ipynb                Per-class metrics, CIs, threshold calibration (DenseNet)
      04_resnet50_baseline.ipynb         ResNet50 training, 224x224
      05_resnet_evaluation.ipynb         ResNet50 scored with the same pipeline
      convnext-tiny-320-training.ipynb   ConvNeXt-Tiny training, 320x320, threshold tuning, calibration, robustness
    docs/
      convnext_experiment_summary.md     Full write-up of the ConvNeXt-Tiny run: config, results, BCE vs focal loss, robustness
    results/                             Per-class thresholds and evaluation output for every model

Notebooks other than `01_data_preparation.ipynb` run on Kaggle. Attach both datasets as inputs, enable a GPU, and run in order.

Data and checkpoints are hosted on Kaggle, not committed here:

- Splits and image index — `iwmm10/chestxray-capstone-splits`
- Model checkpoints — `iwmm10/baseline`
- Images — `nih-chest-xrays/data` (public)

---

## Models trained so far

Three single-model candidates have been trained and scored on the same patient-level test split. All three predict the same 14 pathologies, evaluated at their own validation-tuned per-class thresholds.

| Model | Resolution | Test Macro AUROC | Notebook | Thresholds |
|---|---|---:|---|---|
| DenseNet-121 | 224x224 | ~0.795 | `02_densenet_baseline.ipynb` / `03_evaluation.ipynb` | `results/thresholds_densenet.json` |
| ResNet50 | 224x224 | ~0.783 | `04_resnet50_baseline.ipynb` / `05_resnet_evaluation.ipynb` | `results/thresholds_resnet50.json` |
| **ConvNeXt-Tiny (BCE)** | 320x320 | **0.8158** | `convnext-tiny-320-training.ipynb` | `results/convnext_thresholds.json` |

**ConvNeXt-Tiny is currently the strongest single-model candidate** and the recommended checkpoint to build on for Grad-CAM / IoBB and downstream work — it has the best AUROC, threshold-tuned macro F1 (0.4236 vs 0.3034 at a naive 0.5 cutoff), and a well-calibrated output (test macro ECE 0.0258, so no temperature scaling was needed). A focal-loss variant was also tried and rejected — it scored marginally higher on macro AUROC (0.8145 vs 0.8121 on validation) but was worse on 8 of 14 individual pathologies, so BCE was kept. Full details, per-class AUROC, and a sex/age/view-position robustness breakdown are in `docs/convnext_experiment_summary.md` and `results/convnext_robustness_summary.csv`.

Known weak spots on the ConvNeXt model, relevant to anyone building on it: Pneumonia (AUROC 0.696) and Infiltration (AUROC 0.708) remain the hardest classes, and AUROC drops for patients 60+ (and especially the 80+ group, though that subgroup is only 54 images).

The ConvNeXt-Tiny checkpoint referenced by the notebook (`convnext_tiny_320_best.pth`) is not committed to git — it lives as a Kaggle notebook output and needs to be re-attached as a Kaggle input dataset for anyone continuing from it.

---

## Status

Complete — dataset construction, patient-level splitting, DenseNet-121 and ResNet50 baselines, ConvNeXt-Tiny 320 training (BCE and focal-loss variants), per-class evaluation with bootstrap confidence intervals, threshold calibration, probability calibration (ECE), and demographic robustness analysis — all for the ConvNeXt model (see table above).

In progress / not started — Grad-CAM and IoBB evaluation against the radiologist bounding boxes (`loc_tune` / `loc_report` splits, see Data splits above), inference pipeline, and interface. These are assigned to other team members and out of scope for this update.

Results are reported once the full pipeline is complete.

---

*Research prototype. Not intended, validated, or suitable for clinical use.*
