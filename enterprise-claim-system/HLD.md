# HLD — Enterprise Claim Management System

## 1. Requirements

### Functional
- Create claim
- Validate policy/coverage
- Upload documents
- Submit claim
- Review/assess claim
- Approve/reject claim
- Initiate payment
- Notify customer
- Track claim history
- Audit all important state changes

### Non-functional
- High availability
- Horizontal scalability
- Secure APIs
- Idempotent critical operations
- Eventual consistency between independent services
- Observability
- Auditability
- Fault isolation

## 2. Service Boundaries

```text
Gateway
  |
  +-- Claim Service ------ Claim DB
  +-- Policy Service ----- Policy DB
  +-- Assessment Service - Assessment DB
  +-- Payment Service ---- Payment DB
  +-- Document Service --- Object Storage + metadata DB
  +-- Notification ------- provider integration
  +-- Audit -------------- Audit DB / event store
```

Use database-per-service to reduce coupling.

## 3. Synchronous vs Asynchronous

Synchronous REST:
- Create claim
- Get claim
- Policy lookup when immediate validation is required

Asynchronous Kafka:
- ClaimSubmitted
- ClaimApproved
- ClaimRejected
- PaymentRequested
- PaymentCompleted
- NotificationRequested

## 4. Reliability

- Timeouts for downstream REST calls
- Retry only transient failures
- Circuit breaker for unstable dependencies
- Idempotency key for create/payment operations
- Kafka retries + DLT for poison messages
- Transactional Outbox for DB + event consistency

## 5. Scaling

- Stateless services
- Load balancer/API Gateway
- Horizontal replicas
- Kafka partitions for event parallelism
- Redis for suitable read-heavy cache use cases
- DB indexes and read replicas where justified
- Object storage for large documents instead of DB blobs

## 6. Security

```text
Client -> Gateway -> JWT/OAuth2 validation -> Service authorization
```

- TLS in transit
- Secrets manager
- RBAC
- Service-to-service authentication
- Input validation
- Audit trail

## 7. Observability

- Structured JSON logs
- Correlation ID / trace ID
- Metrics: latency, error rate, throughput, Kafka lag
- Distributed tracing
- Alerts for payment failures, claim backlog and dependency failures

## 8. Failure Scenarios

### Payment service unavailable
Claim remains APPROVED/PAYMENT_PENDING. Event can be retried without losing the claim.

### Kafka consumer crashes
Another consumer can take ownership after rebalance and resume from committed offset.

### Duplicate payment event
Use claim/payment idempotency key and a unique business transaction identifier.

### DB succeeds but Kafka publish fails
Use Transactional Outbox.

## 9. Capacity Estimation Template

During an interview ask for:
- claims/day
- peak requests/sec
- average claim/document size
- retention period
- read/write ratio
- availability target

Then estimate:
```text
Peak RPS ≈ average RPS × peak factor
Storage ≈ events/day × average event size × retention
```

Do not invent business numbers unless the interviewer provides them; state assumptions explicitly.
