# Intrinsic vs. Post-hoc Explainability in Credit Risk Prediction

**A Quantitative Evaluation of Faithfulness, Stability, and Predictive Performance**

---

## Overview

This repository contains the code, data, and results for a comparative study of intrinsic and post-hoc explainability methods applied to credit risk prediction. Using the UCI **Default of Credit Card Clients** dataset, we benchmark five machine learning and deep learning models, each paired with an appropriate explanation technique, and evaluate them across three dimensions:

- **Predictive Performance** — ROC-AUC, PR-AUC, F1-Score, Brier Score, ECE
- **Explanation Faithfulness** — Feature-removal perturbation (|ΔP| at k = 1, 3, 5, 10)
- **Explanation Stability** — Spearman rank correlation of attributions across seeds and folds

---

## Models and Explanation Methods

| Model | Type | Explanation Method | Category |
|---|---|---|---|
| Logistic Regression | Linear Baseline | Model Coefficients | Intrinsic |
| LightGBM | Gradient Boosting | TreeSHAP | Post-hoc |
| CatBoost | Gradient Boosting | TreeSHAP | Post-hoc |
| MLP | Deep Neural Network | Integrated Gradients | Post-hoc |
| TabNet | Attention-based DNN | TabNet Masks | Intrinsic |

---

## Key Results

All values below are taken directly from the experiment output files and represent mean scores across 5-fold cross-validation with 3 random seeds (15 runs total).

### Predictive Performance

| Model | ROC-AUC (mean ± std) | PR-AUC (mean ± std) | F1 (mean ± std) |
|---|---|---|---|
| CatBoost | 0.7890 ± 0.0095 | 0.5666 ± 0.0150 | 0.5481 ± 0.0129 |
| MLP | 0.7828 ± 0.0088 | 0.5563 ± 0.0146 | 0.5405 ± 0.0118 |
| LightGBM | 0.7825 ± 0.0104 | 0.5555 ± 0.0147 | 0.5421 ± 0.0123 |
| Logistic Regression | 0.7741 ± 0.0076 | 0.5407 ± 0.0124 | 0.5367 ± 0.0120 |
| TabNet | 0.7731 ± 0.0099 | 0.5350 ± 0.0135 | 0.5311 ± 0.0143 |

### Explanation Faithfulness (mean |ΔP| at k=5)

| Model + Method | |ΔP| at k=5 |
|---|---|
| MLP + Integrated Gradients | 0.2377 |
| CatBoost + TreeSHAP | 0.2072 |
| TabNet + TabNet Masks | 0.1776 |
| LightGBM + TreeSHAP | 0.1056 |

### Explanation Stability (mean Spearman ρ)

| Model + Method | Spearman ρ |
|---|---|
| LightGBM + TreeSHAP | 0.8683 |
| CatBoost + TreeSHAP | 0.8584 |
| MLP + Integrated Gradients | 0.7877 |
| TabNet + TabNet Masks | 0.2394 |

### Calibration (Post-calibration)

| Model | Brier Score | ECE |
|---|---|---|
| MLP (Isotonic) | 0.1348 | 0.0059 |
| MLP (Platt) | 0.1347 | 0.0104 |
| LightGBM | 0.1537 | 0.1250 |
| CatBoost | 0.1770 | 0.2000 |

### Runtime

| Model | Training Time (mean) | Explanation Time per Sample (mean) |
|---|---|---|
| LightGBM | 0.21 s | 0.046 ms |
| TabNet | 30.85 s | 0.036 ms |
| MLP | 9.91 s | 0.614 ms |
| CatBoost | 10.70 s | 0.722 ms |

---

## Dataset

**Default of Credit Card Clients** — UCI Machine Learning Repository

| Property | Value |
|---|---|
| Total Samples (after cleaning) | 29,965 |
| Total Features (engineered) | 38 |
| Numerical Features | 29 |
| Categorical Features | 9 |
| Default Rate | 22.1% |
| Class Distribution (non-default : default) | 23,335 : 6,630 |

The original dataset contains 30,000 credit card clients from Taiwan (April–September 2005). After removing 35 exact duplicate rows to prevent train/test leakage, 29,965 samples were retained. Feature engineering added 15 derived features (e.g., average utilization, payment-to-bill ratio, delay statistics).

