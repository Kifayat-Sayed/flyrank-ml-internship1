# Human-in-the-Loop Content Refresh Opportunity Scoring
## A Client-Grouped Evaluation of Search-Content Review Prioritization

**Author:** Kifayat Sayed  
**Lane:** Content Refresh / Opportunity Scoring  
**Repository:** https://github.com/Kifayat-Sayed/flyrank-ml-internship1  
**Date:** November 2026

## 0. Abstract

This study asks whether historical content and search-performance signals can help editorial teams prioritize content items for human review, including for clients not represented during model training. It analyzes 30,000 anonymized content records across 32 pseudonymized clients, with a binary decline label derived from the supplied trend direction. A leakage-conscious preprocessing pipeline and a 120-tree Random Forest are compared with a recency-and-impressions rule baseline using Precision@20 and Precision@50, with a client-grouped holdout as the primary evaluation. On the held-out clients, the rule baseline achieves 80% Precision@20 and 70% Precision@50, exceeding the Random Forest at 65% and 62%, respectively, while the random-row diagnostic produces substantially more optimistic model results. The resulting workflow is an experimental decision-support and review-queue prototype, not a proven prospective predictor or evidence that refreshing content causes improved search performance.

## 1. Problem framing

The unit of analysis is a content item. The system produces an ordered review queue so editorial teams can decide which pages to inspect first when review capacity is limited. A human editor must verify accuracy, intent, quality, business value and whether a refresh is appropriate. The cost of a false positive is wasted editorial effort or an unnecessary change; the cost of a false negative is a potentially useful review opportunity being delayed.

## 2. Data safety and data contract

- Source: `data/raw/content_refresh_anonymized.csv`, supplied for the FlyRank ML internship.
- Size: 30,000 rows, 44 source columns and 45 columns after deriving the target.
- Unit: one content item per row; 32 pseudonymized clients.
- Duplicate content IDs: 0.
- Overall observed decline-label prevalence: 54.2%.
- Supplied data includes trailing 90-day metrics and recent 30-day comparisons. Exact calendar dates are not established by the available documentation.
- Target: `is_declining_label = 1` when `trend_direction == "down"`; otherwise 0.

Excluded from model features: `content_id`, `client_id`, `trend_direction`, `trend_pct`, recent/previous 30-day impressions, clicks and sessions, `provider_used`, and `model_used`. The client identifier is used only for grouped validation, never as a predictor. The target is derived from supplied trend information and is not an independently verified future event. No client names, raw URLs, private queries, credentials, row-level predictions or content identifiers are included in this report.

## 3. Baseline

The Week 4 rule baseline selects content items whose last update was at least 180 days ago and whose trailing 90-day impressions are at least 3,000. Eligible items are ranked by 90-day impressions. It is transparent, easy to audit and evaluated on the same records and ranking metrics as the model.

## 4. Model and analysis

The feature frame contains 32 predictors: 23 numeric and 9 categorical. Numeric values use median imputation with missingness indicators. Categorical values use most-frequent imputation and one-hot encoding with unknown-category handling and minimum frequency 10. Preprocessing is fitted inside the training pipeline.

The model is a Random Forest with 120 trees, minimum leaf size 5, `class_weight="balanced_subsample"`, and seed 42. Its positive-class probability is used as the ranking score.

## 5. Evaluation

The primary evaluation uses one client-grouped holdout with a 20% test fraction and seed 42. Training contains 23,837 rows from 25 clients; testing contains 6,163 rows from 7 unseen clients. Client overlap is zero. The test-set decline-label prevalence is 3,149 / 6,163 = 51.1%.

| Method | Precision@20 | Precision@50 |
|---|---:|---:|
| Week 4 rule baseline | 80% | 70% |
| Random Forest | 65% | 62% |
| Model minus baseline | −15 percentage points | −8 percentage points |

A random-row split is reported only as a diagnostic: the Random Forest achieves 100% Precision@20 and 98% Precision@50, while the baseline achieves 60% and 64%. These much stronger random-row results are not the headline estimate because the split permits client overlap. At a 0.50 threshold on the grouped holdout, accuracy is 59%; positive-class precision and recall are 0.60 and 0.61.

## 6. Interpretation and limitations

The rule baseline outperforms the Random Forest on both primary grouped ranking metrics. The study therefore does not support replacing the transparent baseline with the model.

Key limitations:
1. The label is derived from supplied trend direction, not a verified future decline event.
2. The precise calendar window and strict feature-before-label chronology are not established.
3. A single grouped holdout tests seven unseen clients; it is not a repeated cross-client or future-time evaluation.
4. The analysis does not estimate the causal effect of refreshing content.
5. Reason codes are transparent heuristics, not SHAP explanations or causal drivers.
6. Missingness and the anonymized single-snapshot structure may limit transportability.

## 7. Ranked recommendations

1. Retain the recency-and-impressions rule as the first-pass shortlist because it outperformed the model on the primary holdout.
2. Require human editorial verification before any content change.
3. Keep the Random Forest as a challenger model for error analysis, not a replacement.
4. Establish explicit feature and label windows and test on a later time period with a future outcome.
5. Monitor performance by client and content segment; require repeated grouped/time-aware evidence before deployment.

## 8. Reproducibility

- Notebook: [work/notebooks/capstone.ipynb](../notebooks/capstone.ipynb)
- Metrics: [work/artifacts/capstone_metrics.json](../artifacts/capstone_metrics.json)
- Chart: [work/artifacts/grouped_precision_at_k.png](../artifacts/grouped_precision_at_k.png)
- Repository: https://github.com/Kifayat-Sayed/flyrank-ml-internship1
- Seed: 42.

Open the capstone notebook in Google Colab and run all cells from top to bottom. It clones the repository, loads the anonymized CSV, performs the random-row diagnostic and client-grouped primary evaluation, and writes the aggregate metrics JSON and comparison chart to `work/artifacts/`. Verify the zero-overlap assertion and reported split sizes before relying on the results.

## 9. Acknowledgments and data credit

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
