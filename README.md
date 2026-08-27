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
      01_data_preparation.ipynb     Patient-level splitting (Colab; run and frozen)
      02_densenet_baseline.ipynb    DenseNet-121 training
      03_evaluation.ipynb           Per-class metrics, CIs, threshold calibration
      04_resnet50_baseline.ipynb    ResNet50 training
      05_resnet_evaluation.ipynb    ResNet50 scored with the same pipeline
    results/                        Per-class thresholds and evaluation output

Notebooks 02–05 run on Kaggle. Attach both datasets as inputs, enable a GPU, and run in order.

Data and checkpoints are hosted on Kaggle, not committed here:

- Splits and image index — `iwmm10/chestxray-capstone-splits`
- Model checkpoints — `iwmm10/baseline`
- Images — `nih-chest-xrays/data` (public)

---

## Status

Complete — dataset construction, patient-level splitting, DenseNet-121 and ResNet50 baselines, per-class evaluation with bootstrap confidence intervals, and threshold calibration.

In progress — Grad-CAM and IoBB evaluation, demographic robustness analysis, probability calibration, inference pipeline and interface.

Results are reported once the full pipeline is complete.

---

*Research prototype. Not intended, validated, or suitable for clinical use.*
