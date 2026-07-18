# ICE Delay Prediction — Project Reasoning & Clear Picture

**Team:** Manikanta Engalligi · abhinay-sambherao  
**Task:** Regression · Target: `delay_in_min` · Metric: **MAE (minutes)**

---

## 1. What is this project?

We predict **how many minutes late** an ICE train stop will be, using:
- **Operational data** (station, time, train, stop position, etc.)
- **Historical weather** (temperature, wind, rain, etc.)

**Not classification** — we predict delay **in minutes**, not just “delayed yes/no”.

---

## 2. What does MAE mean?

**MAE = Mean Absolute Error (in minutes)**

- It is **not** “the train is X minutes late”
- It is **how wrong the prediction is on average**

Example: MAE = 10.3 → on average, our guess is off by ~10.3 minutes.

**Lower MAE = better model.**

---

## 3. Train vs test (months)

| Split | Months | Purpose |
|-------|--------|---------|
| **Train** | July + August 2024 (~255k rows) | Model learns |
| **Test** | September 2024 (~122k rows) | Model checked (never seen in training) |

**Time-based split** — no random shuffle. Realistic: train on past, test on future month.

---

## 4. What data do we store?

| Data | Where | In GitHub? |
|------|-------|------------|
| Raw DB (~54 GB) | Downloaded locally (Hugging Face) | **No** |
| Processed Parquet | `Notebooks/data/processed/` | **No** (too big) |
| Results JSON | `Notebooks/data/reference/` | **Yes** |
| Final model | `final_model.joblib` | Usually local |

**Why Parquet?** Compressed, fast, keeps types, proves merged data exists **on disk**.

**Merge key:** `eva` (station) + planned departure hour.

---

## 5. Two feature sets (weather ablation)

Same rows, different columns:

1. **Operational only** — no weather (answers RQ1)
2. **Operational + weather** — full features (answers RQ2)

We compare MAE with vs without weather on the **same model**.

---

## 6. Model groups — why each?

| Group | Models | Why |
|-------|--------|-----|
| **Naive baseline** | Mean, Median | “Can ML beat always guessing one number?” |
| **Linear + regularisation** | Linear, Ridge, Lasso, Elastic Net | Course topics; fair linear comparison |
| **Bagging** | Random Forest (+ tuned) | Strong tabular non-linear baseline |
| **Boosting** | Gradient Boosting, XGBoost, LightGBM | Same family, 3 engines — fair compare |

**Why not others?**
- No classification models (wrong task)
- No clustering (unsupervised)
- No CNN/deep nets (tabular data → trees work better)
- No KNN/SVM at scale (slow / weak on 250k+ categorical rows)

---

## 7. Final results (September test)

| Result | Test MAE | Meaning |
|--------|----------|---------|
| **Naive median** (always 4 min) | **9.48 min** | Best overall — still hard to beat |
| **Best ML: XGBoost + weather** | **10.31 min** | Best learned model |
| Tuned Random Forest + weather | 10.42 min | Good, but worse than XGBoost |
| Ridge + weather | ~10.59 min | Linear ceiling |
| Linear operational only | ~11.2 min | Weak vs trees |

---

## 8. Is naive linear?

**No.**

- **Naive median** = always predict **4** (constant). No features. Not ML.
- **Linear models** = use features with weighted sum.

Naive wins because many stops are near “a few minutes late” and exact prediction is noisy.

---

## 9. What did we prove?

1. **ML helps vs simple linear** — trees/boosting beat Ridge/Linear.
2. **Boosting beat RF** — XGBoost best ML (~10.31 vs ~10.42–10.48).
3. **Weather helped XGBoost a little** — not much for RF in summer months.
4. **Nobody beat naive median** — delay magnitude is hard; constant guess is strong baseline.
5. **Honest conclusion** — features add value, but **perfect delay prediction remains difficult**.

---

## 10. Research questions — short answers

| RQ | Answer |
|----|--------|
| **RQ1** | Operational features help partially — XGBoost ~10.31 min, but not better than naive 9.48 min |
| **RQ2** | Weather: small gain for XGBoost; negligible for tuned RF in Jul–Sep |
| **RQ3** | Temperature & wind matter most in importances, but overall MAE effect is weak in summer |

---

## 11. Notebook pipeline (01–09)

| NB | Role |
|----|------|
| 01 | Project definition, RQs, MAE |
| 02–04 | Get, clean, merge data → Parquet on disk |
| 05 | EDA |
| 06 | Features + train/test split |
| 07 | All models + leaderboard |
| 08 | RF tuning, full leaderboard, pick winner |
| 09 | Save final model + project summary |

---

## 12. Why 80k sample for boosting?

- Full train = 255k rows → very slow for sklearn GB + one-hot categories
- **80k random sample from Jul–Aug** for fair compare of 3 boosters
- **Test still full September (~122k)** — evaluation unchanged

RF Section 2 used full train; boosting Section 3 used sample for speed (documented).

---

## 13. One-line project story (for viva opening)

> “We merged ICE operational data with hourly weather, split Jul–Aug train and Sep test, compared naive baselines through linear, Random Forest, and three boosting models using MAE. XGBoost with weather was our best ML model at 10.31 minutes error, but the naive median at 9.48 minutes still performed best overall.”