# Hypo/Hyperglycemia Detection from Wearable ECG

This project implements an ECG-based pipeline to detect **hypoglycemia** (CGM < 70 mg/dL) and **hyperglycemia** (CGM > 180 mg/dL) using wearable data.  
ECG is used to extract features, while CGM is used only to assign labels.

## Pipeline
- Load wearable **ECG (250 Hz)** plus device summary signals (HR, HRConfidence) and **CGM (5-min)** data.
- Clean ECG and detect **R-peaks** (NeuroKit2).
- Apply quality control using **HRConfidence > 90** and session-level discard rules.
- Delineate fiducial points (P/Q/R/S/T) and extract:
  - morphology features (amplitudes, intervals, slopes, distances)
  - RR intervals + time-domain HRV
  - circadian time encoding (sin/cos hour)
- Label beats by aligning each beat to the **next CGM sample** (forward merge) with **15 min tolerance**.
- Aggregate features into **non-overlapping 1-minute windows** (main evaluation unit).

## Models
All classifiers are **Random Forest**. Training is **per subject** with **5-fold temporal validation** using **1-hour blocks** to reduce leakage.

Implemented variants include morphology-only, HRV(+time), morphology+HRV, and a fusion model.

### Multi-threshold fusion (MF Fusion)
- Train multiple beat-level models at different glucose cutoffs.
- Aggregate beat probabilities per minute into “probability descriptors”.
- Fuse descriptors with HRV + time features at interval level.

Cutoffs used:
- Hypo side: 55, 60, 65, 70, 75, 80, 85, 90 mg/dL  
- Hyper side: 150, 165, 180, 200, 225, 250 mg/dL

## Results (from the report)
MF Fusion performs best on average:
- Hypoglycemia: **AUC 0.813 ± 0.155**
- Hyperglycemia: **AUC 0.755 ± 0.156**

## Data expected
The notebook expects a dataset folder containing:
- ECG files (e.g., `*_ECG.csv`)
- wearable sensor summary files
- CGM glucose files (e.g., `glucose.csv`)

Update the dataset root path in the notebook before running.

## How to run
1. Create a Python environment.
2. Install dependencies: `numpy`, `pandas`, `scipy`, `scikit-learn`, `neurokit2`, `matplotlib` (plus `tqdm`, `joblib`).
3. Run the notebook: `Code_SignalLab.ipynb`.

## Repo contents
- `Code_SignalLab.ipynb` — full pipeline (data indexing → QC → features → models → evaluation).
- `Report_SignalLab.pdf` — method and results.

## Notes
Research/replication code. Not a medical device.
