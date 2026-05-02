# Final Run Report

## What Changed
- Added CatBoost-focused paired statistical test output.
- Corrected cost-sensitive evaluation to use validation-only threshold selection per run.
- Added raw-vs-engineered ablation, model-specific random faithfulness baselines, extended fairness diagnostics, and richer local case studies.
- Rewrote the paper interpretation so it depends on saved results instead of fixed favorable language.

## Key Updated Results
- Highest mean ROC-AUC: **CatBoost** (0.7890).
- Lowest validation-threshold test cost won most often by **CatBoost** (3/3 scenarios).
- **CatBoost** was the strongest predictive/cost-sensitive option on this run.
- Highest mean |ΔP| at k=5: **MLP + IntegratedGradients** (0.2377).
- **MLP + Integrated Gradients** was the strongest faithfulness result on this run.
- Model-specific random baseline comparison at k=5: CatBoost vs Random-CatBoost: Δ|ΔP|=0.1624; LightGBM vs Random-LightGBM: Δ|ΔP|=0.0868; MLP vs Random-MLP: Δ|ΔP|=0.1968; TabNet vs Random-TabNet: Δ|ΔP|=0.1295.
- Highest available stability: **LightGBM + TreeSHAP** (mean Spearman ρ=0.8683).
- **LightGBM + TreeSHAP** was the strongest stability result on this run.
- Best calibration by Brier/ECE ordering: **MLP (Platt)** (Brier=0.1347, ECE=0.0104).
- Fastest training runtime: **LightGBM** (0.21s mean).
- The combined stability/calibration/runtime edge was mixed on this run, so the notebook reports the individual winners above instead of forcing a single claim.
- Fastest explanation runtime per sample: **TabNet + TabNetMasks** (0.036 ms/sample).
- **TabNet masks** were fast, but this run does not support a claim of intrinsic superiority over stronger post-hoc alternatives.
- Engineered features improved ROC-AUC for all ablation models in this run.
- Groupwise descriptive diagnostics were computed for **EDUCATION, MARRIAGE, SEX**, including group size, default rate, predicted positive rate, TPR, FPR, FNR, Brier score, and ROC-AUC where defined.
- These fairness outputs are descriptive diagnostics only and should not be interpreted as causal or normative fairness claims.

## Warnings
- preprocessing: Removed 35 exact duplicate rows to prevent train/test leakage.

## Remaining Limitations
- DEBUG mode is suitable for pipeline verification but not for final inferential claims.
- Stability and paired significance results may be underpowered when only one seed/fold is available.
- Fairness diagnostics remain descriptive and dataset-bound.