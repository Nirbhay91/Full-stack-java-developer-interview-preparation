# Enterprise Claim System — Interview Script

## Project Introduction

> I worked on an enterprise claim management platform built using Java, Spring Boot and microservices. The platform manages the claim lifecycle from submission and policy validation through assessment, approval or rejection, payment and notification.

## My 5-Year Developer Role

> My primary ownership was the Claim Service and its integrations. I worked on requirement analysis, REST API design, business validation, claim lifecycle logic, PostgreSQL database design, Spring Data JPA/Hibernate, Kafka integration, exception handling, unit/integration testing, code reviews, debugging and production support. For cross-service architecture I contributed to design discussions and integrations rather than claiming ownership of every service.

## End-to-End Flow

```text
Client -> API Gateway -> Claim Service
                         |
                         +-> Policy Service
                         |
                         +-> PostgreSQL + Outbox
                                      |
                                    Kafka
                         +------------+------------+
                         |                         |
                    Assessment                 Notification
                         |
                       Payment
```

## REST API

```text
POST /api/v1/claims
GET  /api/v1/claims/{claimId}
GET  /api/v1/claims?page=0&size=20&status=SUBMITTED
PUT  /api/v1/claims/{claimId}
POST /api/v1/claims/{claimId}/submit
POST /api/v1/claims/{claimId}/approve
POST /api/v1/claims/{claimId}/reject
```

Use DTOs, Bean Validation, HTTP status codes and `@RestControllerAdvice` for consistent errors.

## Database

```text
claim
- id PK
- claim_number UNIQUE
- policy_id
- customer_id
- claim_type
- amount
- status
- created_at
- updated_at
- version

claim_status_history
- id PK
- claim_id FK
- old_status
- new_status
- changed_by
- changed_at
- reason

outbox_event
- id PK
- event_id UNIQUE
- aggregate_id
- event_type
- payload
- status
- created_at
```

Create indexes according to query patterns, for example claim number, policy ID, customer ID and status/date combinations.

## One Million Records

> I would not return one million records from one REST request. I would use pagination, select only required columns using DTO/projection, and verify indexes with `EXPLAIN ANALYZE`. For very large offsets I would consider keyset/cursor pagination.

Example:

```java
Pageable pageable = PageRequest.of(page, size);
return claimRepository.findByStatus(status, pageable);
```

## Kafka

> REST is used when an immediate synchronous response is required. Kafka is used for asynchronous events such as claim-approved. Payment and Notification can consume the event independently.

Use event IDs and idempotent consumers to handle duplicate delivery safely.

## Transactional Outbox

> If the database update succeeds but Kafka publishing fails, a direct dual write creates inconsistency. I store the business change and outbox event in the same database transaction. A publisher then sends the event to Kafka. Consumers remain idempotent because a publisher crash can still cause a duplicate publish.

## Failure Handling

- Transient failure → bounded retry + backoff.
- Permanent invalid event → DLT/remediation.
- Unavailable downstream service → timeout + circuit breaker where appropriate.
- Duplicate event/payment → idempotency key/event ID + database uniqueness protection.
- Concurrent claim update → optimistic locking with `@Version`.

## Patterns

- Repository pattern
- DTO pattern
- Strategy pattern for claim assessment rules
- State-transition approach for claim lifecycle
- Transactional Outbox
- Circuit Breaker
- Retry with backoff
- Saga-style event workflow

## Security

> Use OAuth2/JWT authentication, role/permission-based authorization, TLS for communication and proper secret management. Do not hard-code credentials.

## Testing

- JUnit 5 + Mockito for unit tests
- MockMvc for controller tests
- `@DataJpaTest` for repository tests
- Spring Boot integration tests for end-to-end paths
- Kafka integration tests where required

## Production Debugging

### High API latency
Check traces/logs, downstream latency, slow SQL, execution plan, connection pool and application metrics before changing code.

### High Kafka lag
Check consumer processing time, partition distribution, consumer count, rebalances, downstream dependency latency and broker health. Scale useful consumer parallelism within the partition limit.

## 90-Second Closing

> My strongest ownership was backend development around the Claim Service. I can explain the REST APIs, database model, transaction boundaries, JPA mappings, Kafka integration, failure handling, scalability decisions, testing and production troubleshooting end-to-end. I clearly separate the components I implemented from team-level architecture contributions.
