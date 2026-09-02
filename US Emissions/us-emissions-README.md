# US Emissions Analytics Dashboard

A geographic and per-capita analytics dashboard on US county-level greenhouse gas (GHG) emissions data, built to practice location-based visualization and per-capita normalization.

## Overview

Explores `emissions.default.emissions_data` — county-level GHG emissions figures (in metric tons of CO2-equivalent) paired with population — to answer three distinct questions: where are emissions physically concentrated, how do they scale with population, and which states contribute the most in absolute terms.

## Dashboard

**Charts:**

| Chart | Type | Query focus | Finding |
|---|---|---|---|
| Emissions for Continental US | Map | `latitude, longitude, GHG emissions mtons CO2e` | Geographic distribution of raw emissions by county |
| Emission vs Population | Scatter/ranked | `population`, `Emissions_per_person` (emissions ÷ population) | Higher-population counties tend to have *lower* emissions per person |
| Total Emissions by mTon of CO2e | Bar (top 10 states) | `SUM(GHG emissions)` grouped by state, `LIMIT 10` | The top 10 emitting states account for 51% of all US emissions |

## Key concepts

- **Per-capita normalization matters for fair comparison** — ranking raw emissions alone would just surface the most populous counties; dividing by population (`Emissions_per_person`) answers a genuinely different, more meaningful question about efficiency/impact per resident, and the finding here (higher population ≠ higher per-person emissions) would be invisible without that normalization step.
- **Data cleaning inside the query itself**: the source emissions figures are stored as comma-formatted strings (e.g. `"1,234"`), requiring `CAST(REPLACE(col, ',', '') AS DOUBLE)` before any arithmetic — a reminder that "numeric-looking" source data isn't always actually numeric-typed, and needs explicit casting before aggregation or division.
- **Concentration finding** (top 10 states = 51% of emissions) is the kind of headline statistic worth surfacing directly in a dashboard description rather than making a viewer infer it from a bar chart alone.
- **Geographic (map) visualization** as a distinct chart type from the other two — useful when the "where" itself is the primary question, not just a breakdown by category.

## Tools used

Databricks SQL, Databricks SQL dashboards (map, scatter, and bar chart types), string-to-numeric data cleaning in SQL.
