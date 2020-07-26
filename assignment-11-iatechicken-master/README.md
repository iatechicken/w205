# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking: 
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze event types.
- This task will be spread out over the last four assignments (9-12).


### Setting up the directory and copying the neccesssary files over
```
mkdir ~/w205/full-stack/
cd ~/w205/full-stack
cp ~/w205/course-content/12-Querying-Data-II/docker-compose.yml .
cp ~/w205/course-content/12-Querying-Data-II/*.py .
```

### Spinning up the docker clusters with the docker-compose.yml file
```
docker-compose up -d
```

### Creating a new topic called 'events'
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

### Let's take a look at the game api script that we'll be running in Flask
```
#!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask, request

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
    event.update(request.headers)
    producer.send(topic, json.dumps(event).encode())


@app.route("/")
def default_response():
    default_event = {'event_type': 'default'}
    log_to_kafka('events', default_event)
    return "This is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "Sword Purchased!\n"

@app.route("/join_guild")
def join_guild():
    join_guild_event = {'event_type': 'joined_guild'}
    log_to_kafka('events', join_guild_event)
    return "Congratulations! You have joined a guild!\n"

@app.route("/purchase_a_knife")
def purchase_a_knife():
    purchase_knife_event = {'event_type': 'purchase_knife', 'description': 'very sharp knife'}
    log_to_kafka('events', purchase_knife_event)
    return "Knife Purchased!\n"
```
- Please note that /join_guild and /puchase_a_knife was added

### Running Flask with our game_api.py file
```
docker-compose exec mids env FLASK_APP=/w205/full-stack/game_api.py flask run --host 0.0.0.0
```

### Open a new terminal to continuously watch kafkacat
```
# omission of -e allows kafkacat to run continuously
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
```
- Now as we execute API calls from another terminal, we can watch the events being logged in real time on this terminal

### From another terminal, let's go ahead and generate multiple events with Apache Bench by curling from the browser
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 5 -H "Host: user1.comcast.com" http://localhost:5000/join_guild
docker-compose exec mids ab -n 5 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_knife
docker-compose exec mids ab -n 7 -H "Host: user1.comcast.com" http://localhost:5000/join_guild
docker-compose exec mids ab -n 3 -H "Host: user1.comcast.com" http://localhost:5000/purchase_sword
docker-compose exec mids ab -n 3 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 2 -H "Host: user1.comcast.com" http://localhost:5000/
docker-compose exec mids ab -n 7 -H "Host: user7.sktelecom.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 8 -H "Host: user8.sktelecom.co.kr" http://localhost:5000/purchase_a_sword
```
- This should generate 12 default events, 31 purchase_a_sword events, 12 join_guild events, and 5 purchase_a_knife events for multiple hosts with different domains

### With the introduction of 'purchase_a_knife' call, we've introduced a new nested schema that was different from other event's schema. Therefore, we will have to selectively filter the events that we're interested in and modify the python script accordingly. Below is the filtered_writes.py script
```
#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf

# Only filtering for "purchase_sword"
@udf('boolean')
def is_purchase(event_as_json):
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


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

    purchase_events = raw_events \
        .select(raw_events.value.cast('string').alias('raw'),
                raw_events.timestamp.cast('string')) \
        .filter(is_purchase('raw'))

    extracted_purchase_events = purchase_events \
        .rdd \
        .map(lambda r: Row(timestamp=r.timestamp, **json.loads(r.raw))) \
        .toDF()
    extracted_purchase_events.printSchema()
    extracted_purchase_events.show()

    extracted_purchase_events \
        .write \
        .mode('overwrite') \
        .parquet('/tmp/purchases')


if __name__ == "__main__":
    main()
```

### Executing the filtered_writes.py script through spark-submit
```
docker-compose exec spark spark-submit /w205/full-stack/filtered_writes.py
```

### Verify that 'purchases' parquet output exists in the hdfs
```
docker-compose exec cloudera hadoop fs -ls /tmp/purchases/
```

### Now that we have the filtered events, let's take a closer look at it in jupyter notebook.

### Starting up jupyter notebook
```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark
```

### Reading 'purchases' parquet file in Jupyter Notebook
```
purchases = spark.read.parquet('/tmp/purchases')
purchases.show()
```

### Registering the table above into a temporary table called 'purchases'
```
purchases.registerTempTable('purchases')
```

### Filtering only for host = 'user8.sktelecom.co.kr'
```
purchases_by_example2 = spark.sql("select * from purchases where host='user8.sktelecom.co.kr'")
purchases_by_example2.show()
```

### Creating a new dataframe with the filtered results for 'user8.sktelecom.co.kr'
```
newdf = purchases_by_example2.toPandas()
newdf.describe()
```

- We should see 8 events in the new dataframe. All of them should have the host name: 'user8.sktelecom.co.kr'
