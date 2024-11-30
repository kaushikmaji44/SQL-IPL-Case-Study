# SQL-IPL-Case-Study

create database ipl;

use ipl;

# 1. Create a table named 'matches' with appropriate data types for columns #
create table iplball
(id int,
inning int, 
overs int,
ball int,
batsman varchar(80),
non_striker varchar(80),
bowler varchar(80),
batsman_runs int,
extra_runs int,
total_runs int, 
is_wicket int,
dismissal_kind varchar(80),
player_dismissed varchar(80),
fielder	varchar(80),
extras_type	varchar(80),
batting_team varchar(80),
bowling_team varchar(80),
venue varchar(80),
match_date varchar(80));

select * from iplball LIMIT 2000;

# 2. Create a table named 'deliveries' with appropriate data types for columns #

create table iplmatches(
id int,
city varchar(80),
date varchar(80),
player_of_match varchar(80),
venue varchar(80),
neutral_venue int,
team1 varchar(80),
team2 varchar(80),
toss_winner varchar(80),
toss_decision varchar(80),
winner varchar(80),
result varchar(80),
result_margin int,
eliminator varchar(80),
method varchar(80),
umpire1 varchar(80),
umpire2 varchar(80)
);

select * from iplmatches LIMIT 2000;

SET SQL_SAFE_UPDATEs = 0;

# 5. Select the top 20 rows of the deliveries table. #

SELECT * FROM iplmatches LIMIT 20;

# 6. Select the top 20 rows of the matches table.# 

select * from iplball limit 20;

# 7. Fetch data of all the matches played on 2nd May 2013.#

select *
from iplball
where match_date = '02-05-2013';

# 8. Fetch data of all the matches where the margin of victory is more than 100 runs#

select	*
from iplmatches
where result_margin > 100 and result = 'runs';

# 9. Fetch data of all the matches where the final scores of both teams tied and order it in descending order of the date #

select *
from iplmatches
where result = 'tie' 
order by date DESC;

# 10. Get the count of cities that have hosted an IPL match#

SELECT COUNT(distinct city) FROM iplmatches;

# 11. Create table deliveries_v02 with additional column ball_result #

create table deliveries_v02 as
select *, 
    case 
        when total_runs >= 4 then 'boundary' 
        when total_runs = 0 then 'dot' 
        else 'other' 
    end as ball_result
from iplball;

select *
from deliveries_v02;

# 12. Write a query to fetch the total number of boundaries and dot balls #

select 
    SUM(CASE WHEN ball_result = 'boundary' THEN 1 ELSE 0 END) AS total_boundaries,
    SUM(CASE WHEN ball_result = 'dot' THEN 1 ELSE 0 END) AS total_dot_balls
from deliveries_v02;

# 13. Write a query to fetch the total number of boundaries scored by each team #

select batting_team, count(*) as total_boundaries
from deliveries_v02
where ball_result = 'boundary'
group by batting_team;

# 14. Write a query to fetch the total number of dot balls bowled by each team #

select bowling_team, count(*) as totaldotballs
from deliveries_v02
where ball_result = 'dot'
group by bowling_team;

# 15. Write a query to fetch the total number of dismissals by dismissal kinds #

SELECT dismissal_kind, COUNT(*) AS total_dismissals
FROM iplball
WHERE player_dismissed IS NOT NULL
GROUP BY dismissal_kind;

# 16. Write a query to get the top 5 bowlers who conceded maximum extra runs#

SELECT bowler, SUM(extra_runs) AS total_extra_runs
FROM iplball
GROUP BY bowler
ORDER BY total_extra_runs DESC
LIMIT 5;

/* 17. Write a query to create a table named deliveries_v03 with all columns of 
deliveries_v02 and two additional columns (venue and match_date) */

create table deliveries_v03 AS SELECT a.*, b.venue, b.match_date from
deliveries_v02 as a
left join (select max(venue) as venue, max(date) as match_date, id from iplball group by
id) as b
on a.id = b.id;

select * from deliveries_v03;

# 18. Write a query to fetch the total runs scored for each venue and order it in the descending order of total runs scored #

SELECT venue, SUM(total_runs) AS total_runs_scored
FROM deliveries_v03
GROUP BY venue
ORDER BY total_runs_scored DESC;

# 19. Write a query to fetch the year-wise total runs scored at Eden Gardens and order it in the descending order of total runs scored #

SELECT EXTRACT(YEAR FROM match_date) AS year, SUM(total_runs) AS total_runs_scored,venue
FROM deliveries_v02
WHERE venue = 'Eden Gardens'
GROUP BY year
ORDER BY total_runs_scored DESC;

# 20.Get unique team1 names from the matches table and create a matches_corrected table #

CREATE TABLE matches_corrected AS
SELECT *,
    CASE 
        WHEN team1 = 'Rising Pune Supergiants' THEN 'Rising Pune Supergiant' 
        ELSE team1 
    END AS team1_corr,
    CASE 
        WHEN team2 = 'Rising Pune Supergiants' THEN 'Rising Pune Supergiant' 
        ELSE team2 
    END AS team2_corr
FROM iplmatches;

SELECT team1_corr, COUNT(*) AS count
FROM matches_corrected
GROUP BY team1_corr;

SELECT team2_corr, COUNT(*) AS count
FROM matches_corrected
GROUP BY team2_corr;

# 21. Create a new table deliveries_v04 with ball_id as the first column #

create table deliveries_v04 as select *, concat(id or '-'or inning or '-'or overs or '-' or ball) as ball_id from deliveries_v02;
select * from deliveries_v04;

#23. Create table deliveries_v05 with all columns of deliveries_v04 and an additional column for row number partition over ball_id #

CREATE TABLE deliveries_v05 AS
SELECT *,
       ROW_NUMBER() OVER (PARTITION BY ball_id ORDER BY id) AS rownum
FROM deliveries_v04;

select *
from deliveries_v05;

#24. Use the r_num created in deliveries_v05 to identify instances where ball_id is repeating #

SELECT * 
FROM deliveries_v05 
WHERE rownum > 1;

# 25. Use subqueries to fetch data of all the ball_id which are repeating #

SELECT * 
FROM deliveries_v05 
WHERE ball_id IN 
(SELECT ball_id 
FROM deliveries_v05 
WHERE rownum > 1);
