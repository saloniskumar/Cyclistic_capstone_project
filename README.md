# Cyclistic_capstone_project
This capstone project analyzes Cyclistic, a bike-share service provider in Chicago. As a junior data analyst on the marketing analysis team, I am tasked with examining the distinct usage patterns of Cyclistic bikes between casual riders and annual members. ​The insights gathered from this analysis will inform the development of a new marketing strategy aimed at converting casual riders into annual members. Cyclistic's financial analysis reveals that annual members contribute significantly more to profitability than casual riders, leading Moreno to conclude that focusing on increasing annual memberships is crucial for the company's future growth strategy.
To achieve our objective, I must deliver insightful visual representations that will assist the team in making more informed business decisions. These visuals will highlight the key differences between annual and casual members.

For this project, I followed the 7 stages of data analysis process: Ask,Prepare,Process,Analyze,Share and Act

## 1. Ask ##
How do annual members and casual riders use Cyclistic bikes differently?

## 2. Prepare ##
The data was downloaded from https://divvy-tripdata.s3.amazonaws.com/index.html
For this analysis, I considered the data of April, May and June 2020 (Q2). The information is structured into separate CSV files, with each file representing a distinct monthly period. 
​Upon my preliminary analysis, I can confidently state that the source data meets the criteria of reliability, originality, comprehensiveness, currency, and proper citation.

I analyzed the CSV files for April, May, and June, focusing on improving the data structure for easier analysis. To achieve this, I split the "Started At" and "Ended At" columns into separate "Started Time" and "Ended Time" columns. Subsequently, I applied filters to each column to identify any blank or null values. To load them into MySQL I saved these modified CSV files in the directory C:\ProgramData\MySQL\MySQL Server 8.0\Uploads. Finally, I created a new database and table in SQL, into which I imported the refined CSV files.

```sql
-- Database Creation
create database cyclist;

-- Creating tables for each month
CREATE TABLE april_20210 (
    ride_id VARCHAR(255),
    rideable_type VARCHAR(255),
    started_at DATE,
    started_time TIME,
    ended_at DATE,
    ended_time TIME,
    start_station_name VARCHAR(255),
    start_station_id VARCHAR(255),
    end_station_name VARCHAR(255), 	
    end_station_id VARCHAR(255),
    start_lat FLOAT NULL,
    start_lng FLOAT NULL,
    end_lat FLOAT NULL,
    end_lng FLOAT NULL,
    member_casual VARCHAR(255)
);

-- Loading csv data into their respective tables 
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/data_April_2020.csv'
INTO TABLE april_2020
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(ride_id, rideable_type, @started_at,started_time, @ended_at,ended_time, start_station_name, start_station_id, end_station_name, end_station_id, @start_lat, @start_lng, @end_lat, @end_lng, member_casual)
SET
    started_at = STR_TO_DATE(@started_at, '%d-%m-%Y'),
    ended_at = STR_TO_DATE(@ended_at, '%d-%m-%Y'),
    start_lat = NULLIF(@start_lat, ''),
    start_lng = NULLIF(@start_lat, ''),
    end_lat = NULLIF(@start_lat, ''),
    end_lng = NULLIF(@start_lat, '');

-- Then I joined the 3 tables together to create Q1

CREATE TABLE IF NOT EXISTS Q1 AS ( 
	SELECT * FROM april_2020
	UNION ALL 
	SELECT * FROM may_2020
	UNION ALL 
	SELECT * FROM june_2020
    );

```
## 3. Prepare ##

​To check how SQL handles null values, I explored the CSV file to find a ride ID with null entries. I then used SQL to query and display all columns of this ride ID, allowing me to observe how SQL is storing the null values

```sql
-- This ride id had 
SELECT * from june_2021 where ride_id= '99FEC93BA843FB20';

-- By this I concluded that SQL is storing null values as blank values. I then deleted the blank values

DELETE FROM april_2021 where 
start_station_id =""
OR end_station_id = ""  
or start_station_name = ""
or end_station_name=""
oR start_lat =""
OR end_lat ="" 
 OR start_lng =""
OR end_lng ="" ;

```

