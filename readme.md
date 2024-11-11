# Formula One Analysis

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

``` SQL
WITH racing_result AS (
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
INTO racing_result
FROM racing_result
```

b) Creating a table consisting of race,result,circuit and drivers

``` SQL
SELECT raceid,
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
ORDER BY YEAR
```

c) FIltered out with just race informations and driver positons

``` SQL
SELECT raceid, year, round, name AS grand_prix, country, circuit_name, driverid, forename, surname, nationality AS racer_nationality, 
CASE WHEN result_time = '\N' THEN null ELSE result_time END AS result_time,  
CASE WHEN position = '\N' THEN null ELSE position END AS position, 
CASE WHEN fastestlaptime = '\N' THEN null ELSE fastestlaptime END AS fastestlaptime,
constructorid
INTO driver_race
FROM race_driver_circuit
ORDER BY year, round, name, position
```

d) JOIN table to constructor table

``` SQL
SELECT raceid, year, round, grand_prix, country, circuit_name,driverid, forename, surname, racer_nationality, 
result_time, position, fastestlaptime, dr.constructorid,
cn.name AS constructor_name,
cn.nationality AS cons_nationality
INTO driver_race_constructor
FROM driver_race AS dr
LEFT JOIN constructors_name AS cn
ON dr.constructorid = cn.constructorid
```

e) FIltering out for the first 3 pole positions in each grandprix

``` SQL
SELECT raceid, year, round, grand_prix, country, circuit_name,driverid, forename, surname, racer_nationality, 
result_time, position :: INTEGER AS race_position, fastestlaptime, constructorid,
constructor_name, cons_nationality
INTO driver_race_const
FROM driver_race_constructor
WHERE position IN ('1','2','3')
```


## Exploratory Data Analysis

The exploratory data analysis involved exploring the formula one data to answer key questions such as:

a) Constructors with the highest number of wins

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

## Data Analysis

a) Constructors with the highest number of wins

``` SQL
SELECT constructor_name, COUNT(race_position) AS total_wins
INTO constructor_most_win
FROM driver_race_const
WHERE race_position = '1'
GROUP BY constructor_name
ORDER BY total_wins DESC
```