---

## Methodology

1. **Data Preprocessing** — Duplicate removal, categorical encoding, and feature engineering (15 derived features added to the original 23).
2. **Stratified K-Fold Cross-Validation** — 5-fold CV repeated across 3 random seeds (42, 2024, 7) for a total of 15 evaluation runs per model.
3. **Model Training** — Each model trained with early stopping where applicable (MLP: patience 8, max 50 epochs; TabNet: patience 10, max 100 epochs).
4. **Explainability Evaluation**:
   - *Faithfulness*: Top-k feature removal perturbation test measuring absolute prediction change (|ΔP|) at k = 1, 3, 5, 10 over 500 samples per fold.
   - *Stability*: Spearman rank correlation of feature attributions computed across 105 seed-fold pairs.
   - *Random Baselines*: Model-specific random attribution baselines to validate that real explanations outperform chance.
5. **Additional Analyses** — Platt/Isotonic calibration, cost-sensitive threshold evaluation, SMOTE ablation, raw vs. engineered feature ablation, and groupwise fairness diagnostics across Education, Marriage, and Sex.

---

## Repository Structure

```
├── dl-vf.ipynb                  # Main experiment notebook
├── dataset/
│   └── default of credit card clients.xls
├── figures/
│   ├── roc_curves.png
│   ├── pr_curves.png
│   ├── calibration_curve.png
│   ├── pooled_confusion_matrices.png
│   ├── faithfulness_curves.png
│   ├── stability_comparison.png
│   ├── runtime_comparison.png
│   └── cost_sensitive_validation_thresholds.png
├── results/
│   ├── config.json              # Experiment configuration
│   ├── metrics/                 # Per-fold and aggregated metrics
│   ├── explanations/            # Feature attributions and importance scores
│   ├── tables/                  # Paper-ready result tables (01–08)
│   ├── models/                  # Saved model checkpoints (.pt, .zip)
│   ├── figures/                 # EDA and model-specific visualizations
│   ├── final_paper_interpretation.md
│   └── final_run_report.md
├── papers/                      # Reference literature
├── proposal.pdf                 # Project proposal
├── LiteratureReviewSheet.pdf    # Literature review summary
└── *.pdf                        # Final paper
```

---

## Experiment Configuration

| Parameter | Value |
|---|---|
| Cross-Validation Folds | 5 |
| Random Seeds | 42, 2024, 7 |
| Inner Validation Split | 15% |
| Explanation Samples | 500 per fold |
| SHAP Background Samples | 100 |
| Integrated Gradients Steps | 25 (ig_steps) / 32 (ig_n_steps) |
| Batch Size | 1024 |
| Feature Mode | Engineered |
| Mixed Precision | Enabled |
| MLP Learning Rate / Weight Decay | 0.001 / 0.0001 |
| TabNet Learning Rate / Weight Decay | 0.02 / 1e-5 |

---

## Requirements

- Python 3.8+
- PyTorch
- PyTorch TabNet
- LightGBM
- CatBoost
- Scikit-learn
- Captum
- SHAP
- Pandas, NumPy
- Matplotlib, Seaborn
- imbalanced-learn

### Installation

```bash
pip install torch pytorch-tabnet lightgbm catboost scikit-learn captum shap pandas numpy matplotlib seaborn imbalanced-learn
```

---

## Usage

1. Clone this repository:
   ```bash
   git clone https://github.com/huzaifakhallid/Intrinsic-vs-Post-hoc-Explainability-in-Credit-Risk-Prediction.git
   cd Intrinsic-vs-Post-hoc-Explainability-in-Credit-Risk-Prediction
   ```

2. Open and run the main notebook:
   ```bash
   jupyter notebook dl-vf.ipynb
   ```

3. Results, metrics, explanations, and figures will be saved to the `results/` directory.

---

## Citation

```bibtex
@misc{khallid2025credit_risk_xai,
  title   = {Intrinsic vs. Post-hoc Explainability in Credit Risk Prediction: A Quantitative Evaluation of Faithfulness, Stability, and Predictive Performance},
  author  = {Khallid, Huzaifa},
  year    = {2025}
}
```

---

## License

This project is intended for academic and research purposes.
