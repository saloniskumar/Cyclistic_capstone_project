# Cyclistic project using MySQL and PowerBI
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
## 3. Process ##

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
To identify duplicate ride IDs, I compared the count of distinct ride IDs with the total count of all ride IDs. Upon discovering a discrepancy between these values, I conducted a closer inspection of the ride IDs and observed that many contained the notation "E+" within them. To determine whether this notation was a result of SQL conversion or if it existed in the original data, I examined the CSV files.

```sql
select count(*) from Q1;
select count(distinct(ride_id)) from Q1;
```
 ​My investigation confirmed that some ride IDs indeed contained "E+" in their original form. Consequently, I deemed these ride IDs as invalid and removed them from the relevant tables in the database.
```sql
delete from Q1 where ride_id like '%+%';
```

## 4. Analyze ##

**SQL:**

To do more in depth analysis I created these columns-

```sql
-- Adding week of the month:
ALTER TABLE Q1 ADD COLUMN week_no INT;
UPDATE Q1
SET week_no = CEIL(DAY(started_at) / 7);

-- Adding day of the week-
ALTER TABLE Q1 ADD COLUMN day_name VARCHAR(20);
UPDATE Q1
SET day_name = DATE_FORMAT(started_at, '%W');

-- Adding name of the month-
ALTER TABLE Q1 ADD COLUMN month_name VARCHAR(20);
UPDATE Q1
SET month_name = MONTHNAME(started_at);

-- Adding duration of rides-
ALTER TABLE Q1
ADD COLUMN duration TIME;
alter table Q1 drop duration;
UPDATE Q1
SET duration = TIMEDIFF(ended_time, started_time);
```
I then observed that some rides had negative duration and deleted them from my sql
```sql
select * from Q1 where duration like '-%';
```

While attempting to compare member and casual rides, I encountered difficulty segregating them using a WHERE query. To address this issue, I proceeded to troubleshoot the problem.
```sql
select * from Q1 where member_casual='casual';
-- By doing the above, I got blanks in my output so I checked length of the member_casual column to see if any hidden character or blank space is present

SELECT DISTINCT member_casual, LENGTH(member_casual) 
FROM Q1;
-- On checking the length I observed that the column stored 7 characters instead of 6 characters. 

-- Next, I examined the column for any hidden hexadecimal values.

SELECT DISTINCT member_casual, HEX(member_casual)
FROM Q1;

-- I got 0D at the end that represents \r so I removed it using replace

UPDATE Q1 
SET member_casual = REPLACE(member_casual, CHAR(13), '');

-- I checked the length again to make sure it is correct now. As the output was 6 characters I can continue with my analysis

SELECT DISTINCT member_casual, LENGTH(member_casual) 
FROM Q1;

```

**Note:** While doing the below analaysis,sometimes MYSQL showed me a time out error. To fix this I ran these queries:

```sql
SET GLOBAL net_read_timeout = 6000;
SET GLOBAL net_write_timeout = 6000;
Or on your keyboard press windows + R,type services.msc , select mysql80 and press restart
```

**Note:** After executing several queries, I received an output indicating that SQL safe updates were enabled.Hence I ran this query: 
```sql
SET SQL_SAFE_UPDATES = 0;
```


Conducting initial data analysis to extract valuable insights:

 casual vs member riders frequency of rides throughout the week 
```sql
select member_casual,day_name,count(ride_id) from Q1 group by day_name,member_casual order by count(ride_id) desc;
```
![image](https://github.com/user-attachments/assets/93f7a205-5c17-442f-982b-d5804ba77f74)


Casual vs member riders month on month growth
```sql
select member_casual,day_name,count(ride_id) from Q1 group by day_name,member_casual order by count(ride_id) desc;
```
![image](https://github.com/user-attachments/assets/479006f0-15e6-4aef-affd-f433a754fc3d)

Start station popopularity among casual riders
```sql
select member_casual,count(member_casual),start_station_name from Q1 where member_casual='casual' group by start_station_name order by count(member_casual) desc;
```
![image](https://github.com/user-attachments/assets/9f9ce243-777e-47fe-a467-5c45db158cb6)

End station popopularity among casual riders

select member_casual,count(member_casual),end_station_name from Q1 where member_casual='casual' group by end_station_name order by count(member_casual) desc;

![image](https://github.com/user-attachments/assets/b037eb07-8bac-4864-8b54-e696b8983443)


**PowerBI:**

Casual riders visualizations for Q1:

![image](https://github.com/user-attachments/assets/ff577db0-97e9-48d9-b18f-f6bb462d8d39)

Member riders visualizations for Q1:

![image](https://github.com/user-attachments/assets/15de7742-7d8b-48ef-acac-3fef09b45e80)

All types of riders visualizations for Q1:

![image](https://github.com/user-attachments/assets/e3e4c03b-b5fb-483b-be00-40c07304a072)


## 5. Share ##

Summary of all the insights I gathered:

**1. Weeks:** In the last two weeks of the quarter (weeks 4 and 5) see a surge in casual riders.

**2. Riding Habits:** Casual riders primarily choose to ride during evenings (4pm - 8pm), suggesting a leisure-oriented usage pattern.

**3. Popular Routes:** The most popular routes for casual riders are characterized by riders taking trips with the same origin and destination station hence concluding that the casual riders are using their cycles for exercise,sightseeing,or running for errands.

**4. Weekends:** Casual riders, likely motivated by leisure, are most active on weekends.

**5. Location:** Stations closest to the Chicago River are the most popular among casual riders.

**6. Duration:** The most common trip duration for casual riders falls between 10-15 minutes.

**7. Growth:** The number of casual riders is growing at a faster rate compared to membership riders.


## 6. Act ##
My recommendations based on the insights I gathered are:

1. Provide discounted membership rates for sign-ups during the popular riding times (4 pm - 8 pm)
2. Reserve a set number of member only cycles at popular stations 
3. Set discounts on rides under duration of 15 minutes to only members
4. Set up promotional booths at popular stations to engage with casual riders and explain membership benefits
5. Introduce flexible membership plans, such as weekend-only memberships
6. Display membership ads or hold cycling events during 4-8 pm on weekends in week 4 and 5 of each month






