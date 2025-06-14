# Apache Kafka Cheatsheet

Apache Kafka is a distributed event streaming platform capable of handling trillions of events a day. It's designed for high-throughput, low-latency real-time data feeds and is widely used for building real-time streaming data pipelines and applications.

## Overview

### Key Features
- **High Throughput** - Handle millions of messages per second
- **Low Latency** - Sub-millisecond latency for real-time applications
- **Durability** - Persistent storage with configurable retention
- **Scalability** - Horizontally scalable across multiple brokers
- **Fault Tolerance** - Built-in replication and automatic failover
- **Stream Processing** - Real-time stream processing with Kafka Streams
- **Multi-Protocol Support** - Native protocols and REST APIs
- **Exactly-Once Semantics** - Guaranteed message delivery
- **Schema Evolution** - Support for evolving data schemas
- **Connect Framework** - Easy integration with external systems
- **Security** - Authentication, authorization, and encryption
- **Multi-Tenancy** - Support for multiple isolated workloads

### Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Producers     │    │   Kafka Cluster │    │   Consumers     │
│                 │    │                 │    │                 │
│ • Applications  │──► │ ┌─────────────┐ │ ──►│ • Applications  │
│ • Services      │    │ │  Broker 1   │ │    │ • Analytics     │
│ • Connectors    │    │ │  Broker 2   │ │    │ • Databases     │
│ • Streams       │    │ │  Broker 3   │ │    │ • Streams       │
└─────────────────┘    │ └─────────────┘ │    └─────────────────┘
                       │                 │
                       │ ┌─────────────┐ │
                       │ │  ZooKeeper  │ │
                       │ │   Cluster   │ │
                       │ └─────────────┘ │
                       └─────────────────┘
                                │
                       ┌─────────────────┐
                       │   Topics &      │
                       │   Partitions    │
                       │                 │
                       │ • Topic A       │
                       │   - Partition 0 │
                       │   - Partition 1 │
                       │ • Topic B       │
                       │   - Partition 0 │
                       │   - Partition 1 │
                       │   - Partition 2 │
                       └─────────────────┘
```

## Installation

### macOS Installation
```bash
# Using Homebrew
brew install kafka

# Or install with Confluent Platform
brew install confluent-platform

# Using binary download
wget https://downloads.apache.org/kafka/2.8.2/kafka_2.13-2.8.2.tgz
tar -xzf kafka_2.13-2.8.2.tgz
sudo mv kafka_2.13-2.8.2 /opt/kafka

# Add to PATH
echo 'export PATH="/opt/kafka/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Verify installation
kafka-topics.sh --version
```

### Linux Installation
```bash
# Ubuntu/Debian
# Install Java (required)
sudo apt update
sudo apt install openjdk-11-jdk

# Download and install Kafka
wget https://downloads.apache.org/kafka/2.8.2/kafka_2.13-2.8.2.tgz
tar -xzf kafka_2.13-2.8.2.tgz
sudo mv kafka_2.13-2.8.2 /opt/kafka
sudo chown -R $USER:$USER /opt/kafka

# Add to PATH
echo 'export KAFKA_HOME=/opt/kafka' >> ~/.bashrc
echo 'export PATH=$PATH:$KAFKA_HOME/bin' >> ~/.bashrc
source ~/.bashrc

# Create systemd services
# ZooKeeper service
sudo cat > /etc/systemd/system/zookeeper.service << EOF
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=forking
User=kafka
Group=kafka
Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh -daemon /opt/kafka/config/zookeeper.properties
ExecStop=/opt/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF

# Kafka service
sudo cat > /etc/systemd/system/kafka.service << EOF
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=forking
User=kafka
Group=kafka
Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ExecStart=/opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF

# Create kafka user
sudo useradd -r -s /bin/false kafka
sudo chown -R kafka:kafka /opt/kafka

