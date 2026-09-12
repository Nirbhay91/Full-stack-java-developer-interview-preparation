# HLD — High-Level Design Interview Notes

> Interview revision sheet for Java/Spring Boot backend and microservices roles.

## 1. What is HLD?
High-Level Design defines the architecture of a system: major components, responsibilities, data flow, communication, storage, scalability, reliability, security, and deployment.

**HLD = What components + How they interact + How the system scales and survives failures.**

### HLD vs LLD
- **HLD:** services, databases, queues, APIs, load balancers, scaling, deployment.
- **LLD:** classes, interfaces, design patterns, methods, object relationships.

## 2. HLD Interview Approach ⭐⭐⭐⭐⭐
Use this order:

1. Clarify requirements
2. Estimate scale
3. Define APIs / important operations
4. Identify core entities/data
5. Draw high-level components
6. Choose database/storage
7. Add caching
8. Add asynchronous messaging where useful
9. Discuss scalability
10. Discuss reliability/fault tolerance
11. Discuss security
12. Discuss observability
13. Discuss bottlenecks and trade-offs

**Memory:** Requirements → Scale → API → Data → Architecture → Scale → Failure → Security → Observability.

## 3. Functional vs Non-Functional Requirements
### Functional
What the system should do.
- Create order
- Search product
- Send notification
- Upload file

### Non-functional
How the system should behave.
- Availability
- Latency
- Throughput
- Scalability
- Durability
- Consistency
- Security
- Reliability

**Interview trick:** Always ask both.

## 4. Capacity Estimation ⭐⭐⭐⭐
Estimate:
- DAU / MAU
- Requests per second
- Peak traffic
- Read/write ratio
- Data generated per day
- Storage growth
- Bandwidth

### Basic formulas
- `RPS = total requests / seconds`
- `Peak RPS = average RPS × peak factor`
- `Storage/day = writes/day × average object size`

Do rough numbers first; exact precision is usually less important than identifying the dominant scale and bottleneck.

## 5. Scalability
### Vertical Scaling
Increase CPU/RAM on one machine.

**Pros:** simple.
**Cons:** hard upper limit, bigger failure domain.

### Horizontal Scaling
Add more instances.

**Pros:** better scale and resilience.
**Cons:** requires distributed coordination, load balancing, statelessness, shared storage/session strategy.

**Memory:** Vertical = Bigger machine. Horizontal = More machines.

## 6. Stateless vs Stateful Services
### Stateless
Each request contains enough context; any instance can process it.

Best for horizontal scaling.

### Stateful
Instance stores client/session state locally.

Can require sticky sessions or external shared state.

**Preferred:** Keep application services stateless where practical; externalize shared state to a durable/shared store.

## 7. Load Balancer ⭐⭐⭐⭐
Distributes requests across healthy instances.

Common strategies:
- Round robin
- Weighted round robin
- Least connections
- Hash-based routing

Responsibilities can include:
- Health checks
- TLS termination
- Traffic distribution
- Failover

## 8. Reverse Proxy
A reverse proxy sits in front of backend servers and forwards client requests.

Can provide:
- TLS termination
- Routing
- Compression
- Caching
- Rate limiting
- Security controls

## 9. API Gateway ⭐⭐⭐⭐⭐
Single entry point for clients into backend services.

Responsibilities:
- Routing
- Authentication/authorization enforcement
- Rate limiting
- Request/response transformation
- Aggregation
- Logging/tracing
- TLS termination

**Why not direct microservice calls from frontend?**
- Exposes internal topology
- Cross-cutting concerns get duplicated
- Harder client evolution
- More coupling

## 10. Service Discovery
Maps logical service names to currently available instances.

Examples of approaches:
- Client-side discovery
- Server-side discovery
- Registry-based discovery

**Why service name instead of IP?**
Instances are dynamic because of autoscaling, deployment, failure, and rescheduling.

## 11. Database Choice ⭐⭐⭐⭐⭐
### SQL / RDBMS
Use when you need:
- Strong transactional guarantees
- Rich joins
- Structured relational data
- Mature constraints/querying

Examples: PostgreSQL, MySQL, Oracle.

### NoSQL
Useful when:
- Access patterns are simple and well-defined
- Massive horizontal scale is needed
- Flexible schema is valuable

