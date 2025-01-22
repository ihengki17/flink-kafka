# Apache Flink 101

This repository is for the **Apache Flink 101** course provided by Confluent Developer.

## What this does

The code in this repo uses Docker Compose to start up a small Flink cluster and Flink's SQL Client.

## How to get it running

First build the image and start all of the containers:

```bash
docker compose up --build -d
```

Once the containers are running,

```bash
docker compose exec -it jobmanager ./bin/sql-client.sh
```

## How to test the Flink-job

Create the table for Kafka on Flink SQL

```bash
CREATE TABLE pageviews_kafka (
  `url` STRING,
  `user_id` STRING,
  `browser` STRING,
  `ts` TIMESTAMP(3)
) WITH (
  'connector' = 'kafka',
  'topic' = 'pageviews',
  'properties.group.id' = 'demoGroup',
  'scan.startup.mode' = 'earliest-offset',
  'properties.bootstrap.servers' = 'broker:29092',
  'properties.security.protocol' = 'PLAINTEXT',
  'value.format' = 'json',
  'sink.partitioner' = 'fixed'
);
```
You could modify the topic name and other properties to connecting with your own Kafka Cluster

Create a data generator using flink faker

```bash
CREATE TABLE `pageviews` (
  `url` STRING,
  `user_id` STRING,
  `browser` STRING,
  `ts` TIMESTAMP(3)
)
WITH (
  'connector' = 'faker',
  'rows-per-second' = '10',
  'fields.url.expression' = '/#{GreekPhilosopher.name}.html',
  'fields.user_id.expression' = '#{numerify ''user_##''}',
  'fields.browser.expression' = '#{Options.option ''chrome'', ''firefox'', ''safari'')}',
  'fields.ts.expression' =  '#{date.past ''5'',''1'',''SECONDS''}'
);
```

Now let's create long-lived Flink Job or it is equivalent to the persistent query on ksqlDB
```bash
INSERT INTO pageviews_kafka SELECT * FROM pageviews;
```

Now you could check on Control Center on the topic pageviews or using the query to select from the Flink Table
```bash
SELECT * FROM pageviews_kafka;
```

You could also check the Flink UI on your environment
```bash
<machine/hostname>:8081
```

You could check the Job that are running as persistent job check the status on the Flink UI

will drop you into the Flink SQL Client, where you can interact with Flink SQL.
