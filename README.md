#Primary Insights

# 1. Top 10 batsmen based on past 3 years total runs scored.

SELECT batsmanName,
sum(runs) as total_runs_scored
FROM ipl_2024.fact_bating_summary
group by batsmanName
order by total_runs_scored desc
limit 10;

# 2. Top 10 batsmen based on past 3 years batting average. (min 60 balls faced in each season)

WITH yearly_stats AS (
    SELECT
        fs.batsmanName,
        dm.matchyear,
        SUM(fs.runs) AS total_runs,
        SUM(fs.balls) AS total_balls
    FROM
        fact_bating_summary fs
    JOIN
        dim_match_summary dm ON fs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
    GROUP BY
        fs.batsmanName,
        dm.matchyear
),
yearly_ball_counts AS (
    SELECT
        batsmanName,
        SUM(CASE WHEN matchyear = 2021 THEN total_balls END) AS balls_2021,
        SUM(CASE WHEN matchyear = 2022 THEN total_balls END) AS balls_2022,
        SUM(CASE WHEN matchyear = 2023 THEN total_balls END) AS balls_2023
    FROM
        yearly_stats
    GROUP BY
        batsmanName
),
qualified_batsmen AS (
    SELECT
        batsmanName
    FROM
        yearly_ball_counts
    WHERE
        balls_2021 >= 60 AND balls_2022 >= 60 AND balls_2023 >= 60
),
overall_stats AS (
    SELECT
        fs.batsmanName,
        SUM(fs.runs) AS overall_runs,
        SUM(fs.out) AS overall_outs
    FROM
        fact_bating_summary fs
    JOIN
        dim_match_summary dm ON fs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
        AND fs.batsmanName IN (SELECT batsmanName FROM qualified_batsmen)
    GROUP BY
        fs.batsmanName
),
final_stats AS (
    SELECT
        os.batsmanName,
        os.overall_runs,
        os.overall_outs,
        round((os.overall_runs / os.overall_outs),2) AS batting_average
    FROM
        overall_stats os
)
SELECT
    batsmanName,
    overall_runs,
    overall_outs,
    batting_average
FROM
    final_stats
ORDER BY
    batting_average DESC
LIMIT 10;

# 3. Top 10 batsmen based on past 3 years strike rate (min 60 balls faced in each season)

WITH yearly_stats AS (
    SELECT
        fs.batsmanName,
        dm.matchyear,
        SUM(fs.runs) AS total_runs,
        SUM(fs.balls) AS total_balls
    FROM
        fact_bating_summary fs
    JOIN
        dim_match_summary dm ON fs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
    GROUP BY
        fs.batsmanName,
        dm.matchyear
),
yearly_ball_counts AS (
    SELECT
        batsmanName,
        SUM(CASE WHEN matchyear = 2021 THEN total_balls END) AS balls_2021,
        SUM(CASE WHEN matchyear = 2022 THEN total_balls END) AS balls_2022,
        SUM(CASE WHEN matchyear = 2023 THEN total_balls END) AS balls_2023
    FROM
        yearly_stats
    GROUP BY
        batsmanName
),
qualified_batsmen AS (
    SELECT
        batsmanName
    FROM
        yearly_ball_counts
    WHERE
        balls_2021 >= 60 AND balls_2022 >= 60 AND balls_2023 >= 60
),
overall_stats AS (
    SELECT
        fs.batsmanName,
        SUM(fs.runs) AS overall_runs,
        SUM(fs.balls) AS overall_balls
    FROM
        fact_bating_summary fs
    JOIN
        dim_match_summary dm ON fs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
        AND fs.batsmanName IN (SELECT batsmanName FROM qualified_batsmen)
    GROUP BY
        fs.batsmanName
),
final_stats AS (
    SELECT
        os.batsmanName,
        os.overall_runs,
        os.overall_balls,
        round((os.overall_runs / os.overall_balls) * 100,2) AS strike_rate
    FROM
        overall_stats os
)
SELECT
    batsmanName,
    overall_runs,
    overall_balls,
    strike_rate
FROM
    final_stats
ORDER BY
    strike_rate DESC
LIMIT 10;

#4. Top 10 bowlers based on past 3 years total wickets taken.

Select bowlername,
SUM(wickets) as total_wickets
from fact_bowling_summary
group by bowlerName
order by total_wickets desc
Limit 10;

#5. Top 10 bowlers based on past 3 years bowling average. (min 60 balls bowled in each season)

