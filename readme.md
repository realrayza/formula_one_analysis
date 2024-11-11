[constructor_most_win.csv](https://github.com/user-attachments/files/17705519/constructor_most_win.csv)# Formula One Analysis

## Project Overview

The project aims to provide insight into the Formula one seasonal records from May 1950 t0 July 2004. We aim to review differnet records set by drivers, constructors and host countries.
This project is important because with the reports, we can easily track drivers who have the highest number of grandprix wins, the drivers that have achieved fastest laptimes in a formula one grand prix and also countries that have produced the highest number of formula one winners.
The dataset was gotten from [kaggle.com](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020).
The dataset contains information about
- Race events and dates
- Race results
- Formula One constructors information and standing
- Driver information and standing
In this analysis, I focused only on the **Grand Prix** races

## Tools used in the analysis
- Pgadmin Postgresql - Data cleanup and analysis [Download here](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
- Microsoft Power Bi - Data visualization

## Data Cleaning/preparation
In the initial phase of the preparation, I performed the following tasks:
1) Setting up the data: I filtered out the required data I needed from the different datasets
 a)  Combining racing_standing and racing result
``WITH racing_result AS (
SELECT rc.raceid,
rc.year,
rc.round,
rc.circuitid,
rc.name,
rc.date,
rc.time,
rc.quali_time,
rc.sprint_date,
rc.sprint_time,
r.resultid,
r.driverid,
r.constructorid,
r.number,
r.grid,
r.position,
r.positiontext,
r.positionorder,
r.points,
r.laps,
r.time AS result_time,
r.fastestlap,
r.rank,
r.fastestlaptime,
r.fastestlapspeed,
r.statusid
FROM racing AS rc
LEFT JOIN result AS r
ON r.raceid = rc.raceid
)
SELECT *
--creating a new table: racing_result
INTO racing_result
FROM racing_result``

b) Creating a table consisting of race,result,circuit and drivers

``SELECT raceid,
year,
round,
rs.name,
date,
time,
rs.resultid,
rs.driverid,
rs.number,
grid,
position,
positiontext,
points,
laps,
result_time,
fastestlap,
rank,
fastestlaptime,
fastestlapspeed,
code,
forename,
surname,
nationality,
c.circuitid,
circuitref,
c.name AS circuit_name,
location,
country,
constructorid
INTO race_driver_circuit
FROM racing_result AS rs
LEFT JOIN drivers AS d
on rs.driverid = d.driverid
LEFT JOIN circuits AS c
on rs.circuitid = c.circuitid
ORDER BY YEAR``

c) FIltered out with just race informations and driver positons

``SELECT raceid, year, round, name AS grand_prix, country, circuit_name, driverid, forename, surname, nationality AS racer_nationality, 
CASE WHEN result_time = '\N' THEN null ELSE result_time END AS result_time,  
CASE WHEN position = '\N' THEN null ELSE position END AS position, 
CASE WHEN fastestlaptime = '\N' THEN null ELSE fastestlaptime END AS fastestlaptime,
constructorid
INTO driver_race
FROM race_driver_circuit
ORDER BY year, round, name, position``

d) JOIN table to constructor table
``SELECT raceid, year, round, grand_prix, country, circuit_name,driverid, forename, surname, racer_nationality, 
result_time, position, fastestlaptime, dr.constructorid,
cn.name AS constructor_name,
cn.nationality AS cons_nationality
INTO driver_race_constructor
FROM driver_race AS dr
LEFT JOIN constructors_name AS cn
ON dr.constructorid = cn.constructorid``

e) FIltering out for the first 3 pole positions in each grandprix
``SELECT raceid, year, round, grand_prix, country, circuit_name,driverid, forename, surname, racer_nationality, 
result_time, position :: INTEGER AS race_position, fastestlaptime, constructorid,
constructor_name, cons_nationality
INTO driver_race_const
FROM driver_race_constructor
WHERE position IN ('1','2','3')``


## Exploratory Data Analysis
The exploratory data analysis involved exploring the formula one data to answer key questions such as:
a) Constructors with the highest number of wins
``SELECT constructor_name, COUNT(race_position) AS total_wins
INTO constructor_most_win
FROM driver_race_const
WHERE race_position = '1'
GROUP BY constructor_name
ORDER BY total_wins DESC``


b) Driver with the most wins
c) Each Driver's fastest times in each grand prix
d) Fastest time in each grandprix
e) Drivers with the fastest laps in each grand prix
f) Drivers with the fastest laps in each grandprix and constructors
g) Driver with the most number of fastest lap
h) Drivers, their  fastest laps, the year and the grand prix
i) Fastest laptime ever
j) Driver with the fastest laptime
k) Grand prix winners
l) Drivers who had the fastest laps and also won the grand prix
m) Most hosted grand prix
n) Country that has hosted the most grand prix
O) Grand prix winners and the different contractors they've driven for
p) Grand prix winners and the number of contractors they've driven for
q) Different grand prix in each country
r) number of grand prix per country
s) Grand prix won by a national of the host country
t) Number of grand prix won by a national of the host country
u) Grand prix won by non national of host country
v) Number of grand prix won by a non national of the host country
w) Host countries and the number of drivers of same country who won their grand prix
X) Grand prix winners by nationality
y) Number of grand prix winners per nationality
z) Total points by each grand prix winner in a grandprix
a1) Total points by driver in a formula one season
a2) Driver with the highest points per season
a3) Overall point tally per driver
a4) Driver with overall point tall
a5) Seasons and total point
a6) Season with most point
a7) How many times a driver won the fastest lap and also won the grand prix
