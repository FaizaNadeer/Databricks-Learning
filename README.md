# Databricks Learning

A collection of hands-on Databricks projects built while learning the platform end-to-end — batch engineering, real-time streaming, machine learning, analytics dashboards, and Unity Catalog governance. Built on Databricks Free Edition, working within its serverless compute and resource-quota constraints.

## Projects

### 🚕 [NYC Taxi Pipeline](./NYC/nyc-taxi-pipeline-README.md)
A batch medallion architecture (Bronze → Silver → Gold) pipeline on NYC taxi trip data, with a Databricks SQL dashboard and a scheduled Workflow orchestrating the full run. Covers PySpark transformations, Delta Lake, data-quality handling for small-sample averages, and dashboard design.
**Notebooks:** `NYC Taxi Learn.ipynb` · **Dashboard:** `NYC Taxi Overview.lvdash.json`

### 📡 [IoT Streaming Pipeline](./IoT%20Streaming/iot-streaming-pipeline-README.md)
A real-time streaming pipeline for simulated IoT sensor data, built entirely with Databricks SQL streaming tables (`CREATE OR REFRESH STREAMING TABLE`) — the SQL-native equivalent of Auto Loader — since Free Edition only exposes a SQL warehouse for this workload. Covers `TRIGGER ON UPDATE`, watermarking, windowed aggregation, and diagnosing real streaming-specific failures (checkpoint/source mismatches, watermark timing).
**Notebook:** `IoT Sensor Monitoring Notebook.ipynb` · **Dashboard:** `IoT Sensor Monitoring Dashboard.lvdash.json`

### 🤖 [Taxi Fare Prediction (ML)](./NYC/taxi-fare-ml-README.md)
A machine learning project predicting taxi fares from trip features, using scikit-learn and MLflow for experiment tracking, comparison, and model registration via the Unity Catalog Model Registry. Includes a genuine error investigation where an initial hypothesis about the model's worst predictions was tested and disproven in favor of a better explanation (a data quality issue, not a modeling blind spot).
**Notebook:** `MLFlow NYC.ipynb`

### 🔐 [Unity Catalog Governance](./unity-catalog-governance-README.md)
Applies documentation (table/column comments, tags), automatic lineage tracing, access control (`GRANT`/`REVOKE`), and dynamic per-user column masking to the existing taxi pipeline tables — in place, without disrupting any live dashboards or jobs. Covers the real tradeoff between governing tables safely vs. physically reorganizing them.

### ✈️ Flight Data Pipeline
A real-time flight tracking pipeline using Lakeflow Declarative Pipelines (`pyspark.pipelines`) and the OpenSky live flight data source, ingesting a continuous stream of aircraft position/velocity events and computing rolling stats (event count, distinct aircraft, max velocity).
**Files:** `transformations/ingest_flights.py`, `transformations/flights_stats.py`

### 🥐 Transaction Analytics
An analytics dashboard on Databricks' built-in `samples.bakehouse.sales_transactions` sample dataset, exploring total sales by product and sales quantity over time.
**Notebook:** `Transaction Notebook.ipynb` · **Dashboard:** `Transaction Dashboard.lvdash.json`

### 🏭 US Emissions Analytics
A geographic and per-capita analytics dashboard on US county-level greenhouse gas emissions data — mapping emissions by location, comparing emissions against population to find per-capita outliers, and ranking the top-emitting states.
**Dashboard:** `Emissions Dashboard.lvdash.json`

## Skills covered across the repo

| Area | Projects |
|---|---|
| Medallion architecture (Bronze/Silver/Gold) | NYC Taxi Pipeline, IoT Streaming |
| Delta Lake | NYC Taxi Pipeline, IoT Streaming, ML |
| Structured Streaming / streaming tables | IoT Streaming, Flight Data Pipeline |
| Lakeflow Declarative Pipelines | Flight Data Pipeline |
| Databricks SQL dashboards | NYC Taxi Pipeline, IoT Streaming, Transaction Analytics, US Emissions |
| Workflows / Jobs orchestration | NYC Taxi Pipeline |
| MLflow (tracking, registry, batch inference) | Taxi Fare Prediction |
| Unity Catalog governance (lineage, access control, masking) | Unity Catalog Governance |
| Geographic / per-capita analysis | US Emissions Analytics |

## Notes

Built entirely on **Databricks Free Edition**, which shaped a few real design decisions worth knowing if you're following along on the same tier: no notebook-based Structured Streaming (streaming tables via SQL warehouse used instead), a hard cap on concurrently active pipelines (drove the decision to collapse Bronze into Silver in the IoT project), and daily serverless compute quotas that occasionally paused work mid-pipeline — each documented in its respective project README along with how it was worked around.
