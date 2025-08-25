# Oura Ring Stress Prediction Project

This repository contains a machine‑learning pipeline for predicting **next‑day stress levels** from daily Oura Ring summaries.

## Quick Start
```bash
# 1) Create environment
pip install -r requirements.txt  # see notebook headers for minimal deps

# 2) Run/inspect model
jupyter notebook stress_predict_extended.ipynb

# 3) Generate plots from saved predictions
python visualize_output.py --input model_predictions.csv --outdir viz/
```
Artifacts produced by the notebook:
- `model_lightgbm.json` — trained LightGBM booster (JSON).
- `shap_values.pkl` — SHAP contributions & metadata.
- `dashboard_data.json` — per‑day features, SHAP, probs, predictions.
- `model_predictions.csv` — per‑day actual vs predicted labels (+probs).

## Data (inputs)
Daily CSV exports from Oura Cloud with a `day` column (YYYY‑MM‑DD):
- **sleep** (`rem_sleep_duration`, `deep_sleep_duration`, `total_sleep_duration`)
- **activity** (`steps`, `high_activity_time`, `medium_activity_time`, `sedentary_time`, `inactivity_alerts`, `active_calories`, `score`)
- **physiology/all_stats** (`temperature_trend_deviation`, `average_hrv`, `average_resting_heart_rate`, `respiratory_rate`)
- **SpO2** (`spo2_percentage`)
- **stress** (`day_summary` ∈ {restored, normal, stressful})

Yesterday’s activity is shifted **+1 day** to predict next‑day stress.

## Features
Fifteen numeric features assembled by merging on `day` and filling any missing values with column means. The target label is the categorical `day_summary` encoded to integers.

## Modeling
- **TimeSeriesSplit (5 folds)** to respect temporal order.
- **LightGBM (multiclass)** as the primary model.
- **Baselines**: Logistic Regression & RandomForest for reference.
- **Class weighting** to reduce imbalance.
- **Explainability**: SHAP (LightGBM `pred_contrib=True`) for global and local importance.

## Notebook Steps
1. Load & clean CSVs; snake‑case columns; shift yesterday’s activity to align with today’s stress.
2. Merge to a single daily table; encode target.
3. EDA: correlation heatmap; class balance check.
4. Baselines: Logistic & RF (weighted F1).
5. LightGBM with class weights; 5‑fold TSCV; collect out‑of‑fold predictions & probabilities.
6. Train final model on full data; export booster JSON.
7. Compute SHAP contributions; save `shap_values.pkl` and `dashboard_data.json`.
8. Write `model_predictions.csv` for downstream plotting.
9. Optional: t‑SNE visualisation of feature space.

## Visualisation
`visualize_output.py` creates:
- Static: confusion matrix (`confusion_matrix.png`) & count plot (`predicted_distribution.png`).
- Interactive (Plotly HTML in `viz/`): confusion matrix, per‑day actual vs predicted scatter, and predicted‑label distribution.

Usage:
```bash
python visualize_output.py --input model_predictions.csv --outdir viz/ --html
```

## Results (example snapshot on current data)
- Imbalanced labels; class‑weighted LightGBM improves weighted F1 vs baselines.
- Top drivers (mean |SHAP|): sleep duration + quality, resting HR, HRV, prior‑day activity score.

> Treat numbers as illustrative; they’ll update as you add more days.

## Reproducibility
- Python ≥3.9
- Key deps: `lightgbm`, `scikit-learn`, `pandas`, `numpy`, `matplotlib`, `plotly`.
- Random seed fixed where applicable.

## Future Work
- Calibrated probabilities (Platt/Isotonic).
- Rolling‑window re‑training & drift checks.
- Add wearable context (tags: illness, travel) as features.

