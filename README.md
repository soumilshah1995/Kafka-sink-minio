# Kafka-sink-minio
Kafka-sink-minio


```
version: '3.7'

services:

  fast-data-dev:
    image: dougdonohoe/fast-data-dev
    ports:
      - "3181:3181"
      - "3040:3040"
      - "7081:7081"
      - "7082:7082"
      - "7083:7083"
      - "7092:7092"
      - "8081:8081"
    depends_on:
      - minio
    environment:
      - ZK_PORT=3181
      - WEB_PORT=3040
      - REGISTRY_PORT=8081
      - REST_PORT=7082
      - CONNECT_PORT=7083
      - BROKER_PORT=7092
      - ADV_HOST=127.0.0.1

  minio:
    image: minio/minio
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=minio
    networks:
      default:
        aliases:
          - warehouse.minio
    ports:
      - 9001:9001
      - 9000:9000
    command: ["server", "/data", "--console-address", ":9001"]

  mc:
    depends_on:
      - minio
    image: minio/mc
    environment:
      - AWS_ACCESS_KEY_ID=admin
      - AWS_SECRET_ACCESS_KEY=password
      - AWS_REGION=us-east-1
    entrypoint: >
      /bin/sh -c "
      until (/usr/bin/mc config host add minio http://minio:9000 admin password) do echo '...waiting...' && sleep 1; done;
      /usr/bin/mc rm -r --force minio/warehouse;
      /usr/bin/mc mb minio/warehouse;
      /usr/bin/mc policy set public minio/warehouse;
      tail -f /dev/null
      "
volumes:
  mongo_data:
```

#### 
It creates reverse SSH tunnels that forward ports 9000 and 9001 from a remote server (abc.serveo.net) to the same ports on your local machine.
```
ssh -R 9000:localhost:9000 -R 9001:localhost:9001 abc.serveo.net
```

# Sink
```
name=s3-sink-nyc_yellow_taxi_trip_data
connector.class=io.confluent.connect.s3.S3SinkConnector
flush.size=1000
schema.compatibility=NONE
topics=nyc_yellow_taxi_trip_data
tasks.max=1
timezone=UTC
format.class=io.confluent.connect.s3.format.json.JsonFormat
storage.class=io.confluent.connect.s3.storage.S3Storage
s3.bucket.name=warehouse
s3.endpoint=http://abc.serveo.net:9000
s3.ssl.enabled=false
s3.access.key=admin
s3.secret.key=password
aws.access.key.id=admin
aws.secret.access.key=password
store.url=http://abc.serveo.net:9000

================================================================================================================================

name=s3-sink-telecom_italia_data
connector.class=io.confluent.connect.s3.S3SinkConnector
flush.size=1000
schema.compatibility=NONE
topics=nyc_yellow_taxi_trip_data
tasks.max=1
timezone=UTC
format.class=io.confluent.connect.s3.format.json.JsonFormat
storage.class=io.confluent.connect.s3.storage.S3Storage
s3.bucket.name=warehouse
s3.endpoint=http://abc.serveo.net:9000
s3.ssl.enabled=false
s3.access.key=admin
s3.secret.key=password
aws.access.key.id=admin
aws.secret.access.key=password
store.url=http://abc.serveo.net:9000
partitioner.class=io.confluent.connect.storage.partitioner.TimeBasedPartitioner
path.format='year'=YYYY/'month'=MM/'day'=dd/'hour'=HH
partition.duration.ms=3600000
locale=en
```
