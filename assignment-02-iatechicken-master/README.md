# Query Project
- In the Query Project, you will get practice with SQL while learning about Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven questions using public datasets housed in GCP. To give you experience with different ways to use those datasets, you will use the web UI (BiqQuery) and the command-line tools, and work with them in jupyter notebooks.
- We will be using the Bay Area Bike Share Trips Data (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement
- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the company running Bay Area Bikeshare. You are trying to increase ridership, and you want to offer deals through the mobile app to do so. What deals do you offer though? Currently, your company has three options: a flat price for a single one-way trip, a day pass that allows unlimited 30-minute rides for 24 hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


## Assignment 02: Querying Data with BigQuery

### What is Google Cloud?
- Read: https://cloud.google.com/docs/overview/

### Get Going

- Go to https://cloud.google.com/bigquery/
- Click on "Try it Free"
- It asks for credit card, but you get $300 free and it does not autorenew after the $300 credit is used, so go ahead (OR CHANGE THIS IF SOME SORT OF OTHER ACCESS INFO)
- Now you will see the console screen. This is where you can manage everything for GCP
- Go to the menus on the left and scroll down to BigQuery
- Now go to https://cloud.google.com/bigquery/public-data/bay-bike-share 
- Scroll down to "Go to Bay Area Bike Share Trips Dataset" (This will open a BQ working page.)


### Some initial queries
Paste your SQL query and answer the question in a sentence.

- What's the size of this dataset? (i.e., how many trips)
SELECT count(distinct trip_id) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
983,648 rows or trips

- What is the earliest start time and latest end time for a trip?
SELECT min(start_date), max(end_date) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
earliest start time was 2013-08-29 09:08:00 UTC and latest end time was 2016-08-31 23:48:00 UTC

- How many bikes are there?
SELECT count(distinct bike_number), max(bike_number) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
In the database, there are 700 unique bikes with trip records, but the highest bike_number was 878. Therefore, we can infer that some bikes were not used from August 2013 to August 2016. 

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.
- Use the SQL tutorial (https://www.w3schools.com/sql/default.asp) to help you with mechanics.

- Question 1: Which bike had the highest # of trips and how many?
  * Answer: Bike # 389 had the highest # of trips with 2872 trips.
  * SQL query: SELECT bike_number, count(distinct trip_id) as trips FROM `bigquery-public-data.san_francisco.bikeshare_trips` group by bike_number order by trips desc

- Question 2: Which month had the highest total # of trips?
  * Answer: August had the highest total # of trips overall with 95576 trips.
  * SQL query: SELECT extract(month from start_date) as month, count(trip_id) as no_trips from `bigquery-public-data.san_francisco.bikeshare_trips` group by month order by no_trips desc

- Question 3: Which day of the week had the lowest total # of trips?
  * Answer: Sunday had the lowest total # of trips with 51375 trips
  * SQL query: SELECT extract(dayofweek from start_date) as dayofweek, count(trip_id) as no_trips from `bigquery-public-data.san_francisco.bikeshare_trips` group by dayofweek order by no_trips asc



