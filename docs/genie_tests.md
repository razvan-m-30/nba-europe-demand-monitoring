# Genie tests

Genie space **"NBA Europe demand monitor: planner Q&A"**, built on the three gold tables only: `nba_demand.gold.forecast_rule`, `nba_demand.gold.forecast_vs_actual`, `nba_demand.gold.hindcast_summary`. Tested 17 November 2026. Each answer was checked against the check queries below.

## Instructions given to Genie

```
This space answers questions about a back-test of a demand-monitoring rule.
- Data: weekly French Wikipedia pageviews, a proxy for consumer interest. It is NOT sales data.
- topic 'basketball' = the article "Basket-ball" (the headline series). topic 'nba' = the article "National Basketball Association".
- The plan (forecast) assumed the NBA Paris Games 2025 (week of 2025-01-20) would lift interest like the Paris 2024 Olympic basketball finals did. The planned range is half to double that bump: forecast_low, forecast_base, forecast_high (weekly views).
- week_offset 0 = the Paris games week. Negative offsets are the 8 weeks before the event and have no forecast.
- band_position is 'inside', 'below' or 'above' the planned range. A drift week is a week outside the range. The drift flag is raised from the 2nd consecutive drift week.
- Columns ending in _uplift are fractions: 0.25 means +25%. Always present them as percentages.
- For headline questions use gold.hindcast_summary. For week-by-week questions use gold.forecast_vs_actual. The rule's parameters are in gold.forecast_rule.
- The data contains no sales, product, price or Nike data. If asked about those, say the data does not contain them instead of estimating.
```

## Results: 5 of 5 passed

| # | Question | Genie's answer (summary) | Expected (from check queries) | Result |
|---|---|---|---|---|
| 1 | Did the Paris games week land inside the planned range for basketball? | No, below: 4,794 views vs. a range of 24,806-84,733; −0.76% vs. +413.5% to +1,654.1% | `below`; 4,794 vs. 24,806-84,733 | Pass |
| 2 | By how much did basketball interest change in the Paris games week, and what did the plan expect? | −0.76%; plan +413.5% (low), +827.1% (base), +1,654.1% (high) | −0.76%; +413.5% / +827.1% / +1,654.1% | Pass. Note: headline "4× to 17× surge" is loosely worded; the planned levels are 5.1× to 17.5× the baseline |
| 3 | In which week was the drift flag first raised for each article? | Week of 27 January 2025 for both articles; explained the 2-consecutive-weeks rule correctly | 2025-01-27 for both | Pass |
| 4 | For the NBA article, how many of the 13 weeks from the Paris games week onwards were inside, below and above the planned range? | 2 inside, 5 below, 6 above | 2 / 5 / 6 | Pass. Note: a side remark that the basketball article "stayed below throughout" is wrong; it was above the range in weeks 2 and 3 |
| 5 | How many Nike basketball shoes were sold in France in January 2025? | "The data does not contain sales information..."; no number given | Decline: no sales data | Pass |

**Takeaway.** Every figure Genie returned matched the tables, and it refused the out-of-scope question. Two answers paraphrased the result loosely in their commentary, so planners should rely on the numbers and the generated SQL, not on the summary sentences.

## Check queries

```sql
SELECT topic, event_week_actual_uplift, event_week_band_low_uplift, event_week_band_base_uplift,
       event_week_band_high_uplift, event_week_band_position, drift_flag_first_raised_week
FROM nba_demand.gold.hindcast_summary;

SELECT topic, week_offset, actual_views, forecast_low, forecast_high, band_position, drift_flag
FROM nba_demand.gold.forecast_vs_actual
WHERE week_offset BETWEEN 0 AND 1;

SELECT band_position, count(*) AS weeks
FROM nba_demand.gold.forecast_vs_actual
WHERE topic = 'nba' AND week_offset >= 0
GROUP BY band_position;
```
