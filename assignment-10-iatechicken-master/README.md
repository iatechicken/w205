# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking: 
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze event types.
- This task will be spread out over the last four assignments (9-12).



## Copying the necessary files over
```
mkdir spark-from-files
cp ../w205/course-content/11-Storing-Data-III/*.py .
cp ../course-content/11-Storing-Data-III/docker-compose.yml .
```

## Spinning up the new Docker instance
```
docker-compose up -d
```

## Checking logs
```
docker-compose logs -f cloudera
```

## First, let's spin up Hadoop and create a new topic called events
```
docker-compose exec cloudera hadoop fs -ls /tmp/
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

## Now, let's start our Flask app
```
docker-compose exec mids env FLASK_APP=/w205/spark-from-files/game_api.py flask run --host 0.0.0.0
```

## From another terminal, we are testing our API with the following curls:
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/join_guild
docker-compose exec mids curl http://localhost:5000/join_guild
```

## Above curls should return
```
127.0.0.1 - - [23/Jul/2019 00:13:02] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [23/Jul/2019 00:13:55] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [23/Jul/2019 00:13:58] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [23/Jul/2019 00:14:14] "GET /join_guild HTTP/1.1" 200 -
127.0.0.1 - - [23/Jul/2019 00:14:18] "GET /join_guild HTTP/1.1" 200 -
```
200 is a sign that API calls wen through successfully. Looks like our test is successful. 

## Let's go ahead and read our API calls from Flask to Kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```

## Let's go ahead and extract these events from Kafka and write them to hdfs, using a simple python script (extract_events.py)
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""

import json
from pyspark.sql import SparkSession


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .getOrCreate()

    raw_events = spark \
        .read \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .option("startingOffsets", "earliest") \
        .option("endingOffsets", "latest") \
        .load()

    events = raw_events.select(raw_events.value.cast('string'))
    extracted_events = events.rdd.map(lambda x: json.loads(x.value)).toDF()

    extracted_events \
        .write \
        .parquet("/tmp/extracted_events")


if __name__ == "__main__":
    main()
```

```
docker-compose exec spark spark-submit /w205/spark-from-files/extract_events.py
```

## After the script ends, let's go ahead and check out our results in Hadoop
```
docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events/
```
Successfully extracted files should exist in .parquet format. 
