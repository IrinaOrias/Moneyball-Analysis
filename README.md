# Moneyball SQL Analysis ⚾ | Strategic Player Value Assessment

**Client Scenario:** Oakland Athletics (2001)  
**Perpective as:** Data Analyst – Recruitment Strategy & Salary Optimization  
**Tools:** SQL (SQLite), Relational Databases, Aggregation, Filtering, Performance Metrics

---

##  Business Objective

In 2001, the Oakland Athletics faced one of Major League Baseball’s most pressing challenges: build a competitive team with a drastically limited budget. My responsibility was to identify undervalued players using historical salary and performance data, and to support strategic decision-making for player acquisition.

---
### Dataset Overview

This analysis is based on the `moneyball.db` SQLite database, which contains four interconnected tables: `players`, `teams`, `salaries`, and `performances`. Together, these datasets enable a comprehensive exploration of player performance, compensation, and team history to uncover value-driven insights in baseball.

## Tables

### `players`
Contains biographical data on 15,622 professional baseball players, including names, batting and throwing handedness, physical attributes (height and weight), and place of birth. It also includes career timeline details such as debut and final game dates, supporting demographic and career span analysis.

### `teams`
Includes 148 records representing team-year combinations from 1871 to 2001. Each entry provides the team name, ballpark, and season year, offering the contextual backbone for analyzing performance and salary data at the team level.

### `salaries`
Comprises 13,959 records of player salaries from 1985 to 2001, with each entry linked to a player, team, and season. Salary figures range from $0 to $22 million, enabling in-depth evaluation of player value, payroll efficiency, and market trends over time.

### `performances`
Consists of 81,989 entries detailing individual player performance statistics such as games played, hits, home runs, RBIs, and stolen bases. Each record is associated with a player, team, and year, forming the quantitative foundation for identifying undervalued players and analyzing on-field impact.

All queries in this project aim to reveal performance-to-cost insights and expose market inefficiencies in player valuation — the essence of the Moneyball strategy.


---

##  Strategic Insights

| # | Focus Area | Key Business Question |
|---|------------|------------------------|
| 1 | Trend Analysis | How have average player salaries evolved over time? |
| 2 | Contract Assessment | What is Cal Ripken Jr.’s salary history? |
| 3 | Performance Scouting | What is Ken Griffey Jr.’s home run trajectory? |
| 4 | Cost Optimization | Who were the 50 lowest-paid players in 2001? |
| 5 | Legacy Research | Which teams did Satchel Paige play for? |
| 6 | Competitive Analysis | Top 5 teams by total hits in 2001? |
| 7 | Salary Benchmarking | Who received the highest salary in MLB history? |
| 8 | Investment Feasibility | Cost of acquiring the best home run hitter in 2001? |
| 9 | Market Intelligence | 5 lowest-paying teams by average salary in 2001? |
|10 | Career Profiling | Player-level yearly stats: home runs & salaries |
|11 | ROI Analysis | 10 best value-for-money players (cost per hit) |
|12 | Dual-Impact Players | Overlap: best cost per hit **and** per RBI |

---

##  Example: ROI Analysis Query (11.sql)

```sql
SELECT p.first_name, p.last_name, ROUND(s.salary / perf.hits, 2) AS "dollars per hit"
FROM salaries s
JOIN players p ON p.id = s.player_id
JOIN performances perf ON s.player_id = perf.player_id AND s.year = perf.year
WHERE s.year = 2001 AND perf.hits > 0
ORDER BY "dollars per hit" ASC, p.first_name, p.last_name
LIMIT 10;

