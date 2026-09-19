# LLD + Database + API Design — Claim System

## 1. Layered Design

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
PostgreSQL
```

Supporting components:
- DTOs
- Mapper
- Validation
- Global Exception Handler
- Kafka Producer
- External client
- Cache abstraction

## 2. Core Domain Model

### Claim
```text
Claim
- id
- claimNumber
- policyId
- customerId
- claimType
- status
- claimedAmount
- approvedAmount
- incidentDate
- description
- version
- createdAt
- updatedAt
```

### ClaimDocument
```text
ClaimDocument
- id
- claimId
- documentType
- objectKey
- fileName
- status
- createdAt
```

### ClaimHistory
```text
ClaimHistory
- id
- claimId
- oldStatus
- newStatus
- changedBy
- reason
- changedAt
```

### Payment
```text
Payment
- id
- claimId
- transactionId
- amount
- status
- retryCount
- createdAt
- updatedAt
```

## 3. Relationships

```text
Customer 1 ---- N Claim
Policy   1 ---- N Claim
Claim    1 ---- N ClaimDocument
Claim    1 ---- N ClaimHistory
Claim    1 ---- 1 Payment
```

For microservices, these relationships are logical business relationships; do not create cross-service foreign keys.

## 4. Important Indexes

```sql
CREATE INDEX idx_claim_policy ON claim(policy_id);
CREATE INDEX idx_claim_customer ON claim(customer_id);
CREATE INDEX idx_claim_status_created ON claim(status, created_at);
CREATE UNIQUE INDEX uk_claim_number ON claim(claim_number);
CREATE UNIQUE INDEX uk_payment_transaction ON payment(transaction_id);
```

Choose indexes based on actual query patterns and verify them using `EXPLAIN ANALYZE`.

## 5. JPA Entity Sketch

```java
@Entity
@Table(name = "claim")
public class Claim {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String claimNumber;

    @Column(nullable = false)
    private String policyId;

    @Column(nullable = false)
    private String customerId;

    @Enumerated(EnumType.STRING)
    private ClaimStatus status;

    @Version
    private Long version;
}
```

`@Version` provides optimistic locking for concurrent updates to the same claim.

## 6. Repository

```java
public interface ClaimRepository extends JpaRepository<Claim, Long> {
    Page<Claim> findByStatus(ClaimStatus status, Pageable pageable);
    Optional<Claim> findByClaimNumber(String claimNumber);
}
```

## 7. Service

```java
@Service
@RequiredArgsConstructor
public class ClaimService {

    private final ClaimRepository repository;
    private final ClaimEventPublisher eventPublisher;

    @Transactional
    public ClaimResponse create(CreateClaimRequest request) {
        // validate business rules
        // persist claim
        // persist outbox event in same transaction
        // return response
        return null;
    }
}
```

## 8. Controller

```java
@RestController
@RequestMapping("/api/v1/claims")
@RequiredArgsConstructor
public class ClaimController {

    private final ClaimService service;

    @PostMapping
    public ResponseEntity<ClaimResponse> create(
            @Valid @RequestBody CreateClaimRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(service.create(request));
    }

    @GetMapping
    public Page<ClaimSummaryResponse> search(
            @RequestParam(required = false) ClaimStatus status,
            @PageableDefault(size = 20) Pageable pageable) {
        return service.search(status, pageable);
    }
}
```

## 9. Validation + Exception Handling

Use:
- Bean Validation (`@NotNull`, `@Positive`, `@Size`)
- Business validation in service layer
- `@RestControllerAdvice` for consistent error responses

Example error:
```json
{
  "code": "CLAIM_NOT_FOUND",
  "message": "Claim was not found",
  "traceId": "abc-123",
  "timestamp": "2026-08-10T10:20:00Z"
}
```

## 10. Idempotency

For critical POST operations:
```text
Idempotency-Key: 7c9e...
```

Persist the key/result or business transaction ID so a retry does not create a duplicate claim/payment.

## 11. Claim State Pattern

Keep transitions explicit:

```text
DRAFT -> SUBMITTED
SUBMITTED -> VALIDATING
VALIDATING -> UNDER_REVIEW
UNDER_REVIEW -> APPROVED
UNDER_REVIEW -> REJECTED
APPROVED -> PAYMENT_PENDING
PAYMENT_PENDING -> PAID
PAID -> CLOSED
```

Implement transition validation in a domain/service component rather than allowing arbitrary status updates.

## 12. LLD Design Patterns

### Strategy
Different claim assessment algorithms by claim type.

```java
interface ClaimAssessmentStrategy {
    AssessmentResult assess(Claim claim);
}
```

### Factory
Select the correct assessment strategy.

### Observer / Event-driven
Claim state changes publish domain events consumed by notification/audit/payment services.

### Adapter
Wrap external payment/notification providers behind internal interfaces.

### Repository
Abstract persistence access.

### Builder
Useful for complex immutable response/domain objects where appropriate.

## 13. Concurrency

Use optimistic locking with `@Version` when simultaneous updates are possible.

If two requests update the same claim:
```text
Request A version=5 -> update succeeds -> version=6
Request B version=5 -> optimistic lock failure
```

Handle this with a controlled API error/retry strategy.