# Enable and start services
sudo systemctl enable zookeeper kafka
sudo systemctl start zookeeper
sudo systemctl start kafka
```

### Docker Installation
```bash
# Single-node setup with Docker Compose
cat > docker-compose.yml << EOF
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9094:9094"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost

  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    container_name: kafka-ui
    depends_on:
      - kafka
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
EOF

docker-compose up -d

# Multi-broker cluster setup
cat > docker-compose-cluster.yml << EOF
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka1:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka1
    container_name: kafka1
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3

  kafka2:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka2
    container_name: kafka2
    depends_on:
      - zookeeper
    ports:
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:29093,PLAINTEXT_HOST://localhost:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3

  kafka3:
    image: confluentinc/cp-kafka:7.4.0
    hostname: kafka3
    container_name: kafka3
    depends_on:
      - zookeeper
    ports:
      - "9094:9094"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:29094,PLAINTEXT_HOST://localhost:9094
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
EOF

docker-compose -f docker-compose-cluster.yml up -d
```

### Kubernetes Installation
```yaml
# Using Strimzi Kafka Operator
kubectl create namespace kafka
kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

# Create Kafka cluster
cat > kafka-cluster.yaml << EOF
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.5.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: "3.5"
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: 100Gi
        deleteClaim: false
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 100Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
EOF

kubectl apply -f kafka-cluster.yaml -n kafka

# Wait for cluster to be ready
kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka
```

## Basic Commands

### Starting Kafka
```bash
# Start ZooKeeper (required for Kafka)
zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties

# Start Kafka server
kafka-server-start.sh $KAFKA_HOME/config/server.properties

# Start in background (daemon mode)
zookeeper-server-start.sh -daemon $KAFKA_HOME/config/zookeeper.properties
kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties

# Stop services
kafka-server-stop.sh
zookeeper-server-stop.sh

# Using systemd (if configured)
sudo systemctl start zookeeper
sudo systemctl start kafka
sudo systemctl stop kafka
sudo systemctl stop zookeeper
```

### Environment Variables
```bash
# Common Kafka environment variables
export KAFKA_HOME=/opt/kafka
export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
export KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35"
export KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:$KAFKA_HOME/config/log4j.properties"
export JMX_PORT=9999
```

## Topic Management

### Topic Operations
```bash
# Create topic
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partitions 3 \
  --replication-factor 1

# Create topic with specific configuration
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partitions 3 \
  --replication-factor 1 \
  --config retention.ms=86400000 \
  --config segment.ms=3600000

# List topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Describe topic
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Describe all topics
kafka-topics.sh --describe --bootstrap-server localhost:9092

# Delete topic
kafka-topics.sh --delete \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Modify topic configuration
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config retention.ms=172800000

# Add partitions to existing topic
kafka-topics.sh --alter \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partitions 6
```

### Topic Configuration
```bash
# List topic configurations
kafka-configs.sh --describe \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic

# Set topic configuration
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config 'cleanup.policy=compact,retention.ms=86400000'

# Remove topic configuration
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --delete-config retention.ms

# Common topic configurations
# retention.ms=604800000              # Retain for 7 days
# segment.ms=86400000                 # Roll segment daily
# cleanup.policy=delete               # Delete old segments
# cleanup.policy=compact              # Log compaction
# min.insync.replicas=2               # Minimum in-sync replicas
# max.message.bytes=1000000           # Max message size
# compression.type=gzip               # Compression type
```

## Producer Operations

### Console Producer
```bash
# Basic console producer
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Producer with key
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property "parse.key=true" \
  --property "key.separator=:"

# Producer with headers
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property "parse.headers=true" \
  --property "headers.delimiter=|" \
  --property "headers.separator=,"

# Producer from file
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic < messages.txt

# Producer with specific partitioner
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner
```

### Producer Performance Testing
```bash
# Performance test producer
kafka-producer-perf-test.sh \
  --topic my-topic \
  --num-records 1000000 \
  --record-size 1024 \
  --throughput 10000 \
  --producer-props bootstrap.servers=localhost:9092

