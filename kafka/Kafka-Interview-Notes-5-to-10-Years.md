# Apache Kafka — Interview Notes (5–10 Years Experience)

> Focus: architecture, internals, reliability, performance, transactions, operations, Spring Boot integration, and scenario-based questions.

## 1. What is Kafka?
Kafka is a distributed event-streaming platform used to publish, store, and process streams of records at scale.

### Core idea
**Producer → Topic/Partition → Consumer**

## 2. Kafka Core Components

- **Broker** — Kafka server that stores and serves records.
- **Topic** — Logical stream/category of records.
- **Partition** — Ordered, append-only log inside a topic.
- **Producer** — Publishes records.
- **Consumer** — Reads records.
- **Consumer Group** — Consumers sharing work for a topic.
- **Offset** — Position of a record within a partition.
- **Controller / KRaft** — Cluster metadata and controller management in modern Kafka.
- **Replica** — Copy of a partition for fault tolerance.

### Memory
**Topic = category, Partition = parallelism + ordering, Offset = position.**

# 3. Why Kafka?

Key reasons:

- High throughput
- Horizontal scalability
- Durable event storage
- Fault tolerance through replication
- Consumer replay using offsets
- Loose coupling between producers and consumers
- Real-time event processing

### Interview answer
> Kafka is useful when we need durable, scalable, high-throughput asynchronous communication with replayable events.

# 4. Topic vs Partition

A topic can contain multiple partitions.

Example:
```text
Orders Topic
  ├── P0
  ├── P1
  └── P2
```

Partitions enable parallel processing.

### Important
Kafka ordering is guaranteed **within a partition**, not globally across the whole topic.

# 5. How does Kafka maintain ordering?

Records written to the same partition are ordered by offset.

If strict ordering is required for an entity, use a stable key:
```java
producer.send(new ProducerRecord<>("orders", orderId, event));
```

The same key is mapped consistently to a partition under the partitioner rules.

### Interview trap
> More partitions do not mean global ordering.

# 6. Producer

Producer publishes records to Kafka.

Important producer concepts:

- `acks`
- retries
- idempotence
- batching
- compression
- linger
- delivery timeout
- buffer memory
- `max.in.flight.requests.per.connection`

### Important `acks`

- `acks=0` — no broker acknowledgement
- `acks=1` — leader acknowledgement
- `acks=all` / `-1` — all in-sync replicas acknowledgement

For critical events, `acks=all` is commonly combined with replication and idempotence.

# 7. Producer Idempotence

Idempotent producer prevents duplicate writes caused by producer retries from being appended more than once within the supported producer semantics.

Typical configuration:
```properties
enable.idempotence=true
acks=all
```

### Important
Producer idempotence does **not** mean your entire end-to-end business workflow is automatically exactly once.

# 8. Delivery Semantics

### At-most-once
Message may be lost, but is not normally redelivered.

### At-least-once
Message is not lost under the delivery model, but duplicates may occur.

### Exactly-once
The system is designed so an event's processing/output is committed atomically within Kafka's transactional model for supported workflows.

### Interview point
> In distributed systems, application-level exactly-once side effects still require careful design, especially when external systems are involved.

# 9. Consumer

Consumer reads records and tracks offsets.

Important settings:

- `group.id`
- `enable.auto.commit`
- `auto.offset.reset`
- `max.poll.records`
- `max.poll.interval.ms`
- `session.timeout.ms`
- `heartbeat.interval.ms`

# 10. Consumer Group

A consumer group allows multiple consumers to process partitions in parallel.

Example:
```text
Topic: 4 partitions

Group A:
Consumer 1 → P0
Consumer 2 → P1
Consumer 3 → P2
Consumer 4 → P3
```

### Important rule
Within one consumer group, **a partition is assigned to at most one consumer at a time**.

If consumers > partitions, some consumers remain idle.

# 11. Partition Rebalancing

A rebalance happens when partition ownership changes within a consumer group.

Possible causes:

- Consumer joins
- Consumer leaves
- Consumer crashes
- Membership changes
- Partition assignment changes

### Interview follow-up
How do you reduce rebalance impact?

- Proper poll configuration
- Avoid long processing inside the consumer thread
- Cooperative assignment strategies where appropriate
- Stable consumer membership
- Right-sized consumer count

# 12. Consumer Offset

Offset is the logical position of a record in a partition.