Examples:
- DynamoDB / key-value/document
- MongoDB / document
- Cassandra / wide-column
- Redis / in-memory key-value

**Interview answer:** Choose based on consistency, access patterns, query requirements, scale, latency, and operational needs—not popularity.

## 12. Database per Service
A microservice should own its data and schema when independent deployment and autonomy are important.

**Benefits:** lower coupling, independent scaling, independent schema evolution.

**Trade-off:** cross-service joins/transactions become harder.

## 13. Database Indexing
Indexes speed up reads by creating searchable data structures, at the cost of storage and write overhead.

Consider indexes for:
- Frequently queried columns
- Join/filter/sort columns
- Composite access patterns

Avoid indexing everything.

## 14. Read Replica
Copies data from a primary to replicas for read scaling.

Common architecture:
```text
Writes → Primary DB
Reads  → Read Replicas
```

**Trade-off:** replication lag can cause stale reads.

## 15. Database Sharding ⭐⭐⭐⭐
Split data horizontally across multiple database partitions/shards.

Example:
```text
User ID 1–1M   → Shard A
User ID 1M–2M  → Shard B
```

### Shard key should:
- Distribute load evenly
- Match access patterns
- Avoid hotspots
- Be stable enough for the system

**Challenges:** cross-shard queries, transactions, resharding, operational complexity.

## 16. Partitioning
Split one logical dataset into partitions based on a key/range/time.

Useful for:
- Large tables
- Time-series data
- Faster maintenance/pruning

**Partitioning ≠ sharding:** partitioning can be inside one database/system; sharding generally distributes partitions across separate database nodes.

## 17. Caching ⭐⭐⭐⭐⭐
Use cache to reduce latency and database load.

Common strategies:
- Cache-aside
- Read-through
- Write-through
- Write-behind

### Cache-aside
1. Check cache
2. On miss, read DB
3. Put result into cache
4. Return result

**Trade-offs:** stale data, eviction, invalidation complexity, cache stampede.

### Cache invalidation
Common approaches:
- TTL
- Explicit delete/update
- Versioning
- Event-driven invalidation

**Memory:** Cache = fast reads, but introduces consistency/invalidation concerns.

## 18. Redis ⭐⭐⭐⭐
Common uses:
- Caching
- Session storage
- Distributed locks with care
- Counters/rate limiting
- Short-lived data
- Sorted sets for rankings

Do not automatically treat Redis as the system of record unless durability/consistency requirements explicitly support it.

## 19. Message Queue vs Event Streaming ⭐⭐⭐⭐⭐
### Queue
One message is generally processed by one consumer from a competing-consumer group.

Good for background jobs and work distribution.

### Event streaming
Events can be retained and consumed by multiple independent consumer groups.

Good for event-driven architecture, replay, analytics, and decoupling.

Examples:
- SQS / RabbitMQ for queues
- Kafka for event streaming

## 20. Kafka Basics ⭐⭐⭐⭐⭐
Core concepts:
- Topic
- Partition
- Producer
- Consumer
- Consumer group
- Offset

### Partition
Unit of parallelism and ordering within a Kafka topic partition.

### Consumer group
Consumers in the same group share partitions; each partition is assigned to one consumer within that group at a time.

**Memory:** Topic → Partitions → Consumers.

## 21. Synchronous vs Asynchronous Communication
### Synchronous
Caller waits for response.

Examples:
- REST
- gRPC

Good for immediate request/response.

### Asynchronous
Producer sends work/event and continues.

Examples:
- Kafka
- SQS
- RabbitMQ

Good for decoupling, buffering, retries, and long-running work.

## 22. REST vs gRPC
### REST
- HTTP-based resource-oriented APIs
- Easy client/browser integration
- Common for public APIs

### gRPC
- Contract-first RPC
- Protobuf
- Efficient binary protocol
- Strong fit for internal service-to-service communication

Choose based on client compatibility, latency, streaming, tooling, and organizational constraints.

## 23. Reliability / Fault Tolerance ⭐⭐⭐⭐⭐
### Timeout
Never wait indefinitely for a downstream dependency.

### Retry
Retry only transient failures and use:
- Exponential backoff
- Jitter
- Maximum attempt limits

Do not blindly retry non-idempotent operations.

### Circuit Breaker
Stops sending calls to a failing dependency temporarily.

States commonly modeled as:
```text
CLOSED → OPEN → HALF_OPEN → CLOSED
```

