# Enterprise Claim System — End-to-End Interview Script

## 1. 90-Second Project Introduction

> "I worked on an enterprise claim management platform in the insurance domain. The system handled claim creation, policy validation, document submission, assessment, approval or rejection, payment and customer notifications. We used Spring Boot microservices with REST APIs for synchronous operations and Kafka for asynchronous event-driven communication. PostgreSQL was used for transactional data, Redis was used selectively for read-heavy cache use cases, and object storage was used for claim documents. My main responsibility was the Claim Service and its integrations. I worked end-to-end on API design, business logic, database design, JPA/Hibernate, Kafka integration, validation, exception handling, unit/integration testing, debugging, code reviews and production support."

## 2. My Roles & Responsibilities — 5 Years Level

Say:

1. Analysed requirements and converted them into technical tasks.
2. Designed REST endpoints and request/response DTOs.
3. Implemented Claim Service using Spring Boot.
4. Implemented business validations and claim state transitions.
5. Designed PostgreSQL tables, relationships and indexes for my service.
6. Used Spring Data JPA/Hibernate for persistence.
7. Implemented pagination for claim search APIs.
8. Integrated Policy Service through REST client patterns.
9. Published/consumed Kafka events for asynchronous workflows.
10. Implemented retry, idempotency and DLT handling where required.
11. Used Transactional Outbox for reliable DB-to-Kafka publication.
12. Implemented global exception handling and validation.
13. Wrote JUnit/Mockito tests and integration tests.
14. Participated in code reviews and defect fixing.
15. Investigated logs, metrics and production issues.
16. Participated in Agile ceremonies and deployment support.

Use "I contributed to" for team-owned architecture/infrastructure work unless you personally owned it.

# 3. Explain One Claim End-to-End

```text
POST /claims
   ↓
API Gateway
   ↓
Claim Controller
   ↓
Validation
   ↓
Claim Service
   ↓
Policy validation
   ↓
PostgreSQL
   ↓
Outbox event
   ↓
Kafka
   ↓
Assessment / Notification / Payment
```

### Script

> "When a customer submits a claim, the request reaches the gateway and then the Claim Service. The controller validates the request DTO and delegates to the service layer. The service validates claim and policy business rules. We persist the claim in PostgreSQL. For an event that needs asynchronous processing, we store the outbox event in the same database transaction. A publisher sends the event to Kafka. Downstream services consume it independently. This keeps the request path fast and reduces synchronous coupling."

# 4. Why Microservices?

> "We separated services around business capabilities. Claim, Policy, Assessment, Payment and Notification have different responsibilities and scaling/failure characteristics. Database-per-service reduces tight coupling. REST is used when an immediate response is required, while Kafka is used for asynchronous workflows."

# 5. Why Kafka?

> "For events such as ClaimSubmitted or PaymentRequested, Kafka provides asynchronous communication, durable event storage, replay through offsets and independent consumer groups. It also allows downstream services to scale independently."

# 6. Why REST + Kafka Together?

> "They solve different problems. REST is appropriate when the caller needs an immediate response. Kafka is appropriate when processing can be asynchronous and we want loose coupling, buffering and event-driven integration."

# 7. Why PostgreSQL?

> "Claims and payments contain transactional business data with consistency requirements and relational queries. PostgreSQL gives us ACID transactions, constraints and indexing. Large documents are better kept in object storage, with metadata in the database."

# 8. Database Design Explanation

```text
claim
  id PK
  claim_number UNIQUE
  policy_id
  customer_id
  status
  claimed_amount
  approved_amount
  incident_date
  version
  created_at
  updated_at

claim_document
  id PK
  claim_id
  document_type
  object_key
  status

claim_history
  id PK
  claim_id
  old_status
  new_status
  changed_by
  reason
  changed_at

payment
  id PK
  claim_id
  transaction_id UNIQUE
  amount
  status
```

### DB interview script

> "I index columns used frequently in filtering and joins, such as policy_id, customer_id and status plus created_at. I avoid indexing every column because indexes have write and storage costs. For a slow query I check the execution plan using EXPLAIN ANALYZE rather than assuming an index will help."

# 9. One Million Records Question

> "I would not return one million records in one REST response. I would use pagination, return only required columns using DTO/projection, add appropriate indexes and set a reasonable page size. For very large offsets, I would consider keyset/cursor pagination. If the requirement is batch processing rather than returning data to a client, I would use batch/streaming processing instead."

# 10. JPA/Hibernate Questions

### N+1 problem
> "N+1 occurs when fetching a list triggers additional queries for associated entities. I identify it from SQL logs/APM and solve it based on the use case using fetch joins, entity graphs or projections, while avoiding blindly making everything eager."

### Lazy vs Eager
> "Lazy loads the association when accessed; eager loads it immediately. I generally prefer controlled fetching and DTO projections for API use cases to avoid unnecessary data and N+1 problems."

