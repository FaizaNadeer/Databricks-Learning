# NYC Taxi Fare Prediction with MLflow

A machine learning project on Databricks using scikit-learn and MLflow to predict taxi fares from trip features, with full experiment tracking, model registration via Unity Catalog, and batch inference — built as a follow-on to an earlier batch/streaming taxi pipeline project.

## Overview

Trains and compares two regression models to predict `fare_amount` from `trip_distance` and `trip_duration_minutes`, tracks both as MLflow experiments, registers the best-performing model to the Unity Catalog Model Registry, and runs batch inference to generate a predictions table — then investigates the model's largest prediction errors to understand *why* they occurred, not just that they did.

## Pipeline

```
silver_taxi (from earlier batch project)
        │
        ▼
  pandas sample (n=5000) → train/test split (80/20)
        │
        ├──► LinearRegression        ─┐
        │    (MLflow run 1)           │
        │                             ├─► Compared in MLflow
        └──► RandomForestRegressor   ─┘   Experiments UI
             (MLflow run 2)
                    │
                    ▼
        Registered as workspace.default.taxi_fare_predictor
                    │
                    ▼
        Batch inference → fare_predictions (Delta table)
```

## Results

| Model | RMSE | R² |
|---|---|---|
| Linear Regression | 3.37 | 0.892 |
| Random Forest (n_estimators=100, max_depth=10) | 3.21 | 0.902 |

Random Forest selected and registered as the production model — modest but consistent improvement on both metrics.

**What the metrics mean:** RMSE is the average prediction error in dollars (lower is better); R² is the fraction of fare variance explained by the model (0-1, higher is better). With just two features, the model explains ~90% of fare variance — fare is largely mechanically determined by distance and time, so this isn't surprising, but it's a useful reminder that simple, obvious features often carry most of the signal.

## Error investigation

Sorting `fare_predictions` by absolute error surfaced a cluster of large misses. Initial hypothesis: airport flat-rate fares (e.g., JFK↔Manhattan's known $52 flat rate) were confusing a distance/duration-only model.

**Investigation disproved the initial hypothesis in the way expected, and revealed a better one:** trips at the $52 flat rate consistently fell in a tight ~16-20 mile / 20-70 minute range, and the model predicted these well (within a few dollars) — because the flat rate happens to correlate strongly with a consistent distance/duration range, not because the model "understands" flat-rate pricing.

The *actual* worst outliers were two specific rows with **near-zero trip_distance/trip_duration_minutes (0.1-0.2 miles, <2 minutes) paired with $52-55 actual fares** — a clear data quality issue (likely a GPS/meter logging error), not a modeling limitation. The model's ~$50 underprediction on these rows is the *correct* response given the (bad) input features.

**Takeaway:** the model has no significant systemic blind spot; its two largest errors trace to bad source data, not weak modeling. Confirming an initial hypothesis was wrong before accepting it was a necessary step — the first plausible-sounding explanation for an anomaly isn't automatically the right one.

## Key concepts learned

- MLflow experiment tracking: `mlflow.start_run()`, `log_param`, `log_metric`, `log_model`, and comparing runs side-by-side in the Experiments UI.
- Unity Catalog model registry requires three-level naming (`catalog.schema.model_name`), consistent with UC table naming.
- Model signatures (via `input_example`) are required for UC registry validation.
- Batch inference pattern: load a registered model by URI (`models:/catalog.schema.model/version`), score a DataFrame, write results to a Delta table for downstream querying/dashboarding.
- Diagnosing model errors by inspecting the underlying feature values for worst-case predictions, rather than accepting the first plausible explanation.

## Tools used

Databricks Serverless (Python) compute, pandas, scikit-learn, MLflow (tracking + Unity Catalog Model Registry), Delta Lake.