WITH yearly_stats AS (
    SELECT
        fbs.bowlerName,
        dm.matchyear,
        SUM(fbs.runs) AS total_runs_conceded,
        SUM(fbs.balls) AS total_balls_bowled
    FROM
        fact_bowling_summary fbs
    JOIN
        dim_match_summary dm ON fbs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
    GROUP BY
        fbs.bowlerName,
        dm.matchyear
),
yearly_balls_bowled AS (
    SELECT
        bowlerName,
        SUM(CASE WHEN matchyear = 2021 THEN total_balls_bowled END) AS balls_bowled_2021,
        SUM(CASE WHEN matchyear = 2022 THEN total_balls_bowled END) AS balls_bowled_2022,
        SUM(CASE WHEN matchyear = 2023 THEN total_balls_bowled END) AS balls_bowled_2023
    FROM
        yearly_stats
    GROUP BY
        bowlerName
),
qualified_bowlers AS (
    SELECT
        bowlerName
    FROM
        yearly_balls_bowled
    WHERE
        balls_bowled_2021 >= 60 AND balls_bowled_2022 >= 60 AND balls_bowled_2023 >= 60
),
overall_stats AS (
    SELECT
        fbs.bowlerName,
        SUM(fbs.runs) AS overall_runs_conceded,
        SUM(fbs.wickets) AS overall_wickets_taken
    FROM
        fact_bowling_summary fbs
    JOIN
        dim_match_summary dm ON fbs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
        AND fbs.bowlerName IN (SELECT bowlerName FROM qualified_bowlers)
    GROUP BY
        fbs.bowlerName
),
final_stats AS (
    SELECT
        os.bowlerName,
        os.overall_runs_conceded,
        os.overall_wickets_taken,
        round(SUM(os.overall_runs_conceded) / SUM(os.overall_wickets_taken),2) AS bowling_avg
    FROM
        overall_stats os
	group by 
        os.bowlerName
)
SELECT
    bowlerName,
    overall_runs_conceded,
    overall_wickets_taken,
    bowling_avg
FROM
    final_stats
ORDER BY
    bowling_avg ASC
LIMIT 10;

#6. Top 10 bowlers based on past 3 years economy rate. (min 60 balls bowled in each season)

WITH yearly_stats AS (
    SELECT
        fbs.bowlerName,
        dm.matchyear,
        SUM(fbs.runs) AS total_runs_conceded,
        SUM(fbs.balls) AS total_balls_bowled
    FROM
        fact_bowling_summary fbs
    JOIN
        dim_match_summary dm ON fbs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
    GROUP BY
        fbs.bowlerName,
        dm.matchyear
),
yearly_balls_bowled AS (
    SELECT
        bowlerName,
        SUM(CASE WHEN matchyear = 2021 THEN total_balls_bowled END) AS balls_bowled_2021,
        SUM(CASE WHEN matchyear = 2022 THEN total_balls_bowled END) AS balls_bowled_2022,
        SUM(CASE WHEN matchyear = 2023 THEN total_balls_bowled END) AS balls_bowled_2023
    FROM
        yearly_stats
    GROUP BY
        bowlerName
),
qualified_bowlers AS (
    SELECT
        bowlerName
    FROM
        yearly_balls_bowled
    WHERE
        balls_bowled_2021 >= 60 AND balls_bowled_2022 >= 60 AND balls_bowled_2023 >= 60
),
overall_stats AS (
    SELECT
        fbs.bowlerName,
        SUM(fbs.runs) AS overall_runs_conceded,
        SUM(fbs.balls) AS overall_balls_bowled
    FROM
        fact_bowling_summary fbs
    JOIN
        dim_match_summary dm ON fbs.match_id = dm.match_id
    WHERE
        dm.matchyear BETWEEN 2021 AND 2023
        AND fbs.bowlerName IN (SELECT bowlerName FROM qualified_bowlers)
    GROUP BY
        fbs.bowlerName
),
final_stats AS (
    SELECT
        os.bowlerName,
        os.overall_runs_conceded,
        os.overall_balls_bowled,
        round(SUM(os.overall_runs_conceded) / SUM(os.overall_balls_bowled/6),2) AS economy_rate
    FROM
        overall_stats os
	group by 
        os.bowlerName
)
SELECT
    bowlerName,
    overall_runs_conceded,
    overall_balls_bowled,
    round(sum(overall_balls_bowled/6)) as overs_bowled,
    economy_rate    
FROM
    final_stats
group by
    bowlerName
ORDER BY
    economy_rate ASC
LIMIT 10;

# 7. Top 5 batsmen based on past 3 years boundary % (fours and sixes).

