# Capstone Report — Content Refresh Opportunity Scoring

- **Author:** Sukanya Bodke
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/sukanya9020/flyrank-ml-internship
- **Date:** September 2026


## 0. Abstract

This project investigates which content items should be prioritized for review or refresh using observed search performance, content age, update history, visibility, and engagement signals. The analysis uses an anonymized dataset containing 30,000 content records and 44 columns from the FlyRank ML Internship dataset. A transparent opportunity-scoring baseline was created and compared with a Random Forest model for predicting recent performance trend. The Random Forest model did not outperform the simple baseline on the held-out test set, with an MAE of 75.64 compared with 71.89 for the baseline. The final output is therefore used as a decision-support ranking that helps content practitioners identify items for human review rather than as a guarantee of future performance improvement.

## 1. Problem framing

The project supports the decision of which content items should be reviewed first for a possible refresh.

**Unit of analysis:** Each row represents one anonymized content item.

**Output:** A ranked opportunity score from 0 to 100, together with an action recommendation and reason codes.

**Human action:** A content or SEO practitioner can use the ranked list to decide which pages should receive closer review, possible refresh consideration, monitoring, or protection from unnecessary changes.

**Cost of a wrong call:** Prioritizing the wrong content can waste editorial resources, while overlooking an important opportunity can delay a useful content review. Therefore, the ranking is intended to support human judgment rather than automatically determine actions.

**Why data and ML help:** The dataset contains multiple signals describing content age, update history, search visibility, engagement, and recent performance. Combining these signals provides a consistent way to prioritize a large number of content items. A machine-learning experiment was also tested to determine whether a more complex model could improve on the transparent baseline.

## 2. Data safety

The analysis uses the public-safe, anonymized FlyRank internship dataset containing 30,000 content records and 44 columns.

The analysis uses content-level performance and content characteristics such as:

- content_age_days
- days_since_last_update
- impressions_90d
- clicks_90d
- ctr
- avg_position
- word_count
- impressions_last_30d
- impressions_prev_30d
- trend_pct

The pseudonymous identifiers `content_id` and `client_id` were not used as predictive features. They are used only to identify or group records where necessary.

No client names, domains, private search queries, credentials, or raw private exports are included in the analysis or recommendations.

### Leakage considerations

The fields `trend_direction` and `trend_pct` describe recent performance and can contain information about the outcome being analyzed. Therefore, they are not used as predictive features in the Random Forest model. The Random Forest predicts `trend_pct` using the six independent features listed in the methodology section.

The baseline opportunity score is different from the predictive model. It intentionally uses `trend_pct` as one of its observed prioritization signals. Therefore, the baseline is treated as an interpretable heuristic rather than a leakage-free predictive model.

The results should be interpreted as decision-support signals based on observed data, not as evidence of causal effects or guaranteed future search performance..

## 3. Baseline

A transparent rule-based opportunity score was created before evaluating the machine-learning model. The purpose of the baseline was to provide an interpretable reference point that could be compared with the more complex Random Forest approach.

The baseline combines five observed signals:

- Content age — 25%
- Days since last update — 25%
- Search visibility measured by impressions — 20%
- Lower CTR — 15%
- Negative recent trend — 15%

Each signal was converted to a percentile-based score and combined into an overall opportunity score from 0 to 100.

The baseline is intentionally simple and interpretable. A higher score indicates that several observed signals suggest the content may deserve closer review.

For the predictive evaluation, a simple mean prediction was also used as the numerical baseline. On the same held-out test set, the simple baseline achieved:

- **MAE:** 71.89
- **R²:** approximately 0.000

The Random Forest achieved an MAE of 75.64 and an R² of -0.015. Therefore, the Random Forest did not improve on the simple baseline in this experiment.

The interpretable opportunity score was retained as the primary prioritization mechanism because it provides clear reasons for why an item received a higher ranking.

## 4. Model / analysis

A Random Forest regression model was tested to determine whether a machine-learning approach could improve prediction of recent content performance.

The model predicts `trend_pct`, which represents the observed recent performance trend.

### Model features

The Random Forest uses exactly these six features:

- `content_age_days`
- `days_since_last_update`
- `impressions_90d`
- `ctr`
- `avg_position`
- `word_count`

The identifiers `content_id` and `client_id` were excluded from the model.

The fields `trend_direction` and `trend_pct` were also excluded from the feature set because they describe the performance outcome being predicted.

The Random Forest was configured with 100 trees, a maximum depth of 8, and `random_state=42`.

The model was compared with a simple mean-prediction baseline using the same target and held-out test data.

