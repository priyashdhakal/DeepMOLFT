# Deep MOLFT

Source code, hyperparameter configurations, and preprocessing scripts for the
**Deep MOLFT** (Multi-Omics Late Fusion Technique) framework. This repository is
provided to ensure full reproducibility of the reported results.

---

## Repository contents

| File | Purpose |
|------|---------|
| `arguments.py` | All configuration (dataset, model, training, Optuna arguments) |
| `dataloader.py` | Data loading, train/test split, correlation-based feature selection, scaling |
| `model.py` | Deep MOLFT late-fusion architecture |
| `main.py` | Training / evaluation entry point |
| `shap_from_logs.py` | SHAP-based biomarker attribution (computed on the held-out test set) |
| `PlotUtils.py` | ROC, confusion matrix, and plotting utilities |
| `best_params.json` | Per-dataset optimized hyperparameters |
| `requirements.txt` | Exact dependency versions |

---

## Implementation details

**Architecture.** Each omics modality (DNA methylation, miRNA, mRNA) is processed by an
independent fully connected branch (128 → 64 → 32 units, ReLU, L1 = 0.001, dropout
0.5 / 0.3 / 0.2). The branch outputs are concatenated and passed through three tuned
fused dense layers, then an output layer (sigmoid for binary, softmax for multiclass).

**Weight initialization.** All `Dense` layers use the Keras default **Glorot (Xavier)
uniform** initializer; biases are zero-initialized (not overridden in code).

**Training.**

| Setting | Value |
|---------|-------|
| Optimizer | Adam |
| Learning rate | tuned per dataset (see `best_params.json`) |
| Loss | binary / categorical cross-entropy |
| Batch size | 32 |
| Max epochs | 500 |
| Early stopping | monitor `val_loss`, patience 50, restore best weights |
| Class imbalance | class-balanced weighting |
| Random seed | 123 |

**Hyperparameter optimization.** Fused-layer sizes/dropout and learning rate optimized
per dataset with **Optuna** (objective: validation accuracy; direction: maximize).
Search ranges: learning rate 1e-5–1e-1 (log); fused sizes 64–256 / 32–128 / 16–64;
dropout 0.1–0.5 each.

---

## Data and preprocessing

- **Datasets:** BRCA, ROSMAP, LUAD, THCA, UCEC.
- **Source:** preprocessed multi-omics data from
  [MOGONET](https://github.com/txWang/MOGONET) and
  [GREMI](https://github.com/Yaolab-fantastic/GREMI). The same preprocessed inputs are
  used for Deep MOLFT and all baselines to ensure an identical-input comparison.
- **Train/test split:** a fixed, predefined partition encoded in each dataset
  (`Split` column: 1 = train, 0 = test), identical to the MOGONET/GREMI benchmark split
  and aligned across modalities (e.g., BRCA: 612 train / 263 test, ~70/30, stratified).
- **Leakage control:** correlation-based feature elimination (threshold **0.85**) and
  `StandardScaler` are **fit on the training partition only**, then applied to the
  held-out test set. The test set is used for neither feature selection nor scaling fit.
- **SHAP:** biomarker attributions are computed on the **held-out test partition**, not
  the training data.

---

## Environment

| Component | Version |
|-----------|---------|
| Python | 3.x |
| TensorFlow / Keras | 2.15.0 |
| CUDA | 11.6 (driver 510.108.03) |
| GPU | NVIDIA TITAN Xp (12 GB) |
| Key libraries | `tensorflow`, `scikit-learn`, `optuna`, `shap`, `numpy`, `pandas` |

A fixed random seed (`123`) is set across NumPy, TensorFlow, and Keras for
deterministic, exactly reproducible runs.

Deep MOLFT is intentionally lightweight: a fully connected late-fusion backbone with
compact modality-specific branches and aggressive upstream feature reduction. 

| Metric | Value |
|--------|-------|
| Hardware | Single NVIDIA TITAN Xp (12 GB) |
| Optimizer / batch size | Adam / 32 |
| Max epochs (early stopping patience) | 500 (50) |
| **Full training time (500 epochs)** | **≈ 19.26 s** |

## Citation

If you use this code, please cite the Deep MOLFT manuscript. (Citation to be added
upon publication.)