Kafka stores committed consumer-group offsets so consumers can resume.

### Important
Offsets are per **consumer group + partition**.

# 13. `auto.offset.reset`

Common values:

- `earliest`
- `latest`
- `none`

It applies when there is no valid committed offset for the consumer group or the committed offset is no longer available.

# 14. Auto Commit vs Manual Commit

### Auto commit
Offsets are committed automatically according to configuration.

### Manual commit
```java
consumer.commitSync();
```

or asynchronous commit.

Manual control is useful when message processing must complete before the offset is committed.

### Key principle
**Commit offset only after the required processing is safely completed.**

# 15. At-Least-Once Processing

Common approach:
```text
poll
 ↓
process
 ↓
commit offset
```

If the consumer crashes after processing but before commit, the record may be delivered again.

Therefore:
> At-least-once systems require idempotent consumers or deduplication where duplicates are possible.

# 16. Idempotent Consumer

An idempotent consumer produces the same business result even if the same event is processed multiple times.

Typical techniques:

- Unique event ID
- Database unique constraint
- Processed-event table
- Upsert
- State/version checks

Example:
```text
eventId = 123
Already processed? → ignore
Not processed? → process + mark processed
```

# 17. Consumer Lag

Consumer lag is the amount of unconsumed backlog relative to the latest available records.

Conceptually:
```text
Lag = latest available offset - consumer position
```

### High lag can indicate

- Slow consumers
- Under-provisioning
- Downstream bottleneck
- Rebalances
- Large processing time
- Broker/network issues

# 18. How to Handle High Consumer Lag?

Possible actions:

1. Increase consumer instances up to useful partition parallelism.
2. Increase partitions when sustained parallelism is required and capacity planning supports it.
3. Optimize processing.
4. Batch downstream operations.
5. Reduce expensive synchronous calls.
6. Monitor broker and consumer metrics.
7. Check rebalances and poll settings.

### Important
Adding consumers beyond partition count does not increase parallel processing for that topic partition set.

# 19. Replication

Kafka replicates partitions across brokers.

Example:
```text
Partition P0
  Leader → Broker 1
  Replica → Broker 2
  Replica → Broker 3
```

Replication factor = 3.

# 20. Leader and Followers

Each partition has one leader and zero or more followers.

- Producer writes to leader.
- Consumers normally read from leader under standard configurations.
- Followers replicate leader data.

Leadership can move after broker failures.

# 21. ISR — In-Sync Replicas

ISR = replicas considered sufficiently caught up with the leader according to Kafka's replication rules.

`acks=all` waits for acknowledgement from all required in-sync replicas.

### Important setting
`min.insync.replicas`

Used with `acks=all` to enforce a minimum number of ISR replicas required for successful writes.

# 22. Replication Factor vs ISR

**Replication Factor** = total configured copies.

**ISR** = replicas currently considered in sync.

Example:
```text
RF = 3
ISR = 3 normally
```

If one replica falls behind:
```text
RF = 3
ISR = 2
```

# 23. Kafka Durability

Durability depends on configuration and operational design, including:

- Replication factor
- `acks`
- `min.insync.replicas`
- Broker storage
- Failure handling
- Producer retry/idempotence strategy

# 24. Kafka Retention

Kafka does not remove a record simply because a consumer read it.

Records remain according to retention configuration.

Common retention policies:

- Time-based retention
- Size-based retention

This is why Kafka supports replaying old events while they remain available.

# 25. Log Compaction

Compaction keeps the latest record for each key, subject to compaction semantics.

Useful for:

- Current state snapshots
- Reference data
- Key-based state restoration

Example:
```text
user-1 → A
user-1 → B
user-1 → C

Compacted log eventually keeps latest value for user-1
```

### Retention vs Compaction

**Retention = keep based on time/size**

**Compaction = keep latest value per key**

# 26. Kafka vs Traditional Message Queue

Kafka:

- Durable log
- Replayable
- Partition-based scaling
- Multiple independent consumer groups can read the same events

Traditional queue semantics often emphasize message delivery to workers.

### Interview answer
> Kafka is better when we need event history, replay, high throughput, and multiple independent consumers.

# 27. Kafka vs REST

### REST
Synchronous request-response.

### Kafka
Asynchronous event-driven communication.

Use Kafka when:

- Consumers can process asynchronously
- Loose coupling is valuable
- Event streaming is required
- Replay is useful

# 28. Kafka in Microservices

