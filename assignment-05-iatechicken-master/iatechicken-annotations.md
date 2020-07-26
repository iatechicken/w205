## opening up new docker after creading docker-compose on redis-standalone
docker-compose up -d
docker-compose ps
docker-compose logs redis

## checking if it works
ipython

## closing docker down
docker-compose down
docker-compose ps

## creating new directory for redis-cluster2
mkdir ~/w205/redis-cluster2
cd ~/w205/redis-cluster2
cp ../course-content/05-Storing-Data-II/example-1-docker-compose.yml docker-compose.yml
ls

## checking the docker-compose.yml file
vim docker-compose.yml

## starting up a new docker
docker-compose up -d
docker-compose ps
docker-compose logs redis

## connecting to mids container
docker-compose exec mids bash

## closing down docker
docker-compose down
docker-compose ps
vim docker-compose.yml

## opening new docker after making the edits to docker-compose.yml file
docker-compose up -d

## opening up jupyter notebook
docker-compose exec mids jupyter notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root

## closing down docker
docker-compose down
docker-compose ps

## getting history for submission
history > richard_ryu-history.txt
history
