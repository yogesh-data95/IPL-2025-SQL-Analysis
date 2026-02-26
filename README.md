# IPL 2025 SQL Data Analysis Project

#Overview
This project analyzes **IPL 2025 ball-by-ball delivery data** using SQL to extract
meaningful cricket insights such as top run scorers, batting averages, highest scores,
and player consistency throughout the tournament.

---

#Dataset
- Source: IPL 2025 Ball by Ball Deliveries Dataset
- Tool Used: MySQL / MySQL Workbench
- Table Name:`ipl_2025_deliveries`
- Key Columns:`striker`, `match_id`, `runs_of_bat`, `player_dismissed`

---

# SQL Concepts Used
- CTE (Common Table Expressions) — WITH clause
- Aggregate Functions — SUM, MAX, COUNT, ROUND
- CASE WHEN statements
- GROUP BY and HAVING filters
- Subquery logic for cricket-specific average calculation

---

# Analysis 1: Top 5 Highest Run Scorers

# Business Question
Who are the top 5 batsmen by total runs scored in IPL 2025, and what are their batting averages, highest scores, and matches played?

# Approach
- Used CTE to calculate runs per player per match
- Applied cricket batting average logic — average = total runs / times out - first i have made mistake that i have taken total match played by the player and later reliased my
  mistake (then not out innings are NOT counted in average denominator)
- Used CASE WHEN to identify whether a batsman got out in each match

# Query
```sql
WITH match1 AS (
    SELECT 
        striker, match_id,
        SUM(runs_of_bat) AS run_per_match,
        MAX(CASE WHEN player_dismissed = striker 
            THEN 1 ELSE 0 END) AS player_out
    FROM ipl_2025_deliveries
    GROUP BY striker, match_id)

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
ORDER BY total_score DESC
LIMIT 5;
```

# Result

| Striker | Total Score | Highest Score | Match Played | Average |
|---------|------------|---------------|--------------|---------|
| Sai Sudharsan | 759 | 108 | 15 | 54.21 |
| Suryakumar Yadav | 717 | 73 | 16 | 65.18 |
| Kohli | 657 | 73 | 15 | 54.75 |
| Shubman Gill | 650 | 93 | 15 | 54.17 |
| Mitchell Marsh | 627 | 117 | 13 | 48.23 |

# Key Insights
- Sai Sudharsan topped the run charts with 759 runs in just 15 matches showing exceptional consistency throughout IPL 2025
  
- Suryakumar Yadav had the highest average of 65.18 among top 5, making him the most consistent performer relative to dismissals
  
- Mitchell Marsh scored the **highest individual innings of 117 among the top 5 batsmen

- All top 5 batsmen maintained averages above 48 — showing that the best run scorers were also the most consistent players
  
- Suryakumar Yadav scored 717 runs in 16 matches compared to Sudharsan's 759 in 15 — very close competition at the top

---

# What Makes This Analysis Special

Most people calculate batting average as total runs divided by matches played. However real cricket average only counts innings where the batsman got out.
Not out innings are excluded from the denominator.

This project correctly implements cricket batting average logic using SQL CASE WHEN statements — making the analysis statistically accurat and professionally meaningful.

---

# How to Run This Project

1. Download the IPL 2025 deliveries dataset from Kaggle
2. Import CSV into MySQL using MySQL Workbench
3. Run queries from `ipl_analysis.sql` file
4. View results in Result Grid

---

## More Analysis Coming Soon
- Bowling Analysis — Top wicket takers, economy rates
- Team Performance — Best powerplay teams, win percentages
- Match Analysis — Highest scoring matches, toss impact
- Powerplay vs Death Overs — Player performance by phase

---

## Connect With Me
- GitHub: github.com/yogesh-data95
- Email: yogesh.kabdal95@gmail.com
- LinkedIn: www.linkedin.com/in/yogesh-kabdal-5b880ba6


*Open to freelance SQL and Data Analysis projects!*