### Bulkhead
Isolate resources so one failing workload does not consume everything.

### Rate Limiting
Protect service from excessive traffic.

**Memory:** Timeout → Retry → Circuit Breaker → Bulkhead → Rate Limit.

## 24. Cascading Failure
One slow/failing service causes callers to pile up, consuming threads/connections and causing further failures.

Prevent with:
- Timeouts
- Circuit breakers
- Bulkheads
- Backpressure
- Bounded queues
- Rate limiting
- Graceful degradation

## 25. Backpressure
When consumers cannot keep up, the system controls the rate of incoming work instead of allowing unbounded accumulation.

Techniques:
- Bounded queues
- Rate limits
- Consumer throttling
- Load shedding

## 26. Distributed Transactions ⭐⭐⭐⭐⭐
Avoid global distributed transactions where possible.

### Saga Pattern
A business transaction is split into local transactions across services.

If a later step fails, compensating actions undo/offset earlier business actions.

### Choreography
Services react to events without a central coordinator.

### Orchestration
A central orchestrator directs the workflow.

**Trade-off:** Saga provides eventual consistency with more application-level coordination.

## 27. Idempotency ⭐⭐⭐⭐⭐
A repeated request produces the same intended business result as a single request.

Example:
```text
POST /payments
Idempotency-Key: 12345
```

Store the key/result so retries do not create duplicate payments/orders.

Useful when clients retry after timeouts.

## 28. Transactional Outbox ⭐⭐⭐⭐⭐
Solve the problem of updating the database and publishing an event reliably.

Pattern:
1. Update business data.
2. Insert event into outbox table in same DB transaction.
3. Separate publisher reads outbox and publishes event.
4. Mark event processed.

Helps avoid dual-write inconsistency.

## 29. Eventual Consistency
Different replicas/services may temporarily have different values but converge over time.

Common in distributed systems using asynchronous events and replication.

**Trade-off:** better availability/decoupling/scalability versus immediate consistency.

## 30. CAP Theorem ⭐⭐⭐⭐⭐
In the presence of a network partition, a distributed system cannot simultaneously guarantee both strong consistency and full availability across the partition.

- **C** = Consistency
- **A** = Availability
- **P** = Partition tolerance

Because partitions can occur in distributed systems, real systems make trade-offs between consistency and availability during partition scenarios.

## 31. ACID
- **Atomicity:** all or nothing
- **Consistency:** transaction preserves defined constraints/invariants
- **Isolation:** concurrent transactions behave according to isolation guarantees
- **Durability:** committed data survives failures according to database durability guarantees

## 32. Isolation Levels ⭐⭐⭐⭐
Common SQL levels:
- Read Uncommitted
- Read Committed
- Repeatable Read
- Serializable

Possible anomalies:
- Dirty read
- Non-repeatable read
- Phantom read

The exact behavior varies by database implementation.

## 33. Strong Consistency vs Eventual Consistency
### Strong consistency
Reads reflect the latest committed write according to the system's consistency model.

### Eventual consistency
Reads may temporarily observe older state but converge.

**Choose based on business correctness, latency, availability, and user experience.**

## 34. High Availability ⭐⭐⭐⭐⭐
Techniques:
- Multiple instances
- Multi-AZ / failure-domain deployment
- Load balancing
- Health checks
- Automatic failover
- Replication
- Stateless application tier
- Avoid single points of failure

**HA ≠ durability.** A highly available system can still lose data if storage durability is poor.

## 35. Disaster Recovery
Important concepts:
- Backup
- Replication
- RTO: Recovery Time Objective
- RPO: Recovery Point Objective

**RTO = how quickly service must recover.**

**RPO = how much data loss is acceptable.**

## 36. Deployment Strategies ⭐⭐⭐⭐
### Rolling
Gradually replace instances.

### Blue-Green
Maintain old and new environments; switch traffic.

### Canary
Send a small percentage of traffic to the new version first.

### Feature Flags
Deploy code separately from enabling functionality.

## 37. Auto Scaling
Scale instances based on metrics such as:
- CPU
- Memory
- Request rate
- Queue depth
- Latency

Good scaling requires:
- Stateless instances or externalized state
- Safe startup/shutdown
- Health checks
- Capacity planning

## 38. CDN
Content Delivery Network caches content near users.

