# template-activity-03


# Query Project

- In the Query Project, you will get practice with SQL while learning about
  Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven
  questions using public datasets housed in GCP. To give you experience with
  different ways to use those datasets, you will use the web UI (BiqQuery) and
  the command-line tools, and work with them in jupyter notebooks.

- We will be using the Bay Area Bike Share Trips Data
  (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the
  company running Bay Area Bikeshare. You are trying to increase ridership, and
  you want to offer deals through the mobile app to do so. What deals do you
  offer though? Currently, your company has three options: a flat price for a
  single one-way trip, a day pass that allows unlimited 30-minute rides for 24
  hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


## Assignment 03 - Querying data from the BigQuery CLI - set up 

### What is Google Cloud SDK?
- Read: https://cloud.google.com/sdk/docs/overview

- If you want to go further, https://cloud.google.com/sdk/docs/concepts has
  lots of good stuff.

### Get Going

- Install Google Cloud SDK: https://cloud.google.com/sdk/docs/

- Try BQ from the command line:

  * General query structure

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

### Queries

1. Rerun last week's queries using bq command line tool (Paste your bq
   queries):

- What's the size of this dataset? (i.e., how many trips)
```
bq query --use_legacy_sql=false SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
983648 trips

- What is the earliest start time and latest end time for a trip?
```
bq query --use_legacy_sql=false SELECT min(start_date),max(end_date) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
earliest start time = 2013/08/29 09:88:00
latest end time = 2016/08/31 23:48:00

- How many bikes are there?
```
bq query --use_legacy_sql=false SELECT count(distinct bike_number) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
700 bikes

2. New Query (Paste your SQL query and answer the question in a sentence):
###AM
```
bq query --use_legacy_sql=false Select count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips` WHERE EXTRACT(HOUR from end_date) BETWEEN 0 AND 11;
```

###PM
```
bq query --use_legacy_sql=false Select count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips` WHERE EXTRACT(HOUR from end_date) BETWEEN 12 AND 23;
```

- How many trips are in the morning vs in the afternoon?
Morning: 400,144 trips
Afternoon: 583,504 trips

### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: Which membership group has the least # of members?

- Question 2: What are the 5 most popular trips that you would call "commuter trips"?

- Question 3: What is the average trip duration?

- ...

- Question n: What is the average trip duration by subscriber type?

### Answers

Answer at least 4 of the questions you identified above You can use either
BigQuery or the bq command line tool.  Paste your questions, queries and
answers below.

- Question 1: Which membership group has the least # of members?
  * Answer: Realized that we don't have the right data to know which customers are daily or single use, therefore went with an attempt to break down all trips by subscriber_type and results are as follows. Among the 983,648 trips in the database:
  Customers: 136,809 (13.91%) trips
  Subscribers: 846,839  (86.09%) trips
  * SQL query:
  ```
  bq query --use_legacy_sql=false         
  SELECT subscriber_type, count(*)        
  FROM `bigquery-public-data.san_francisco.bikeshare_trips` group by subscriber_type
  ```

- Question 2: What are the 5 most popular trips that you would call "commuter trips"?
  * Answer: Top 5 most popular commuter trips are:
  #1. Harry Bridges Plaza (Ferry Building) to Embarcadero at Sansome (9150 trips)
  #2. San Francisco Caltrain 2 (330 Townsend) to Townsend at 7th (8505 trips)
  #3. 2nd at Townsend to Harry Bridges Plaza (Ferry Building) (7620 trips)
  #4. Harry Bridges Plaza (Ferry Building) to 2nd at Townsend (6888 trips)
  #5. Embarcadero at Sansome to Steuart at Market (6874 trips)
  * SQL query:
```
bq query --use_legacy_sql=false 
SELECT CONCAT(b.start_station_name, ',', b.end_station_name) as tripss, count(trip_id) as count_a 
FROM `bigquery-public-data.san_francisco.bikeshare_trips` as b 
where b.start_station_name != b.end_station_name 
group by tripss 
order by count_a desc
```

- Question 3: What is the average trip duration?
  * Answer: 1018.93 seconds per trip
  * SQL query:
```
bq query --use_legacy_sql=false 
SELECT AVG(duration_sec) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips` limit 10
```
- ...

- Question n: What is the average trip duration by subscriber type?
  * Answer: For Customers, 3718.79 seconds per trip
  * SQL query: For Subscribers, 582.76 seconds per trip
```
bq query --use_legacy_sql=false 
SELECT subscriber_type, AVG(duration_sec) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips` 
group by subscriber_type
```