Typical architecture:
```text
Order Service
     ↓
   Kafka
   ├── Payment Service
   ├── Inventory Service
   └── Notification Service
```

Advantages:

- Loose coupling
- Independent scaling
- Async processing
- Event-driven architecture

# 29. Topic Design

Questions to ask:

- What is the event/business domain?
- What should the partition key be?
- What ordering is required?
- How many partitions are needed?
- How long should data be retained?
- How many consumer groups exist?
- Is replay required?

# 30. Partition Key Selection

Good partition key should balance:

- Ordering requirements
- Even distribution
- Business grouping

Example:

If all events for one customer must remain ordered:
```text
key = customerId
```

### Risk
Hot key / hot partition if one key receives disproportionate traffic.

# 31. Hot Partition

Occurs when records are unevenly distributed and one partition receives much more traffic.

Solutions depend on business constraints:

- Better key distribution
- Composite keys
- Splitting hot entities
- More partitions
- Reconsider ordering granularity

# 32. Producer Batching

Kafka producers can batch multiple records before sending.

Relevant settings:

- `batch.size`
- `linger.ms`
- compression settings

Trade-off:

**More batching → potentially better throughput**

but:

**more waiting → potentially higher latency**

# 33. Compression

Common codecs include:

- gzip
- snappy
- lz4
- zstd

Compression can reduce network/storage pressure at the cost of CPU.

# 34. Kafka Transactions

Kafka transactions can atomically publish records to multiple partitions/topics and commit consumer offsets together in supported read-process-write workflows.

Typical configuration:
```properties
transactional.id=...
```

Consumer isolation:
```properties
isolation.level=read_committed
```

### Interview point
Transactions are useful for Kafka-to-Kafka exactly-once style processing.

# 35. Exactly-Once Processing

Typical flow:
```text
Consume
 ↓
Process
 ↓
Produce result
 ↓
Commit offsets + output atomically
```

Kafka transactions help combine these operations within Kafka.

### Caveat
External database or third-party side effects require additional patterns/design.

# 36. Kafka Streams

Kafka Streams is a Java library for processing Kafka data.

Features:

- Stateless transformations
- Stateful processing
- Windowing
- Joins
- Aggregations
- Exactly-once processing support

Common operations:
```text
filter
map
flatMap
groupBy
aggregate
join
window
```

# 37. Kafka Streams vs Consumer API

### Consumer API
You build the processing logic manually.

### Kafka Streams
Provides abstractions for stream processing, state stores, joins, windows, etc.

# 38. Dead Letter Topic (DLT)

When a message repeatedly fails processing, the application can publish it to a dead-letter topic.

```text
Main Topic
    ↓
Consumer
    ↓
Processing fails
    ↓
DLT
```

Use DLT for:

- Poison messages
- Invalid payloads
- Business validation failures

# 39. Retry Strategy

Possible models:

### Immediate retry
Retry quickly.

### Delayed retry
Wait before retrying.

### Retry topics
Move failed events through delayed/retry topics.

### DLT
Send permanently failing messages to dead-letter topic.

### Interview point
Do not retry non-transient business validation errors forever.

# 40. Poison Message

A message that repeatedly fails processing and blocks normal progress.

Solutions:

- Retry with limit
- Backoff
- DLT
- Error classification
- Manual remediation

# 41. Kafka Error Handling

Classify failures:

### Transient
Network timeout, temporary dependency issue.

→ Retry/backoff.

### Permanent
Invalid schema, impossible business rule.

→ DLT / remediation.

# 42. Schema Management

Kafka payloads should have a controlled schema.

Common approaches:

- JSON
- Avro
- Protobuf
- JSON Schema

Schema Registry is commonly used to manage schema versions and compatibility.

### Why?

Avoid producers and consumers breaking each other.

# 43. Schema Evolution

Common compatibility models:

- Backward
- Forward
- Full

Interview answer:
> Schema evolution should maintain compatibility so consumers and producers can be deployed independently.

# 44. Spring Boot Kafka

Typical producer:
```java
kafkaTemplate.send("orders", orderId, event);
```

Consumer:
```java
@KafkaListener(topics = "orders", groupId = "payment-service")
public void consume(OrderEvent event) {
    // process
}
```

# 45. Kafka Consumer Offset in Spring

Important considerations:

- Ack mode
- Manual acknowledgement
- Error handler
- Retry configuration
- DLT handling
- Consumer concurrency

