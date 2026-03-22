# Interpretable Machine Learning on Protein-Protein Interaction Networks

## Overview
Hybrid deep learning approach combining Graph Attention Networks (GAT) 
and ESM-2 Protein Language Model for PPI prediction.

## Results
- Accuracy: 97.72% (Hybrid Fusion)
- AUC: 0.994
- False Positive reduction: 21.2% vs GAT alone
- Generalization Gap: 0.66% (no overfitting)

## Notebooks
- `Hybrid_Model.ipynb` — GAT + ESM-2 Fusion, XAI & Attention Analysis
- `XGBoost.ipynb` — XGBoost baseline with ESM-2 embeddings
- `esm2.ipynb` — ESM-2 biochemical analysis + MLP classifier

## Dataset
[PPI Dataset](https://www.kaggle.com/datasets/spandansureja/ppi-dataset)
- 73,110 protein pairs (balanced)
- 10,344 unique proteins
- Hard Negatives validated (71.4% overlap)

## University of Patras — Computer Engineering Dept. — 2026
