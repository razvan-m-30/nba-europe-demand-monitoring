# CLAUDE.md

Portfolio project: a weekly lakehouse pipeline on Databricks Free Edition that monitors a demand signal against a plan and flags drift. The data is a vehicle; a working, documented pipeline matters more than forecast sophistication.

- Pipeline: Wikipedia pageviews (fr.wikipedia `Basket-ball`, `National Basketball Association`) -> bronze (raw API responses in Delta, never cleaned) -> silver (daily typed and deduplicated, gap fill, outlier flag; weekly Monday-start sums, 2019 = 100 index) -> gold (forecast band vs actual, drift flag) -> AI/BI dashboard and Genie space.
- Hindcast: forecast the NBA Paris Games (week of 2025-01-20) from the Paris 2024 Olympic basketball finals (week of 2024-08-05). The exact rule is pre-registered in docs/decision_log.md; never change it.
- Scope: one market (France), no control market.
- Notebooks are authored in a Databricks Git folder and numbered in run order (00_setup, 01_bronze, 02_silver, 03_gold). Non-notebook files are edited locally.
- Code reused from github.com/razvan-m-30/nba-europe-demand-plan @ 498752b (src/ and reference/project1/). Never reuse that project's output numbers.
- Environment: Windows 11, PowerShell, Python 3.13.1, venv in .venv.
- Rules: never invent data, dates or article titles; keep changes small and scoped; no data files or secrets in git; the repo is public, keep it clean.
