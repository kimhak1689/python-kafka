# Getting Started with Apache Kafka in Python: A Practical Guide

Apache Kafka has emerged as a cornerstone technology for building real-time data pipelines and streaming applications. In this tutorial, I'll walk you through setting up a simple yet complete Kafka environment with Python producers and consumers.

## Project Structure
```
project/
├── kafka-single-node/
│   └── docker-compose.yml
├── kafka_producer.py
└── kafka_consumer.py
```

## Step 1: Setting Up Kafka with Docker
First, we need to set up a Kafka environment. The easiest way to do this is with Docker Compose. Create a `docker-compose.yml` file in a directory called `kafka-single-node`:

```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kafdrop:
    image: obsidiandynamics/kafdrop:latest
    restart: always
    ports:
      - "9001:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka:29092
      JVM_OPTS: "-Xms32M -Xmx64M"
    depends_on:
      - kafka
```

### Start the Kafka environment:
```bash
cd kafka-single-node
docker-compose up -d
```

This will start:
- **ZooKeeper**: Required for Kafka's coordination
- **Kafka**: The message broker itself
- **Kafdrop**: A web UI for monitoring Kafka (accessible at [http://localhost:9001](http://localhost:9001))

## Step 2: Creating a Kafka Producer
Create a file named `kafka_producer.py`:

```python
import json
from kafka import KafkaProducer
import time

def create_producer():
    """Create a connection to Kafka broker"""
    producer = KafkaProducer(
        bootstrap_servers=['localhost:9092'],
        value_serializer=lambda x: json.dumps(x).encode('utf-8')
    )
    return producer

def send_order(producer, order_id):
    """Send a simple order message to Kafka"""
    order = {
        'order_id': order_id,
        'customer_id': f'customer-{order_id % 10}',
        'product': f'product-{order_id % 5}',
        'quantity': (order_id % 3) + 1,
        'total': ((order_id % 3) + 1) * 10.99,
        'timestamp': time.time()
    }
    
    producer.send('orders', key=str(order_id).encode('utf-8'), value=order)
    print(f"Sent order: {order}")

def run_producer():
    producer = create_producer()
    
    try:
        for i in range(1, 11):
            send_order(producer, i)
            time.sleep(1)
    finally:
        producer.flush()
        producer.close()
        print("Producer closed")

if __name__ == "__main__":
    run_producer()
```

## Step 3: Creating a Kafka Consumer
Create a file named `kafka_consumer.py`:

```python
import json
from kafka import KafkaConsumer

def create_consumer():
    """Create a connection to Kafka broker as a consumer"""
    consumer = KafkaConsumer(
        'orders',
        bootstrap_servers=['localhost:9092'],
        value_deserializer=lambda x: json.loads(x.decode('utf-8')),
        group_id='order-processing-group',
        auto_offset_reset='earliest'
    )
    return consumer

def process_order(order):
    """Simple function to process an order"""
    print(f"Processing order {order['order_id']} for {order['quantity']} units of {order['product']}")

def run_consumer():
    consumer = create_consumer()
    
    try:
        print("Consumer started. Waiting for messages...")
        for message in consumer:
            order = message.value
            process_order(order)
            print(f"Received from partition {message.partition}, offset {message.offset}\n---")
    except KeyboardInterrupt:
        print("Consumer stopped by user")
    finally:
        consumer.close()
        print("Consumer closed")

if __name__ == "__main__":
    run_consumer()
```

## Step 4: Installing Dependencies
Before running the code, install the required Python package:
```bash
pip install kafka-python
```

## Step 5: Running the Example
Open two terminal windows:

### Start the consumer:
```bash
python kafka_consumer.py
```
The consumer will start and wait for messages.

### Start the producer:
```bash
python kafka_producer.py
```
The producer will send 10 order messages to Kafka.

## Understanding the Key Concepts

### Producer
- Connects to Kafka via bootstrap servers
- Serializes data
- Sends messages to specific topics

### Consumer
- Subscribes to topics to receive messages
- Deserializes data
- Processes messages
- Tracks offsets

### Consumer Groups
- Multiple consumers in the same group share the workload
- Each partition is assigned to only one consumer in a group
- Different consumer groups receive all messages independently

## Real-World Benefits
- **Decoupling**: Producers and consumers operate independently
- **Durability**: Messages are stored safely in Kafka
- **Scalability**: Multiple consumers can process messages in parallel
- **Resilience**: Consumers can resume from where they left off

## Exploring With Kafdrop
Visit [http://localhost:9001](http://localhost:9001) to:
- View available topics
- Monitor consumer groups
- Browse messages

## Conclusion
You have successfully set up a Kafka environment and implemented a basic producer and consumer in Python. This foundation can be extended to build more complex real-time streaming applications.

### Next Steps:
- **Exactly-once processing**
- **Stream processing with Kafka Streams**
- **Connector APIs for integrating external systems**
- **Schema management with Avro and Schema Registry**

Happy streaming! 🚀
