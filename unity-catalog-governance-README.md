# Unity Catalog Governance on the Taxi Pipeline

Applying Unity Catalog governance features — documentation, lineage, access control, and dynamic masking — to the existing NYC Taxi batch pipeline, without disturbing any live dashboards, jobs, or dependent assets.

## Overview

Rather than build new tables for this project, governance was applied *in place* to tables already in production use from an earlier project (`silver_taxi`, `gold_taxi_by_zip`). This was a deliberate choice: comments, tags, and grants are metadata/permission operations with no data-movement risk, unlike physically relocating tables into new schemas — which would have broken dashboard queries and streaming table checkpoints with live dependents. Real teams face this exact tradeoff when governing tables that are already in use.

## What was applied

### Documentation
- Table-level comments via `ALTER TABLE ... SET TBLPROPERTIES ('comment' = ...)` on `silver_taxi` and `gold_taxi_by_zip`, explaining what each table is and why it exists.
- Column-level comments (`ALTER TABLE ... ALTER COLUMN ... COMMENT`) documenting business rules not obvious from the column name alone — e.g. `fare_amount` documented as "validated > 0".
- A classification tag (`SET TAGS ('classification' = 'location_data')`) on `pickup_zip`, flagging it as location-related data for future governance tooling.

### Lineage (automatic, zero configuration)
Unity Catalog's Lineage tab reconstructed real dependency graphs from query history alone:
- **Upstream** of `gold_taxi_by_zip`: traced back to `silver_taxi`. Notably, `bronze_taxi` did *not* appear in the upstream graph — lineage tracking was less complete for tables written via the PySpark DataFrame API (`.write.format('delta').saveAsTable()`) compared to SQL-based `CREATE TABLE AS SELECT`, a real limitation worth knowing rather than assuming lineage is always complete.
- **Downstream (Consumers)** of `gold_taxi_by_zip`: correctly identified the saved dashboard query, the exploratory notebook, and the scheduled Workflow job that all read from this table — a concrete, provable answer to "what would break if I changed this table," rather than relying on memory.

### Access control
```sql
GRANT SELECT ON TABLE gold_taxi_by_zip TO `account users`;
REVOKE SELECT ON TABLE silver_taxi FROM `account users`;
```
Encodes a real policy directly as data: the gold layer (cleaned, business-ready) is broadly readable; the silver layer (intermediate, less curated) is not granted broad access, following the common real-world pattern of narrowing access as data moves upstream toward raw/intermediate layers.

### Dynamic column masking
```sql
CREATE VIEW silver_taxi_masked AS
SELECT
  trip_distance, trip_duration_minutes, fare_amount,
  CASE WHEN current_user() = '<owner_email>' THEN pickup_zip ELSE 'REDACTED' END AS pickup_zip
FROM silver_taxi;
```
`current_user()` is evaluated per-query, per-user — not baked in at view creation time — so the same view returns real or redacted `pickup_zip` values depending on who's querying it, with no duplicated data and no manual sync. This is a simplified, hand-rolled version of the row/column-level security pattern Databricks supports more fully via group-membership functions (`is_account_group_member(...)`).

## Key concepts learned

- Metadata operations (comments, tags, grants) are low-risk on live tables; physical schema reorganization is not, and requires the same coordinated-migration thinking as any production system.
- Lineage completeness can depend on *how* a table was written (SQL DDL vs. DataFrame API) — worth verifying, not assuming.
- `SHOW GRANTS` returning no rows means "no explicit grant exists," not an error — owner access is implicit and doesn't appear as a row.
- Access control policy can be read directly off the data (via `SHOW GRANTS`) rather than living only in a person's memory or a wiki page.
- Dynamic masking views evaluate identity functions per-query, enabling one physical table to serve different views of the same data to different users.

## Free Edition note

Being a single-user environment, access control here could only be demonstrated by writing and inspecting the correct grant/revoke commands — not by observing a second user actually being allowed or blocked. The commands and resulting `SHOW GRANTS` output are the same as they would be in a multi-user workspace.

## Tools used

Databricks Unity Catalog (Catalog Explorer, lineage graph, `GRANT`/`REVOKE`, `TBLPROPERTIES`, column tags, dynamic views), building on the existing batch taxi pipeline.
