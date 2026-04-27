# SimCLR + ResNet50-ViT + Attention-MIL for csPCa Detection

**CSC 8851 Deep Learning — Spring 2026**

---

## What This Project Does

This project detects clinically significant prostate cancer (csPCa) from multi-parametric MRI scans using a weakly-supervised deep learning pipeline. Instead of requiring radiologists to manually annotate tumors at the voxel level, we only use biopsy outcome labels — positive or negative — to train the model.

The pipeline combines two stages: first, a SimCLR self-supervised pre-training step that teaches the encoder to learn useful MRI representations without any labels, and second, a Gated Attention Multiple Instance Learning (MIL) classifier that aggregates slice-level features into a patient-level prediction.

---

## Dataset

We use the publicly available **PI-CAI dataset**, specifically **Fold 0**, which contains 184 patients — 77 diagnosed with csPCa and 107 benign cases. Each patient has three MRI sequences: T2-weighted (T2W), Apparent Diffusion Coefficient (ADC), and High b-value (HBV), which are stacked as a 3-channel input. The dataset is split 80/20 into training and validation sets using stratified sampling to preserve the class balance.

### Downloading the Dataset

The dataset is hosted on Kaggle. Here is how to get it:

1. Visit the dataset page: https://www.kaggle.com/datasets/varshithpsingh/prostate-cancer-pi-cai-dataset
2. Log into your Kaggle account and click **Download**
3. Extract the ZIP file — the folder structure should look like this:

```
prostate-cancer-pi-cai-dataset/
├── fold0/
│   ├── 10000/
│   ├── 10001/
│   ├── 10002/
│   └── ...
└── Metadata with lesion info.csv
```

Each numbered folder is a patient ID and contains their MRI files in `.mha` or `.nii.gz` format.

4. Upload the extracted folder to your **Google Drive**. We recommend organizing it like this:

```
MyDrive/
└── PI-CAI/
    ├── fold0/
    └── Metadata with lesion info.csv
```

> **Note:** This project only uses Fold 0. You do not need to download or upload the other folds.

---

## How to Run

1. Open `csPCa_real.ipynb` in **Google Colab**
2. Go to **Runtime → Change runtime type** and select **A100 GPU**
3. Run Cell 2 — it will prompt you to mount your Google Drive
4. In **Cell 3**, update the two paths to match where you saved the dataset:
   - `IMG_DIR` — path to the `fold0/` folder, e.g. `/content/drive/MyDrive/PI-CAI/fold0`
   - `LABEL_CSV` — path to the metadata file, e.g. `/content/drive/MyDrive/PI-CAI/Metadata with lesion info.csv`
5. Run all cells from top to bottom (Cell 1 through Cell 13)

The full pipeline takes roughly 2–3 hours on an A100 depending on Colab availability.

---

## Notebook Structure

| Cell | What It Does |
|------|--------------|
| 1 | Installs required packages (monai, SimpleITK, scikit-learn) |
| 2 | Mounts Google Drive and imports all libraries |
| 3 | Sets all configuration — file paths and hyperparameters |
| 4 | Defines the dataset loader (PiCAIFold0Dataset) |
| 5 | Defines MRI-safe SimCLR data augmentations |
| 6 | Builds the model (ResNet50 + ViT encoder + Gated Attention MIL head) |
| 7 | Runs SimCLR self-supervised pre-training with NT-Xent loss |
| 8 | Fine-tunes the MIL classifier with Focal loss |
| 9 | Evaluates the model — generates ROC and PR curves, prints results table |
| 10 | Visualizes attention saliency maps over MRI slices |
| 11–13 | Ablation experiments (No SSL, No ViT, Average/Max Pooling MIL) |

---

## Model Architecture

The encoder is a **ResNet50 backbone** with a **Vision Transformer (ViT)** head, totaling approximately 67.6 million parameters. Slice-level embeddings are aggregated into a bag-level representation using **Gated Attention MIL**, which learns to weight each slice by its relevance to the final diagnosis.

Pre-training uses **SimCLR** with the NT-Xent contrastive loss at temperature 0.07. Fine-tuning uses **Focal Loss** (α=0.75, γ=2.0) to handle the class imbalance between positive and benign cases.

---

## Hyperparameters

| Parameter | Value |
|-----------|-------|
| Image size | 224 × 224 |
| Embedding dimension | 768 |
| ViT attention heads | 8 |
| ViT transformer layers | 6 |
| SimCLR pre-training epochs | 30 (with early stopping) |
| MIL fine-tuning epochs | 30 (with early stopping) |
| MIL learning rate | 1e-4 |

---

## Results

All results are reported on the PI-CAI Fold 0 validation set.

| Method | Supervision | AUROC | AUPRC |
|--------|-------------|-------|-------|
| PI-CAI nnU-Net † | Voxel-level masks | 0.872 | 0.658 |
| Attn-MIL (no SSL) | Biopsy label only | 0.791 | 0.521 |
| **SimCLR + ResNet50-ViT-MIL (Ours)** | Biopsy label only | **0.735** | **0.701** |

† The nnU-Net baseline requires dense voxel-level tumor annotations at training time, which are expensive to produce. Our method uses only patient-level biopsy labels.

Our model achieves notably higher AUPRC than the no-SSL baseline, demonstrating that SimCLR pre-training meaningfully improves precision-recall performance even in the weakly supervised setting.

---

## Requirements

```
monai==1.3.0
SimpleITK
scikit-learn
torch
torchvision
pandas
numpy
matplotlib
```

All dependencies are installed automatically in Cell 1 of the notebook. No manual setup is needed beyond having a Colab environment with GPU access.

---

## Acknowledgements

Dataset provided by the PI-CAI challenge organizers. This project was completed as part of CSC 8851 Deep Learning at [Your University], Spring 2026.
