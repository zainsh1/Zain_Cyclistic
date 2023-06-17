# Zain_Shareef Cyclistic Data Analysis with MS SQL Server & Tableau

![image](https://github.com/zainsh1/Zain_Cyclistic/assets/131926841/63bd0e39-95d1-4bf3-b9c7-2829cad397b2)

# Zain_Cyclistic Data Analyst with MS SQL Server & Tableau

Background
For this case study, I am working as a junior data analyst in the marketing analyst team at Cyclistic, a fictional bike-share company based in Chicago. Since 2016, the company has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other stations within the network at any time.
Cyclistic has three pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchased single-ride or full-day passes are considered casual riders. And customers who purchased annual memberships are considered Cyclistic members.
In order to increase revenue for the company, the company has to come up with marketing strategies aimed at converting casual riders to annual members. I am tasked with analyzing bike usage data to understand the difference between casual riders and annual members in order to help create a data-driven marketing strategy.

# Data Prep

The original dataset was imported into Microsoft SQL Server with 5,667,717 rows of data when all 12 months of data have been merged. I also added a few new columns in order to help with the analysis.

These are: Date, Start_time, End_time, Trip_duration & Day_of_week.The date column is the date when the trip took place. 
The Trip_duration_minute column is the trip duration calculated in minutes.
The Start_time column is the time when the trip starts. 
The End_time column is when the trip ends
The Day_of_week column is the day of the week when the trip took place. 
For a practical analysis, I removed all trips that lasted < 5 minutes for more precise results and remove any potential errors.

Firstly, i created a new table called 'Bikedata' after uploading all 12 months of data. I then proceeded to use the 'Union All' function to combine them into one big table for easier analysis.

This was possible because all tables of data had the same number of fields in the result set.

Once all merged, i ran some basic queries to question the data and make sure everything came across.

ALL SQL code for process/cleaning available below

Phase 3 – Process

INSERT INTO [dbo].[Bikedata]
SELECT *
FROM [dbo].[202104-divvy-tripdata]
UNION ALL
SELECT * 
FROM [dbo].[202105-divvy-tripdata]
UNION ALL
SELECT *
FROM [dbo].[Bikedata]

Continuing upto [dbo].[202203-divvy-tripdata]

-- Get count of records
SELECT COUNT(*)
FROM [dbo].[Bikedata]

-- Get count of records where ended_at is earlier than started_at
SELECT COUNT(trip_duration)
FROM [dbo].[Bikedata]
WHERE ended_at < started_at;

-- Get count of records where ride_id is duplicate
SELECT COUNT(ride_id) ride_id_count
FROM [dbo].[Bikedata]
GROUP BY ride_id HAVING COUNT(ride_id) > 1;

-- Get count of records with either null start_station_name or end_station_name
SELECT COUNT(*) AS records_count
FROM [dbo].[Bikedata]
WHERE start_station_name IS NULL OR end_station_name IS NULL;

-- Check for spaces before and after string values
SELECT rideable_type,
start_station_name,
end_station_name,
member_casual
FROM [dbo].[Bikedata]
WHERE rideable_type LIKE ' %' OR rideable_type LIKE '% '
OR start_station_name LIKE ' %' OR rideable_type LIKE '% '
OR end_station_name LIKE ' %' OR rideable_type LIKE '% ';

-- Check for misspellings in rideable_type, member_casual
SELECT rideable_type,
member_casual
FROM [dbo].[Bikedata]
GROUP BY rideable_type, member_casual;

-- Get count of records with test name in either null start_station_name or end_station_name
SELECT COUNT(*) AS records_count
FROM [dbo].[Bikedata]
WHERE start_station_name LIKE ('%test%') OR end_station_name LIKE ('%test%');

SELECT COUNT(*) AS records_count
FROM [dbo].[Bikedata]
WHERE start_station_name LIKE ('%TEST%') OR end_station_name LIKE ('%TEST%');

SELECT COUNT(*) AS records_count
FROM [dbo].[Bikedata]
WHERE start_station_name LIKE ('%Test%') OR end_station_name LIKE ('%Test%');

-- Delete records where ended_at is earlier than started_at
DELETE FROM [dbo].[Bikedata]
WHERE ended_at < started_at;

-- Delete records with duplicate ride_id
DELETE FROM [dbo].[Bikedata]
WHERE ride_id IN (SELECT ride_id
FROM [dbo].[Bikedata]
GROUP BY ride_id HAVING COUNT(ride_id) > 1);

-- Delete records with either null start_station_name or end_station_name
DELETE FROM [dbo].[Bikedata]
WHERE start_station_name IS NULL OR end_station_name IS NULL;

-- Delete records with - 'TEST' or 'test' in either start_station_name or end_station_name
DELETE FROM [dbo].[Bikedata]
WHERE start_station_name LIKE ('%test%') OR end_station_name LIKE ('%test%')
OR start_station_name LIKE ('%TEST%') OR end_station_name LIKE ('%TEST%');

Phase 4 - Analyse

-- Member type percentage distribution
SELECT member_casual, COUNT(member_casual) AS member_casual_count
FROM [dbo].[Bikedata]
GROUP BY member_casual;

-- Type of preferred bike per member type
SELECT member_casual,
rideable_type,
COUNT(rideable_type) AS rideable_type_count
FROM [dbo].[Bikedata]
GROUP BY member_casual, rideable_type;

-- Number of rides by day of the week per member type
SELECT member_casual, day_of_week, COUNT(ride_id) AS rides_count
FROM [dbo].[Bikedata]
GROUP BY member_casual, day_of_week
ORDER BY member_casual;


Dashboard at https://public.tableau.com/views/Zain_ShareefCyclisticAnalysis/Zain_ShareefCyclisticAnalysis?:language=en-US&:display_count=n&:origin=viz_share_link

Conclusions:

  1) As we can see from the dashboard, there are slightly more members than casuals 52% vs 48% respectively, however we can see that on average and total trip duration,
  casuals spend almost double the amount of time on average per session (15.36 mins vs 30.09 mins).
  
  2) Casual riders use Cyclistic bikes more during the weekend versus members who use them more during the weekdays, this is most probably because members utilize the bikes to ride and from work.
  We can also see that for both categories, most active usages are at about 5pm. Also the spike early in the morning 7-8am seems to suggest members use it for work purposes.
  
  3) Stations near or in tourist areas are more likely to be used by casual riders than members for a more leisure use. Also we can see that most active days for Casual riders are from Friday to Sunday.
  Mostly peaking on Saturday & Sunday, where as for members it's relatively consistent throughout the week, actually Sunday is also the least popular day for members to ride out.
  
How could we use these trends to help Cyclistic to create a marketing strategy to increase revenue?

  1) Partner with the city’s department of tourism/businesses to create promotion offers for Cyclistic members to encourage more cyclers.
  2) Create advertisements in popular stations and stations near tourist areas, where it's proven to attract the most amount of people interested in cycling.
  3) Create a members-only reward program to incentivize casual riders to sign up as a member, especially targetted to regular casual members.
  4) Create a membership plan for the weekends only where the prices are cheaper. Create a cost analysis for semi-regular customers so as to benefit them with an active membership.
