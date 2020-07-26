
## Setting up Flask with Kafka
```
mkdir flask-with-kafka2
cd ~/w205/flask-with-kafka2
cp ../course-content/09-Ingesting-Data/docker-compose.yml .
cp ../course-content/09-Ingesting-Data/*.py .
```

### Spinning up docker in the "flask-with-kafka2" directory and creating a new topic called "jobs" for event logging
```
docker-compose up -d
docker-compose exec kafka kafka-topics --create --topic jobs --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

### Let's take a quick look at the APIs available for this mobile game
```
vim game_api.py
```
```
#!/usr/bin/env python
from kafka import KafkaProducer
from flask import Flask
app = Flask(__name__)
event_logger = KafkaProducer(bootstrap_servers='kafka:29092')
events_topic = 'events'

@app.route("/")
def default_response():
    event_logger.send(events_topic, 'default'.encode())
    return "This is the default response!"

@app.route("/purchase_a_sword")
def purchase_sword():
    # business logic to purchase sword
    # log event to kafka
    event_logger.send(events_topic, 'purchased_sword'.encode())
    return "Sword Purchased!"
```

### Spinning up the FLASK from mids docker container
```
docker-compose exec mids env FLASK_APP=/w205/flask-with-kafka2/game_api.py flask run
```

## Now, we will open up a new terminal and pretend that we're the user of this mobile game for testing purposes.

### Executing various api calls for testing
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
```

### Using kafkacat to consume events from the "jobs" topic that we've set up for event logging
```
docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t jobs -o beginning -e"
```