# Producer with acks configuration
kafka-producer-perf-test.sh \
  --topic my-topic \
  --num-records 100000 \
  --record-size 1024 \
  --throughput -1 \
  --producer-props \
    bootstrap.servers=localhost:9092 \
    acks=all \
    retries=2147483647 \
    max.in.flight.requests.per.connection=5 \
    enable.idempotence=true
```

### Producer Configuration Examples
```bash
# High throughput producer
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --producer-property acks=1 \
  --producer-property batch.size=16384 \
  --producer-property linger.ms=10 \
  --producer-property compression.type=gzip

# Reliable producer (exactly-once)
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --producer-property acks=all \
  --producer-property retries=2147483647 \
  --producer-property max.in.flight.requests.per.connection=5 \
  --producer-property enable.idempotence=true \
  --producer-property transactional.id=my-transactional-id
```

## Consumer Operations

### Console Consumer
```bash
# Basic console consumer
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Consumer from beginning
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --from-beginning

# Consumer with group
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --group my-consumer-group

# Consumer with key and headers
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property print.key=true \
  --property print.headers=true \
  --property key.separator=":" \
  --from-beginning

# Consumer with timestamp
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property print.timestamp=true \
  --property print.partition=true \
  --property print.offset=true

# Consumer from specific partition
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partition 0 \
  --offset 100

# Consumer with max messages
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --max-messages 10
```

### Consumer Groups
```bash
# List consumer groups
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list

# Describe consumer group
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --describe

# Reset consumer group offset to earliest
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --reset-offsets \
  --to-earliest \
  --topic my-topic \
  --execute

# Reset consumer group offset to latest
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --reset-offsets \
  --to-latest \
  --topic my-topic \
  --execute

# Reset to specific offset
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --reset-offsets \
  --to-offset 1000 \
  --topic my-topic \
  --execute

# Reset by datetime
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --reset-offsets \
  --to-datetime 2023-12-01T00:00:00.000 \
  --topic my-topic \
  --execute

# Delete consumer group
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --delete
```

### Consumer Performance Testing
```bash
# Performance test consumer
kafka-consumer-perf-test.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --messages 1000000 \
  --threads 1

# Consumer with specific fetch size
kafka-consumer-perf-test.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --messages 100000 \
  --consumer-props \
    fetch.min.bytes=1024 \
    fetch.max.wait.ms=500
```

## Data Management

### Log Segments
```bash
# View log segments
ls -la /tmp/kafka-logs/my-topic-0/

# Dump log segment
kafka-dump-log.sh \
  --files /tmp/kafka-logs/my-topic-0/00000000000000000000.log \
  --print-data-log

# Dump log with offsets
kafka-dump-log.sh \
  --files /tmp/kafka-logs/my-topic-0/00000000000000000000.log \
  --print-data-log \
  --verify-index-only

# Get log start and end offsets
kafka-get-offsets.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# Get offsets for specific time
kafka-get-offsets.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --time 1640995200000
```

### Log Cleanup
```bash
# Trigger log cleanup
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config segment.ms=1000

# Force log segment roll
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config segment.bytes=1024

# Compact topic (log compaction)
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config cleanup.policy=compact
```

### Reassign Partitions
```bash
# Generate partition reassignment plan
echo '{"topics":[{"topic":"my-topic"}],"version":1}' > topics.json

kafka-reassign-partitions.sh \
  --bootstrap-server localhost:9092 \
  --topics-to-move-json-file topics.json \
  --broker-list "0,1,2" \
  --generate

# Execute partition reassignment
kafka-reassign-partitions.sh \
  --bootstrap-server localhost:9092 \
  --reassignment-json-file reassignment.json \
  --execute

# Verify partition reassignment
kafka-reassign-partitions.sh \
  --bootstrap-server localhost:9092 \
  --reassignment-json-file reassignment.json \
  --verify
```

## Cluster Management

### Broker Information
```bash
# List brokers
kafka-broker-api-versions.sh --bootstrap-server localhost:9092

