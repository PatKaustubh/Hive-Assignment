# 🚗 Hive Assignment: NYC Parking Violations Analysis

## 📌 Overview
The objective of this assignment is to apply Hive concepts to a real-world dataset. The dataset consists of parking violation tickets issued in New York City during the fiscal year 2017. The analysis will help derive insights into parking violations by examining trends across time, geography, and seasonality.

## 🎯 **Objective**
- Gain familiarity with Hive for data analytics.
- Perform exploratory and aggregation analysis on NYC parking violation data.
- Optimize Hive queries for performance.

## 🛠️ **Setup Instructions**
### **Step 1: Download the Dataset**
The dataset is stored in an S3 bucket and can be downloaded using:
```bash
wget https://hive-assignment-bucket.s3.amazonaws.com/Parking_Violations_Issued_-_Fiscal_Year_2017.csv
```
Alternatively, if the S3 link is down, use:
```bash
aws s3 cp s3://hiveassignmentdatabde/Parking_Violations_Issued_-_Fiscal_Year_2017.csv s3://<your-bucket-name>
```

Download the data dictionary:
[Data Dictionary](https://data.cityofnewyork.us/City-Government/Parking-Violations-Issued-Fiscal-Year-2017/2bnn-yakx)

### **Step 2: Load Data into Hive**
1. Launch Hive and create a database:
   ```sql
   CREATE DATABASE nyc_parking;
   USE nyc_parking;
   ```
2. Create a table with fields matching the CSV schema. The delimiter is `|`:
   ```sql
   CREATE EXTERNAL TABLE parking_tickets (
       summons_number STRING,
       plate_id STRING,
       registration_state STRING,
       issue_date STRING,
       violation_code INT,
       violation_time STRING,
       street_code1 INT,
       street_code2 INT,
       street_code3 INT
   ) ROW FORMAT DELIMITED
   FIELDS TERMINATED BY '|'
   LINES TERMINATED BY '\n'
   STORED AS TEXTFILE;
   ```
3. Load the dataset into the table:
   ```sql
   LOAD DATA LOCAL INPATH '<local path>' INTO TABLE parking_tickets;
   ```

## 🔍 **Part-I: Data Exploration**
1. **Total number of tickets issued in 2017:**
   ```sql
   SELECT COUNT(*) FROM parking_tickets;
   ```
2. **Total number of states issuing tickets:**
   ```sql
   SELECT COUNT(DISTINCT registration_state) FROM parking_tickets;
   ```
3. **Tickets without address information:**
   ```sql
   SELECT COUNT(*) FROM parking_tickets
   WHERE street_code1 IS NULL OR street_code2 IS NULL OR street_code3 IS NULL;
   ```

## 📊 **Part-II: Aggregation Tasks**
### **1. Parking Violations Across Different Times of the Day**
- Convert `violation_time` to a time format and group into six equal time bins.
  ```sql
  SELECT violation_code,
         CASE
             WHEN violation_time BETWEEN '00:00' AND '03:59' THEN 'Midnight-4AM'
             WHEN violation_time BETWEEN '04:00' AND '07:59' THEN '4AM-8AM'
             WHEN violation_time BETWEEN '08:00' AND '11:59' THEN '8AM-Noon'
             WHEN violation_time BETWEEN '12:00' AND '15:59' THEN 'Noon-4PM'
             WHEN violation_time BETWEEN '16:00' AND '19:59' THEN '4PM-8PM'
             ELSE '8PM-Midnight'
         END AS time_bin,
         COUNT(*) AS count
  FROM parking_tickets
  GROUP BY violation_code, time_bin
  ORDER BY count DESC
  LIMIT 3;
  ```

### **2. Most Common Violation Codes for Top Time Bins**
  ```sql
  SELECT time_bin, violation_code, COUNT(*) AS count
  FROM (Previous Query as Subquery)
  GROUP BY time_bin, violation_code
  ORDER BY count DESC;
  ```

### **3. Seasonal Analysis of Tickets**
- Classify months into seasons and count violations per season:
  ```sql
  SELECT CASE
             WHEN MONTH(issue_date) IN (3,4,5) THEN 'Spring'
             WHEN MONTH(issue_date) IN (6,7,8) THEN 'Summer'
             WHEN MONTH(issue_date) IN (9,10,11) THEN 'Fall'
             ELSE 'Winter'
         END AS season,
         COUNT(*) AS count
  FROM parking_tickets
  GROUP BY season;
  ```

- **Most common violations per season:**
  ```sql
  SELECT season, violation_code, COUNT(*) AS count
  FROM (Previous Query as Subquery)
  GROUP BY season, violation_code
  ORDER BY season, count DESC
  LIMIT 3;
  ```

## 🚀 **Optimizations**
- Convert tables into **ORC format** for efficiency:
  ```sql
  CREATE TABLE parking_tickets_orc STORED AS ORC AS SELECT * FROM parking_tickets;
  ```
- **Partitioning** by issue date:
  ```sql
  CREATE TABLE parking_tickets_partitioned (
      summons_number STRING,
      plate_id STRING,
      registration_state STRING,
      violation_code INT,
      violation_time STRING,
      street_code1 INT,
      street_code2 INT,
      street_code3 INT
  ) PARTITIONED BY (issue_date STRING)
  STORED AS ORC;
  ```