# 46. Kafka + Database Consistency

Classic problem:
```text
DB update succeeds
Kafka publish fails
```

or:
```text
Kafka publish succeeds
DB update fails
```

This creates inconsistency.

### Common solution
Transactional Outbox pattern.

```text
Application
   ↓
DB Transaction
   ├── Business data
   └── Outbox event
          ↓
     Publisher
          ↓
        Kafka
```

# 47. Transactional Outbox

Steps:

1. Update business data.
2. Insert event into outbox table in same DB transaction.
3. A publisher reads outbox records.
4. Publish to Kafka.
5. Mark outbox event as published.

### Benefits

- Reliable event publication
- Avoids dual-write inconsistency

# 48. Outbox Duplicates

Outbox publisher can publish duplicates if it crashes after Kafka publish but before marking the outbox row complete.

Therefore:
> Downstream consumers should still be idempotent.

# 49. Kafka vs Saga

Saga coordinates a distributed business transaction through local transactions and events/commands.

Kafka can be used as the transport/event backbone for Saga communication.

Kafka itself is **not** the Saga pattern.

# 50. Kafka Security

Important areas:

- TLS/SSL for encryption in transit
- SASL for authentication
- ACLs for authorization
- Secrets management
- Certificate/key rotation
- Network isolation

### Interview answer
> Kafka security is typically designed around encryption, client authentication, authorization and controlled network access.

# 51. Kafka Monitoring

Monitor:

- Consumer lag
- Under-replicated partitions
- Offline partitions
- Request latency
- Throughput
- Broker CPU/memory/disk
- Network
- ISR changes
- Producer errors
- Consumer rebalances

# 52. High Availability

Use:

- Multiple brokers
- Replication factor > 1
- Proper ISR settings
- Rack awareness where appropriate
- Monitoring
- Capacity planning

Avoid a single-broker production dependency.

# 53. What happens if a broker goes down?

If the broker hosts leaders for partitions:

1. Kafka detects failure.
2. A suitable in-sync replica can become leader.
3. Producers/consumers refresh metadata.
4. Processing resumes according to availability and configuration.

# 54. What happens if a consumer crashes?

Kafka detects membership change.

Its partitions can be reassigned during rebalance.

The new consumer resumes from the last committed offsets.

# 55. What happens if a producer sends a duplicate event?

With idempotent producer configuration, retry-induced duplicates can be prevented within Kafka producer semantics.

For business-level duplicates from application retries, use event IDs/idempotent consumers.

# 56. Kafka Performance Tuning

### Producer

- Batch size
- Linger
- Compression
- Async sends
- Appropriate acknowledgements

### Consumer

- Batch processing
- `max.poll.records`
- Fetch sizes
- Concurrency
- Efficient downstream calls

### Broker

- Disk throughput
- Network
- Partition count
- Replication
- JVM/runtime health

# 57. Too Many Partitions

More partitions provide more parallelism but also increase:

- Metadata
- File handles
- Recovery cost
- Rebalance overhead
- Operational complexity

### Interview line
> Partition count should be driven by throughput, consumer parallelism and operational capacity—not simply by making it as large as possible.

# 58. Consumer Scaling

To scale consumption:
```text
Partitions = 12

Consumers = 3
→ each can handle multiple partitions

Consumers = 12
→ maximum direct partition-level parallelism

Consumers = 20
→ 8 consumers may be idle
```

# 59. Backpressure

If producer rate > consumer processing rate, backlog increases.

Possible solutions:

- Increase consumer parallelism
- Batch processing
- Reduce processing cost
- Scale downstream dependencies
- Control producer rate
- Use buffering/retry strategy

# 60. Kafka Scenario Questions — 5–10 Years

## Scenario 1
**Consumer lag suddenly becomes very high. What do you check?**

Answer structure:
```text
Check → Consumer health
      → Processing latency
      → Rebalances
      → Partition distribution
      → Downstream dependency
      → Broker health
      → Consumer capacity
```

## Scenario 2
**How do you guarantee order for customer events?**

> Use `customerId` as partition key so all events for a customer go to the same partition.

## Scenario 3
**How do you avoid duplicate payment processing?**

Use:

- Unique transaction/event ID
- Idempotency
- Database constraint
- Processed-event tracking
- Appropriate Kafka delivery semantics

## Scenario 4
**DB update and Kafka publish must remain consistent.**

