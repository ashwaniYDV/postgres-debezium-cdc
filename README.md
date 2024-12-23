### Steps

Set up docker containers
```
docker-compose up -d
```

Setup DB
```
docker exec -it postgres-debezium-cdc-postgres-1 psql -U admin -d exampledb

CREATE TABLE students (id integer primary key, name varchar);

ALTER TABLE public.students REPLICA IDENTITY FULL;
```

Create the connector
```
curl -X POST -H "Content-Type: application/json" --data @./debezium-cdc-connector.json localhost:8083/connectors
```

Tails the messages from kafka table
```
docker run --tty --network postgres-debezium-cdc_default confluentinc/cp-kafkacat kafkacat -b kafka:9092 \
-C -s key=s -s value=avro -r http://schema-registry:8081 -t postgres.public.students

Note: If this command gives error = "Broker: Leader not available", then run this command again
```

Create some entries in the table
```
INSERT INTO STUDENTS VALUES (1, 'ashwani');
```


### Notes
* `wal_level` = `logical` is required for debezium cdc to work. Default wal_level is `replica`.
* Since for postgres we are using `debezium/postgres:13` image, so we do not have to update the wal_level from replica to logical

### wal_level
```
SHOW wal_level;

The output will display the current WAL level, which could be one of the following:
a) minimal: Generates the least amount of WAL data but does not support replication or point-in-time recovery.
b) replica: Supports streaming replication and point-in-time recovery.
c) logical: Supports logical replication.
```

### Resources
* https://youtu.be/YZRHqRznO-o?si=LmaaCXaY-G6iYRNz
