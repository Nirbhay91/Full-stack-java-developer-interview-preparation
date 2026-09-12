# Microservices — Interview Notes

## 1. What is Microservices?
Microservices architecture splits an application into small, independently deployable services. Each service focuses on a specific business capability, can own its data, and communicates with other services through well-defined APIs or messaging.

**Memory:** Small service + single business responsibility + independent deployment.

## 2. Monolith vs Microservices
| Monolith | Microservices |
|---|---|
| One deployable application | Multiple independently deployable services |
| Usually one codebase/process | Multiple codebases/processes |
| Scaling often at application level | Services can scale independently |
| Simple initially | More distributed-system complexity |
| Failure can affect large area | Failures can be isolated with proper design |

## 3. Why use Microservices?
- Independent deployment and scaling
- Team autonomy
- Technology flexibility where justified
- Fault isolation
- Clear business boundaries

**Trade-off:** Operational, network, testing, observability, security, and data-consistency complexity increases.

## 4. How do you identify service boundaries?
Use business capabilities, domain boundaries, data ownership, change frequency, and team ownership. Avoid splitting only by technical layers such as controller/service/repository.

**Interview line:** "I prefer business-capability and domain boundaries over arbitrary technical decomposition."

## 5. Database per Service ⭐
Each service should ideally own its data and schema so that other services do not directly modify its database.

Benefits:
- Loose coupling
- Independent schema evolution
- Service autonomy

Challenge:
- Cross-service transactions and joins become harder.

## 6. How do Microservices communicate?
### Synchronous
- REST/HTTP
- gRPC

### Asynchronous
- Kafka
- RabbitMQ
- Other message brokers

**Rule of thumb:** Use synchronous calls when an immediate response is required; use asynchronous messaging for decoupling, event-driven workflows, and eventual processing.

## 7. REST communication
Service A:
```text
Order Service -> Payment Service -> /payments
```

Problems with synchronous chains:
- Network latency
- Timeout propagation
- Cascading failures
- Tight runtime dependency

## 8. Service Discovery ⭐⭐⭐
Service Discovery lets services find the current network location of other service instances dynamically.

Instead of:
```text
http://10.10.1.20:8081
```
use a logical service identity such as:
```text
payment-service
```

A registry keeps track of available instances.

**Memory:** Service name -> Registry -> Healthy instance.

## 9. Why not hard-code IP addresses?
Instances can start/stop, scale horizontally, move hosts, or change ports. Hard-coded addresses create tight coupling and make scaling/failover difficult.

## 10. Client-side vs Server-side Discovery
### Client-side
Client asks registry for instances and selects one.

### Server-side
Client calls a load balancer/gateway, which performs discovery and routing.

## 11. API Gateway ⭐⭐⭐
API Gateway is the entry point for clients into a microservices system.

Typical responsibilities:
- Routing
- Authentication/token validation
- Rate limiting
- TLS termination
- Request/response transformation
- Aggregation where appropriate
- Logging/correlation

**Memory:** One door to many services.

## 12. Why use API Gateway instead of directly calling services?
- Hide internal topology
- Centralize cross-cutting concerns
- Reduce client complexity
- Consistent security and routing
- Easier versioning and traffic control

## 13. Frontend to Microservices flow
```text
Frontend
   |
   v
API Gateway
   |
   +--> User Service
   +--> Order Service
   +--> Payment Service
```

The frontend normally does not need to know internal service addresses.

## 14. Response Aggregation
When a screen needs data from multiple services, an API gateway/BFF/aggregator can call multiple services and compose a client-friendly response.

**Example:** Dashboard = User + Orders + Recommendations.

Be careful about latency amplification and partial failures.

## 15. Load Balancing ⭐⭐⭐
Distributes requests across multiple healthy instances.

Common strategies:
- Round robin
- Weighted round robin
- Least connections
- Least response time

Load balancing can exist at gateway, service-mesh, client, or infrastructure levels.

## 16. High Traffic Handling ⭐⭐⭐
For a suddenly overloaded service:
1. Scale horizontally.
2. Put/load-balance traffic across instances.
3. Cache frequently read data.
4. Use asynchronous processing where appropriate.
5. Apply rate limiting/backpressure.
6. Protect dependencies with timeouts and circuit breakers.
7. Monitor CPU, latency, errors, and saturation.

## 17. Auto Scaling
Automatically adds/removes instances based on demand or metrics such as CPU, memory, request rate, or latency, depending on the platform.

