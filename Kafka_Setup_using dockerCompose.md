# Setting Up Kafka with Docker

## Step 1: Create a `docker-compose.yml` File
Create and edit the `docker-compose.yml` file:

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka-broker:
    image: confluentinc/cp-kafka:latest
    container_name: kafka-broker
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_INTERNAL:PLAINTEXT
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,PLAINTEXT_INTERNAL://localhost:29092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker:9092,PLAINTEXT_INTERNAL://localhost:29092
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_MESSAGE_MAX_BYTES: 10485760
```

## Step 2: Start Kafka and Zookeeper
Run the following command to start the services:

```sh
docker-compose up -d
```

## Step 3: Verify Running Containers
Check if the containers are running:

```sh
docker ps
```

**Expected output:**

```bash
CONTAINER ID   IMAGE                              STATUS          NAMES
xxxxx          confluentinc/cp-kafka:latest       Up XX seconds   kafka-broker
xxxxx          confluentinc/cp-zookeeper:latest   Up XX seconds   zookeeper
```

## Step 4: Create a Kafka Topic
To create a new topic inside the running Kafka container, use:

```sh
docker exec -it kafka-broker kafka-topics --create \
  --bootstrap-server kafka-broker:9092 \
  --replication-factor 1 --partitions 1 \
  --topic my-topic
```

### Explanation:
- `--bootstrap-server kafka-broker:9092` → Connects to the Kafka broker.
- `--replication-factor 1` → Since it's a single-node Kafka, this must be 1.
- `--partitions 1` → Creates one partition.
- `--topic my-topic` → Topic name.

## Step 5: Verify Topic Creation
To confirm the topic was created, run:

```sh
docker exec -it kafka-broker kafka-topics --list \
  --bootstrap-server kafka-broker:9092
```

**Expected output:**

```bash
my-topic
```

## Step 6: Start a Kafka Producer
To send messages to the topic, run:

```sh
docker exec -it kafka-broker kafka-console-producer \
  --bootstrap-server kafka-broker:9092 \
  --topic my-topic
```

After running this, type messages and press Enter to send.

## Step 7: Start a Kafka Consumer
To read messages from the topic, open another terminal and run:

```sh
docker exec -it kafka-broker kafka-console-consumer \
  --bootstrap-server kafka-broker:9092 \
  --topic my-topic \
  --from-beginning
```

Now you can send messages from the producer and receive them in the consumer. 🚀
