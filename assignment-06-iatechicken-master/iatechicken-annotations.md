
### Making a new directory (spark-with-kafka2) and copying the docker-compose.yml file from course-content
```
mkdir ~/w205/spark-with-kafka2
cd ~/w205/spark-with-kafka2
cp ../course-content/07-Sourcing-Data/docker-compose.yml .
```

### Bringing the clusters up for kafka, zookeeper, mids
```
docker-compose up -d
docker-compose logs -f kafka
```

### Creating a new topic called assessment
```
docker-compose exec kafka kafka-topics --create --topic assessment --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
docker-compose exex kafka kafka-topics --describe --topic assessment --zookeeper zookeeper:32181
```

### Curling data for assignment 6
```
curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp
```

### Reviewing the data
```
docker-compose exec mids bash -c "cat /w205/spark-with-kafka2/assessment-attempts-20180128-121051-nested.json | jq '.'"
```

### Publishing the data to assessment topic with kafkacat
```
docker-compose exec mids bash -c "cat /w205/spark-with-kafka2/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessment && echo 'Produced many messages.'"
```

### Using spark to by running the spark container to read data from kafka
```
docker-compose exec spark pyspark
```

### Taking down the cluster
```
docker-compose down
```
