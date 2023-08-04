
# NBA-2023
SQLite Examples with NBA data

# What original teams are still playing in the NBA? 

SELECT nickname, city, abbreviation, year_founded<br>
FROM team<br>
WHERE year_founded =<br> 
 &emsp; (SELECT MIN(year_founded) FROM team)<br>
ORDER BY abbreviation ASC<br>
;<br>
**Results:** <br>
<img width="281" alt="image" src="https://github.com/sarahhardy/NBA-2023/assets/7597401/89afe0e9-543a-4478-9d59-9c1b1aaf34a6">

-----------------------------------
# Foreign Talent

## Over the last 20 years, which non-US countries have had the most players play in the NBA?

Note: Includes only those countries with 2 or more NBA players.

-----------------------------------------------------------------------
SELECT count(*) AS count, country<br>
FROM common_player_info<br>
WHERE country NOT LIKE 'USA' and STRFTIME('%Y','now') - from_year <= 20<br>
GROUP BY country<br>
HAVING count >=2<br>
ORDER BY count DESC<br>
;<br>
**Results:** <br>
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
**Results:** 26 currently active players came to the NBA from a country other than the USA. <br>

### Who are the currently active foreign players and how long have they been playing? Who has been playing for the longest?

SELECT cpi.person_id,<br>
   		&emsp; cpi.display_first_last AS name,<br>
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

**Results:** Kyrie Irving from Australia,who has been playing in the NBA for 11 years, is the foreign player who has been playing the longest.
The complete list of all 26 foreign players can be found in [current_foreign_players.csv](current_foreign_players.csv).

----------------------------
# Traded Players
## How many players played for more than one team during the 2022-2023 season?

SELECT player, count_teams<br>
 FROM <br>
 ( <br>
&emsp;	SELECT count(*) AS count_teams,  player <br>
&emsp;	FROM player_stats <br>
&emsp;	GROUP BY player <br>
&emsp;	HAVING count(*) > 1 <br>
&emsp;	ORDER BY count_teams DESC, player <br>
) <br>
; <br>
**Result:** 70 players played for 2 teams during the 2022-2023 season. No players played for 3 teams.
The complete list of all 70 players can be found in  [traded_players.csv](traded_players.csv).

------------------------

## Which teams did the players who played for more than one team in 2022-2023 play for?

Note: Each team the player played for is included.

SELECT player, tm <br>
FROM player_stats <br>
WHERE player IN <br>
&emsp;	( <br>
&emsp;SELECT player <br>
	&emsp;	FROM <br>
		&emsp;	&emsp;( <br>
		&emsp;	&emsp; SELECT count(*) AS count_teams, player <br>
		&emsp;	&emsp;	FROM player_stats <br>
		&emsp;	&emsp;	GROUP BY player <br>
		&emsp;	&emsp;	HAVING count(*) > 1 <br>
		&emsp;	&emsp;	ORDER BY count_teams DESC <br>
		&emsp;	&emsp;	) <br>
	) <br>
ORDER BY player <br>
;<br>
**Result:** Dataset idenitifying both teams the 70 traded players played for:  [traded_players_with_teams.csv](traded_players_with_teams.csv).<br>

------------------

## How many teams were involved in midseason trades? Which team had the most traded players on their roster?
Notes: 
1. Any one trade tranaction will result in 2 or more players on the roster involved in a trade. Trades in this output refers to traded players on the roster, not to trade transactions.
   
-----------------------   
SELECT team.full_name,<br>
&emsp;       abbreviation,<br>
&emsp;	   -- Replace the NULLs with zeros for the teams that had no trades<br>
&emsp;	  ifnull(trades.count_teams,0) AS count_trades	<br>
FROM team<br>
LEFT OUTER JOIN (<br>
&emsp;		SELECT count(*) AS count_teams,<br> 
&emsp;	&emsp;			   ps.tm,<br>
&emsp;	&emsp;			   team.full_name <br>
&emsp;		FROM player_stats AS ps <br>
&emsp;		INNER JOIN team <br>
&emsp;	&emsp;			ON team.abbreviation = ps.tm <br>	
&emsp;		WHERE ps.player IN <br>
&emsp;	&emsp;			(SELECT player <br>
&emsp;	&emsp;				 FROM <br>
&emsp;	&emsp;					( <br>
&emsp;	&emsp;	&emsp;				SELECT count(*) AS count_teams, player <br>
&emsp;	&emsp;	&emsp;					FROM player_stats <br>
&emsp;	&emsp;	&emsp;						GROUP BY player <br>
&emsp;	&emsp;	&emsp;						HAVING count(*) > 1 <br>
&emsp;	&emsp;	&emsp;						ORDER BY count_teams DESC <br>
&emsp;	&emsp;	&emsp;						)
&emsp;	&emsp;			) <br>
		GROUP BY tm) AS trades <br>
ON trades.tm = team.abbreviation <br>
ORDER BY count_trades DESC <br>
; <br>

**Results:** 27 of the 30 team were involved in a trade. The Los Angeles Lakers, with 13 traded players on their roster, had the most traded players. From the ESPN website:<br>

-------------------------------------

### Query that returns all 30 teams with a count of how many players on their 2022-23 roster were traded during the season and their winning percentage.

Note: 
1. Any one trade will result in 2 or more players on the roster involved in a trade. This query returns all 30 teams, including 3 that did not make trades. For these 3 teams, count_trades is originally NULL, but the NULL is replaced
with a zero.
----------------------------------------------------
SELECT team.full_name,<br>
&emsp;       abbreviation,<br>
&emsp; 	   ts.WIN_pct,<br>
&emsp;	   -- Replace the NULLs with zeros for the teams that had no trades<br>
&emsp;	  ifnull(trades.count_teams,0) AS count_trades	<br>
FROM team<br>
LEFT OUTER JOIN (<br>
&emsp;		SELECT count(*) AS count_teams,<br> 
&emsp;	&emsp;			   ps.tm,<br>
&emsp;	&emsp;			   team.full_name <br>
&emsp;		FROM player_stats AS ps <br>
&emsp;		INNER JOIN team <br>
&emsp;	&emsp;			ON team.abbreviation = ps.tm <br>	
&emsp;		WHERE ps.player IN <br>
&emsp;	&emsp;			(SELECT player <br>
&emsp;	&emsp;				 FROM <br>
&emsp;	&emsp;					( <br>
&emsp;	&emsp;	&emsp;				SELECT count(*) AS count_teams, player <br>
&emsp;	&emsp;	&emsp;					FROM player_stats <br>
&emsp;	&emsp;	&emsp;						GROUP BY player <br>
&emsp;	&emsp;	&emsp;						HAVING count(*) > 1 <br>
&emsp;	&emsp;	&emsp;						ORDER BY count_teams DESC <br>
&emsp;	&emsp;	&emsp;						)
&emsp;	&emsp;			) <br>
		GROUP BY tm) AS trades <br>
ON trades.tm = team.abbreviation <br>
INNER JOIN team_stats AS ts -- Add in team_stats to get WIN_pct <br>
ON ts.abbreviation = team.abbreviation
ORDER BY count_trades DESC <br>
; <br>

**Result:**

----------------

# Data Sources:
1. Walsh, Wyatt (2023, June). NBA Database, Daily Updated SQLite Database. Retrieved June 21, 2023 from
   https://www.kaggle.com/datasets/wyattowalsh/basketball.
2. Ugur, Serhat (2023). NBAstuffer, 2022-2023 Player Stats, Regular Season. Retrieved June 23, 2023 from https://www.nbastuffer.com/2022-2023-nba-player-stats/.

   
