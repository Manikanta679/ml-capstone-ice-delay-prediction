# ICE Train Delay Prediction — ML Capstone

Predict **how many minutes late** an ICE train stop will be, using Deutsche Bahn operational data and Open-Meteo historical weather.

**Team:** Manikanta Engalligi · abhinay-sambherao

**Status:** Complete (Notebooks 01–09)

---

## Problem

| Item | Value |
|------|--------|
| **Task** | Regression |
| **Target** | `delay_in_min` (minutes; negative = early) |
| **Primary metric** | **MAE** (Mean Absolute Error, in minutes) |
| **Scope** | ICE trains only · **2024-07, 2024-08, 2024-09** |

Classification (delayed / not delayed) is **not** a primary modeling goal. RMSE is not used as a required metric.

---

## Results (September 2024 test set)

| Model | Test MAE |
|-------|----------|
| Naive median (always 4 min) | **9.48 min** ← best overall |
| **Final ML: Tuned Random Forest + weather** | **10.42 min** ← best ML model |
| Ridge + weather (baseline) | 10.59 min |
| Linear / Ridge (operational only) | ~11.21 min |

**Train / test split (time-based, no random shuffle):**
- **Train:** July + August 2024 (255,062 rows)
- **Test:** September 2024 (121,964 rows)

**Weather ablation (RQ2):** Same tuned Random Forest with vs without weather → **+0.0026 min** MAE gain (no meaningful improvement in Jul–Sep).

Full details: `Notebooks/data/reference/final_model_selection.json`

---

## Research questions

| RQ | Question | Finding |
|----|----------|---------|
| **RQ1** | Can operational features predict delay in minutes? | Partially — RF (~10.42 min) beats linear models, not naive median (9.48 min) |
| **RQ2** | Does weather improve prediction? | No meaningful gain for tuned RF (+0.003 min MAE) |
| **RQ3** | Which weather variables matter most? | Temperature & wind — weak overall effect in summer |

---

## Data sources

| Source | Role | Link |
|--------|------|------|
| Deutsche Bahn | ICE delays, stations, times | [piebro/deutsche-bahn-data](https://huggingface.co/datasets/piebro/deutsche-bahn-data) |
| Open-Meteo | Hourly weather | [Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) |
| Nominatim / OSM | Station geocoding (`eva` → lat/lon) | [OpenStreetMap](https://nominatim.openstreetmap.org/) |

Raw DB data (~54 GB) is **not** in this repo. Notebooks download monthly Parquet from Hugging Face and save ICE subsets locally.

---

## Merge strategy

Left join railway → weather on:
1. **Station:** `eva`
2. **Time:** planned departure floored to hour (`departure_planned_hour_naive`)

Merged data saved as **`ice_weather_merged_YYYY-MM.parquet`** on disk.

---

## Notebooks

| # | File | Status |
|---|------|--------|
| 01 | `01_Project_Definition.ipynb` | Done |
| 02 | `02_Data_Acquisition.ipynb` | Done |
| 03 | `03_Data_Cleaning.ipynb` | Done |
| 04 | `04_Data_Integration.ipynb` | Done |
| 05 | `05_Exploratory_Data_Analysis.ipynb` | Done |
| 06 | `06_Feature_Engineering.ipynb` | Done |
| 07 | `07_Baseline_Models.ipynb` | Done |
| 08 | `08_Model_Tuning_and_Evaluation.ipynb` | Done |
| 09 | `09_Final_Pipeline_Export.ipynb` | Done |

Run in order from `Notebooks/`. Each notebook loads `data/reference/*.json` and reads/writes Parquet in `data/processed/`.

---

## Data on disk (Parquet)

**Why Parquet?** Compressed, fast, keeps column types, reloadable across notebooks — proves merged data exists on disk (not only in memory).

### `Notebooks/data/processed/` (gitignored — regenerate locally)

| File | Description |
|------|-------------|
| `ice_raw_YYYY-MM.parquet` | ICE-only raw data |
| `ice_standardized_YYYY-MM.parquet` | Timestamps → Europe/Berlin |
| `ice_cleaned_YYYY-MM.parquet` | Canceled stops removed |
| `weather_by_station_YYYY-MM.parquet` | Hourly weather per station |
| **`ice_weather_merged_YYYY-MM.parquet`** | **Merged ICE + weather** |
| `ice_modeling_ready_YYYY-MM.parquet` | ML-ready features |
| `ice_train.parquet` | Train (Jul + Aug) |
| `ice_test.parquet` | Test (Sep) |
| `models/final_rf_model.joblib` | Saved final model |

### `Notebooks/data/reference/` (JSON — commit to git)

| File | Description |
|------|-------------|
| `project_config.json` | Scope, target, MAE, months |
| `final_model_selection.json` | Final model + MAE + RQ answers |
| `weather_ablation_report.json` | With vs without weather |
| `full_project_pipeline_summary.json` | Master pipeline summary |
| `project_complete.json` | Completion flag |

---

## Setup

```bash
pip install -r requirements.txt