### Optimistic locking
> "We use @Version when concurrent updates to the same claim are possible. If another transaction has already changed the row, the stale update fails instead of silently overwriting newer data."

# 11. Kafka Failure Question

> "If a consumer processes an event and crashes before committing the offset, the event may be delivered again. Therefore the business operation must be idempotent. For poison messages, I use bounded retries and a DLT."

# 12. DB + Kafka Consistency Question

> "I would use the Transactional Outbox pattern. The business update and outbox event are committed in one DB transaction. A publisher then sends the outbox event to Kafka. Because publication can still be retried, consumers should remain idempotent."

# 13. Payment Duplicate Question

> "Payment is a critical operation, so I would use an idempotency key or unique business transaction ID. The database can enforce uniqueness, and repeated requests return the existing result instead of creating another payment."

# 14. High Traffic Question

> "I would keep services stateless and scale horizontally behind a load balancer. I would use Kafka partitions for event parallelism, cache appropriate read-heavy data, optimize DB indexes and queries, and scale downstream dependencies. I would verify bottlenecks through metrics rather than scaling blindly."

# 15. Security Script

> "At the API boundary we validate authentication tokens and apply authorization based on roles/permissions. Communication uses TLS. Sensitive configuration is kept in a secrets-management solution rather than source control. Input validation, least privilege and audit logging are applied to sensitive operations."

# 16. Testing Script

> "For unit tests I isolate the service logic with Mockito. For controller tests I validate request/response and HTTP behaviour. Integration tests verify repository and service integration with the database. Kafka integration tests verify serialization and event flow where needed."

Example:
```java
@Test
void shouldRejectNegativeClaimAmount() {
    // arrange
    // act
    // assert
}
```

# 17. Production Issue Script

> "I first reproduce or identify the affected endpoint/event and correlation ID. Then I check application logs, metrics, traces, database performance and Kafka lag. I isolate whether the bottleneck is application, database, downstream dependency or messaging. After the fix I add a regression test and monitor the deployment."

# 18. HLD Walkthrough Order

When asked to design the system, follow this exact order:

```text
1. Requirements
2. Scale / assumptions
3. APIs
4. High-level components
5. Data model
6. Sync vs async communication
7. Kafka/event design
8. Caching
9. Database/indexing
10. Reliability
11. Security
12. Observability
13. Deployment/scaling
14. Failure scenarios
15. Trade-offs
```

# 19. LLD Walkthrough Order

```text
Controller
   ↓
DTO + Validation
   ↓
Service
   ↓
Domain/Strategy
   ↓
Repository
   ↓
Database

Cross-cutting:
Exception Handler
Kafka Publisher
External Client
Cache
Audit
```

# 20. Design Patterns Used

### Strategy
Assessment logic differs by claim type.

### Factory
Creates/selects the appropriate strategy.

### Adapter
Wraps external payment/notification providers.

### Repository
Encapsulates persistence operations.

### Observer/Event-driven
Downstream services react to claim events.

### State-style transition handling
Controls valid claim status transitions.

# 21. Rapid Interview Questions

**Why not one large monolith?**
> Business boundaries, independent deployment/scaling and failure isolation can justify service separation, but microservices add operational complexity.

**Why database-per-service?**
> Service ownership and loose coupling; cross-service data is accessed through APIs/events rather than direct DB joins.

**Why Redis?**
> Reduce repeated reads for suitable data where caching provides measurable benefit; define TTL and invalidation carefully.

**Why S3/object storage for documents?**
> Large binary files are better suited to object storage; DB stores metadata and references.

**How do you handle duplicate Kafka events?**
> Idempotent consumer using event/business ID and durable deduplication/unique constraint.

**How do you handle slow downstream service?**
> Timeout, circuit breaker, bounded retry/backoff and asynchronous processing where appropriate.

**How do you monitor Kafka?**
> Consumer lag, under-replicated/offline partitions, throughput, latency, producer errors and rebalances.

**How do you handle concurrent claim updates?**
> Optimistic locking with @Version where suitable, plus business conflict handling.

# 22. Final 60-Second Summary

> "My core ownership was the Claim Service and its integrations. I worked across the complete lifecycle: requirement analysis, REST API design, Spring Boot implementation, validation and exception handling, PostgreSQL/JPA design, Kafka events, idempotency and retry handling, testing, debugging and production support. At system level, the platform uses API Gateway, independent microservices, database-per-service, Kafka for asynchronous workflows, Redis for selected cache use cases and object storage for documents. The key design concerns are scalability, consistency, reliability, security and observability."

## Important Interview Rule

Never memorize only the architecture diagram. For every component you mention, be prepared to answer:

**Why? How? Failure? Scale? Trade-off? How did you test it? What exactly did you own?**