Good for:
- Images
- Static JS/CSS
- Video/content delivery
- Cacheable API responses in suitable cases

Benefits:
- Lower latency
- Reduced origin traffic
- Better global performance

## 39. Object Storage
Useful for files/blobs such as:
- Images
- Videos
- Documents
- Backups

Example: Amazon S3.

Keep large binary files out of relational database rows unless requirements justify it.

## 40. Authentication vs Authorization
**Authentication:** Who are you?

**Authorization:** What are you allowed to do?

Common approaches:
- OAuth 2.0
- OpenID Connect
- JWT
- RBAC
- ABAC

## 41. JWT in Distributed Systems
A client obtains an access token and sends it to backend services/gateway.

Services validate:
- Signature
- Expiration
- Issuer/audience as applicable
- Required claims/scopes

Avoid putting sensitive secrets in JWT payloads.

## 42. Secrets Management
Do not hard-code passwords or API keys.

Use a secret manager/KMS-backed solution and rotate secrets.

Examples:
- AWS Secrets Manager
- HashiCorp Vault
- KMS for key management/encryption use cases

## 43. Encryption
### In transit
TLS/HTTPS.

### At rest
Disk/database/object encryption.

### Application-level encryption
Used when certain fields require additional protection.

## 44. Observability ⭐⭐⭐⭐⭐
Three pillars:

### Logs
What happened?

Use structured logs and correlation/request IDs.

### Metrics
How much/how often/how fast?

Examples:
- RPS
- Error rate
- Latency
- CPU
- Memory
- Queue depth

### Traces
Where did time go across distributed services?

**Memory:** Logs = events, Metrics = numbers, Traces = journey.

## 45. Health Checks
### Liveness
Is the process alive?

### Readiness
Can this instance receive traffic?

Readiness should consider whether the service is actually ready to serve requests.

## 46. Distributed Tracing
A request gets a trace/context ID propagated across services.

Example:
```text
Gateway
  ↓
Order Service
  ↓
Payment Service
  ↓
Notification Service
```

Tracing helps identify latency and failures across the call chain.

## 47. Logging Best Practices
- Structured JSON logs
- Correlation ID / trace ID
- Appropriate log levels
- Avoid secrets/PII in logs
- Centralized collection
- Retention policy

## 48. API Rate Limiting
Controls how frequently a client can call an API.

Common algorithms:
- Fixed window
- Sliding window
- Token bucket
- Leaky bucket

**Token bucket** is commonly used where burst traffic should be allowed within limits.

## 49. API Pagination ⭐⭐⭐⭐
### Offset pagination
```text
?page=2&size=20
```

Easy, but deep pages can become expensive and unstable under changing data.

### Cursor/keyset pagination
```text
?after=abc123&limit=20
```

Often better for large datasets and stable traversal.

## 50. API Versioning
Approaches:
- URI versioning: `/v1/orders`
- Header versioning
- Content negotiation

Focus on backward compatibility and controlled deprecation.

## 51. Cache Stampede
Many requests miss the same expired key and all hit the database simultaneously.

Mitigations:
- Request coalescing/single-flight
- Jittered TTL
- Pre-warming
- Locking with care
- Stale-while-revalidate patterns

## 52. Hot Key / Hot Partition
One popular key/partition receives disproportionate traffic.

Solutions:
- Better partition key design
- Key salting where appropriate
- Local caching
- Replication/read scaling
- Traffic shaping

## 53. Connection Pooling
Creating DB/network connections repeatedly is expensive.

A connection pool reuses connections.

Important settings:
- Maximum pool size
- Minimum idle
- Connection timeout
- Idle timeout
- Max lifetime

**Too large a pool can overload the database.**

## 54. Thread Pool vs Connection Pool
**Thread pool:** reusable execution threads.

**Connection pool:** reusable DB/network connections.

Both avoid repeated setup cost and bound concurrency.

## 55. Queue for Traffic Spikes
If traffic suddenly increases:

```text
Clients
  ↓
API
  ↓
Queue
  ↓
Workers
  ↓
Database / External service
```

The queue buffers bursts and lets workers process at a controlled rate.

## 56. Synchronous vs Asynchronous Trade-off
Synchronous:
- Simple request/response
- Immediate result
- Tighter coupling
- Failure propagates directly

