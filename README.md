# IPL 2025 Batting Statistics Analysis

I was watching IPL 2025 and got curious about which players were actually performing well beyond just the orange cap holder. I did this SQL-based analysis project that computes key batting performance metrics for IPL 2025 players using ball-by-ball delivery data which taken from kaggle. Here are some findings you can see and suggetion is always welcome.

---

# Overview

This query analyzes the 'ipl_2025_deliveries' table to produce a batting scorecard for top-performing batters in the tournament. It filters for players who meet a minimum performance threshold, making it useful for identifying consistent and high-impact batters.

---

# Why This Query Matters

Most cricket fans and analysts judge a batter purely by total runs scored — the bigger the number, the better the player. But that view is incomplete and often unfair.

Consider a lower-middle-order batter who comes in at No. 6 or No. 7. They face fewer balls per innings by design, so their total run tally will naturally be lower than an opener who bats longer. Ranking players only by total runs penalizes these players despite them being efficient and impactful innings.

This query fixes that by applying a three filter:

- Average > 30 — the batter scores consistently and doesn't throw their wicket away cheaply
- Matches played > 10 — the performance is sustained across the tournament, not a one-match flash
- Total score > 300 — ensures the batter has made a meaningful run contribution overall

Together, these three conditions can find players who are consistent, durable, and efficient, not just those who happened to bat at the top of the order for every game.

# Real-World Application: IPL Auction Strategy

This analysis is directly useful for franchise decision making at the IPL auction. Teams often overpay for high-profile names with big total run tallies, while undervaluing players who:

- Bat in tough lower-order positions
- Maintain excellent averages under pressure
- Contribute across many matches without being the headline act

By identifying these under-the-radar performers, franchises can make smarter, value-driven investments — picking up quality players before rival teams recognize their worth.

---

# Dataset from kaggle

Table: `ipl_2025_deliveries`

| Column | Description |
|---|---|
| `striker` | Batter facing the delivery |
| `match_id` | Unique identifier for each match |
| `runs_of_bat` | Runs scored off the bat on that delivery |
| `player_dismissed` | Name of the player dismissed (if any) on that delivery |

---

# Output Columns

| Column | Description |
|---|---|
| `striker` | Batter's name |
| `total_score` | Total runs scored across all matches |
| `highest_score` | Best individual innings score |
| `match_played` | Number of matches played |
| `average` | Batting average (total runs / times dismissed); `NULL` if never dismissed |

---

# How It Works

The query runs in two stages using a Common Table Expression (CTE):

Stage 1 — `match1` CTE

For each batter per match, it calculates:
- `run_per_match` — runs scored in that match
- `player_out` — whether the batter was dismissed (1 = out, 0 = not out)

Stage 2 — Aggregation**

Aggregates across all matches to compute career-level stats, then filters using `HAVING`:

```sql
match_played > 10    -- played in more than 10 matches
total_score > 300    -- scored more than 300 runs total
average > 30         -- batting average above 30
```

Results are sorted by `total_score` in descending order.

---

# Key SQL Concepts Used

- CTE (`WITH` clause) — breaks the logic into readable stages
- Conditional aggregation — `MAX(CASE WHEN ...)` to detect dismissals per innings
- `HAVING` clause — filters on aggregated values post-grouping
- `NULLIF`-style guard — avoids division by zero for not-out batters

---

# Usage

Run the query against any SQL engine (MySQL, PostgreSQL, BigQuery, etc.) with access to the `ipl_2025_deliveries` table:

```sql
WITH match1 AS (
  SELECT
    striker,
    match_id,
    SUM(runs_of_bat) AS run_per_match,
    MAX(CASE WHEN player_dismissed = striker THEN 1 ELSE 0 END) AS player_out
  FROM ipl_2025_deliveries
  GROUP BY striker, match_id
)
SELECT
  striker,
  SUM(run_per_match) AS total_score,
  MAX(run_per_match) AS highest_score,
  COUNT(match_id) AS match_played,
  CASE
    WHEN SUM(player_out) = 0 THEN NULL
    ELSE ROUND(SUM(run_per_match) / SUM(player_out), 2)
  END AS average
FROM match1
GROUP BY striker
HAVING match_played > 10
  AND total_score > 300
  AND average > 30
ORDER BY total_score DESC;
```

---

#  Project Structure

```
ipl-2025-analysis/
│
├── README.md
└── batting_stats.sql       # Main query file
```

License
MIT License
