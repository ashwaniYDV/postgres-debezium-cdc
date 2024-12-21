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