Asynchronous:
- Decoupled
- Better burst handling
- Easier retries
- Eventual consistency
- Harder debugging and workflow tracking

## 57. Single Point of Failure (SPOF)
A component whose failure can take down the whole system.

Examples:
- One DB instance
- One gateway
- One message broker node

Remove SPOFs through redundancy/failover where justified.

## 58. Bottleneck Analysis
Typical bottlenecks:
- Database CPU/IO
- Lock contention
- Connection pool exhaustion
- Network latency
- Slow third-party API
- Hot partition
- Cache miss storm
- Queue backlog

Always identify **what saturates first**.

## 59. Graceful Degradation
When dependency fails, provide reduced functionality instead of total failure.

Examples:
- Show cached product details
- Skip non-critical recommendations
- Queue email instead of blocking checkout

## 60. Retry Pitfalls
Bad retry behavior can amplify load.

Avoid:
- Infinite retries
- Immediate synchronized retries
- Retrying permanent failures
- Retrying unsafe non-idempotent operations

Prefer bounded exponential backoff with jitter.

## 61. N+1 Problem at System Level
A service makes one call to fetch a list, then one downstream call per item.

Example:
```text
1 request + N service calls
```

Fix with:
- Batch APIs
- Aggregation
- Bulk queries
- Precomputed views/caches

## 62. Fan-out / Fan-in
### Fan-out
One request triggers multiple downstream requests.

### Fan-in
Aggregate multiple responses into one final result.

Risk: latency becomes dependent on slowest critical dependency.

Mitigations:
- Parallel calls
- Timeouts
- Partial results
- Bulkheads
- Caching

## 63. API Aggregation
Gateway/backend-for-frontend can call multiple services and combine results.

Useful when the client would otherwise need many round trips.

Trade-off: aggregator becomes more complex and can become a bottleneck.

## 64. Backend for Frontend (BFF)
Separate backend tailored for each client type.

Example:
```text
Mobile BFF → mobile-optimized APIs
Web BFF    → web-optimized APIs
```

Useful when client needs differ significantly.

## 65. WebSocket vs Polling
### Polling
Client repeatedly asks for updates.

### WebSocket
Persistent bidirectional connection.

Use WebSocket for real-time interactive updates; polling is simpler and may be sufficient for low-frequency updates.

## 66. Long Polling
Client sends request and server holds it until an event/data is available or timeout occurs.

A middle-ground between ordinary polling and WebSocket.

## 67. Search System
For large-scale search:
- Dedicated search engine/index
- Inverted indexes
- Async indexing
- Read-optimized search model

Do not force complex full-text search into the primary transactional database when scale/query requirements outgrow it.

## 68. Read-Heavy System
Typical architecture:
```text
Clients
  ↓
Load Balancer
  ↓
Stateless Services
  ↓
Cache → DB / Read Replicas
```

Focus on caching, read replicas, CDN, indexing, and horizontal scale.

## 69. Write-Heavy System
Focus on:
- Partitioning/sharding
- Batching
- Async processing
- Queueing
- Efficient indexes
- Avoiding unnecessary synchronous downstream calls

## 70. Real-Time System
Typical components:
- WebSocket/gateway
- Pub/Sub or event streaming
- Fast state store/cache
- Event-driven processing

## 71. File Upload Architecture
Prefer:
```text
Client → Object Storage (pre-signed URL)
            ↓
       Event / Queue
            ↓
      Async Processing
```

This keeps large file transfer away from application servers.

## 72. Notification System
Typical flow:
```text
Business Service
      ↓
   Event/Queue
      ↓
Notification Service
  ↙       ↓       ↘
Email    SMS      Push
```

Advantages:
- Decoupling
- Independent scaling
- Retry handling

## 73. Order System
Typical services:
```text
API Gateway
    ↓
Order Service
 ↙        ↘
Inventory  Payment
    ↓        ↓
 Database  Payment Provider
       \    /
     Event / Saga
          ↓
   Notification
```

Need:
- Idempotency
- Saga/compensation
- Outbox/event publishing
- Inventory consistency
- Payment timeout/retry handling

## 74. URL Shortener
Core idea:
```text
Short URL → ID/Key → Original URL
```

Important design points:
- Key generation
- Redirect latency
- Hot keys
- Cache
- Read/write ratio
- Analytics asynchronously

## 75. Rate Limiter System
Possible architecture:
```text
Client
 ↓
Gateway
 ↓
Rate Limiter
 ↓
Service
```

