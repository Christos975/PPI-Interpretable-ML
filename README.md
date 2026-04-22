# Interpretable Machine Learning on Protein-Protein Interaction Networks

<p align="center">
  <img src="figures/figure4_architecture.png" width="900"/>
</p>

> **Diploma Thesis** — University of Patras, Dept. of Computer Engineering & Informatics, 2026
> 
> **Title:** Ερμηνεύσιμη Μηχανική Μάθηση σε Δίκτυα Αλληλεπίδρασης Πρωτεϊνών (PPI)

---

## Overview

This repository contains the implementation of a **Hybrid Deep Learning model** for Protein-Protein Interaction (PPI) prediction, combining:

- **GAT** (Graph Attention Networks) — extracts topological/structural features from the PPI graph
- **ESM-2** (Protein Language Model) — extracts biochemical features from amino acid sequences
- **MLP Classifier** — final binary classifier (interaction / no interaction)

The model is complemented by a full **Explainability (XAI) pipeline** using DeepSHAP, Integrated Gradients, and GAT Attention Weights, as well as **biological validation** via UniProt BLAST and STRING-db.

---

## Key Results

| Model | Accuracy | F1-Score | Precision | Recall | AUC-ROC |
|-------|----------|----------|-----------|--------|---------|
| ESM-2 (Sequence Only) | 80.85% | 76.89% | 97.24% | 63.58% | 0.9422 |
| GAT (Topology Only) | 96.79% | 96.71% | 99.14% | 94.40% | 0.9934 |
| **Hybrid (ESM-2 + GAT)** | **98.22%** | **98.21%** | **99.07%** | **97.37%** | **0.9949** |

### Comparison with External Baselines (same dataset)

| Model | Accuracy | AUC |
|-------|----------|-----|
| ProtBERT + MLP | 79.50% | 0.8978 |
| XGBoost + ESM-2 | 90.67% | 0.9642 |
| AlphaFold-based (Gallo et al.) | 94.62% | 0.9874 |
| **Hybrid (GAT + ESM-2)** | **98.22%** | **0.9949** |

### Overfitting Check
| Model | Train AUC | Test AUC | Gap |
|-------|-----------|----------|-----|
| ESM-2 | 0.9518 | 0.9385 | 1.33% ✅ |
| GAT | 0.9982 | 0.9932 | 0.49% ✅ |
| Hybrid | 1.0000 | 0.9950 | 0.50% ✅ |

---

## Explainability (XAI)

Three complementary methods were applied at two levels of the model:

### DeepSHAP & Integrated Gradients (MLP Classifier level)
Both methods consistently agree:
- **GAT (Topology): ~74%** of the total contribution
- **ESM-2 (Sequence): ~26%** of the total contribution

| Method | GAT Contribution | ESM-2 Contribution |
|--------|-----------------|-------------------|
| DeepSHAP | 73.5% | 26.5% |
| Integrated Gradients | 74.3% | 25.7% |

### GAT Attention Weights (Encoder level)
- Highly **skewed distribution** (mean: 0.1468, 95th percentile: 0.4862)
- The GAT focuses selectively on a small set of **hub proteins**
- Consistent with known biological network topology

---

## Biological Validation

Top-20 SHAP pairs validated via **UniProt BLAST** + **STRING-db**:

| Rank | Protein A | Protein B | STRING Score | Pathway |
|------|-----------|-----------|-------------|---------|
| 1 | DAAM1 | DVL1 | 0.999 ✅ | Wnt Signaling |
| 7 | OXSR1 | SLC12A2 | 0.901 ✅ | Ion Homeostasis |
| 8 | STK11 | STRADA | 0.999 ✅ | Tumor Suppression |
| 12 | NUMB | PRKCZ | 0.741 ✅ | Cell Polarity |
| 15 | PARP9 | DTX3L | 0.999 ✅ | DNA Damage Response |
| 16 | REST | RCOR1 | 0.999 ✅ | Transcriptional Repression |
| 19 | TCERG1 | U2AF2 | 0.767 ✅ | mRNA Splicing |
| 20 | DHFR | FKBP1A | 0.650 🟡 | Metabolism |

**50% of top-20 SHAP pairs** confirmed by STRING-db (score ≥ 0.4)

---

## Repository Structure

```
PPI-Interpretable-ML/
│
├── README.md
├── requirements.txt
├── LICENSE
│
├── notebooks/
│   ├── 01_main_model.ipynb        # GAT + ESM-2 Hybrid, Explainability, Attention
│   ├── 02_baselines.ipynb         # XGBoost + ProtBERT baselines
│   └── 03_esm2_analysis.ipynb     # ESM-2 embeddings & MLP
│
├── figures/
│   ├── figure2_exp_vs_comp.png    # Experimental vs Computational methods
│   ├── figure3_flowchart.png      # Methodology flowchart
│   ├── figure4_architecture.png   # Hybrid model architecture
│   ├── results_thesis.png         # Final comparison table
│   ├── shap_summary.png           # DeepSHAP summary plot
│   ├── shap_contribution.png      # GAT vs ESM-2 contribution (SHAP)
│   ├── ig_analysis.png            # Integrated Gradients analysis
│   ├── attention_analysis.png     # GAT Attention Weights
│   └── overfitting_check.png      # Train/Test learning curves
│
└── data/
    └── README.md                  # Instructions to download from Kaggle
```

---

## Installation

```bash
git clone https://github.com/Christos975/PPI-Interpretable-ML.git
cd PPI-Interpretable-ML
pip install -r requirements.txt
```

---

## Dataset

The dataset is publicly available on Kaggle:

🔗 [PPI Dataset — Kaggle](https://www.kaggle.com/datasets/spandansureja/ppi-dataset)

**Statistics:**
- 73,118 protein pairs (balanced: 36,630 PPI + 36,488 Non-PPI)
- 10,344 unique proteins
- Train/Test split: 80% / 20% (seed=42)

Download and place the files in the `data/` folder:
```
data/
├── positive_protein_sequences.csv
├── negative_protein_sequences.csv
├── esm2_embeddings_1.npy
└── esm2_embeddings_2.npy
```

---

## Model Architecture

The hybrid model processes each protein pair through two parallel branches:

```
Protein A + B
     │
     ├── GAT Encoder ──────────────── emb_A [64] + emb_B [64] → [128]
     │   (2 GATConv layers,                                           │
     │    heads=4, dim=64)                                            ├── Concat [1088] → MLP → P(interaction)
     │                                                                │
     └── ESM-2 Encoder ─────────────  emb_A [480] + emb_B [480] → [960]
         (Pre-trained PLM,
          480-dim embeddings)
```

**MLP Classifier:**
```
Linear(1088→128) → BatchNorm1d → ReLU → Dropout(0.2)
→ Linear(128→64) → ReLU → Linear(64→1) → Sigmoid
```

**Training:** 200 epochs, Adam (lr=0.005), BCEWithLogitsLoss (pos_weight=1.2), seed=42

---

## Reproducibility

All experiments use `seed=42` for full reproducibility:

```python
def set_seed(seed=42):
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    np.random.seed(seed)
    random.seed(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
```

---

## Citation

If you use this work, please cite:

```bibtex
@mastersthesis{ppi_interpretable_2026,
  title  = {Ερμηνεύσιμη Μηχανική Μάθηση σε Δίκτυα Αλληλεπίδρασης Πρωτεϊνών (PPI)},
  author = {[Όνομα Φοιτητή]},
  school = {University of Patras, Dept. of Computer Engineering & Informatics},
  year   = {2026}
}
```

---

## License

MIT License — see [LICENSE](LICENSE) for details.