WITH boundary_stats AS (
    SELECT
        fbs.batsmanName,
        SUM(fbs.Fours * 4 + fbs.Sixes * 6) AS boundary_runs, -- Total boundary runs
        SUM(fbs.runs) AS total_runs -- Total runs scored
    FROM
        fact_bating_summary fbs
    INNER JOIN
        dim_match_summary dms ON fbs.match_id = dms.match_id
    WHERE
        dms.matchyear BETWEEN YEAR(CURDATE()) - 3 AND YEAR(CURDATE()) - 1
    GROUP BY
        fbs.batsmanName
),
boundary_percentage AS (
    SELECT
        batsmanName,
        boundary_runs,
        total_runs,
        round((sum(boundary_runs) / sum(total_runs) * 100),2) AS boundary_percent -- Calculating boundary percentage
    FROM
        boundary_stats
    WHERE
        total_runs > 500
	group by
        batsmanName
)
SELECT
    batsmanName,
    boundary_runs,
    total_runs,
    boundary_percent
FROM
    boundary_percentage
ORDER BY
    boundary_percent DESC
LIMIT 5;

# 8. Top 5 bowlers based on past 3 years dot ball %.

WITH yearly_performance AS (
    SELECT
        dms.matchyear,
        fbs.bowlerName,
        SUM(fbs.Zeros) AS total_dot_balls,
        SUM(fbs.balls) AS total_balls
    FROM
        fact_bowling_summary fbs
    JOIN
        dim_match_summary dms ON fbs.match_id = dms.match_id
    WHERE
        dms.matchyear IN (2021, 2022, 2023)
    GROUP BY
        dms.matchyear, fbs.bowlerName
),
bowlers_active_all_years AS (
    SELECT
        bowlerName
    FROM
        yearly_performance
    GROUP BY
        bowlerName
    HAVING
        COUNT(DISTINCT matchyear) = 3
),
overall_performance AS (
    SELECT
        yp.bowlerName,
        SUM(yp.total_dot_balls) AS total_dot_balls,
        SUM(yp.total_balls) AS total_balls
    FROM
        yearly_performance yp
    WHERE
        yp.bowlerName IN (SELECT bowlerName FROM bowlers_active_all_years)
    GROUP BY
        yp.bowlerName
),
dot_ball_percentage AS (
    SELECT
        op.bowlerName,
        op.total_dot_balls,
        op.total_balls,
        round((op.total_dot_balls / op.total_balls)* 100,2) AS dotball_percent
    FROM
        overall_performance op
)
SELECT
    dbp.bowlerName,
    dbp.total_dot_balls,
    dbp.total_balls,
    dbp.dotball_percent
FROM
    dot_ball_percentage dbp
ORDER BY
    dbp.dotball_percent DESC
LIMIT 5;

# 9. Top 4 teams based on past 3 years winning %.

WITH team_wins AS (
    SELECT
        team1 AS team,
        matchyear,
        COUNT(*) AS wins
    FROM
        dim_match_summary
    WHERE
        winner = team1
        AND matchyear BETWEEN 2021 AND 2023
    GROUP BY
        team1, matchyear
    UNION ALL
    SELECT
        team2 AS team,
        matchyear,
        COUNT(*) AS wins
    FROM
        dim_match_summary
    WHERE
        winner = team2
        AND matchyear BETWEEN 2021 AND 2023
    GROUP BY
        team2, matchyear
),
team_matches AS (
    SELECT
        team1 AS team,
        matchyear,
        COUNT(*) AS matches
    FROM
        dim_match_summary
    WHERE
        matchyear BETWEEN 2021 AND 2023
    GROUP BY
        team1, matchyear
    UNION ALL
    SELECT
        team2 AS team,
        matchyear,
        COUNT(*) AS matches
    FROM
        dim_match_summary
    WHERE
        matchyear BETWEEN 2021 AND 2023
    GROUP BY
        team2, matchyear
)
SELECT
    tw.team,
    SUM(tw.wins) AS total_wins,
    SUM(tm.matches) AS total_matches,
    round((SUM(tw.wins) / SUM(tm.matches)) * 100, 2) AS winning_percentage
FROM
    team_wins tw
JOIN
    team_matches tm ON tw.team = tm.team AND tw.matchyear = tm.matchyear
GROUP BY
    tw.team
ORDER BY
    winning_percentage DESC
LIMIT 4;

# 10. Top 2 teams with the highest number of wins achieved by chasing targets over the past 3 years.

With chasing_wins as (
    Select
          team2 as chasing_team,
          matchyear,
          count(*) as wins
	from
        dim_match_summary
	Where
         winner = team2 and
         matchyear between 2021 and 2023
	group by
         team2, matchyear
)
    select
         cw.chasing_team,
         sum(cw.wins) as total_wins
    from
         chasing_wins cw
    group by
         cw.chasing_team
    order by
         total_wins desc
	limit 2;