[Constructor with the most win](https://github.com/realrayza/formula_one_analysis/blob/main/constructor_most_win.csv)


![contructor_most_win](https://github.com/user-attachments/assets/ec9e5819-27f7-45e2-afcd-fc2a370b9b60)



b) Driver with the most wins

``` SQL
SELECT driverid,CONCAT(forename,' ',surname) AS driver, COUNT(race_position) AS total_wins
INTO driver_most_wins
FROM driver_race_const
WHERE race_position = '1'
GROUP BY driverid,driver
ORDER BY total_wins DESC
```
[Driver with most wins](https://github.com/realrayza/formula_one_analysis/blob/main/driver_most_wins.csv)

![driver_most_win](https://github.com/user-attachments/assets/3a2bdf74-3292-461a-8bda-4c64bd164465)



c) Each Driver's fastest times in each grand prix

``` SQL
SELECT year,drc.raceid, grand_prix,drc.driverid,CONCAT(forename,' ',surname) AS driver,MIN(time :: INTERVAL) AS laptimes
INTO grandprix_drivers_fastesttime
FROM driver_race_constructor AS drc
INNER JOIN laptimes AS lps
ON drc.raceid = lps.raceid
AND drc.driverid = lps.driverid
GROUP BY year,drc.raceid, grand_prix,drc.driverid,driver
ORDER BY year,drc.raceid, grand_prix,laptimes
```

[Drivers and their fastest lap in each grand prix](https://github.com/realrayza/formula_one_analysis/blob/main/grandprix_drivers_fastesttime.csv)

d) Fastest time in each grandprix

``` sql
SELECT raceid,year,grand_prix,MIN(laptimes) AS laptimes
INTO grandprix_fastest_laptimes
FROM grandprix_drivers_fastesttime
GROUP BY raceid,year,grand_prix
ORDER BY year,raceid,grand_prix
```

[Fastest laptimes in each grand prix](https://github.com/realrayza/formula_one_analysis/blob/main/grandprix_fastest_laptimes.csv)

e) Drivers with the fastest laps in each grand prix

``` sql
WITH race_laptimes AS (
SELECT raceid,laptimes
FROM grandprix_fastest_laptimes
)
SELECT rl.raceid, year,grand_prix,driverid,driver,rl.laptimes
INTO grandprix_fastest_drivers
FROM race_laptimes AS rl
LEFT JOIN grandprix_drivers_fastesttime AS gdf
ON rl.raceid = gdf.raceid AND rl.laptimes = gdf.laptimes
ORDER by year, raceid, grand_prix
```

[Drivers with fastest lap in each grand prix](https://github.com/realrayza/formula_one_analysis/blob/main/grandprix_fastest_drivers.csv)


f) Drivers with the fastest laps in each grandprix and constructors

``` sql
SELECT gfd.year,gfd.raceid,gfd.grand_prix,gfd.driverid,driver,constructorid,constructor_name,laptimes
INTO fastest_driver_constructor
FROM grandprix_fastest_drivers AS gfd
LEFT JOIN driver_race_const AS drc
ON gfd.raceid = drc.raceid 
AND gfd.driverid =drc.driverid
ORDER by gfd.year, gfd.raceid, gfd.grand_prix
```
[Drivers with fastest laps in each grand prix and their constructors](https://github.com/realrayza/formula_one_analysis/blob/main/fastest_driver_constructor.csv)

g) Driver with the most number of fastest lap

``` sql
SELECT driverid,driver, COUNT(driver) AS driver_with_most_fastest_lap,MIN(laptimes) AS fastest_lap
INTO highest_num_of_fastest_lap
FROM fastest_driver_constructor
GROUP BY driverid,driver
ORDER BY driver_with_most_fastest_lap DESC
```
[Driver with the most number of fastest laps](https://github.com/realrayza/formula_one_analysis/blob/main/highest_num_of_fastest_lap.csv)

![driver_most_fastest_lap](https://github.com/user-attachments/assets/cfdb45e0-d039-4619-bc97-011faf238b3f)


h) Drivers, their  fastest laps, the year and the grand prix

``` sql
SELECT hno.driver,hno.driverid,gfd.laptimes,raceid,year,grand_prix
INTO driver_fastest_info
FROM highest_num_of_fastest_lap AS hno
LEFT JOIN grandprix_fastest_drivers gfd
ON hno.driverid = gfd.driverid
AND hno.driver = gfd.driver
AND hno.fastest_lap = gfd.laptimes
ORDER BY driver
```
[Drivers, their  fastest laps, the year and the grand prix](https://github.com/realrayza/formula_one_analysis/blob/main/driver_fastest_info.csv)

i) Fastest laptime ever

``` sql
SELECT MIN(laptimes) AS fastest_laptimes
INTO fastest_laptime_ever
FROM driver_fastest_info
```
[Fastest laptime ever](https://github.com/realrayza/formula_one_analysis/blob/main/fastest_laptime_ever.csv)

j) Driver with the fastest laptime

``` sql
SELECT driver,driverid,laptimes AS laptime,raceid,year,grand_prix
INTO fastest_driver_ever
FROM fastest_laptime_ever AS fle
LEFT JOIN fastest_driver_constructor AS fdc
ON fle.fastest_laptimes = fdc.laptimes
```
[Driver with the fastest laptime](https://github.com/realrayza/formula_one_analysis/blob/main/fastest_driver_ever.csv)

k) Grand prix winners

``` sql
SELECT raceid,year,round,grand_prix,country,driverid,CONCAT(forename,' ',surname) AS driver,racer_nationality,constructor_name
INTO grand_prix_winners
FROM public.driver_race_const
WHERE race_position = 1
```
[Grand prix winners](https://github.com/realrayza/formula_one_analysis/blob/main/grand_prix_winners.csv)

l) Drivers who had the fastest laps and also won the grand prix

``` sql
SELECT gpw.raceid,gpw.year,gpw.grand_prix,country,gpw.driverid,gpw.driver,laptimes,constructor_name
INTO racewinner_and_also_fastestlap
FROM grand_prix_winners AS gpw
LEFT JOIN grandprix_fastest_drivers AS gfd
ON gpw.raceid = gfd.raceid
AND gpw.driverid = gfd.driverid
WHERE gfd.raceid IS NOT NULL
```
[Drivers who had the fastest laps and also won the grand prix]()
m) Most hosted grand prix

``` sql
SELECT grand_prix, COUNT(grand_prix) AS grand_prix_hosted
INTO grandprix_highest_host
FROM grand_prix_winners
GROUP BY grand_prix
ORDER BY grand_prix_hosted DESC
```
n) Country that has hosted the most grand prix

``` sql
SELECT country, COUNT(country) AS most_host_country
INTO most_host_country
FROM grand_prix_winners
GROUP BY country
ORDER BY most_host_country DESC
```
O) Grand prix winners and the different contractors they've driven for