The model experiment is treated as an evaluation of whether additional model complexity provides useful predictive value. Because the Random Forest did not outperform the baseline, it is not used as the primary recommendation mechanism.

## 5. Evaluation

The Random Forest was evaluated using a held-out test set created with an 80/20 train-test split and `random_state=42`.

After cleaning missing and invalid values, the model dataset contained 22,301 usable records:

- **Training records:** 17,840
- **Test records:** 4,461

The same test set was used to evaluate both the Random Forest and the simple mean-prediction baseline.

### Evaluation metrics

| Method | MAE | R² |
|---|---:|---:|
| Simple baseline | 71.89 | approximately 0.000 |
| Random Forest | 75.64 | -0.015 |

The Random Forest produced a higher MAE than the simple baseline. Its negative R² also indicates that it did not provide useful additional predictive performance on the held-out data compared with the baseline.

This evaluation is therefore treated as a negative model result rather than evidence that the Random Forest is superior.

A limitation of the evaluation is that the experiment used a random train-test split rather than a time-based or client-grouped split. Future work could test time-aware validation and grouped validation to better reflect how the system might be used in practice.
## 6. Interpretation

The analysis shows that the highest-ranked content opportunities generally combine several observed signals: older content, a longer period since the last update, meaningful search visibility, lower CTR, and negative recent performance.

The highest-ranked content item received an opportunity score of approximately **84.53**. It was **504 days old**, had not been updated for **104 days**, had **2,294 impressions** over the trailing 90 days, a **0.00% CTR**, and a recent trend of **-88.8%**.

The baseline score is intentionally interpretable. Each component represents a specific observed signal, making it possible for a practitioner to understand why an item received a high opportunity score.

The machine-learning experiment produced an important negative result. The Random Forest did not outperform the simple baseline, so the model is not treated as the main source of recommendations.

A key observation is that more complex modeling did not automatically provide better predictive performance. This supports keeping the final prioritization approach simple and explainable for this dataset and use case.

The rankings should still be reviewed by a human practitioner because the dataset does not capture every factor that can affect a content decision, such as content quality, business importance, search intent, or strategic priorities.
## 7. Recommendation

The final output is a ranked list of content items based on the transparent opportunity score. Each item receives an opportunity score, a recommended action, and reason codes that describe the signals contributing to its ranking.

The four recommendation categories are:

- **Refresh:** Higher opportunity signals that justify detailed content review and possible refresh consideration.
- **Review:** Moderate opportunity signals that should be examined by a practitioner.
- **Monitor:** Weaker opportunity signals that may be observed over time.
- **Protect:** Lower opportunity scores where unnecessary changes may be avoided.

A practitioner could use the output by starting with the highest-ranked items, checking the reason codes, reviewing the actual content, and then considering business importance and search intent before taking action.

The recommendations are not automatic instructions. A high opportunity score does not prove that a page needs a refresh, and a refresh is not guaranteed to improve search performance.

The confidence in the ranking should therefore be considered **moderate for prioritization purposes and limited for predicting future outcomes**. The main evidence is the consistency and interpretability of the observed signals, while the Random Forest experiment did not provide additional predictive performance over the simple baseline.

The final system is best viewed as a human-in-the-loop prioritization tool rather than an autonomous content decision system.

## 8. Reproducibility

The analysis was developed in Google Colab using the notebook:

`work/notebooks/capstone.ipynb`

The notebook contains the complete analysis workflow, including data loading, baseline scoring, model training, evaluation, recommendations, and visualizations.

### Reproduction steps

1. Clone or download the repository.
2. Open `work/notebooks/capstone.ipynb` in Google Colab or Jupyter Notebook.
3. Run the notebook from top to bottom.
4. The notebook loads the public-safe anonymized dataset from the repository.
5. The baseline opportunity score and Random Forest experiment are generated.
6. The evaluation metrics and ranked recommendations are produced.

### Main libraries

The analysis uses:

- Python
- pandas
- NumPy
- scikit-learn
- Matplotlib

### Randomness and reproducibility

The train-test split uses `random_state=42`.

The Random Forest model also uses `random_state=42` and 100 estimators with a maximum depth of 8.

The notebook was successfully executed from top to bottom without errors before submission.

The evaluation uses a held-out test set, but the experiment does not claim a sealed or blind evaluation. Future work should consider time-aware and client-grouped validation for a stronger assessment of generalization.


## 9. Acknowledgments & Data Credit

This project was completed as part of the FlyRank ML Internship and uses the public-safe anonymized dataset provided for the internship.

Built on the FlyRank ML Internship dataset: https://flyrank.ai

The analysis follows the public-safe requirements of the internship and does not expose client-identifying information, private queries, credentials, domains, or raw private exports.
