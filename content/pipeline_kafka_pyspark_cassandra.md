## Overview
to set up a pipeline Kafka Pyspark Cassandra

## Steps

### 1. kafka broker

kafka 7.5

```
docker container prune -f

docker run -d --rm \
--name zookeeper \
-p 2181:2181 \
confluentinc/cp-zookeeper:7.5.0 \
bash -c "zookeeper-server-start /etc/kafka/zookeeper.properties"

docker run -d --rm \
--name kafka \
-p 9092:9092 \
--link zookeeper \
-e KAFKA_BROKER_ID=1 \
-e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
-e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
confluentinc/cp-kafka:7.5.0
```

### 2. python kafka producer

kafka-python 2.3.0

```
in projects/kafka_projects/simple_producer.py
pip install kafka-python
pip list

from kafka import KafkaProducer  
import json, time  
  
producer = KafkaProducer(  
    bootstrap_servers='localhost:9092',  
    value_serializer=lambda v: json.dumps(v).encode('utf-8')  
)  
  
while True:  
    msg = {  
        "contract_id": "123",  
        "client_id": "456",  
        "status": "updated",  
        "timestamp": "2026-03-25T10:00:00"  
    }  
    producer.send("tpc_contract", msg)  
    time.sleep(1)
    
docker exec -it kafka kafka-console-consumer   --topic tpc_contract   --bootstrap-server localhost:9092   --from-beginning
```

### 3.  Spark app

spark 3.5

```
from pyspark.sql import SparkSession  
from pyspark.sql.functions import col, from_json, to_timestamp  
from pyspark.sql.types import StructType, StringType  
  
# Spark session  
spark = SparkSession.builder \  
    .appName("KafkaToCassandraPipeline") \  
    .config(  
    "spark.jars.packages",  
    "org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0,"  
    "com.datastax.spark:spark-cassandra-connector_2.12:3.5.0") \  
    .config("spark.cassandra.connection.host", "127.0.0.1") \  
    .getOrCreate()  
  
print(spark.version)  
  
spark.sparkContext.setLogLevel("WARN")  
  
# Define schema of incoming JSON  
schema = StructType() \  
    .add("contract_id", StringType()) \  
    .add("client_id", StringType()) \  
    .add("status", StringType()) \  
    .add("timestamp", StringType())  
  
# Read stream from Kafka  
kafka_df = spark.readStream \  
    .format("kafka") \  
    .option("kafka.bootstrap.servers", "localhost:9092") \  
    .option("subscribe", "tpc_contract") \  
    .option("startingOffsets", "earliest") \  
    .load()  
  
# Convert Kafka value (binary) → JSON  
json_df = kafka_df.selectExpr("CAST(value AS STRING) as json_value")  
  
parsed_df = json_df.select(  
    from_json(col("json_value"), schema).alias("data")  
).select("data.*")  
  
# Convert timestamp to proper type  
parsed_df = parsed_df.withColumn(  
    "timestamp",  
    to_timestamp(col("timestamp"))  
)  
  
# Deduplication (stateful)  
dedup_df = parsed_df.dropDuplicates(["contract_id", "timestamp"])  
  
# Write to Cassandra  
query = dedup_df.writeStream \  
    .format("org.apache.spark.sql.cassandra") \  
    .option("keyspace", "mykeyspace") \  
    .option("table", "contract_changes") \  
    .option("checkpointLocation", "/tmp/spark_checkpoint_contracts") \  
    .outputMode("append") \  
    .start()  
  
query.awaitTermination()
```

### 4. Cassandra

cassandra 4.1

```
docker run -d --name cassandra -p 9042:9042 cassandra:4.1

docker logs cassandra

docker exec -it cassandra bash
cqlsh
OR
docker exec -it cassandra cqlsh

CREATE KEYSPACE mykeyspace
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

desc keyspaces;
desc keyspace mykeyspace;

USE mykeyspace;

CREATE TABLE contract_changes (
    contract_id TEXT,
    timestamp TIMESTAMP,
    client_id TEXT,
    status TEXT,
    PRIMARY KEY (contract_id, timestamp)
);

desc tables;
desc table contract_changes;
```

### 5. final

![[Pasted image 20260327153226.png]]