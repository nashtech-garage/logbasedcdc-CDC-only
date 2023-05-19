# Context: only CDC

- We have a Users table (fields: user_id, first_name, last_name) in a source PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a soure PostgreSQL database.
- We have a Users table (fields: user_id, first_name, last_name) in a sink PostgreSQL database.
- We have a Wages table (fields: user_id, wage) in a sink PostgreSQL database.
- We want to synchronize the Change Data Capture (CDC) to the tables Users and Wages in a sink PostgreSQL database when the tables Users and Wages are inserted, updated, or deleted in the source PostgreSQL database.
- We expect this synchonization happenning in near-real-time.

![alt text](https://github.com/bao2902/logbasedcdc/blob/main/LogBasedCDC_3.png)

![alt text](https://github.com/bao2902/logbasedcdc/blob/main/LogBasedCDC_1.PNG)

# Environment

"docker-compose.yml" includes the following containers:
- Zookeeper
- Kafka
- PostgreSQL source database 1
- PostgreSQL source database 2
- PostgreSQL sink database

"config" folder includes the following configurations:
- PostgreSQL source config for Users and Wages tables (connect-postgres-source.properties)
- PostgreSQL sink config for Users table (connect-postgres-sink-users.properties)
- PostgreSQL sink config for Wages table (connect-postgres-sink-wages.properties)

"plugins" folder includes the following plugins:
- Debezium PostgreSQL source connector (debezium-connector-postgres)
- Debezium JDBC sink connector (confluentinc-kafka-connect-jdbc)


# Steps to start containers:

1. Build Dockerfile

sudo docker build -t nashtech/kafka .

2. Start Docker Compose

sudo docker-compose up -d


# Steps to create PosgreeSQL source 1 tables:

1. Access PostgreSQL source container 1

sudo docker exec -it  postgres-source-1 /bin/bash

psql -U postgres

2. Create Users table and insert 1 record

CREATE TABLE users(user_id INTEGER, first_name VARCHAR(200), last_name VARCHAR(200), PRIMARY KEY (user_id));

INSERT INTO users VALUES(1, 'first 1', 'last 1');


# Steps to create PosgreeSQL source 2 tables:

1. Access PostgreSQL source container 2

sudo docker exec -it  postgres-source-2 /bin/bash

psql -U postgres

2. Create Wages table and insert 1 record

CREATE TABLE wages(user_id INTEGER, wage integer, PRIMARY KEY (user_id));

INSERT INTO wages VALUES(1, '1000');



# Steps to create PosgreeSQL sink tables:

1. Access PostgreSQL sink container

sudo docker exec -it  postgres-sink /bin/bash

psql -U postgres

2. Create Users table

CREATE TABLE users(user_id INTEGER, first_name VARCHAR(200), last_name VARCHAR(200), PRIMARY KEY (user_id));

3. Create Wages table 

CREATE TABLE wages(user_id INTEGER, wage integer, PRIMARY KEY (user_id));




# Steps to create Kafka source topics:

1. Access Kafka container

sudo docker exec -t -i kafka /bin/bash

2. Start Kafka standalone cluster

cd /bin

connect-standalone /config/connect-standalone.properties /config/connect-postgres-source.properties /config/connect-postgres-source-2.properties /config/connect-postgres-sink-users.properties /config/connect-postgres-sink-wages.properties

3. Check Kafka source topics

kafka-topics --list --bootstrap-server localhost:9092



# Steps to test the context:

1. Test case of "insert"

Access PostgreSQL sink container and check if a record with "user_id = 1" is created in Users and Wages table

select * from Users;

select * from Wages;

2. Test case of "update"

Access PostgreSQL source 1 container and update the record with "user_id = 1"

UPDATE users SET first_name = 'first 1 updated' WHERE user_id = 1;

Access PostgreSQL source 2 container and update the record with "user_id = 1"

UPDATE wages SET wage = 1000 + 1 WHERE user_id = 1;

Access PostgreSQL sink container and check if the record with "user_id = 1" is updated

select * from Users;

select * from Wages;

3. Test case of "delete"

Access PostgreSQL source 1 container and delete the record with "user_id = 1"

DELETE FROM users WHERE user_id = 1;

Access PostgreSQL source 2 container and delete the record with "user_id = 1"

DELETE FROM wages WHERE user_id = 1;

Access PostgreSQL sink container and check if the records with "user_id = 1" are deleted

select * from Users;

select * from Wages;