``` sql
SELECT driverid, driver, constructor_name
INTO grandprix_winners_contractors_driven
FROM grand_prix_winners
GROUP BY driverid,driver,constructor_name
ORDER BY driver
```
p) Grand prix winners and the number of contractors they've driven for

``` sql
SELECT driverid, driver, COUNT(driver) AS number_of_contractors_driven
INTO driver_number_of_contractors_driven
FROM grandprix_winners_contractors_driven
GROUP BY driverid,driver
ORDER BY number_of_contractors_driven DESC
```
q) Different grand prix in each country

``` sql
SELECT country,grand_prix
INTO countries_and_grand_prix
FROM grand_prix_winners
GROUP BY country,grand_prix
ORDER BY country
```
r) number of grand prix per country

``` sql
SELECT country,COUNT(country) AS number_of_grandprix_per_country
INTO number_of_grandprix_per_country
FROM countries_and_grand_prix
GROUP BY country
ORDER BY number_of_grandprix_per_country DESC
```
s) Grand prix won by a national of the host country

``` sql
SELECT raceid,year,grand_prix,country,gpw.driverid,gpw.driver,nationality
INTO grandprix_won_by_national_of_hosting_country
FROM grand_prix_winners AS gpw
LEFT JOIN driver_bio AS db
ON gpw.country = db.nationality
AND gpw.driverid = db.driverid
WHERE db.nationality IS NOT NULL
```
t) Number of grand prix won by a national of the host country

``` sql
SELECT COUNT(*) AS no_of_gradnprix_won_by_national_of_host_country
INTO no_of_gradnprix_won_by_national_of_host_country
FROM grand_prix_winners AS gpw
LEFT JOIN driver_bio AS db
ON gpw.country = db.nationality
AND gpw.driverid = db.driverid
WHERE db.nationality IS NOT NULL
```
u) Grand prix won by non national of host country

``` sql
SELECT raceid,year,grand_prix,country,gpw.driverid,gpw.driver,db2.nationality
INTO grandprix_won_by_nonnational_of_hosting_country
FROM grand_prix_winners AS gpw
LEFT JOIN driver_bio AS db
ON gpw.country = db.nationality
AND gpw.driverid = db.driverid
LEFT JOIN driver_bio AS db2
ON gpw.driverid = db2.driverid
WHERE db.nationality IS NULL
ORDER BY year,raceid
```
v) Number of grand prix won by a non national of the host country

``` sql
SELECT COUNT(*) AS no_of_gradnprix_won_by_nonnational_of_host_country
-- INTO no_of_gradnprix_won_by_nonnational_of_host_country
-- FROM grand_prix_winners AS gpw
-- LEFT JOIN driver_bio AS db
-- ON gpw.country = db.nationality
-- AND gpw.driverid = db.driverid
-- WHERE db.nationality IS NULL
```
w) Host countries and the number of drivers of same country who won their grand prix