State can be stored in a distributed low-latency store such as Redis.

Important questions:
- Per-user or per-IP?
- Global or per-service?
- Burst allowed?
- What happens when limiter storage is unavailable?

## 76. Idempotency + Retry Scenario ⭐⭐⭐⭐⭐
Question: Payment request timed out. Client retries. How avoid double charge?

Answer:
1. Client sends idempotency key.
2. Payment service stores key + final result.
3. Retry with same key returns previous result.
4. Downstream provider integration must also honor idempotency where possible.

## 77. Database Failure Scenario
If primary DB goes down:
- Detect failure
- Failover to standby/replica if supported
- Keep application retries bounded
- Protect DB from retry storms
- Restore normal traffic after health checks

Discuss data-loss implications separately using RPO.

## 78. Service Failure Scenario
If Payment Service is down:
- Timeout
- Circuit breaker opens
- Do not block all threads indefinitely
- Return appropriate response or move workflow to async state
- Retry only when safe
- Reconcile later when service recovers

## 79. Third-Party API Failure
Use:
- Timeout
- Bounded retries
- Circuit breaker
- Bulkhead
- Fallback/degraded behavior
- Monitoring/alerts

## 80. Distributed Lock
Used when only one process should perform a critical distributed action.

Possible implementations use Redis/database/coordination services, but the lock must have:
- Ownership
- Expiration/lease
- Safe release
- Failure handling

Prefer database uniqueness/atomic operations over distributed locks when they can solve the requirement more simply.

## 81. Consistent Hashing
Maps keys to nodes so that adding/removing nodes moves relatively few keys.

Used in distributed caches/storage/routing systems.

🧠 **Normal hashing:** many keys can move when node count changes.

🧠 **Consistent hashing:** minimize movement.

## 82. Quorum
A distributed read/write can require responses from a subset of nodes.

Concepts often described as:
- Read quorum
- Write quorum

The exact consistency properties depend on the system/protocol.

## 83. Leader Election
When one node should coordinate work, systems may elect a leader.

Used in coordination/cluster management.

Requirements include:
- Unique leadership
- Failure detection
- Re-election
- Avoiding split-brain

## 84. Split Brain
Multiple nodes believe they are the active leader/primary at the same time.

Can cause conflicting writes or duplicate work.

Prevent with robust consensus/coordination mechanisms and fencing/epochs where appropriate.

## 85. Data Duplication
Denormalizing or replicating data can improve read performance.

Trade-off:
- Faster reads
- More storage
- More complicated updates/consistency

## 86. CQRS
Command Query Responsibility Segregation:
- Commands change state.
- Queries read state.

Can use separate read models for complex/high-scale query needs.

CQRS does **not** automatically mean separate databases or microservices.

## 87. Event-Driven Architecture
Components communicate through events.

Benefits:
- Loose coupling
- Async processing
- Independent consumers

Challenges:
- Event ordering
- Duplicate delivery
- Schema evolution
- Debugging
- Eventual consistency

## 88. At-Least-Once Delivery
A message may be delivered more than once.

Consumers should therefore be **idempotent** where possible.

Other delivery models may include at-most-once and effectively-once semantics depending on the technology and end-to-end design.

## 89. Ordering
Ordering is usually guaranteed only within a defined scope, such as a Kafka partition.

If per-user order is required, choose a stable partitioning key such as user ID where appropriate.

## 90. Data Retention
Decide how long logs/events/files should remain.

Factors:
- Compliance
- Recovery needs
- Storage cost
- Replay requirements

## 91. Observability Metrics for an API
Track:
- Request rate
- Error rate
- P50/P95/P99 latency
- Saturation
- Dependency latency
- Timeout count
- Circuit breaker state

**Golden signals:** latency, traffic, errors, saturation.

## 92. Security in HLD
Consider:
- TLS everywhere appropriate
- Authentication
- Authorization
- Secret management
- Encryption at rest
- Network segmentation
- Least privilege
- Input validation
- Rate limiting
- Audit logging

## 93. Zero Trust Principle
Do not automatically trust a request because it came from an internal network.

Authenticate and authorize services/users based on identity and policy.

## 94. HLD Trade-offs ⭐⭐⭐⭐⭐
Strong HLD answers do not say "this is always best."

