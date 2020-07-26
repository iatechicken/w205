### Setting up new directory for the test data
```
mkdir ~/w205/spark-with-kafka-and-hdfs2
cd ~/w205/spark-with-kafka-and-hdfs2
cp ~/w205/course-content/08-Querying-Data/docker-compose.yml .
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp
```

### Spinning up dockers
```
docker-compose up -d
docker-compose logs -f kafka
```

### Creating a new kafka topic
```
docker-compose exec kafka kafka-topics --create --topic reviews --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

### Using kafkacat to produce messages for topic reviews
```
docker-compose exec mids bash -c "cat /w205/spark-with-kafka-and-hdfs2/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t reviews"
```

### Spinning up pyspark
```
docker-compose exec spark pyspark
```

### Saving the raw exam data in dataframe and caching it
```
raw_reviews = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe", "reviews").option("startingOffsets", "earliest").option("endingOffsets", "latest").load()
raw_reviews.cache()
```

### Casting the raw data as string into new variable and taking a quick look at it
```
reviews = raw_rewviews.select(raw_reviews.value.cast('string'))
reviews.show()
```

### Let's do some formatting work before landing it in hdfs
```
import sys
sys.stdout = open(sys.stdout.fileno(), mode='w', encoding='utf8', buffering=1)
import json
### taking a quick peek here ###
reviews.rdd.map(lambda x: json.loads(x.value)).toDF().show()
extracted_reviews = reviews.rdd.map(lambda x: json.loads(x.value)).toDF()
```

### Landing the formatted data in hdfs
```
extracted_reviews.write.parquet("/tmp/extracted_reviews")
```
