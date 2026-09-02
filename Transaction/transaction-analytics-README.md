# Transaction Analytics Dashboard

A Databricks SQL analytics dashboard built on the built-in `samples.bakehouse.sales_transactions` sample dataset, exploring product-level sales performance and sales volume over time.

## Overview

A focused, dashboard-first project: rather than building a pipeline from raw data, this uses Databricks' bundled Bakehouse sample dataset directly to practice query design and dashboard construction — useful for quickly prototyping analytics without needing a data ingestion step first.

## Dashboard

**Datasets:**
- `sales_transactions` — direct pass-through of `samples.bakehouse.sales_transactions`
- `Product_Sales_Desc` — `SELECT product, SUM(totalPrice) FROM samples.bakehouse.sales_transactions GROUP BY product ORDER BY SUM(totalPrice) DESC`

**Charts:**
| Chart | Type | Insight |
|---|---|---|
| Total Price by Product | Bar | Golden Gate Ginger is the highest-selling product 8 years running |
| Quantity of Sales over Time | Line/time-series | Sales volume trend across the dataset's date range |

## Key concepts

- Working directly with a **Databricks-provided sample schema** (`samples.bakehouse`) rather than an externally sourced dataset — a fast way to practice dashboarding without a data engineering step first.
- Aggregating and sorting in the dataset SQL itself (`GROUP BY product ORDER BY ... DESC`) rather than relying on the chart builder to do it — keeping the heavier logic in SQL keeps the visualization layer simple.
- Writing a dashboard **description that states a specific, checkable finding** ("Golden Gate Ginger is our highest selling product 8 years in a row") rather than a generic label — turning the chart into an actual claim a viewer can verify at a glance.

## Tools used

Databricks SQL, Databricks sample datasets (`samples.bakehouse`), Databricks SQL dashboards.
