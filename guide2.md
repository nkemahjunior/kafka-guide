# Zeco-Eats Kafka Cluster Configuration

**Final Setup Notes for `apache/kafka:3.9.0` in KRaft Mode**  
*Created by Grok 3, xAI*  
*March 09, 2025*

## Welcome Back!

Hey! This is your guide to the Kafka cluster powering `zeco-eats`—your food ordering app. We’ve built this for production (AWS-ready!), and I’ll explain your exact config, why we went with **isolated mode** instead of combined, and what each setting does. Everything’s here in Markdown—copy this into a `.md` file, and you’ll pick it up fast months later. Let’s go!

## Your Configuration

### The `docker-compose.yml` You Provided

Here’s the exact setup you gave me—6 nodes, isolated roles, tailored for `zeco-eats`.

```yaml
networks:
  zeco-eats-network:
    driver: bridge

services:
  kafka-controller-1:
    networks:
      - zeco-eats-network
    image: apache/kafka:3.9.0
    container_name: kafka-controller-1
    environment:
      KAFKA_NODE_ID: 1
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      KAFKA_PROCESS_ROLES: controller
      KAFKA_LISTENERS: CONTROLLER://:9093
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_LOG_DIRS: /var/lib/kafka/data
    volumes:
      - kafka-controller-1-data:/var/lib/kafka/data

  kafka-controller-2:
    networks:
      - zeco-eats-network
    image: apache/kafka:3.9.0
    container_name: kafka-controller-2
    environment:
      KAFKA_NODE_ID: 2
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      KAFKA_PROCESS_ROLES: controller
      KAFKA_LISTENERS: CONTROLLER://:9093
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_LOG_DIRS: /var/lib/kafka/data
    volumes:
      - kafka-controller-2-data:/var/lib/kafka/data

  kafka-controller-3:
    networks:
      - zeco-eats-network
    image: apache/kafka:3.9.0
    container_name: kafka-controller-3
    environment:
      KAFKA_NODE_ID: 3
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      KAFKA_PROCESS_ROLES: controller
      KAFKA_LISTENERS: CONTROLLER://:9093
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_LOG_DIRS: /var/lib/kafka/data
    volumes:
      - kafka-controller-3-data:/var/lib/kafka/data

  kafka-broker-1:
    image: apache/kafka:3.9.0
    container_name: kafka-broker-1
    networks:
      - zeco-eats-network
    ports:
      - 29092:9092
    environment:
      KAFKA_NODE_ID: 4
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      KAFKA_PROCESS_ROLES: broker
      KAFKA_LISTENERS: 'PLAINTEXT://:19092,PLAINTEXT_HOST://:9092'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka-broker-1:19092,PLAINTEXT_HOST://localhost:29092'
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_LOG_DIRS: /var/lib/kafka/data
    volumes:
      - kafka-broker-1-data:/var/lib/kafka/data
    depends_on:
      - kafka-controller-1
      - kafka-controller-2
      - kafka-controller-3

  kafka-broker-2:
    image: apache/kafka:3.9.0
    container_name: kafka-broker-2
    networks:
      - zeco-eats-network
    ports:
      - 29093:9092
    environment:
      KAFKA_NODE_ID: 5
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      KAFKA_PROCESS_ROLES: broker
      KAFKA_LISTENERS: 'PLAINTEXT://:19092,PLAINTEXT_HOST://:9092'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka-broker-2:19092,PLAINTEXT_HOST://localhost:29093'
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_LOG_DIRS: /var/lib/kafka/data
    volumes:
      - kafka-broker-2-data:/var/lib/kafka/data
    depends_on:
      - kafka-controller-1
      - kafka-controller-2
      - kafka-controller-3

  kafka-broker-3:
    image: apache/kafka:3.9.0
    container_name: kafka-broker-3
    networks:
      - zeco-eats-network
    ports:
      - 29094:9092
    environment:
      KAFKA_NODE_ID: 6
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      KAFKA_PROCESS_ROLES: broker
      KAFKA_LISTENERS: 'PLAINTEXT://:19092,PLAINTEXT_HOST://:9092'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka-broker-3:19092,PLAINTEXT_HOST://localhost:29094'
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
      KAFKA_LOG_DIRS: /var/lib/kafka/data
    volumes:
      - kafka-broker-3-data:/var/lib/kafka/data
    depends_on:
      - kafka-controller-1
      - kafka-controller-2
      - kafka-controller-3
```