Explain trade-offs:
- SQL vs NoSQL
- Sync vs async
- Strong vs eventual consistency
- Cache vs freshness
- Monolith vs microservices
- REST vs gRPC
- Read replica vs sharding
- Simplicity vs scalability

## 95. Common HLD Questions
Practice these:

1. Design URL Shortener
2. Design Rate Limiter
3. Design Notification System
4. Design URL/Link Preview Service
5. Design File Storage/Upload System
6. Design Chat System
7. Design News Feed
8. Design Ride Booking System
9. Design Food Delivery System
10. Design E-commerce Order System
11. Design Payment System
12. Design Search Autocomplete
13. Design Logging System
14. Design Metrics/Monitoring System
15. Design Ticket Booking System
16. Design Distributed Cache
17. Design Job Scheduler
18. Design Video Streaming Platform
19. Design Social Media Feed
20. Design API Gateway

## 96. Interview Scenario: Sudden 10x Traffic ⭐⭐⭐⭐⭐
Answer framework:
1. Load balancer distributes traffic.
2. Auto scale stateless app instances.
3. Cache hot/read-heavy data.
4. Rate limit abusive traffic.
5. Queue non-critical async work.
6. Protect DB with connection limits and replicas.
7. Apply circuit breakers/timeouts to dependencies.
8. Monitor saturation and error rates.
9. Degrade non-critical features.

## 97. Interview Scenario: DB is Bottleneck
Think in this order:

```text
Query optimization
      ↓
Indexes
      ↓
Caching
      ↓
Read replicas
      ↓
Partitioning / Sharding
```

Choose the least-complex solution that meets the required scale.

## 98. Interview Scenario: One Microservice is Slow
Check:
- Downstream latency
- DB queries
- Thread pool saturation
- Connection pool exhaustion
- CPU/memory
- External API calls
- GC/resource pressure

Then use tracing + metrics to isolate the bottleneck.

## 99. Interview Scenario: Queue is Growing
Possible causes:
- Producers faster than consumers
- Consumer failures
- Downstream dependency slow
- Too few workers

Actions:
- Scale consumers
- Increase processing efficiency
- Fix downstream bottleneck
- Apply backpressure
- Monitor lag

## 100. 2-Minute HLD Revision
> **“In an HLD interview, I first clarify functional and non-functional requirements, then estimate traffic and storage. I identify APIs and core data, design the major components, choose the database based on access patterns and consistency needs, and then add caching, asynchronous messaging, load balancing and horizontal scaling where required. I also discuss availability, timeouts, retries, circuit breakers, idempotency, security, observability, deployment and disaster recovery. Finally, I explain the key trade-offs and likely bottlenecks.”**

# 🧠 Ultimate HLD Memory Map
```text
REQUIREMENTS
     ↓
SCALE / ESTIMATION
     ↓
API + DATA
     ↓
LOAD BALANCER
     ↓
SERVICES
  ↙    ↓    ↘
CACHE  DB   QUEUE
       ↓      ↓
   REPLICA   WORKERS
       ↓
 SHARD/PARTITION

CROSS-CUTTING
├── Security
├── Timeout
├── Retry
├── Circuit Breaker
├── Rate Limit
├── Idempotency
└── Observability

FAILURE
├── HA
├── Failover
├── Backup
├── RTO/RPO
└── Graceful Degradation
```

# 🔥 15 Must-Remember Lines
1. **Horizontal scaling = add more instances.**
2. **Stateless services are easier to scale horizontally.**
3. **Cache reduces latency and DB load but creates invalidation/consistency concerns.**
4. **Queue absorbs traffic spikes and decouples producers from consumers.**
5. **Kafka partitions provide parallelism and ordering within a partition.**
6. **Timeout prevents indefinite waiting.**
7. **Retry should be bounded, backoff-based and safe for the operation.**
8. **Circuit breaker prevents repeated calls to a failing dependency.**
9. **Bulkhead isolates resources.**
10. **Idempotency prevents duplicate business effects during retries.**
11. **Outbox helps reliably publish DB-backed events.**
12. **Saga coordinates distributed business transactions using local transactions and compensation.**
13. **RTO = recovery time; RPO = acceptable data loss.**
14. **Logs tell what happened, metrics quantify it, traces show the request journey.**
15. **Good HLD is about trade-offs, not one universally correct technology.**
