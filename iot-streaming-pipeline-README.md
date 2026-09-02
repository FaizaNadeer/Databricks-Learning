# Real-Time IoT Sensor Monitoring Pipeline

A streaming data pipeline built entirely with Databricks SQL streaming tables (Auto Loader-equivalent, SQL-native), simulating a fleet of IoT sensors and processing readings incrementally with watermarking and windowed aggregation — built and run on Databricks Free Edition.

## Overview

Since Free Edition compute is limited to a serverless SQL warehouse (no notebook/cluster compute for PySpark Structured Streaming), this pipeline uses `CREATE OR REFRESH STREAMING TABLE` with `read_files`/`STREAM` sources — the pure-SQL path to incremental, self-refreshing streaming pipelines.

## Architecture

```
sensor_raw_landing (plain table, simulates arriving data via periodic INSERTs)
        │
        ▼
  silver_sensor_readings   — STREAMING TABLE
        │                     filters invalid temps, applies WATERMARK
        │                     (2 min delay), TRIGGER ON UPDATE
        ▼
  gold_sensor_5min_avg     — STREAMING TABLE
                              5-minute tumbling window aggregation
                              per device (avg temp, avg humidity),
                              own WATERMARK for stateful aggregation

        ▼
  Databricks SQL Dashboard — line charts (temp/humidity trend per
                              device), bar chart (overall avg per device)
```

Note: an original `bronze_sensor_readings` layer was collapsed into `silver_sensor_readings` after hitting Free Edition's active-pipeline quota — a deliberate two-layer simplification rather than three, a legitimate tradeoff under resource constraints.

## Key concepts and gotchas

- **`TRIGGER ON UPDATE AT MOST EVERY INTERVAL 1 minute`** replaces `SCHEDULE EVERY` (which only supports HOUR/DAY/WEEK granularity) for sub-hour freshness. Reacts to new upstream data rather than polling on a fixed clock.
- **`WATERMARK` syntax** attaches to the source in the `FROM` clause (`FROM STREAM x WATERMARK col DELAY OF INTERVAL n`), not as a filter condition — it's a property of the data source, not a row filter.
- **Windowed aggregations require their own watermark** — a watermark defined upstream does not carry through automatically; each stateful aggregation query needs it declared explicitly.
- **A window only finalizes once the watermark passes `window_end`** — seeing 0 rows in a freshly created aggregate table is often correct behavior (not enough time has elapsed/enough new data has landed), not a bug.
- **Changing a streaming table's source requires a full `DROP TABLE` + recreate**, not just a redefinition — Spark refuses to reconcile a changed source against an existing checkpoint (`DIFFERENT_DELTA_TABLE_READ_BY_STREAMING_SOURCE`), a deliberate exactly-once safety check.
- **Struct columns (e.g. `window` structs) don't chart cleanly** — extract fields explicitly (`time_window.start AS window_start`) before visualizing.

## Free Edition constraints encountered

- Serverless *generic* compute (notebooks) cannot create streaming tables — requires a SQL warehouse specifically (`STREAMING_TABLE_OPERATION_NOT_ALLOWED.ST_NOT_ENABLED_ON_SERVERLESS_GENERIC_COMPUTE`).
- Each streaming table spins up its own pipeline, counting against a hard active-pipeline cap — exceeding it returns `RESOURCE_EXHAUSTED`, and resets on a daily cycle.

## Tools used

Databricks SQL (streaming tables, `read_files`, `STREAM`, `WATERMARK`), Delta Lake, Databricks SQL dashboards.