# Get cluster metadata
kafka-metadata-shell.sh --snapshot /tmp/kafka-logs/__cluster_metadata-0/00000000000000000000.log

# Check broker configuration
kafka-configs.sh --describe \
  --bootstrap-server localhost:9092 \
  --entity-type brokers \
  --entity-name 0

# Modify broker configuration
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type brokers \
  --entity-name 0 \
  --add-config log.retention.hours=168
```

### Replication
```bash
# Check under-replicated partitions
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --under-replicated-partitions

# Check unavailable partitions
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --unavailable-partitions

# Preferred replica election
kafka-leader-election.sh \
  --bootstrap-server localhost:9092 \
  --election-type preferred \
  --all-topic-partitions

# Unclean leader election
kafka-leader-election.sh \
  --bootstrap-server localhost:9092 \
  --election-type unclean \
  --topic my-topic \
  --partition 0
```

### Log Directory
```bash
# Describe log directories
kafka-log-dirs.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --json

# Move log directory
echo '{
  "version": 1,
  "partitions": [
    {
      "topic": "my-topic",
      "partition": 0,
      "replicas": [0],
      "log_dirs": ["/tmp/kafka-logs-new"]
    }
  ]
}' > log-dirs.json

kafka-reassign-partitions.sh \
  --bootstrap-server localhost:9092 \
  --reassignment-json-file log-dirs.json \
  --execute
```

## Security Configuration

### SSL/TLS Configuration
```bash
# Generate keystore and truststore
keytool -keystore kafka.server.keystore.jks \
  -alias localhost \
  -validity 365 \
  -genkey \
  -keyalg RSA

# Create certificate authority
openssl req -new -x509 -keyout ca-key -out ca-cert -days 365

# Import CA into truststore
keytool -keystore kafka.server.truststore.jks \
  -alias CARoot \
  -import \
  -file ca-cert

# Server properties for SSL
cat >> server.properties << EOF
listeners=PLAINTEXT://localhost:9092,SSL://localhost:9093
security.inter.broker.protocol=SSL
ssl.keystore.location=/path/to/kafka.server.keystore.jks
ssl.keystore.password=password
ssl.key.password=password
ssl.truststore.location=/path/to/kafka.server.truststore.jks
ssl.truststore.password=password
ssl.client.auth=required
EOF
```

### SASL Configuration
```bash
# SASL/PLAIN configuration
cat > kafka_server_jaas.conf << EOF
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret"
  user_alice="alice-secret";
};
EOF

# Server properties for SASL
cat >> server.properties << EOF
listeners=SASL_PLAINTEXT://localhost:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN
EOF

# Start Kafka with JAAS
export KAFKA_OPTS="-Djava.security.auth.login.config=kafka_server_jaas.conf"
kafka-server-start.sh config/server.properties
```

### ACL (Access Control Lists)
```bash
# Add ACL for user
kafka-acls.sh \
  --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:alice \
  --operation Read \
  --topic my-topic

# Add producer ACL
kafka-acls.sh \
  --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:bob \
  --operation Write \
  --topic my-topic

# Add consumer group ACL
kafka-acls.sh \
  --bootstrap-server localhost:9092 \
  --add \
  --allow-principal User:alice \
  --operation Read \
  --group my-consumer-group

# List ACLs
kafka-acls.sh \
  --bootstrap-server localhost:9092 \
  --list

# Remove ACL
kafka-acls.sh \
  --bootstrap-server localhost:9092 \
  --remove \
  --allow-principal User:alice \
  --operation Read \
  --topic my-topic
```

## Kafka Connect

### Connect Cluster
```bash
# Start Connect in standalone mode
connect-standalone.sh \
  config/connect-standalone.properties \
  config/connect-file-source.properties

# Start Connect in distributed mode
connect-distributed.sh config/connect-distributed.properties

