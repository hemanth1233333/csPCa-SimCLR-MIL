# SimCLR + ResNet50-ViT + Attention-MIL for csPCa Detection

**CSC 8851 Deep Learning — Spring 2026**  
Hemanth Kumar Mulluri · Shiv Brahmbhatt · Georgia State University

## Overview
Weakly-supervised detection of clinically significant prostate cancer (csPCa)
from multi-parametric MRI (mpMRI) using SimCLR self-supervised pre-training
combined with a Gated Attention Multiple Instance Learning (MIL) classifier.
Trained using biopsy labels only — no voxel-level annotations required.

## Dataset
- **PI-CAI** public dataset — Fold 0
- 184 patients (77 csPCa positive / 107 benign)
- MRI sequences used: T2W, ADC, HBV (3 channels)
- 80/20 stratified train/val split

## How to Run
1. Open `csPCa_real.ipynb` in **Google Colab**
2. Set runtime to **A100 GPU** (Runtime → Change runtime type → A100)
3. Mount your Google Drive when prompted
4. Update these paths in **Cell 3** to match your Drive:
   - `IMG_DIR` → folder containing PI-CAI fold0 patient images
   - `LABEL_CSV` → path to `Metadata with lesion info.csv`
5. Run all cells in order (Cell 1 → Cell 13)

## Notebook Structure
| Cell | Description |
|------|-------------|
| 1 | Install dependencies (monai, SimpleITK, scikit-learn) |
| 2 | Mount Drive + imports |
| 3 | Configuration (paths, hyperparameters) |
| 4 | Dataset loader (PiCAIFold0Dataset) |
| 5 | MRI-safe SimCLR augmentations |
| 6 | Model architecture (ResNet50-ViT + Gated Attention MIL) |
| 7 | SimCLR pre-training (NT-Xent loss) |
| 8 | Attention-MIL fine-tuning (Focal loss) |
| 9 | Evaluation — ROC, PR curves, results table |
| 10 | Attention saliency visualization |
| 11-13 | Ablation study (No SSL, No ViT, Avg/Max Pool MIL) |

## Model Architecture
- **Encoder:** ResNet50 backbone + Vision Transformer (67.6M parameters)
- **SSL:** SimCLR with NT-Xent loss (temperature=0.07)
- **Aggregation:** Gated Attention MIL
- **Loss:** Focal Loss (α=0.75, γ=2.0)

## Key Hyperparameters
| Parameter | Value |
|-----------|-------|
| Image size | 224×224 |
| Embed dim | 768 |
| ViT heads | 8 |
| ViT layers | 6 |
| SimCLR epochs | 30 (early stop) |
| MIL epochs | 30 (early stop) |
| MIL learning rate | 1e-4 |

## Results (PI-CAI Fold 0)
| Method | Supervision | AUROC | AUPRC |
|--------|-------------|-------|-------|
| PI-CAI nnU-Net † | Voxel-level masks | 0.872 | 0.658 |
| Attn-MIL (no SSL) | Biopsy label only | 0.791 | 0.521 |
| **SimCLR + ResNet50-ViT-MIL (Ours)** | Biopsy label only | **0.735** | **0.701** |

† Requires dense voxel-level annotations

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
