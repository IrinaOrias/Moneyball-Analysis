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

```sql
-- Check for duplicates

SELECT * , COUNT(*)
FROM players
GROUP BY
    id, first_name, last_name, bats, throws , weight, height, debut, final_game, birth_year, birth_month, birth_day, birth_city, birth_state, birth_country
HAVING COUNT(*) > 1;

SELECT * , COUNT(*)
FROM teams
GROUP BY
    id, year, name, park
HAVING COUNT(*) > 1;

SELECT * , COUNT(*)
FROM salaries
GROUP BY
    id, player_id, team_id, year, salary
HAVING COUNT(*) > 1;

SELECT * , COUNT(*)
FROM performances
GROUP BY
    id, player_id, team_id, year, G, AB, H, 2B, 3B, HR, RBI, SB
HAVING COUNT(*) > 1;


-- 1. How have average player salaries evolved over time?

SELECT 
    year,
    ROUND(AVG(salary), 2) AS "average salary"
FROM salaries
GROUP BY year
ORDER BY year;


-- 2. What is Cal Ripken Jr.’s salary history?

SELECT 
    year,
    salary
FROM salaries
JOIN players ON salaries.player_id = players.id
WHERE players.first_name = 'Cal' AND players.last_name = 'Ripken'
ORDER BY year;


-- 3. What is Ken Griffey Jr.’s home run trajectory?

SELECT
    pf.year,
    pf.hr
FROM performances pf
JOIN players p ON pf.player_id = p.id
WHERE p.first_name = 'Ken' 
AND p.last_name = 'Griffey'
AND p.birth_year = 1969
ORDER BY pf.year;


-- 4. Who were the 50 lowest-paid players in 2001?

SELECT
    p.first_name,
    p.last_name,
    s.salary
FROM players p 
JOIN salaries s ON s.player_id = p.id
WHERE s.year = 2001
ORDER BY 
    s.salary,
    p.first_name,
    p.last_name
LIMIT 50;


-- 5. Which teams did Satchel Paige play for?

SELECT DISTINCT 
    t.name
FROM teams t
JOIN performances pf ON t.id = pf.team_id AND t.year = pf.year
JOIN players p ON pf.player_id = p.id
WHERE p.first_name = 'Satchel' AND p.last_name = 'Paige';


-- 6. Top 5 teams by total hits in 2001?

SELECT
    t.name,
    SUM(p.H) AS "total hits"
FROM teams AS t
JOIN performances AS p ON t.id = p.team_id AND t.year = p.year
WHERE p.year = 2001
GROUP BY t.name
ORDER BY "total hits" DESC
LIMIT 5;


-- 7. Who received the highest salary in MLB history?

SELECT
    pl.first_name,
    pl.last_name
FROM salaries AS s
JOIN players AS pl ON s.player_id = pl.id
ORDER BY s.salary DESC
LIMIT 1;


-- 8. Cost of acquiring the best home run hitter in 2001?

SELECT
    s.salary
FROM performances AS p
JOIN salaries AS s ON p.player_id = s.player_id AND p.year = s.year
WHERE p.year = 2001
ORDER BY p.HR DESC
LIMIT 1;


-- 9. 5 lowest-paying teams by average salary in 2001?

SELECT
    t.name,
    ROUND(AVG(s.salary), 2) AS "average salary"
FROM salaries AS s
JOIN teams AS t ON s.team_id = t.id AND s.year = t.year
WHERE s.year = 2001
GROUP BY t.name
ORDER BY "average salary" ASC
LIMIT 5;


-- 10. Player-level yearly stats: home runs & salaries

SELECT
    pl.first_name,
    pl.last_name,
    s.salary,
    p.HR,
    s.year
FROM players AS pl
JOIN salaries AS s ON pl.id = s.player_id
JOIN performances AS p ON s.player_id = p.player_id AND s.year = p.year
ORDER BY
    pl.id ASC,
    s.year DESC,
    p.HR DESC,
    s.salary DESC;


-- 11. 10 best value-for-money players (cost per hit)

SELECT
    p.first_name,
    p.last_name,
    ROUND(s.salary / pf.H, 2) AS "dollars per hit"
FROM players AS p
JOIN salaries AS s ON p.id = s.player_id
JOIN performances AS pf ON s.player_id = pf.player_id AND s.year = pf.year
WHERE s.year = 2001 AND pf.H > 0
ORDER BY "dollars per hit" ASC, p.first_name ASC, p.last_name ASC
LIMIT 10;


-- 12. Overlap: best cost per hit and per RBI

SELECT
    p.first_name,
    p.last_name
FROM players AS p
WHERE p.id IN (
    SELECT p.id
    FROM players AS p
    JOIN salaries AS s ON p.id = s.player_id
    JOIN performances AS pf ON s.player_id = pf.player_id AND s.year = pf.year
    WHERE s.year = 2001 AND pf.H > 0
    ORDER BY s.salary * 1.0 / pf.H ASC
    LIMIT 10
)
AND p.id IN (
    SELECT p.id
    FROM players AS p
    JOIN salaries AS s ON p.id = s.player_id
    JOIN performances AS pf ON s.player_id = pf.player_id AND s.year = pf.year
    WHERE s.year = 2001 AND pf.RBI > 0
    ORDER BY s.salary * 1.0 / pf.RBI ASC
    LIMIT 10
)
ORDER BY p.id ASC;
```