# Isolated vs. Combined Mode

## We Chose Isolated Mode
**What It Is:**  
We’ve split duties—3 controllers (`kafka-controller-1`, `2`, `3`) manage metadata, 3 brokers (`kafka-broker-1`, `2`, `3`) handle data like Zeco Eats orders.

**Easy Way to Think:**  
*Imagine a restaurant—managers (controllers) plan, cooks (brokers) cook—separate teams.*

---

## How Combined Mode Would Look
**What It Is:**  
Each node does both jobs—controller and broker—fewer nodes, more multitasking.

### Example Config: Here’s one node from a 3-node combined setup:
**here is what makes it a combined setup:  KAFKA_PROCESS_ROLES: controller,broke**
**each kafka node acts as a broker and controller**

```yaml
kafka-1:
  image: apache/kafka:3.9.0
  container_name: kafka-1
  ports:
    - "9092:9092"  # Broker
    - "9093:9093"  # Controller
  environment:
    KAFKA_NODE_ID: 1
    CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
    KAFKA_PROCESS_ROLES: controller,broker  # Both! 
    KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092
    KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
    KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
    KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
    KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-1:9093,2@kafka-2:9093,3@kafka-3:9093
    KAFKA_LOG_DIRS: /var/lib/kafka/data
  volumes:
    - kafka-1-data:/var/lib/kafka/data
```


# Environment Variables Explained

Let’s pull vars from `kafka-controller-1` and `kafka-broker-1`—they tell the story of why this works.

---

## one of the Controllers: kafka-controller-1

### YAML Config:
```yaml
environment:
  KAFKA_NODE_ID: 1
  KAFKA_PROCESS_ROLES: controller
  KAFKA_LISTENERS: CONTROLLER://0.0.0.0:9093
  KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
  KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
  KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
  KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
  KAFKA_LOG_DIRS: /var/lib/kafka/data
  KAFKA_RAFT_ELECTION_TIMEOUT_MS: 5000
```



### KAFKA_NODE_ID: 1
- **Why:** The unique identifier for this node within the Kafka cluster.
- **Use:** Kafka uses this ID to track the node in the Raft consensus quorum, correlating with `1@...` in the `QUORUM_VOTERS` configuration.
- **Recall:** “Node ID = my unique identifier in the cluster.”

---

### KAFKA_PROCESS_ROLES: controller
- **Why:** Specifies the role of this node in the Kafka cluster, indicating that it is solely a controller and does not handle broker responsibilities.
- **Use:** This role limits the node’s responsibilities to managing metadata (such as deciding which broker should be the leader for a specific partition).
- **Recall:** “Role = this node is responsible for coordination, not data storage.”

---

### KAFKA_LISTENERS: CONTROLLER://0.0.0.0:9093
- **Why:** Defines the communication port for the Raft protocol between controllers. `0.0.0.0` allows the controller to listen on all available network interfaces.
- **Use:** Other controllers communicate with this node on port `9093` for coordination and metadata updates.
- **Recall:** “Listeners = the communication channel for Raft protocol between controllers.”

---

### KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
- **Why:** This is a placeholder that applies to brokers, not controllers.
- **Use:** Although controllers do not use this listener, it ensures consistent configuration across Kafka nodes.
- **Recall:** “Inter-broker = used by brokers for communication, not relevant for controllers.”

---

### KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
- **Why:** Specifies the name of the listener for the Raft protocol, linking to the `LISTENERS` configuration for controllers.
- **Use:** This setting tells brokers that they should use the `CONTROLLER` listener to communicate with the controllers for metadata updates.
- **Recall:** “Controller listener name = the communication channel for metadata updates.”

---

### KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
- **Why:** Lists the controllers that participate in the Raft consensus quorum. A majority of these controllers (2 out of 3) must agree on important cluster decisions, such as leader election.
- **Use:** Ensures that decisions are made in a distributed and fault-tolerant manner, allowing for leader elections and maintaining the cluster's state.
- **Recall:** “Quorum = the set of controllers that vote on key decisions in the cluster.”

