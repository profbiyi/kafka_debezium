## Debezium Quick Start Guide with Docker

This guide provides a step-by-step approach to setting up a Debezium environment using Docker.

### Prerequisites

- Docker installed on your machine. Preferably, docker desktop

### Step-by-Step Guide

#### Step 1: Start ZooKeeper

```bash
docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:0.9
```

#### step 2:  Start Kafka

```bash
docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:0.9
```

#### step 3: Start Kafka Connect

```bash
docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka debezium/connect:0.9
```


#### Step 4:  Start Kafdrop to see the sync data.

```bash
docker run -it --rm --name kafdrop -p 9000:9000 --link kafka:kafka obsidiandynamics/kafdrop:latest
```



### Using the Kafka Connect REST API 
To use the Kafka Connect REST API, you can make HTTP requests from any client that supports making HTTP requests (e.g., curl).
To use the Kafka Connect REST API, you need to know the IP address and port of the Kafka Connect service. In this example,
To interact with Kafka Connect, you can use its REST API. The following example demonstrates how to create and list connectors using curl commands


#### step 5: check if Kafka connect is working

```bash
curl -H "Accept:application/json" localhost:8083/ 
```
 
#### Step 6:  Check plugins

```bash
curl -H "Accept:application/json" http://localhost:8083/connector-plugins
```
  

#### Step 7:  Check connectors

```bash
curl -H "Accept:application/json" localhost:8083/connectors/
```
  
  
#### step 8: Create /Add a kafka connect connector. we are using this one to test. you run by copying everthing to your terminal and run it

```bash 
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" \
    localhost:8083/connectors/ -d '
    {
  "name": "mongo-connector",
  "config": {
    "connector.class": "io.debezium.connector.mongodb.MongoDbConnector",
    "tasks.max": "1",
    "mongodb.hosts": "atlas-pk8ne0-shard-0/ac-axucnil-shard-00-00.ov5lfdz.mongodb.net:27017,ac-axucnil-shard-00-01.ov5lfdz.mongodb.net:27017,ac-axucnil-shard-00-02.ov5lfdz.mongodb.net:27017",
    "mongodb.name": "dbserver1",
    "mongodb.user": "biyiguy",
    "mongodb.password": "1111",
    "mongodb.ssl.enabled": "true",
    "database.whitelist": "sample_airbnb",
    "collection.whitelist": "sample_airbnb.*",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "topic.prefix":"test_"
  }
}
'
```


#### Step 9:  Do step 7 again to check if connector is up and running

```bash
curl -H "Accept:application/json" localhost:8083/connectors/
```



#### Step 10:  Check connector status

```bash
curl -i -X GET -H "Accept:application/json" http://localhost:8083/connectors/mmongo-connector/status
```

#### Step 11: use `http://localhost:9000/` to check kafdrop.  
`in kafdrop, click on topic and see the data coming in`


##### Extras

>To delete a connector. if you want to delete a connector. Please dont do it unless you have a reason to do
>

```bash
curl -X DELETE http://localhost:8083/connectors/mongo-source-connector
```


>> Do not worry about sinking to destination for now. we need to get this part very well.


> 
Clean up. You can delete the whole container by running the code below
>
```bash
docker connect kafka zookeeper
```

After all of this, we can then redo, the whole process, this time instead using docker compose below


safe the below in a docker-compose.yml file. then run docker compose up on your system. When it is up, you can 
then do step 5 to the end, as the docker compose would have help in taking care of step 1 to 4


```bash
version: '3'

services:
  zookeeper:
    image: debezium/zookeeper:0.9
    ports:
      - "2181:2181"
      - "2888:2888"
      - "3888:3888"

  kafka:
    image: debezium/kafka:0.9
    ports:
      - "9092:9092"
    environment:
      ZOOKEEPER_CONNECT: zookeeper:2181

  connect:
    image: debezium/connect:0.9
    ports:
      - "8083:8083"
    environment:
      GROUP_ID: 1
      CONFIG_STORAGE_TOPIC: my_connect_configs
      OFFSET_STORAGE_TOPIC: my_connect_offsets
      STATUS_STORAGE_TOPIC: my_connect_statuses
      BOOTSTRAP_SERVERS: kafka:9092

  kafdrop:
    image: obsidiandynamics/kafdrop:latest
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka:9092
    depends_on:
      - kafka

```





> You can run Steps 5, 6, 7 and 9 on your browser without the curl. eg 
>>`curl -H "Accept:application/json" localhost:8083/connectors/` becomes `localhost:8083/connectors/`

if you browser complains, 
>>change `localhost:8083/connectors/` to `0.0.0.0:8083/connectors/`
>



credits: https://debezium.io/documentation/reference/0.9/tutorial.html
