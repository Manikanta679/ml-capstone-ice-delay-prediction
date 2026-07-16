# ICE Train Delay Prediction — ML Capstone

Predict **how many minutes late** an ICE train stop will be, using Deutsche Bahn operational data and Open-Meteo historical weather.

**Team:** Manikanta Engalligi · abhinay-sambherao

## Problem

- **Task:** Regression  
- **Target:** `delay_in_min`  
- **Primary metric:** MAE (Mean Absolute Error, in minutes)  
- **Scope:** ICE trains only · months **2024-07, 2024-08, 2024-09**

Classification (delayed / not delayed) is **not** a primary modeling goal for this project.

## Data sources

| Source | Link |
|--------|------|
| Deutsche Bahn | [piebro/deutsche-bahn-data](https://huggingface.co/datasets/piebro/deutsche-bahn-data) |
| Weather | [Open-Meteo Historical API](https://open-meteo.com/en/docs/historical-weather-api) |

Raw DB data (~54 GB) is **not** in this repo. Notebooks download monthly Parquet shards from Hugging Face and save filtered ICE files locally.

## Notebooks

| # | Notebook | Status |
|---|----------|--------|
| 01 | Project Definition & Data Understanding | Updating (regression + MAE + multi-month) |
| 02 | Data Acquisition & Initial Inspection | Updating (3 months) |
| 03 | Data Cleaning & Standardization | Updating (3 months) |
| 04 | Data Integration (Merge DB + Weather) | Updating (3 months) |
| 05 | Exploratory Data Analysis | Updating (delay minutes focus) |
| 06 | Feature Engineering | Planned |
| 07 | Baseline Regression Models (MAE) | Planned |
| 08 | Advanced Regression Models | Planned |
| 09 | Evaluation, Tuning & Weather Ablation | Planned |
| 10 | Final Pipeline & Conclusion | Planned |

## Setup

```bash
pip install -r requirements.txt
```

Run notebooks from the `Notebooks/` folder in order. Each notebook loads shared config from `Notebooks/data/reference/*.json` and writes Parquet / JSON artifacts to disk.

## Project structure

```
ML Project/
├── Notebooks/           # Jupyter notebooks (01–10)
│   └── data/
│       ├── reference/   # JSON configs & reports (committed)
│       └── processed/   # Parquet files (gitignored — regenerate locally)
├── requirements.txt
└── README.md
```

## License & attribution

- Deutsche Bahn data: CC BY 4.0  
- Open-Meteo: open API with attribution  
- OpenStreetMap/Nominatim: ODbL (station geocoding)