**Memory:** Traffic up -> instances up; traffic down -> instances down.

## 18. Caching ⭐⭐
Caching reduces repeated database/network calls.

Common use cases:
- Reference/master data
- Frequently read objects
- Expensive computations

Trade-offs:
- Stale data
- Invalidation complexity
- Memory usage

## 19. Fault Tolerance ⭐⭐⭐
A microservice should fail gracefully and prevent one dependency failure from taking down the whole system.

Key patterns:
- Timeout
- Retry
- Circuit breaker
- Bulkhead
- Rate limiting
- Fallback
- Idempotency

## 20. Timeout
Never let a service wait forever for a dependency.

**Interview line:** "Every remote call should have a bounded timeout appropriate to the business operation."

## 21. Retry
Retry transient failures such as temporary network problems.

Important:
- Use limited attempts.
- Add backoff/jitter.
- Do not blindly retry non-transient failures.
- Be careful with non-idempotent operations.

## 22. Circuit Breaker ⭐⭐⭐
Circuit breaker stops repeatedly calling an unhealthy dependency.

States:
```text
CLOSED -> NORMAL calls
OPEN -> Calls blocked/fail fast
HALF_OPEN -> Limited trial calls
```

**Memory:** Too many failures -> open circuit -> recover -> half-open.

## 23. Bulkhead
Isolates resources so one overloaded dependency or workload does not consume everything.

**Example:** Separate thread pools/connection limits for critical dependencies.

## 24. Cascading Failure ⭐⭐⭐
A failure in one service causes dependent services to slow/fail, which then overloads more services.

```text
Payment slow
   -> Order waits
      -> Threads exhausted
         -> Gateway timeouts
```

## 25. How to prevent Cascading Failures?
- Timeouts
- Circuit breakers
- Bulkheads
- Rate limiting
- Bounded queues/thread pools
- Load shedding
- Backpressure
- Graceful degradation

## 26. Saga Pattern ⭐⭐⭐⭐⭐
Saga manages a business transaction spanning multiple microservices by splitting it into a sequence of local transactions.

Each local transaction commits independently. If a later step fails, compensating actions are triggered for earlier steps.

**Memory:** Local transactions + compensation.

## 27. Why Saga instead of distributed DB transaction?
Distributed two-phase commit can be expensive and tightly couples services. Saga favors local transactions and eventual consistency, which fits many microservice architectures better.

## 28. Choreography Saga
Services publish/consume events without a central orchestrator.

Example:
```text
OrderCreated
   -> PaymentProcessed
      -> InventoryReserved
         -> ShipmentCreated
```

Failure triggers compensating events.

**Risk:** Event flow can become difficult to understand as the number of participants grows.

## 29. Orchestration Saga
A central orchestrator tells participating services what action to perform and decides what compensation is needed.

**Choreography:** decentralized.

**Orchestration:** centralized workflow coordination.

## 30. Compensation Transaction
A business action that semantically reverses/compensates a previous successful action.

Example:
```text
Debit account -> later step fails -> issue refund
```

It is not necessarily a database rollback; it is a new business operation.

## 31. Data Consistency in Microservices ⭐⭐⭐
Because each service owns its database, strong cross-service ACID consistency is not always available.

Common approach:
- Local ACID transactions within each service
- Events/Saga for cross-service workflow
- Eventual consistency
- Idempotent consumers
- Reliable message publication patterns

## 32. Transactional Outbox ⭐⭐⭐
Problem: database update succeeds but event publish fails.

Solution:
1. In one local DB transaction, update business data and write an outbox event.
2. A separate publisher reads the outbox and publishes the event.
3. Mark the event as published / manage retries.

This improves reliability between database changes and event publication.

## 33. Idempotency ⭐⭐⭐
An operation is idempotent when repeating the same request does not create additional unintended effect after the first successful application.

Useful for:
- Payment APIs
- Message consumers
- Retryable requests

Example: use an idempotency key to prevent duplicate payment processing.

## 34. Eventual Consistency
Different services may temporarily have different views of data, but the system converges after successful event processing.

**Interview line:** "We accept eventual consistency across services where immediate global consistency is not required."

## 35. Message Delivery Semantics
Possible semantics include:
- At-most-once
- At-least-once
- Exactly-once semantics under specific system guarantees and constraints

Most application designs should assume duplicates can occur with at-least-once delivery and make consumers idempotent.

