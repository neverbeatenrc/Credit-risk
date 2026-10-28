# Power BI Dashboard Guide

Run the numbered notebooks in order, ending with `05_powerbi_prep.ipynb`. Import the five CSVs from `powerbi_exports/` into Power BI Desktop.

## Export tables

| Table name | File | Purpose |
| --- | --- | --- |
| `PortfolioSegments` | `portfolio_segments.csv` | Overall portfolio totals plus loan-purpose and purpose/home-ownership segments. |
| `WoeIvDetail` | `woe_iv_detail.csv` | Binned counts, WOE, IV contributions, and observed default rates. |
| `ScoredTest` | `scored_test_customers.csv` | Out-of-sample model probabilities, scorecard-style points, risk bands, and actual test outcomes. |
| `ModelPerformance` | `model_performance.csv` | Cross-validation, held-out metrics, threshold trade-offs, and random-split PSI in one table. |
| `FeatureImportance` | `feature_importance.csv` | Logistic-regression coefficients and absolute coefficient sizes. |

These five exports are the final Power BI tables. Keep them separate because they have different levels of aggregation. `PortfolioSegments` contains overall, loan-purpose, and purpose/home-ownership rows; filter by `segment_level` so aggregate levels are not added together. The PSI row compares random train and test splits only; it is not temporal monitoring.

## Suggested pages

1. **Portfolio overview:** filter `segment_level` to `overall` for totals, or `loan_intent` / `loan_intent_home` for segments.
2. **Model performance:** ROC AUC, Gini, KS, average precision, Brier score, and the threshold trade-off. Label these as held-out test results.
3. **Risk segments:** record count and observed default rate by the relative low/medium/high bands in `ScoredTest`.
4. **Explainability:** coefficient table or bar chart and WOE/IV summary.
5. **Monitoring concept:** filter `ModelPerformance` to `evaluation = random_train_vs_test` for the PSI demonstration. Real PSI needs dated comparison periods.

## Example DAX measures

Create these measures after naming the imported tables as above:

```DAX
Scored Test Records = COUNTROWS(ScoredTest)

Observed Defaults (Test) = SUM(ScoredTest[actual_default])

Observed Default Rate (Test) =
DIVIDE([Observed Defaults (Test)], [Scored Test Records])

Average Predicted Default Probability =
AVERAGE(ScoredTest[predicted_default_probability])

Average Scorecard Points =
AVERAGE(ScoredTest[scorecard_points])

High Relative-Risk Records =
CALCULATE(
    COUNTROWS(ScoredTest),
    ScoredTest[risk_band_relative] = "High"
)

Portfolio Records =
CALCULATE(
    SUM(PortfolioSegments[records]),
    PortfolioSegments[segment_level] = "overall"
)

Portfolio Defaults =
CALCULATE(
    SUM(PortfolioSegments[defaults]),
    PortfolioSegments[segment_level] = "overall"
)

Portfolio Default Rate = DIVIDE([Portfolio Defaults], [Portfolio Records])
```

Format rates and probabilities as percentages. The scorecard-style points use illustrative scaling parameters (600 points at 50:1 good-to-bad odds, 20 points per doubling of default odds); they are not regulated scorecard points. The `risk_band_relative` values are quantiles of this particular test sample, not portable underwriting cutoffs. The 0.50 predicted-default flag is a teaching example and must not be presented as an approval or rejection policy.