# Connect REST API endpoints
curl http://localhost:8083/                         # Cluster info
curl http://localhost:8083/connectors               # List connectors
curl http://localhost:8083/connector-plugins        # List plugins
```

### Connector Management
```bash
# Create file source connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "file-source",
    "config": {
      "connector.class": "FileStreamSource",
      "file": "/tmp/input.txt",
      "topic": "file-topic",
      "tasks.max": 1
    }
  }'

# Create file sink connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "file-sink",
    "config": {
      "connector.class": "FileStreamSink",
      "file": "/tmp/output.txt",
      "topics": "file-topic",
      "tasks.max": 1
    }
  }'

# Get connector status
curl http://localhost:8083/connectors/file-source/status

# Get connector config
curl http://localhost:8083/connectors/file-source/config

# Update connector config
curl -X PUT http://localhost:8083/connectors/file-source/config \
  -H "Content-Type: application/json" \
  -d '{
    "connector.class": "FileStreamSource",
    "file": "/tmp/input2.txt",
    "topic": "file-topic",
    "tasks.max": 2
  }'

# Pause/Resume connector
curl -X PUT http://localhost:8083/connectors/file-source/pause
curl -X PUT http://localhost:8083/connectors/file-source/resume

# Delete connector
curl -X DELETE http://localhost:8083/connectors/file-source
```

### JDBC Connector Example
```bash
# JDBC Source Connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "jdbc-source",
    "config": {
      "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
      "connection.url": "jdbc:mysql://localhost:3306/mydb",
      "connection.user": "user",
      "connection.password": "password",
      "table.whitelist": "users",
      "mode": "incrementing",
      "incrementing.column.name": "id",
      "topic.prefix": "mysql-",
      "tasks.max": 1
    }
  }'

# JDBC Sink Connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "jdbc-sink",
    "config": {
      "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
      "connection.url": "jdbc:postgresql://localhost:5432/mydb",
      "connection.user": "user",
      "connection.password": "password",
      "topics": "mysql-users",
      "auto.create": true,
      "tasks.max": 1
    }
  }'
```

## Kafka Streams

### Streams Application Example (Java)
```java
// WordCount Streams Application
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.Produced;

import java.util.Arrays;
import java.util.Properties;
import java.util.regex.Pattern;

public class WordCountApplication {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        
        KStream<String, String> textLines = builder.stream("text-input");
        
        KTable<String, Long> wordCounts = textLines
            .flatMapValues(textLine -> Arrays.asList(textLine.toLowerCase().split("\\W+")))
            .groupBy((key, word) -> word)
            .count();

