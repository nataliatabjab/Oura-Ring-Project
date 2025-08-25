# Data Folder Documentation

CSV exports from **Oura Cloud** summarised **per day**. All files include a `day` column in `YYYY‑MM‑DD` format.

> Date range in current snapshot: **2024‑12‑25 → 2025‑08‑23**.

## Files

### `sleep_*.csv`
- `rem_sleep_duration` — minutes in REM sleep.
- `deep_sleep_duration` — minutes in deep sleep.
- `total_sleep_duration` — total minutes asleep (all stages).

### `dailyactivity_*.csv`
- `steps` — total step count.
- `high_activity_time`, `medium_activity_time` — minutes by intensity.
- `sedentary_time` — minutes sedentary.
- `inactivity_alerts` — Oura inactivity prompts.
- `active_calories` — activity energy expenditure (kcal).
- `score` — Oura activity score (0–100).

### `oura_stats.csv` (readiness/physiology)
- `temperature_trend_deviation` — °C deviation from baseline.
- `average_hrv` — RMSSD (ms).
- `average_resting_heart_rate` — bpm.
- `respiratory_rate` — breaths/min.
> These typically correspond to Oura’s **Readiness** view.

### `dailyspo2_*.csv`
- `spo2_percentage` — nightly average SpO₂ (%).

### `dailystress_*.csv`
- `day_summary` — label for the day: `restored`, `normal`, `stressful`.

## How the model reads these
1. All files are merged on `day` after renaming columns to snake_case.
2. **Yesterday’s activity** is shifted forward one day to align with **today’s** stress.
3. Missing values (if any) are imputed with the column mean.
4. Target is `day_summary` (categorical → integer codes).

## Tips for exporting
- From Oura Cloud → Trends → Export data → choose date range and categories.
- Keep original filenames with date ranges; place all CSVs in `data/`.
- New exports can simply replace current files; the notebook will pick them up.