``` sql
SELECT gpw.country AS grandprix_host_countries,count(gpw.driver) AS no_of_nationals_winning_the_grandprix 
INTO host_country_no_of_nationals_winning
FROM grand_prix_winners AS gpw
LEFT JOIN driver_bio AS db
ON gpw.country = db.nationality
AND gpw.driverid = db.driverid
WHERE db.nationality IS NOT NULL
GROUP BY gpw.country
ORDER BY no_of_nationals_winning_the_grandprix DESC
```
X) Grand prix winners by nationality

``` sql
SELECT gpw.driverid,db.driver,nationality
INTO Grandprix_winners_by_country
FROM grand_prix_winners AS gpw
LEFT JOIN driver_bio AS db
ON gpw.driverid = db.driverid
GROUP BY gpw.driverid,db.driver,nationality
ORDER BY nationality,db.driver
```
y) Number of grand prix winners per nationality

``` sql
SELECT nationality,COUNT(driver) AS no_of_winners_per_country
INTO no_of_grandprix_winners_by_country
FROM Grandprix_winners_by_country 
GROUP BY nationality
ORDER BY no_of_winners_per_country DESC
```
z) Total points by each grand prix winner in a grandprix

``` sql
SELECT year, gpw.raceid,grand_prix,gpw.driverid,driver,constructor_name,SUM(points) AS total_points, SUM(wins) AS wins
INTO winner_point_and_wins_per_grandprix
FROM grand_prix_winners AS gpw
LEFT JOIN driver_standing AS ds
ON gpw.raceid = ds.raceid
AND gpw.driverid = ds.driverid
GROUP BY year, gpw.raceid,grand_prix,gpw.driverid,driver,constructor_name
ORDER BY year,gpw.raceid
```
a1) Total points by driver in a formula one season

``` sql
SELECT year,driverid,driver,SUM(total_points) AS total_points
INTO f1_season_driver_points
FROM winner_point_and_wins_per_grandprix
GROUP BY year,driver,driverid
ORDER BY year
```
a2) Driver with the highest points per season

``` sql
WITH max_point AS (
SELECT year, MAX(total_points) AS total_points
FROM f1_season_driver_points
GROUP BY year
ORDER BY year
)
SELECT mp.year,driverid,driver,mp.total_points
INTO driver_highest_points_per_season
FROM max_point AS mp
LEFT JOIN f1_season_driver_points AS f1p
ON mp.year = f1p.year
AND mp.total_points = f1p.total_points
ORDER BY mp.year
```
a3) Overall point tally per driver

``` sql
SELECT driverid,driver, SUM(total_points)  AS overall_won_points
INTO drivers_with_highest_won_points
FROM driver_highest_points_per_season
GROUP BY driverid,driver
ORDER BY overall_won_points DESC
```
a4) Driver with overall point tall

``` sql
SELECT driverid,driver, overall_won_points
INTO driver_with_highest_won_points
FROM drivers_with_highest_won_points
WHERE overall_won_points = (
SELECT MAX(overall_won_points)
FROM drivers_with_highest_won_points
)
```
a5) Seasons and total point

``` sql
SELECT year,SUM(total_points) AS total_points
INTO season_total_points
FROM f1_season_driver_points
GROUP BY year
ORDER BY year
```
a6) Season with most point

``` sql
SELECT * 
INTO season_with_highest_point
FROM season_total_points
WHERE total_points = (
SELECT MAX(total_points)
FROM season_total_points
)
```
a7) Number of times a driver won the fastest lap and also won the grand prix

``` sql
SELECT driverid,driver, COUNT(driver) AS number_of_times_winning_grandprix_and_fastestlap
INTO no_of_racewinner_and_also_fastestlap
FROM racewinner_and_also_fastestlap
GROUP BY driver,driverid
ORDER BY number_of_times_winning_grandprix_and_fastestlap DESC
```
[Number of times a driver won the fastest lap and also won the grand prix](https://github.com/realrayza/formula_one_analysis/blob/main/no_of_racewinner_and_also_fastestlap.csv)
![fastesttime_grandprix_win](https://github.com/user-attachments/assets/260bbf15-7c7e-4c5c-bbed-67d8176e1dde)