---

### KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
- **Why:** Sets the delay for rebalance operations when a new consumer group starts consuming messages.
- **Use:** With this set to `0`, consumers can immediately start consuming messages, which is important for efficient processing in systems that require low-latency data flow.
- **Recall:** “Rebalance delay = 0ms means consumers start consuming instantly.”

---

### KAFKA_LOG_DIRS: /var/lib/kafka/data
- **Why:** Specifies the directory where Kafka stores its metadata, such as partition leader information.
- **Use:** Ensures that the data is stored persistently, even if the node is restarted, as opposed to temporary storage locations like `/tmp`.
- **Recall:** “Log dirs = where the persistent metadata is stored safely.”

---

### KAFKA_RAFT_ELECTION_TIMEOUT_MS: 5000
- **Why:** Defines the maximum amount of time (in milliseconds) a controller will wait before initiating a leader election if it loses communication with the current leader.
- **Use:** This timeout ensures that if the current leader becomes unresponsive, a new leader can be elected quickly, maintaining cluster availability.
- **Recall:** “Election timeout = 5000ms to quickly elect a new leader if needed.”



### One of the Brokers: kafka-broker-1

```yaml
environment:
  KAFKA_NODE_ID: 4
  KAFKA_PROCESS_ROLES: broker
  KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:19092,PLAINTEXT_HOST://0.0.0.0:9092
  KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker-1:19092,PLAINTEXT_HOST://localhost:29092
  KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
  KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
  KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
  KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka-controller-1:9093,2@kafka-controller-2:9093,3@kafka-controller-3:9093
  KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
  KAFKA_DEFAULT_REPLICATION_FACTOR: 3
  KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3
  KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3
  KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2
  KAFKA_LOG_DIRS: /var/lib/kafka/data
  KAFKA_LOG_RETENTION_HOURS: 720
  KAFKA_MESSAGE_MAX_BYTES: 10485760
  KAFKA_MAX_CONNECTIONS_PER_IP: 100
```


### Kafka Configuration Breakdown
#### **KAFKA_NODE_ID: 4**
- **Why:** A unique identifier for this Kafka node within the cluster.
- **Use:** The controller tracks this ID to associate it with specific partitions or roles, e.g., “Broker #4 is responsible for `orders-0`.”
- **Recall:** “Node ID = the unique identifier assigned to this node in the Kafka cluster.”

---

#### **KAFKA_PROCESS_ROLES: broker**
- **Why:** Specifies that this node is a broker, responsible for handling data storage and replication.
- **Use:** This node will focus on managing and storing Kafka topics and partitions, and not handling metadata management (which is done by controllers).
- **Recall:** “Role = this node handles data storage and retrieval, not coordination.”

---

#### **KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:19092,PLAINTEXT_HOST://0.0.0.0:9092**
- **Why:** Defines two listening ports—one for internal communication between brokers (`19092`), and one for external communication with clients (`9092`).
- **Use:** Internal communication uses `19092` for tasks like replication, while `9092` is used for client requests, allowing communication from external services like the application to the Kafka brokers.
- **Recall:** “Listeners = the ports for broker-to-broker and client communication.”

---

#### **KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker-1:19092,PLAINTEXT_HOST://localhost:29092**
- **Why:** Specifies how this broker should advertise its presence to other brokers and clients, including both internal and external addresses.
- **Use:** Internally, `kafka-broker-1:19092` is used for communication with other brokers, while externally, `localhost:29092` can be used by client applications. (In a cloud environment, `localhost` will be replaced by an appropriate IP address.)
- **Recall:** “Advertised listeners = the way external and internal services can contact this broker.”

---

#### **KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT**
- **Why:** Names the internal communication line used between brokers.
- **Use:** This setting tells Kafka brokers to use the `PLAINTEXT` protocol on port `19092` for communication, such as replicating partitions.
- **Recall:** “Inter-broker = the communication channel between brokers.”

---

