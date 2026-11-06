# Leverage Index & Pitcher KPI Analysis

A Python project that measures how pitchers perform in high-pressure situations. It starts with a simple rule-based definition of leverage, then builds a full **Leverage Index (LI)** model grounded in win expectancy to compare pitcher performance across low-, medium-, and high-leverage plate appearances.

## Overview
The project evolved in two stages. First, a transparent baseline: flag high-leverage situations with a simple baserunner/outs rule and compare each pitcher's KPIs to his overall performance. Then a more rigorous approach: a Leverage Index based on the expected change in win expectancy given the inning, half-inning, score differential, outs, and base-runner state. The LI model is built from three seasons (2023–2025) of Power 5 game data, then applied to 2025 pitch and plate-appearance data to see how pitchers perform when the game is on the line.

An LI of 1.00 is an average-leverage situation. Values above 1.00 are above average; values below 1.00 are below average.

## Repository Structure
| Notebook | Description |
|---|---|
| `leverage kpis basic.ipynb` | Baseline analysis: rule-based leverage definition and pitcher KPIs, overall vs. leverage |
| `Leverage Index.ipynb` | Builds the win expectancy table, event probabilities, and Leverage Index for each game state |
| `Leverage KPI Analysis.ipynb` | Joins LI to 2025 pitch data, buckets plate appearances into Low/Medium/High LI, and calculates pitcher KPIs and visuals |

`Leverage Index Project Documentation` contains the full written project documentation.

## Methodology

### Part 1: Baseline — Rule-Based Leverage KPIs
The analysis began with a simple, easy-to-explain definition of a high-leverage situation: **a runner on second or third base (a runner in scoring position) with fewer than two outs.** In these spots, a single hit or a deep fly ball can score a run, while the defense still has outs to spare, so every pitch carries extra weight.

1. **Load pitch-level data** — Trackman pitch and play-by-play data are pulled from SQL Server.
2. **Flag leverage pitches** — Each pitch is tagged as leverage or non-leverage using the baserunner/outs rule above.
3. **Calculate each KPI twice** — For every pitcher, the metrics are computed both overall and in leverage situations: Strike%, In-Zone%, Heart%, Chase%, BABIP, BB%, and Ideal Contact Rate.
4. **Apply sample-size filters** — Minimum pitch counts (overall and in leverage) keep small samples out of the results.
5. **Visualize the gap** — A reusable plotting function charts the difference (delta) between overall and leverage performance for each pitcher, sorted by delta, annotated with leverage sample size, and compared against the league-average delta. It can be filtered to a single team.

**Why move beyond it:** The baseline is easy to interpret, but it treats every scoring-position, fewer-than-two-out situation as equally important. It ignores innings and scores and misses high-pressure spots that fall outside the rule, such as a tie game in the ninth with the bases loaded and two outs. That limitation motivated a model of leverage that quantifies how much each situation actually matters.

### Part 2: Leverage Index Model
1. **Load and prepare game data** — Trackman play/pitch data, play-by-play outcomes, game information, and baserunner data. The model uses plate-appearance-ending events.
2. **Score before each play** — Runs are accumulated per team in chronological order and shifted so each play uses the score *before* it. Score differential = home minus away.
3. **Game outcome** — A home-team-win indicator is the outcome used to estimate win expectancy.
4. **Base states** — Baserunner columns become eight base states (empty, each single runner, each pair, loaded).
5. **Win expectancy table** — Calculated by inning, half-inning, score differential (capped at ±7), outs, and base state. Extra innings are grouped into an inning-10 bucket. Smoothing lets rare states borrow information from broader contexts.
6. **Event probabilities** — Strikeout, other out, walk, single, double, triple, home run, based on observed frequency.
7. **Leverage Index** — For each game state, every possible event is simulated, the resulting win expectancy is looked up, and the absolute change is weighted by event probability to get an expected win-expectancy swing. LI = that swing divided by the average swing across all states.
8. **Validation** — Highest-leverage states with sample sizes, the LI distribution, and average LI by inning, score differential, base state, and outs.

### Part 3: Pitcher KPI Analysis
1. **Join LI to pitches** — Each plate appearance's final LI is joined to all of its pitches.
2. **LI buckets** — Low / Medium / High by LI percentile (equal thirds by default, configurable in one place).
3. **Pitch-level KPIs** — Strike%, Heart%, Whiff%, CSW%, Chase%.
4. **Plate-appearance KPIs** — K%, BB%, Ideal Contact Rate (runner-only events excluded).
5. **Run Value** — Total and per 100 pitches, from a run-value lookup by pitch result, count, and outs, signed so positive is good for the pitcher.
6. **wOBA allowed** — Calculated at the plate-appearance level.
7. **Qualification thresholds** — Minimum pitches, plate appearances, and balls in play before results are charted or ranked.
8. **Visualizations** — Pitcher profile across LI buckets vs. team average, a High-LI bubble chart (wOBA, RV/100, K%, sample size), a Low-to-High LI wOBA change chart, and a High-LI KPI heatmap.

## Tools
Python, pandas, NumPy, matplotlib, seaborn, SQL Server (T-SQL)

## Data
The underlying Trackman and game data is proprietary and used under a non-disclosure agreement, so **no data, outputs, or results are included in this repository**. Notebooks have had their outputs cleared, and database credentials and file paths have been removed. They expect a SQL Server connection (`db_conn`) and two local files (`li_by_pa.parquet`, `pbp_outcome_run_values.csv`) that are not provided.

## Author
Coleman Dukes | [LinkedIn](https://www.linkedin.com/in/coleman-dukes/)