        wordCounts.toStream().to("word-count-output", Produced.with(Serdes.String(), Serdes.Long()));

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

### Streams Operations
```java
// Filtering
KStream<String, String> filtered = stream.filter((key, value) -> value.length() > 5);

// Mapping
KStream<String, String> mapped = stream.mapValues(value -> value.toUpperCase());

// Joining streams
KStream<String, String> joined = leftStream.join(rightStream,
    (leftValue, rightValue) -> leftValue + ", " + rightValue,
    JoinWindows.of(Duration.ofMinutes(5)));

// Aggregating
KTable<String, Long> aggregated = stream
    .groupByKey()
    .aggregate(
        () -> 0L,
        (key, value, aggregate) -> aggregate + 1,
        Materialized.with(Serdes.String(), Serdes.Long())
    );

// Windowing
KTable<Windowed<String>, Long> windowed = stream
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(1)))
    .count();
```

### Stream Reset
```bash
# Reset Kafka Streams application
kafka-streams-application-reset.sh \
  --application-id wordcount-application \
  --bootstrap-servers localhost:9092 \
  --input-topics text-input \
  --intermediate-topics wordcount-application-KSTREAM-AGGREGATE-STATE-STORE-0000000003-repartition
```

## Schema Registry

### Schema Registry Setup
```bash
# Start Schema Registry (Confluent)
schema-registry-start etc/schema-registry/schema-registry.properties

# Schema Registry configuration
cat > schema-registry.properties << EOF
listeners=http://0.0.0.0:8081
kafkastore.bootstrap.servers=localhost:9092
kafkastore.topic=_schemas
debug=false
EOF
```

### Schema Management
```bash
# Register schema
curl -X POST http://localhost:8081/subjects/user-value/versions \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{
    "schema": "{\"type\":\"record\",\"name\":\"User\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"name\",\"type\":\"string\"}]}"
  }'

# List subjects
curl http://localhost:8081/subjects

# Get schema versions
curl http://localhost:8081/subjects/user-value/versions

# Get specific schema version
curl http://localhost:8081/subjects/user-value/versions/1

# Get latest schema
curl http://localhost:8081/subjects/user-value/versions/latest

# Delete schema version
curl -X DELETE http://localhost:8081/subjects/user-value/versions/1

# Delete subject
curl -X DELETE http://localhost:8081/subjects/user-value
```

### Avro Producer/Consumer
```bash
# Avro console producer
kafka-avro-console-producer \
  --bootstrap-server localhost:9092 \
  --topic user-topic \
  --property schema.registry.url=http://localhost:8081 \
  --property value.schema.id=1

# Avro console consumer
kafka-avro-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic user-topic \
  --property schema.registry.url=http://localhost:8081 \
  --from-beginning
```

## Monitoring and Observability

### JMX Metrics
```bash
# Enable JMX
export JMX_PORT=9999
kafka-server-start.sh config/server.properties

# Common JMX metrics
# kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
# kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec
# kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec
# kafka.server:type=ReplicaManager,name=LeaderCount
# kafka.server:type=ReplicaManager,name=PartitionCount
# kafka.server:type=KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent

# Query JMX using jconsole or custom tools
jconsole localhost:9999
```

### Kafka Manager/AKHQ
```bash
# Using AKHQ (Kafka GUI)
docker run -p 8080:8080 \
  -e AKHQ_CONFIGURATION='|
    akhq:
      connections:
        docker-kafka-server:
          properties:
            bootstrap.servers: "host.docker.internal:9092"
  ' \
  tchiotludo/akhq
```

### Lag Monitoring
```bash
# Monitor consumer lag
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-consumer-group \
  --describe

# Export lag metrics
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --all-groups \
  --describe \
  --state
```

### Log Analysis
```bash
# Kafka logs location
tail -f $KAFKA_HOME/logs/server.log
tail -f $KAFKA_HOME/logs/kafkaServer.out

# Log configuration
cat $KAFKA_HOME/config/log4j.properties

# Change log level dynamically
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type broker-loggers \
  --entity-name 0 \
  --add-config 'kafka.controller=DEBUG'
```

## Performance Tuning

### Broker Configuration
```bash
# High-performance broker configuration
cat >> server.properties << EOF
# Network
num.network.threads=8
num.io.threads=16
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Log
num.partitions=3
default.replication.factor=3
min.insync.replicas=2
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

# Replication
replica.fetch.max.bytes=1048576
replica.fetch.wait.max.ms=500
replica.high.watermark.checkpoint.interval.ms=5000
replica.socket.timeout.ms=30000
replica.socket.receive.buffer.bytes=65536
replica.lag.time.max.ms=30000

# Group Coordinator
group.initial.rebalance.delay.ms=3000
group.max.session.timeout.ms=1800000
group.min.session.timeout.ms=6000

# Compression
compression.type=gzip

# JVM tuning
export KAFKA_HEAP_OPTS="-Xmx6g -Xms6g"
export KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true"
EOF
```

### Producer Tuning
```bash
# High-throughput producer
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --producer-property acks=1 \
  --producer-property batch.size=65536 \
  --producer-property linger.ms=100 \
  --producer-property compression.type=gzip \
  --producer-property buffer.memory=67108864

# Low-latency producer
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --producer-property acks=1 \
  --producer-property batch.size=1 \
  --producer-property linger.ms=0 \
  --producer-property compression.type=none
```

### Consumer Tuning
```bash
# High-throughput consumer
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --consumer-property fetch.min.bytes=50000 \
  --consumer-property fetch.max.wait.ms=500 \
  --consumer-property max.partition.fetch.bytes=1048576

# Low-latency consumer
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --consumer-property fetch.min.bytes=1 \
  --consumer-property fetch.max.wait.ms=0
```

## Troubleshooting

### Common Issues
```bash
# Check if Kafka is running
netstat -tlnp | grep 9092

# Check ZooKeeper connection
zookeeper-shell.sh localhost:2181 <<< "ls /brokers/ids"

# Check topic creation issues
kafka-topics.sh --describe --bootstrap-server localhost:9092

# Check consumer lag
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --describe

# Check disk usage
df -h /tmp/kafka-logs
du -sh /tmp/kafka-logs/*

# Check open file descriptors
lsof -p $(pgrep -f kafka.Kafka) | wc -l

# Check network connections
netstat -an | grep 9092
```

### Log Analysis
```bash
# Check for errors in Kafka logs
grep ERROR $KAFKA_HOME/logs/server.log
grep "FATAL\|ERROR\|WARN" $KAFKA_HOME/logs/server.log | tail -50

# Check for specific issues
grep "OutOfOrderSequenceException" $KAFKA_HOME/logs/server.log
grep "NotLeaderForPartitionException" $KAFKA_HOME/logs/server.log
grep "LeaderNotAvailableException" $KAFKA_HOME/logs/server.log

# Check replication issues
grep "Shrinking ISR" $KAFKA_HOME/logs/server.log
grep "Partition.*is not available" $KAFKA_HOME/logs/server.log
```

### Recovery Procedures
```bash
# Recover from unclean shutdown
# 1. Stop all Kafka brokers
# 2. Clean up log.cleaner.dedupe.buffer.size if needed
# 3. Start brokers one by one

# Reset consumer group (if stuck)
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group stuck-group \
  --reset-offsets \
  --to-latest \
  --all-topics \
  --execute

# Trigger log compaction manually
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name compacted-topic \
  --add-config 'segment.ms=1000,min.cleanable.dirty.ratio=0.01'

# Recover from corrupted index
# Stop broker, delete .index and .timeindex files, restart
rm /tmp/kafka-logs/my-topic-0/*.index
rm /tmp/kafka-logs/my-topic-0/*.timeindex
```

### Performance Debugging
```bash
# Check broker performance
kafka-producer-perf-test.sh \
  --topic perf-test \
  --num-records 100000 \
  --record-size 1024 \
  --throughput -1 \
  --producer-props bootstrap.servers=localhost:9092

kafka-consumer-perf-test.sh \
  --bootstrap-server localhost:9092 \
  --topic perf-test \
  --messages 100000

# Monitor JVM performance
jstat -gc $(pgrep -f kafka.Kafka) 5s
jstack $(pgrep -f kafka.Kafka) > kafka-threads.txt

# Check file system performance
iostat -x 1
sar -u 1
```

## Backup and Migration

### Topic Backup
```bash
# Mirror topics between clusters
kafka-mirror-maker.sh \
  --consumer.config source-consumer.properties \
  --producer.config target-producer.properties \
  --whitelist ".*"

# MirrorMaker 2.0 configuration
cat > mm2.properties << EOF
clusters = source, target
source.bootstrap.servers = source-cluster:9092
target.bootstrap.servers = target-cluster:9092

source->target.enabled = true
source->target.topics = .*
EOF

connect-mirror-maker.sh mm2.properties
```

### Data Export/Import
```bash
# Export topic data
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --from-beginning \
  --property print.key=true \
  --property key.separator=":" > topic-backup.txt

# Import topic data
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property "parse.key=true" \
  --property "key.separator=:" < topic-backup.txt
```

---

*For more detailed information, visit the [official Apache Kafka documentation](https://kafka.apache.org/documentation/)*

