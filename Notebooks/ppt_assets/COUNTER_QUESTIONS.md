# ICE Delay Prediction — Counter Questions & Answers (Viva Prep)

Use these as **professor cross-questions** with ready answers.

---

## A. Project basics

**Q: Why regression and not classification?**  
**A:** Professor feedback + RQ focus on **delay magnitude in minutes**. Classification loses how late (5 vs 50 min).

**Q: Why MAE and not RMSE?**  
**A:** MAE is in **plain minutes**, easy to interpret. RMSE punishes big errors more — not required for our scope.

**Q: Why only ICE and 3 months?**  
**A:** Scope control — ICE subset, Jul–Aug–Sep 2024 for manageable data and clear train/test split.

---

## B. Data

**Q: Where is the 54 GB data?**  
**A:** Not in Git. Downloaded from Hugging Face to local disk when running Notebook 02.

**Q: How do you prove merged data exists?**  
**A:** `ice_weather_merged_YYYY-MM.parquet` saved on disk; left join on station + hour documented in JSON.

**Q: Why time split and not random split?**  
**A:** Delays depend on time. Random split leaks future patterns. Sep test = realistic forecast month.

**Q: What if test month is unusual?**  
**A:** Valid limitation — one test month (Sep). Results are for that period only; summer weather effect may differ from winter.

---

## C. Naive baseline

**Q: Why is naive median so good?**  
**A:** Median delay on train is ~4 min. Many stops cluster near small delays. Always guessing 4 is hard to beat when exact prediction is noisy.

**Q: Is naive a machine learning model?**  
**A:** **No.** It uses no features — fixed constant prediction. It is a **benchmark**, not ML.

**Q: Should you report naive as “best model”?**  
**A:** Report honestly: **best overall = naive median**, **best ML = XGBoost**. Different categories.

---

## D. Model choice

**Q: Why these models and not SVM / KNN / Neural nets?**  
**A:** Tabular data at 250k rows with categories — tree ensembles work well. SVM/KNN slow or weak here. NNs need more tuning/data; not course focus.

**Q: Why three boosting models?**  
**A:** Same **boosting family**, different implementations — fair within-family comparison (sklearn vs XGBoost vs LightGBM).

**Q: Why Random Forest AND boosting?**  
**A:** RF = **bagging** ensemble. Boosting = different ensemble idea. Compare both tree families.

**Q: Why Lasso AND Elastic Net if Ridge exists?**  
**A:** Complete regularisation story: L2 (Ridge), L1 (Lasso feature selection), both (Elastic Net).

---

## E. Results interpretation

**Q: Does MAE 10.31 mean trains are 10.31 minutes late?**  
**A:** **No.** It means predictions are **wrong by ~10.31 minutes on average**, not the actual delay.

**Q: Why didn’t ML beat naive?**  
**A:** Delay has high noise, many small delays, weak signal in features for exact minutes. Constant median is a strong simple strategy.

**Q: XGBoost beat RF — why?**  
**A:** Boosting often fits complex tabular patterns better; XGBoost is optimized for speed + regularisation on large data.

**Q: Weather helped XGBoost but not RF much — why?**  
**A:** Different models use features differently. Summer Jul–Sep weather signal is weak overall; XGBoost captured a small gain (~0.24 min), RF did not.

**Q: Is 0.24 min weather gain meaningful?**  
**A:** Small but real for XGBoost. For RF ~0.003 min — **not meaningful** in practice.

---

## F. Methodology / fairness

**Q: Is comparison fair if boosting used 80k and RF used 255k?**  
**A:** Limitation we document. Boosting used same 80k for all three boosters; test always full Sep. RF/full-data comparison is approximate — could retrain XGBoost on full train for production.

**Q: Why one-hot encode station/train?**  
**A:** Categorical features — tree/linear pipelines need numeric input. `handle_unknown="ignore"` for unseen categories in test.

**Q: Did you tune XGBoost?**  
**A:** NB08 tuned **Random Forest** only. XGBoost won with default-ish params from NB07 — honest scope; RF tuning was original plan.

**Q: Why is final model XGBoost if you tuned RF?**  
**A:** NB08 Section 2 picks **best ML on full leaderboard** — XGBoost had lowest test MAE (~10.31 vs tuned RF ~10.42).

---

## G. Leakage & validity

**Q: Any data leakage?**  
**A:** Time-based split reduces leakage. Features built from planned times + weather at planned hour — no future actual delay in features (verify in NB06 feature manifest).

**Q: Negative delay_in_min?**  
**A:** Means **early** (ahead of schedule). MAE still works on signed values.

**Q: Class imbalance?**  
**A:** Regression — not binary classes. Delay distribution is skewed (median 4, mean ~11) — why median naive works well.

---

## H. Technical / tools

**Q: Why Parquet not CSV?**  
**A:** Size, speed, type preservation, standard for ML pipelines.

**Q: What is saved in joblib?**  
**A:** Full pipeline: preprocessing (imputer, one-hot, scaler) + trained model — load once, predict on new rows.

**Q: Pylance says xgboost not found but code runs?**  
**A:** Editor Python path ≠ notebook kernel. `%pip install` in kernel fixes runtime; select same interpreter for linting.

---

## I. Honest limitations (good to say yourself)

1. Single test month (September 2024 only)  
2. Boosting trained on 80k sample for speed  
3. No hyperparameter tuning for XGBoost/LightGBM  
4. Naive median beats all ML models on MAE  
5. Summer months — weather effect may differ in winter  
6. ICE only — not all Deutsche Bahn trains  

---

## J. Quick rapid-fire

| Question | Short answer |
|----------|--------------|
| Target? | `delay_in_min` |
| Metric? | MAE (minutes) |
| Train months? | Jul + Aug 2024 |
| Test month? | Sep 2024 |
| Best overall? | Naive median 9.48 min |
| Best ML? | XGBoost + weather 10.31 min |
| Merge key? | `eva` + departure hour |
| Final saved model? | `final_model.joblib` |
| RQ2 conclusion? | Small weather gain for XGBoost; weak for RF |
| Main proof? | Fair model comparison on same split + ablation |

---

## K. Counter-question YOU can ask professor (optional)

- “Should we treat beating naive median as success, or is strong ML vs linear enough for capstone?”  
- “Would expanding test to Oct–Nov change weather conclusions?”  
- “Is reporting both best-overall and best-ML the right framing?”  

*(Shows you understand nuance — use only if discussion is open.)*