#### **KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER**
- **Why:** Defines the listener name that the broker uses to communicate with the controller.
- **Use:** This setting specifies that the broker should use the `CONTROLLER` listener (as defined in the controller's configuration) to fetch metadata updates, like which partition leader to elect.
- **Recall:** “Controller listener name = the communication channel used by brokers to reach controllers.”

---

#### **KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT**
- **Why:** Maps listener names to security protocols, with no encryption (PLAINTEXT) being used for all communication types.
- **Use:** Ensures that communication across the controller, broker-to-broker, and client-to-broker interfaces uses the same plaintext protocol, as no encryption is configured at this point.
- **Recall:** “Protocol map = defines the communication protocol for each listener (all plaintext).”

---

#### **KAFKA_CONTROLLER_QUORUM_VOTERS: ...**
- **Why:** Specifies the set of Raft quorum voters that this broker communicates with to participate in the consensus process.
- **Use:** This broker will check with the other members of the quorum to synchronize important information like partition leadership.
- **Recall:** “Quorum = the set of controllers involved in making decisions about the cluster.”

---

#### **KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0**
- **Why:** This setting disables any delay in consumer group rebalancing, ensuring that consumers can immediately start processing messages as soon as they join a group.
- **Use:** This results in a faster start-up for consumer groups, which is important in systems that need to process messages with minimal delay.
- **Recall:** “Rebalance delay = 0ms means consumers start immediately without waiting.”

---

#### **KAFKA_DEFAULT_REPLICATION_FACTOR: 3**
- **Why:** Defines the default replication factor for newly created topics, meaning that Kafka will create 3 replicas of each partition for fault tolerance.
- **Use:** This ensures that if one broker fails, there are still two other copies of the partition available to prevent data loss.
- **Recall:** “Replication factor = 3 copies of each partition for redundancy.”

---

#### **KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3**
- **Why:** Ensures that the special `__consumer_offsets` topic, which tracks consumer group progress, is replicated 3 times.
- **Use:** This provides fault tolerance for consumer progress tracking, ensuring that consumers can resume from where they left off in case of broker failures.
- **Recall:** “Offsets topic = 3 replicas of the consumer progress logs for reliability.”

---

#### **KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3**
- **Why:** Configures the replication factor for the transaction log, which records the state of transactional operations.
- **Use:** This ensures that transactional operations, such as order payments, are reliably stored with redundancy, so they are not lost if a broker fails.
- **Recall:** “Transaction logs = 3 copies to ensure data integrity.”

---

#### **KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 2**
- **Why:** Defines the minimum number of in-sync replicas required for a transaction log to be considered successful.
- **Use:** Ensures that even if one replica fails, the transaction will still be committed as long as two replicas confirm the write.
- **Recall:** “Min ISR = 2 replicas must confirm the transaction for it to be considered successful.”

---

#### **KAFKA_LOG_DIRS: /var/lib/kafka/data**
- **Why:** Specifies the directory where Kafka stores its log files, which include message data and metadata.
- **Use:** This directory is used for persistent storage, ensuring that log data survives restarts or failures, and is typically mapped to cloud storage like EBS.
- **Recall:** “Log dirs = where Kafka stores data persistently.”

---

#### **KAFKA_LOG_RETENTION_HOURS: 720**
- **Why:** Configures the retention period for logs, keeping them for 720 hours (30 days) instead of the default 168 hours (7 days).
- **Use:** This ensures that logs (e.g., order data) are kept for a longer period for auditing or replay purposes.
- **Recall:** “Retention = 720 hours (30 days) of log history.”

---

#### **KAFKA_MESSAGE_MAX_BYTES: 10485760**
- **Why:** Specifies the maximum size for a Kafka message, allowing for larger messages (up to 10MB instead of the default 1MB).
- **Use:** This allows larger messages, such as those containing complex order data, to be transmitted within Kafka without being truncated.
- **Recall:** “Max message size = 10MB allows larger data payloads.”

---

#### **KAFKA_MAX_CONNECTIONS_PER_IP: 100**
- **Why:** Limits the number of concurrent connections a single client can make to a broker, preventing overload from a single IP address.
- **Use:** This ensures that a single client (e.g., an application) does not overwhelm the broker by opening too many connections.
- **Recall:** “Max connections = 100 connections per IP address to avoid overload.”


