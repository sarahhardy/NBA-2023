
{:toc}

# NBA-2023
SQLite Examples with NBA data
# Table of Contents
- [Original teams](#Original teams)
  - 
# Original teams
# What original teams are still playing in the NBA? 
SELECT nickname, city, abbreviation, year_founded<br>
FROM team<br>
WHERE year_founded =<br> 
 &emsp; (SELECT MIN(year_founded) FROM team)<br>
ORDER BY abbreviation ASC<br>
;<br>
Results:<br>
<img width="281" alt="image" src="https://github.com/sarahhardy/NBA-2023/assets/7597401/89afe0e9-543a-4478-9d59-9c1b1aaf34a6">

# Foreign Talent
## Over the last 20 years, which non-US countries have had the most players play in the NBA?
<b>Include only those with 2 or more players.</b>

SELECT count(*) AS count, country<br>
FROM common_player_info<br>
WHERE country NOT LIKE 'USA' and STRFTIME('%Y','now') - from_year <= 20<br>
GROUP BY country<br>
HAVING count >=2
ORDER BY count DESC<br>
;<br>
Results:<br>
<img width="114" alt="image" src="https://github.com/sarahhardy/NBA-2023/assets/7597401/6e6265fe-c15e-48ec-9d59-9f003f79876d">

## Currently active foreign talent
### How many currently active players came to the NBA from a country other than the USA?
SELECT count(*) AS count_players<br>
FROM common_player_info AS cpi<br>
INNER JOIN player AS p<br>
	            &emsp; ON cpi.person_id = p.id<br>
WHERE cpi.country NOT LIKE 'USA'<br>
	            &emsp;  AND p.is_active = 1<br>
;<br>
Results: 26 <br>
### Who are the currently active players came to the NBA from a country other than the USA?

SELECT cpi.person_id,<br>
   		&emsp cpi.display_first_last AS name,<br>
   		&emsp; cpi.country,<br>
   		&emsp; cpi.from_year,<br>
   		&emsp; (cpi.to_year - cpi.from_year) AS nba_years,<br>
   		&emsp; p.id AS player_id, p.is_active<br>
FROM common_player_info AS cpi<br>
INNER JOIN player AS p<br>
   	&emsp; ON cpi.person_id = p.id<br>
WHERE cpi.country NOT LIKE 'USA'<br>
   	  &emsp; AND is_active = 1<br>
ORDER BY nba_years DESC<br>
;<br>
# Data Sources:
1. Walsh, Wyatt (2023, June). NBA Database, Daily Updated SQLite Database. Retrieved June 21, 2023 from
   https://www.kaggle.com/datasets/wyattowalsh/basketball.
2. Ugur, Serhat (2023). NBAstuffer, 2022-2023 Player Stats, Regular Season. Retrieved June 23, 2023 from https://www.nbastuffer.com/2022-2023-nba-player-stats/.

   
