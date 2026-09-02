# NYC Taxi Trip Analytics Pipeline

A batch data engineering pipeline built on Databricks, following the medallion (Bronze → Silver → Gold) architecture, with a Databricks SQL dashboard and a scheduled Workflow on top.

## Overview

This project ingests NYC taxi trip data (from Databricks' built-in `samples.nyctaxi.trips` dataset), cleans and transforms it, aggregates it into business-ready summary tables, and surfaces the results in an interactive dashboard — all orchestrated as a scheduled, unattended job.

## Architecture

```
samples.nyctaxi.trips (raw sample data)
        │
        ▼
  bronze_taxi          — raw ingestion, no transformation
        │
        ▼
  silver_taxi           — cleaned: nulls/invalid fares removed,
        │                 duplicates dropped, trip_duration_minutes derived
        ▼
  ┌─────┴─────┐
  ▼           ▼
gold_taxi_   gold_taxi_
by_zip       by_hour     — business-ready aggregates
  │           │
  └─────┬─────┘
        ▼
  Databricks SQL Dashboard — bar chart (trips by zip),
                              line chart (fare/volume by hour)
```

## Key steps

1. **Bronze** — raw data copied as-is into a managed Delta table for lineage/history tracking (`DESCRIBE HISTORY`).
2. **Silver** — PySpark transformation: filters invalid fares/distances, casts timestamps, derives `trip_duration_minutes`, removes duplicates.
3. **Gold** — two SQL aggregation tables:
   - `gold_taxi_by_zip`: trip count, avg fare/distance/duration per pickup zip, with a `HAVING COUNT(*) >= 20` filter to exclude statistically unreliable low-volume zips.
   - `gold_taxi_by_hour`: trip count and avg fare by hour of day.
4. **Dashboard** — "NYC Taxi Trip Patterns: Pickup Zones & Hourly Demand," built on the gold tables.
5. **Orchestration** — a Databricks Workflow (`nyc_taxi_pipeline`) chains bronze → silver → gold_zip + gold_hour (parallel branches) on a daily schedule.

## Key findings

- **Manhattan (100xx zip codes)** dominates trip volume with short, low-cost rides (~2-2.5 miles, ~$10-11 avg fare).
- **Airport zones (11xxx)** show far fewer trips but the highest average fares, driven by distance.
- **Trip volume** rises through the evening and drops after midnight.
- **Average fare peaks at 5-6am** (likely fewer, longer-distance rides such as early airport runs) and dips during the 7-8pm evening rush, when trips are shorter and more frequent — a genuinely counterintuitive pattern worth digging into further with `avg_distance_miles` by hour.

## Data quality note

An early version of `gold_taxi_by_zip` showed wildly unstable average fares (ranging ~$10-$46) because low-trip-count zip codes were skewing the average. Fixed by filtering to zips with `trip_count >= 20`, a reminder that averages are only trustworthy at sufficient sample size.

## Tools used

Databricks notebooks (PySpark + SQL), Delta Lake, Databricks SQL dashboards, Databricks Workflows (Jobs).
