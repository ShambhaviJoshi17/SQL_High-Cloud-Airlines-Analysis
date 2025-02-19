# SQL_High-Cloud-Airlines-Analysis

High Clouds Case study:
![image](https://github.com/user-attachments/assets/4fc63768-1578-45c4-af47-e598b6244c80)


create database projecthighcloude;
use projecthighcloude;
show tables;
create database maindata ;
use maindata;
CREATE TABLE `maindata` (
  `%Airline ID` int DEFAULT NULL,
  `%Carrier Group ID` int DEFAULT NULL,
  `%Unique Carrier Code` text,
  `%Unique Carrier Entity Code` int DEFAULT NULL,
  `%Region Code` text,
  `%Origin Airport ID` int DEFAULT NULL,
  `%Origin Airport Sequence ID` int DEFAULT NULL,
  `%Origin Airport Market ID` int DEFAULT NULL,
  `%Origin World Area Code` int DEFAULT NULL,
  `%Destination Airport ID` int DEFAULT NULL,
  `%Destination Airport Sequence ID` int DEFAULT NULL,
  `%Destination Airport Market ID` int DEFAULT NULL,
  `%Destination World Area Code` int DEFAULT NULL,
  `%Aircraft Group ID` int DEFAULT NULL,
  `%Aircraft Type ID` int DEFAULT NULL,
  `%Aircraft Configuration ID` int DEFAULT NULL,
  `%Distance Group ID` int DEFAULT NULL,
  `%Service Class ID` text,
  `%Datasource ID` text,
  `# Departures Scheduled` int DEFAULT NULL,
  `# Departures Performed` int DEFAULT NULL,
  `# Payload` int DEFAULT NULL,
  `Distance` int DEFAULT NULL,
  `# Available Seats` int DEFAULT NULL,
  `# Transported Passengers` int DEFAULT NULL,
  `# Transported Freight` int DEFAULT NULL,
  `# Transported Mail` int DEFAULT NULL,
  `# Ramp-To-Ramp Time` int DEFAULT NULL,
  `# Air Time` int DEFAULT NULL,
  `Unique Carrier` text,
  `Carrier Code` text,
  `Carrier Name` text,
  `Origin Airport Code` text,
  `Origin City` text,
  `Origin State Code` text,
  `Origin State FIPS` int DEFAULT NULL,
  `Origin State` text,
  `Origin Country Code` text,
  `Origin Country` text,
  `Destination Airport Code` text,
  `Destination City` text,
  `Destination State Code` text,
  `Destination State FIPS` int DEFAULT NULL,
  `Destination State` text,
  `Destination Country Code` text,
  `Destination Country` text,
  `Year` int DEFAULT NULL,
  `Month (#)` int DEFAULT NULL,
  `Day` int DEFAULT NULL,
  `From - To Airport Code` text,
  `From - To Airport ID` text,
  `From - To City` text,
  `From - To State Code` text,
  `From - To State` text
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `maindata`
--

LOCK TABLES `maindata` WRITE;
/*!40000 ALTER TABLE `maindata` DISABLE KEYS */;
select count(*) from maindata;

create view	ord_date as
select
concat(year,'-',Month,'-',day) as order_date,
`# Transported Passengers`,
`# Available Seats`,
`Carrier Name`,
`From - To City`,
`%Distance Group ID`
from maindata;
desc ord_date;
 select `Month (#)`as month from maindata;
 desc maindata;
 alter table maindata rename column `Month (#)` to `Month`;
 alter table `distance groups` rename column `﻿%Distance Group ID` to distance_group_id;
  alter table `distance groups` rename column `Distance Interval` to distance_interval;

select * from ord_date limit 10;

/*KPI1*/
CREATE view kpi1 as 
select year(order_date) as year_number,
Month(order_date) as month_number,
day(order_date) as day_number,
monthname( order_date) as month_name,
concat("Q",quarter(order_date)) as quarter_number,
concat(year(order_date),'-',monthname(order_date)) as year_month_number,
weekday(order_date)as weekday_number,
dayname(order_date) as day_name,
case
when quarter(order_date)=1 then "FQ4"
when quarter(order_date)=2 then "FQ1"
when quarter(order_date)=3 then "FQ2"
when quarter(order_date)=4 then "FQ3"
end as financial_quarter,

case 
when Month(order_date)=1 then "10"
when Month(order_date)=2 then "11"
when Month(order_date)=3 then "12"
when Month(order_date)=4 then "1"
when Month(order_date)=5 then "2"
when Month(order_date)=6 then "3"
when Month(order_date)=7 then "4"
when Month(order_date)=8 then "5"
when Month(order_date)=9 then "6"
when Month(order_date)=10 then "7"
when Month(order_date)=11 then "8"
when Month(order_date)=12 then "9"
end as financial_month,
case
when weekday(order_date) in (5,6) then "weekend"
when weekday(order_date) in (0,1,2,3,4) then "weekday"
end as weekend_weekday,
`# Transported Passengers`,
`# Available Seats`,
`Carrier Name`,
`From - To City`,
`%Distance Group ID`
from ord_date;
select * from kpi1;
select count(*) from kpi1;
------------------------------------------------------------------------------------------------------------------------------------
/*2. Find the load Factor percentage on a yearly , Quarterly , Monthly basis ( Transported passengers / Available seats)*/
select year_number,sum(`# Transported Passengers`),sum(`# Available Seats`),
(sum(`# Transported Passengers`)/sum(`# Available Seats`)*100) as Load_factor
from kpi1 group by year_number;

select month_name,sum(`# Transported Passengers`),sum(`# Available Seats`),
(sum(`# Transported Passengers`)/sum(`# Available Seats`)*100) as Load_factor
from kpi1 group by month_name order by load_factor desc;

select quarter_number,sum(`# Transported Passengers`),sum(`# Available Seats`),
(sum(`# Transported Passengers`)/sum(`# Available Seats`)*100) as "Load factor"
from kpi1 group by quarter_number order by quarter_number;
-----------------------------------------------------------------------------------------------------------

/*3. Find the load Factor percentage on a Carrier Name basis ( Transported passengers / Available seats)*/
select `Carrier Name` as carrier_name ,sum(`# Transported Passengers`),sum(`# Available Seats`),
(sum(`# Transported Passengers`)/sum(`# Available Seats`)*100) as Load_factor
from kpi1 group by Carrier_Name order by load_factor desc;

---------------------------------------------------------------------------------------

/*4. Identify Top 10 Carrier Names based passengers preference */
select  `Carrier Name` as carrier_name ,sum(`# Transported Passengers`)
from kpi1 group by  carrier_name order by sum(`# Transported Passengers`)  desc limit 10;

----------------------------------------------------------------------------------

/*5. Display top Routes ( from-to City) based on Number of Flights */
select `From - To City`,count(`From - To City`) as Number_of_flight from kpi1
group by `From - To City` order by count(`From - To City`) desc limit 10;

-------------------------------------------------------------------------------

/*6. Identify the how much load factor is occupied on Weekend vs Weekdays*/
select weekend_weekday,sum(`# Transported Passengers`),sum(`# Available Seats`),
(sum(`# Transported Passengers`)/sum(`# Available Seats`)*100) as Load_factor
from kpi1 group by weekend_weekday;

---------------------------------------------------------------------------------

/*7. Identify number of flights based on Distance group*/
select `%Distance Group ID`,count(*) as number_of_flights
from maindata
group by`%Distance Group ID`
order by `%Distance Group ID` asc;


