# Moneyball SQL Analysis ⚾ | Strategic Player Value Assessment

**Client Scenario:** Oakland Athletics (2001)  
**Perspective as:** Data Analyst – Recruitment Strategy & Salary Optimization  
**Tools:** SQL (SQLite), Relational Databases, Aggregation, Filtering, Performance Metrics

Project developed as part of Harvard University’s CS50’s Introduction to Databases with SQL.

---

##  Business Objective

In 2001, the Oakland Athletics faced one of Major League Baseball’s most pressing challenges: build a competitive team with a drastically limited budget. My responsibility was to identify undervalued players using historical salary and performance data, and to support strategic decision-making for player acquisition.

---
### Dataset Overview

This analysis is based on the `moneyball.db` SQLite database, which contains four interconnected tables: `players`, `teams`, `salaries`, and `performances`. Together, these datasets enable a comprehensive exploration of player performance, compensation, and team history to uncover value-driven insights in baseball.

### Time Coverage

The data spans more than 150 years of professional baseball history. Player careers range from May 4, 1871, to as recently as October 4, 2022, reflecting updates beyond the Moneyball era. However, salary and team performance data are concentrated between 1985 and 2001, aligning with the core focus of the Moneyball strategy. This period is ideal for exploring player value, performance efficiency, and market inefficiencies in player compensation.


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

# Important or Relevant Insights

🔹 Salaries Have Risen Rapidly (1985–2001)
Average salary in 1985: $476,000
By 2001: $3.4 million yearly by player?
Salary growth significantly outpaced performance growth, indicating inefficiencies in the player market

🔹 Undervalued Players Deliver High ROI
A.J. Pierzynski had the best cost per hit: ~$1,909 per hit
Other players like Torii Hunter appeared in both top 10 cost-efficiency lists (per hit and per RBI)

🔹 Position Doesn’t Always Predict Pay
Some low-profile players produced more hits or RBIs per salary dollar than star players
Demonstrated that raw stats and efficiency metrics outperform reputation in valuation

🔹 Top Performer in HR vs. Salary
The top home run hitter in 2001 earned $10.3M
While this player delivered results, multiple others achieved strong HR numbers at a fraction of the cost

🔹 Low-Paying Teams Still Competitive
Teams like Anaheim, Arizona, and Atlanta had low average salaries but strong performance metrics
Validated the idea that team strategy can overcome budget constraints

🔹 Legacy Insight: Cal Ripken Jr.
His salary rose as his career advanced, but his later performance didn’t justify the high pay—demonstrating aging curve inefficiency in contracts


# Reccomendations

## 1. Prioritize Performance-Based Scouting Over Reputation
With evidence that less famous players often deliver better ROI (e.g., A.J. Pierzynski and Torii Hunter), teams should:
Adopt advanced efficiency metrics (e.g., cost per hit, cost per RBI) to evaluate talent.
Shift focus from star power to data-driven player performance when making recruitment or contract decisions.
Regularly reassess veteran contracts to avoid overpaying based on past performance or reputation.

## 2. Build Competitive Teams on Smaller Budgets
Low-salary teams like Anaheim, Arizona, and Atlanta showed strong results, proving that:
Strategic roster planning and efficiency-focused metrics can compensate for limited payroll.
Investing in player development, scouting, and analytics can uncover undervalued talent and maintain competitiveness.
Encourage a "Moneyball" philosophy to optimize team building—focusing on output per dollar rather than big names.

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
FROM players p
WHERE p.id IN (
    SELECT sub1.id
    FROM (
        SELECT p.id
        FROM players AS p
        JOIN salaries AS s ON p.id = s.player_id
        JOIN performances AS pf ON s.player_id = pf.player_id AND s.year = pf.year
        WHERE s.year = 2001 AND pf.H > 0
        ORDER BY s.salary * 1.0 / pf.H ASC
        LIMIT 10
    ) AS sub1
)
AND p.id IN (
    SELECT sub2.id
    FROM (
        SELECT p.id
        FROM players AS p
        JOIN salaries AS s ON p.id = s.player_id
        JOIN performances AS pf ON s.player_id = pf.player_id AND s.year = pf.year
        WHERE s.year = 2001 AND pf.RBI > 0
        ORDER BY s.salary * 1.0 / pf.RBI ASC
        LIMIT 10
    ) AS sub2
)
ORDER BY p.id ASC;

```
