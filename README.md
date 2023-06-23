# SpaceLaunch
Space_Missions Data set from https://mavenanalytics.io/data-playground?page=7

--select the data being used and have an overlook

SELECT *
 FROM `scudders-project-1.space_launch.space_missions` LIMIT 1000

 --looks like there are some nulls in time and price. time seems to correlate to mission stauts 'failure'
 --checking null values

 SELECT Company, MissionStatus, Time, Price
 FROM `scudders-project-1.space_launch.space_missions`
 WHERE Price IS NULL
 OR Time IS NULL

--Time is null where mission status is failure. (maybe change to no launch)
--Price is missing in many instances so finding avreage cost may have to be calculated for companies without missing entries.
--replace null with undisclosed for price.

UPDATE `scudders-project-1.space_launch.space_missions`
SET Price = IFNULL(Price, 'Undisclosed')
WHERE Price IS NULL;

--Not able to make the adjustment
--After more data exploration I may try to add these details allong with others to a new table with Altertable, createtable or a temptable.

--total number of missions

SELECT COUNT(Mission) as total_num_missions
FROM `scudders-project-1.space_launch.space_missions`;

--result 4630

--total number of missions by company

SELECT Count(*) as total_missions
FROM
  `scudders-project-1.space_launch.space_missions`
GROUP BY Company
ORDER BY Company;

--add new row total_missions by company ref(23)

--Most launches by decade

--Most launches by decade
--Have to figure out




--percent of successful missions

SELECT *, (Mission/MissionStatus)*100 as SuccessRate
 FROM `scudders-project-1.space_launch.space_missions`
WHERE MissionStatus = '1'

--Problem missionstatus is a string so need to run a alternate query.

SELECT
  COUNT(*) AS TotalMissions,
  COUNTIF(MissionStatus = 'Success') AS SuccessfulMissions,
  (COUNTIF(MissionStatus = 'Success') / COUNT(*)) * 100 AS SuccessRate
FROM `scudders-project-1.space_launch.space_missions`;

-- A high level of success, will check again in visulisation process.
--lets look for each company

SELECT
Company,
  COUNT(*) AS TotalMissions,
  COUNTIF(MissionStatus = 'Success') AS SuccessfulMissions,
  (COUNTIF(MissionStatus = 'Success') / COUNT(*)) * 100 AS SuccessRate
FROM `scudders-project-1.space_launch.space_missions`
GROUP BY Company
ORDER BY Company DESC;

--avg cost

SELECT 
AVG (Price) AS AvgBuildCost,
FROM `scudders-project-1.space_launch.space_missions`
GROUP BY Company
ORDER BY Company DESC;

--results include nulls which i do not want 
--Check max and min values as well. combine all into a window function and remeber to drop the group by clause.

SELECT 
Company,
AVG (Price) OVER (PARTITION BY Company) AS AvgBldCost,
MIN (Price) OVER (PARTITION BY Company) AS MinBldCost,
MAX (Price) OVER (PARTITION BY Company) AS MaxBldCost
FROM `scudders-project-1.space_launch.space_missions`
WHERE Price IS NOT NULL
ORDER BY Company DESC;

--seperate location from launch site

SELECT 
  SUBSTRING(Location, 1, STRPOS(Location, ',') - 1) AS LaunchSite,
  SUBSTRING(Location, STRPOS(Location, ',') + 2) AS LauncLocation
FROM `scudders-project-1.space_launch.space_missions`;

--Still a coma deliminating between LaunchLocation and country name.

--most missions by country

--Total Active Rockets by company

SELECT
  Company,
  COUNT(*) AS TotalActive,
  (SELECT COUNT(*) FROM `scudders-project-1.space_launch.space_missions` 
  WHERE Company = t.Company AND RocketStatus = 'Active') AS ActiveRocket
FROM
  `scudders-project-1.space_launch.space_missions` AS t
GROUP BY
  Company;
