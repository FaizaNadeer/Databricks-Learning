# Real-Time Flight Data Pipeline

A real-time flight tracking pipeline built with Lakeflow Declarative Pipelines, ingesting live aircraft position and velocity data from the OpenSky Network and computing rolling summary statistics.

## Overview

Unlike the IoT project (SQL streaming tables, built to work around Free Edition's SQL-warehouse-only compute), this pipeline uses Databricks' **Lakeflow Declarative Pipelines** Python API (`pyspark.pipelines`) directly, ingesting from a live external data source rather than simulated data — real aircraft currently in flight, tracked via the open OpenSky Network API.

## Pipeline

```
OpenSky Network (live external API)
        │
        ▼
  ingest_flights          — streaming table, raw ingestion via a
        │                    custom PySpark data source
        ▼
  flights_stats            — aggregated table: event count,
                              distinct aircraft, max velocity
```

## Implementation

**Ingestion** (`transformations/ingest_flights.py`):
```python
from pyspark import pipelines as dp
from pyspark_datasources import OpenSkyDataSource
spark.dataSource.register(OpenSkyDataSource)

@dp.table
def ingest_flights():
    return spark.readStream.format("opensky").load()
```
Uses a custom-registered PySpark data source (`OpenSkyDataSource`) to stream live flight state vectors — no landing zone or file simulation needed, since this is a genuinely live external feed.

**Aggregation** (`transformations/flights_stats.py`):
```python
from pyspark import pipelines as dp
from pyspark.sql.functions import *

@dp.table
def flights_stats():
    df = spark.read.table("ingest_flights")
    return (
        df.agg(
            count("*").alias("num_events"),
            countDistinct("icao24").alias("distinct_aircraft"),
            max("velocity").alias("max_velocity"),
        )
    )
```
Computes total events received, the number of distinct aircraft (identified by their ICAO 24-bit address), and the fastest velocity observed in the current data.

## Key concepts

- The `@dp.table` decorator API is Databricks' **Lakeflow Declarative Pipelines** syntax — a Python-native alternative to defining streaming tables in SQL (as used in the IoT project), useful when transformation logic benefits from full PySpark rather than SQL expressions.
- Registering a **custom PySpark data source** (`spark.dataSource.register`) is what makes `spark.readStream.format("opensky")` possible — this is how Spark can be extended to stream from arbitrary external APIs, not just files or message queues.
- Aggregating directly over a full streaming table (rather than a windowed aggregation, as in the IoT project) computes a continuously updated running total, illustrating a different aggregation pattern than time-windowed rollups.

## Tools used

Databricks Lakeflow Declarative Pipelines, PySpark, custom Spark data sources, the OpenSky Network live flight tracking API.
