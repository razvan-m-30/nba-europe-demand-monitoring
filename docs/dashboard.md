# Dashboard: NBA Europe demand monitor

AI/BI dashboard built in the Databricks workspace on the gold tables only. One page: title text, a page filter on `topic` (default `basketball`, applied to both datasets), three counters, a line chart and a week-by-week table.

## Datasets

`weekly`
```sql
SELECT topic, article, week_offset, target_week_start AS week_start,
       actual_views, forecast_low, forecast_base, forecast_high,
       band_position, drift_flag
FROM nba_demand.gold.forecast_vs_actual
ORDER BY topic, week_offset
```

`summary`
```sql
SELECT topic, article,
       round(event_week_actual_uplift * 100, 1)    AS actual_change_pct,
       round(event_week_band_base_uplift * 100, 0) AS planned_change_pct,
       round(event_week_band_low_uplift * 100, 0)  AS planned_low_pct,
       round(event_week_band_high_uplift * 100, 0) AS planned_high_pct,
       event_week_band_position,
       drift_weeks, weeks_compared,
       drift_flag_first_raised_week
FROM nba_demand.gold.hindcast_summary
```

## Widgets

- Counters (dataset `summary`): actual change in the Paris games week (%), planned change, base case (%), drift flag raised (week of).
- Line chart (dataset `weekly`): x = `week_start`; y = actual, planned low, planned base, planned high (weekly views, log scale).
- Table (dataset `weekly`): week, actual, planned low/base/high, position, drift flag.