## 36. Authentication in Microservices ⭐⭐⭐
Typical flow:
```text
Client -> API Gateway -> Authentication/Token validation -> Service
```

JWT can carry claims such as subject, roles/scopes, and expiry. Services validate the token and enforce authorization.

## 37. Service-to-Service Authentication
Common approaches include:
- OAuth 2.0 client credentials
- mTLS
- Short-lived service identity tokens

Do not assume that internal network location alone is a security boundary.

## 38. Authorization
Authentication answers **who are you?**

Authorization answers **what are you allowed to do?**

Common models:
- RBAC
- ABAC
- Scope/permission based

## 39. Secure Microservices
Key practices:
- TLS in transit
- Strong authentication/authorization
- Secrets management
- Least privilege
- Input validation
- Secure headers
- Audit logging
- Token validation
- Network segmentation where appropriate

## 40. API Versioning
Common approaches:
```text
/v1/orders
/v2/orders
```

or headers/media types.

Prefer backward compatibility where practical and deprecate older versions deliberately.

## 41. Rate Limiting
Controls request volume from a client/user/system.

Benefits:
- Protect services
- Prevent abuse
- Preserve capacity

Common algorithms:
- Token bucket
- Leaky bucket
- Fixed/sliding windows

## 42. Observability ⭐⭐⭐
Three major pillars:
- Logs
- Metrics
- Traces

Important distributed-system concepts:
- Correlation ID / Trace ID
- Centralized logging
- Request latency
- Error rate
- Throughput
- Saturation

## 43. Distributed Tracing
Tracks one request as it travels through multiple services.

```text
Gateway trace
   -> Order span
      -> Payment span
      -> Inventory span
```

Useful for locating latency and failure points.

## 44. Health Checks
Common concepts:
- Liveness: process is alive.
- Readiness: instance is ready to receive traffic.

Traffic should generally be routed only to ready instances.

## 45. Service Resilience vs Availability
Resilience = ability to continue or recover gracefully under failures.

Availability = proportion of time a service is operational and accessible.

A design can improve resilience through isolation and graceful degradation, but no design eliminates all failures.

## 46. Event-Driven Architecture
Services communicate through events such as:
```text
OrderCreated
PaymentCompleted
InventoryReserved
```

Advantages:
- Loose temporal coupling
- Scalability
- Async processing

Challenges:
- Debugging
- Ordering
- Duplicate events
- Event schema evolution

## 47. Kafka in Microservices
Typical use:
```text
Producer Service -> Kafka Topic -> Consumer Services
```

Benefits:
- Durable event log
- Partitioned scalable processing
- Consumer groups

Interview concepts:
- Topic
- Partition
- Offset
- Consumer group
- Ordering within a partition

## 48. API Gateway vs Service Discovery
**API Gateway:** controls/routs incoming client traffic.

**Service Discovery:** helps services locate service instances.

They solve different problems and are often used together.

## 49. API Gateway vs Load Balancer
**Load balancer:** distributes traffic among instances.

**API Gateway:** provides application/API-level capabilities such as routing by path, authentication, rate limiting, transformation, and sometimes aggregation.

A gateway may use a load balancer internally.

## 50. Distributed Transaction vs Local Transaction
Local transaction:
```text
Order DB -> BEGIN -> UPDATE -> COMMIT
```

Distributed business transaction:
```text
Order -> Payment -> Inventory -> Shipping
```

The second is usually coordinated using Saga/event-driven patterns rather than one global local transaction.

## 51. Handling Partial Failure
Suppose Order succeeds but Payment times out.

Possible strategy:
- Use timeout.
- Make payment operation idempotent.
- Persist workflow state.
- Retry transient failure carefully.
- Use Saga compensation if business rules require it.
- Surface pending status rather than falsely reporting success/failure.

## 52. Graceful Degradation
Return a useful reduced experience when a non-critical dependency is unavailable.

Example:
```text
Product page works
Recommendations temporarily unavailable
```

## 53. Backpressure
Controls how quickly producers send work when consumers cannot keep up.

Useful with queues, reactive pipelines, and high-throughput systems.

## 54. Bulkhead vs Circuit Breaker
**Bulkhead:** limits blast radius/resources.

**Circuit breaker:** stops repeated calls to an unhealthy dependency.

They are complementary.

## 55. Retry vs Circuit Breaker
**Retry:** gives a transient failure another chance.

