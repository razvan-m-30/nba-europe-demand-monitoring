# Decision log

Decisions for this project, oldest first. The forecast rule below was committed before any data was pulled into this project; the commit timestamp is the evidence that the rule was not tuned to the result.

## Pre-registered forecast rule (2026-11-15)

**Honesty note.** The outcome is not blind: the earlier project (github.com/razvan-m-30/nba-europe-demand-plan) already measured French Wikipedia interest around the January 2025 Paris games. Pre-registration guards against tuning the rule, not against prior knowledge.

**Events**
- Analog: Paris 2024 Olympic basketball finals, Bercy Arena. Men's final Saturday 10 August 2024 (USA 98-87 France); women's final Sunday 11 August 2024 (USA 67-66 France). Event week: the week starting Monday 2024-08-05.
- Target: NBA Paris Games 2025, Accor Arena, Indiana Pacers vs San Antonio Spurs, 23 and 25 January 2025. Event week: the week starting Monday 2025-01-20.

**Data**
- French Wikipedia daily pageviews (Wikimedia Pageviews API, `user` agent) for `Basket-ball` (headline series) and `National Basketball Association` (secondary series).
- Daily views are summed into Monday-start weeks; only complete 7-day weeks are used. The forecast uses raw views, not the outlier-capped column.

**Rule** (applied to each series separately)
1. Baseline B = median weekly views over the 8 complete weeks before the event week, computed separately for the analog and the target.
2. Analog uplift: u(k) = analog views in week (event week + k) / B_analog - 1, for k = 0 to 12.
3. Target forecast for week k: base = B_target x (1 + u(k)); low = B_target x (1 + 0.5 x u(k)); high = B_target x (1 + 2 x u(k)). If u(k) is negative, low and high are the smaller and larger of those two values.
4. A week is a drift week if actual views fall outside [low, high]; the direction (above or below) is recorded.
5. The market's drift flag is raised from the second consecutive drift week onwards.
6. Headline verdict: whether `Basket-ball` in the target event week (k = 0) falls inside the band, plus the number of drift weeks for k = 0 to 12.
7. Summary metrics reported alongside: analog peak uplift (max u(k) for k = 0 to 4), half-life (first week after the peak where uplift falls below half the peak, within k <= 12, otherwise none), persistence u(12).

**Expected result, stated in advance.** The earlier project found that national-team success moves basketball interest far more than NBA games in Europe, so the Olympics-based band is expected to overshoot and the Paris games to land below it, firing the drift flag.

## Decisions

| # | Date | Decision | Why |
|---|---|---|---|
| 1 | 2026-10-26 | Build a medallion (bronze, silver, gold) pipeline on Databricks; the data topic is a vehicle | The project's purpose is demonstrating the lakehouse stack |
| 2 | 2026-10-26 | Re-ingest raw data through bronze rather than reusing the earlier project's processed files | A copied file would make bronze fake |
| 3 | 2026-10-26 | Hindcast: forecast a past event from an earlier one and compare with what happened | The real NBA Europe plan covers 2027-28, so no actuals exist yet |
| 4 | 2026-10-26 | Low/base/high band instead of a point forecast | Honest about uncertainty; inside/outside is the headline metric |
| 5 | 2026-10-26 | One analog, one target, one market (France, fr.wikipedia) | Time; same country avoids cross-country transfer |
| 6 | 2026-10-26 | Copy code from the earlier project at a fixed commit with a source header; never reuse its output numbers | Standalone repo; the earlier outputs already include the target event |
| 7 | 2026-11-11 | Olympics analog = the basketball finals only, not the whole tournament | One clear event week |
| 8 | 2026-11-11 | No control market; v0 monitors one market | Scope; stated as a limitation |
| 9 | 2026-11-11 | Band = base uplift x 0.5 (low) and x 2 (high) | Symmetric in ratio terms; "within a factor of two" is easy to read |
| 10 | 2026-11-11 | Silver keeps raw `views` plus `views_capped` and `was_capped` (cap at 5x the trailing 56-day median); the forecast uses raw views | Flag, don't overwrite: the cap must not shrink the spikes being measured |
| 11 | 2026-11-11 | Databricks Free Edition | Serverless compute only, restricted outbound internet, one 2X-Small SQL warehouse; designs must fit these limits |
| 12 | 2026-11-15 | Drift flag = two or more consecutive weeks outside the band | Avoids flagging a one-week blip |
| 13 | 2026-11-15 | Notebooks live in a Databricks Git folder linked to this repo | Pipeline code is versioned in GitHub without manual export |
| 14 | 2026-11-15 | Not in v0: a previous-year seasonality check and a reconciliation against the earlier project's weekly numbers | Time budget; neither is needed to demonstrate the pipeline. Listed as limitations |
| 15 | 2026-11-16 | One bronze table (`bronze.api_responses`) for both API endpoints (title lookup and pageviews) | Same grain (one API response) and schema; one append per batch keeps each batch atomic. The usual convention is a table per source; split if a source with a different shape is added |
| 16 | 2026-11-16 | Bronze re-pulls the full history (2019 to yesterday) every run; silver reads only the latest batch per article | About 0.5 MB per call, so a full snapshot is simpler than incremental loading and picks up corrections to past numbers; at larger volumes, load incrementally |
| 17 | 2026-11-16 | `silver.pageviews_daily` loaded with MERGE (rows updated only when values change, with `source_batch_id` and `updated_at`); weekly silver and gold tables rebuilt in full each run | MERGE makes re-runs idempotent and gives row-level lineage on the table that receives new data; the derived tables are small, so a full rebuild is simplest |
| 18 | 2026-11-16 | The 2019 = 100 index baseline uses complete weeks only | A week with missing days would understate the baseline |
| 19 | 2026-11-16 | Rule parameters stored as a row in `gold.forecast_rule`; the dashboard and Genie read only the gold schema | The rule is visible in the data and changeable in one place; consumers only see curated tables |
| 20 | 2026-11-17 | Results reported as computed; the rule was not changed after seeing them. The headline uses the event week and the first drift flag | Weeks 2-12 are confounded by seasonality (summer analog baseline, August holiday, NBA season start), stated as a limitation |
| 21 | 2026-11-17 | Genie limited to the three gold tables, with instructions defining the business terms; tested with 5 questions including one it should refuse (sales) | Fewer tables and explicit definitions mean fewer ways to answer wrongly; the refusal test checks it doesn't invent answers |