> Use Transactional Outbox.

## Scenario 5
**One event fails repeatedly.**

> Classify the failure, retry transient errors with backoff, and route permanently failing messages to a DLT.

## Scenario 6
**Need exactly-once Kafka-to-Kafka processing.**

> Use Kafka transactions and appropriate consumer isolation, while considering external side effects separately.

## Scenario 7
**One customer generates huge traffic.**

> Check for a hot partition. Reconsider partition-key strategy while preserving the required ordering semantics.

## Scenario 8
**Can we guarantee global ordering?**

> Not across multiple partitions. Kafka guarantees ordering within a partition.

# 61. Kafka Interview Rapid-Fire

### Topic?
Logical stream of records.

### Partition?
Ordered append-only log used for scalability and parallelism.

### Offset?
Position of a record inside a partition.

### Consumer Group?
Consumers sharing partition-processing work.

### Replication Factor?
Number of copies of each partition.

### ISR?
Replicas currently considered in sync.

### Consumer Lag?
Backlog relative to the latest available data.

### Kafka retention?
How long/under what size rules records remain available.

### Compaction?
Retains latest value per key subject to compaction semantics.

### `acks=all`?
Producer waits for acknowledgement from all required in-sync replicas.

### `min.insync.replicas`?
Minimum ISR replicas required for successful writes with appropriate acknowledgements.

### Idempotent producer?
Avoids duplicate writes caused by producer retries within Kafka producer semantics.

### At-least-once?
Duplicates possible; design consumer for idempotency.

### Partition ordering?
Ordering only within a partition.

### DLT?
Destination for messages that cannot be processed successfully after configured handling.

### Hot partition?
Uneven partition load caused by skewed keys.

### Outbox?
Reliable DB-to-Kafka event publication pattern.

### Kafka Streams?
Library for processing Kafka data streams.

# 62. Most Important Questions for 5–10 Years

Before interview, be able to explain these deeply:

1. Kafka architecture
2. Topic vs partition
3. Producer internals
4. `acks`
5. Idempotent producer
6. Delivery semantics
7. Consumer groups
8. Rebalancing
9. Offset management
10. Consumer lag
11. Partition key selection
12. Ordering guarantees
13. Replication / ISR
14. `min.insync.replicas`
15. Retention vs compaction
16. Retry and DLT
17. Exactly-once semantics
18. Kafka transactions
19. Transactional Outbox
20. Idempotent consumer
21. Schema evolution
22. Kafka security
23. Performance tuning
24. High availability
25. Kafka + Spring Boot
26. Kafka + Microservices
27. Kafka + Saga
28. High traffic / hot partition
29. Failure recovery
30. Production monitoring

# 63. 2-Minute Interview Summary

> **Kafka is a distributed event-streaming platform. Data is stored in topics, which are divided into partitions for scalability and parallelism. Producers write records and consumers read them through consumer groups. Ordering is guaranteed within a partition, and partition keys are therefore important when business ordering is required. Kafka provides durability through replication and ISR. Consumer offsets allow replay and recovery. For reliability we consider acknowledgements, idempotent producers, retries, DLTs and idempotent consumers. For Kafka-to-Kafka processing, Kafka transactions can provide exactly-once processing semantics. When coordinating database changes with Kafka, Transactional Outbox is a common solution. In production I would monitor consumer lag, broker health, replication, throughput, latency and rebalances.**

# 🧠 Kafka Master Memory Map

```text
KAFKA
│
├── Core
│   ├── Broker
│   ├── Topic
│   ├── Partition
│   ├── Offset
│   └── Consumer Group
│
├── Producer
│   ├── acks
│   ├── retries
│   ├── idempotence
│   ├── batching
│   └── compression
│
├── Consumer
│   ├── offset
│   ├── lag
│   ├── commit
│   └── rebalance
│
├── Reliability
│   ├── replication
│   ├── ISR
│   ├── DLT
│   ├── retry
│   └── idempotency
│
├── Transactions
│   ├── EOS
│   ├── Kafka Transaction
│   └── Outbox
│
├── Performance
│   ├── partitions
│   ├── batching
│   ├── compression
│   └── consumer scaling
│
├── Security
│   ├── TLS
│   ├── SASL
│   └── ACL
│
└── Operations
    ├── Monitoring
    ├── HA
    ├── Capacity
    └── Failure Recovery
```