**Circuit breaker:** stops calling when failure rate is high.

Bad retries can amplify outages, so combine retries with timeouts and backoff and protect them with circuit breakers.

## 56. Microservices Testing Strategy
- Unit tests for business logic
- Integration tests for service + infrastructure interactions
- Contract tests for API compatibility
- Component/service tests
- End-to-end tests for critical flows

Avoid making every scenario depend on slow full-system tests.

## 57. Contract Testing
Verifies that a provider and consumer agree on request/response/event contracts without requiring the entire distributed system for every test.

## 58. Deployment Strategies
Common:
- Rolling deployment
- Blue-green deployment
- Canary deployment

**Canary:** release to a small percentage first, observe, then expand.

## 59. Configuration Management
Keep environment-specific configuration outside application code where possible.

Examples:
- Environment variables
- Config servers
- Secret managers

Do not commit secrets to source control.

## 60. Centralized Configuration
Common goals:
- Consistent configuration management
- Environment separation
- Controlled rollout/change

But avoid turning configuration into an unprotected single point of failure; applications should have sensible startup/failure behavior.

## 61. Common Microservices Interview Scenario
**Question:** Port of a microservice changes. How does another service find it?

**Answer:**
> "I would avoid hard-coding the IP and port. The service registers its instance with service discovery. The caller uses the logical service name, and the discovery mechanism returns a healthy instance. A load balancer or client-side load balancing then routes the request."

## 62. Common Scenario: One Service Gets Huge Traffic
> "First I would identify whether the bottleneck is CPU, DB, network, or a downstream dependency. Then I would scale horizontally, load-balance traffic, cache read-heavy data, rate-limit where required, and protect downstream systems using timeouts, circuit breakers and bulkheads. I would verify the improvement using metrics and traces."

## 63. Common Scenario: Payment Service Down
> "I would use a timeout and circuit breaker to avoid long waits and cascading failures. For a business operation that can be retried safely, I would use bounded retries with backoff. I would persist the workflow state and use an asynchronous retry or Saga compensation strategy as appropriate."

## 64. Top 20 Rapid-Fire Questions
1. What is microservices architecture?
2. Monolith vs microservices?
3. How do you define service boundaries?
4. Why database per service?
5. How do services communicate?
6. What is service discovery?
7. Client-side vs server-side discovery?
8. What is API Gateway?
9. Why API Gateway?
10. What is load balancing?
11. What is circuit breaker?
12. What are cascading failures?
13. How do you prevent cascading failures?
14. What is Saga?
15. Choreography vs orchestration?
16. What is compensation transaction?
17. What is eventual consistency?
18. What is idempotency?
19. How do you secure service-to-service communication?
20. How do you observe/debug distributed systems?

## 🧠 Final Memory Map
```text
MICROSERVICES
│
├── Design
│   ├── Business boundaries
│   ├── Database per service
│   └── Independent deployment
│
├── Communication
│   ├── REST / gRPC
│   └── Kafka / Messaging
│
├── Routing
│   ├── API Gateway
│   ├── Service Discovery
│   └── Load Balancer
│
├── Data
│   ├── Local transactions
│   ├── Saga
│   ├── Outbox
│   └── Eventual consistency
│
├── Resilience
│   ├── Timeout
│   ├── Retry
│   ├── Circuit Breaker
│   ├── Bulkhead
│   └── Rate Limit
│
├── Security
│   ├── OAuth/JWT
│   ├── mTLS
│   └── Authorization
│
├── Scale
│   ├── Load balancing
│   ├── Auto scaling
│   └── Caching
│
└── Observability
    ├── Logs
    ├── Metrics
    ├── Traces
    └── Correlation ID
```

## ⭐ 2-Minute Interview Answer
> "Microservices is an architecture where an application is decomposed into independently deployable services aligned to business capabilities. Each service generally owns its data and communicates through synchronous APIs or asynchronous messaging. For routing and discovery, I would use an API Gateway, service discovery, and load balancing. For distributed transactions, I prefer local transactions with Saga and eventual consistency rather than tightly coupling services through a global transaction. For resilience, I use timeouts, bounded retries, circuit breakers, bulkheads, rate limiting, and idempotency. For security, I use strong authentication and authorization such as OAuth/JWT or service identity mechanisms, with TLS for communication. Finally, I rely on centralized logs, metrics, distributed tracing, and correlation IDs for observability and troubleshooting."
