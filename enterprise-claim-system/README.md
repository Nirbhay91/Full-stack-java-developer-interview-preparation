# Enterprise Claim Management System — Interview Project

## 1. Project Overview

An enterprise insurance-claim platform built with Java, Spring Boot, REST APIs, PostgreSQL, Kafka and AWS-oriented deployment patterns.

### Business flow
```text
Customer / Agent
      |
      v
 API Gateway
      |
      +--------------------+
      |                    |
      v                    v
 Claim Service        Policy Service
      |
      +----------+---------+
                 |
          Kafka / Events
       +---------+----------+
       |         |          |
       v         v          v
 Assessment  Payment   Notification
 Service     Service      Service
       |         |
       +-----> PostgreSQL

Object Storage -> claim documents/images
Redis          -> frequently used policy/claim data
Observability  -> logs + metrics + tracing
```

## 2. Services

- API Gateway — routing, authentication boundary, rate limiting.
- Claim Service — create/update/submit/track claims.
- Policy Service — policy validation and coverage checks.
- Assessment Service — eligibility/risk assessment and claim review workflow.
- Payment Service — approved claim payment workflow.
- Document Service — upload/metadata for supporting documents.
- Notification Service — email/SMS/event notifications.
- Audit Service — immutable business audit trail.

## 3. Core Claim Lifecycle

```text
DRAFT
  -> SUBMITTED
  -> VALIDATING
  -> UNDER_REVIEW
  -> APPROVED / REJECTED
  -> PAYMENT_PENDING
  -> PAID
  -> CLOSED
```

Invalid transitions are rejected by business rules.

## 4. Main REST APIs

### Claims
```http
POST   /api/v1/claims
GET    /api/v1/claims/{claimId}
GET    /api/v1/claims?page=0&size=20&status=SUBMITTED
PUT    /api/v1/claims/{claimId}
POST   /api/v1/claims/{claimId}/submit
POST   /api/v1/claims/{claimId}/approve
POST   /api/v1/claims/{claimId}/reject
GET    /api/v1/claims/{claimId}/history
```

### Policy
```http
GET /api/v1/policies/{policyId}
GET /api/v1/policies/{policyId}/coverage
```

### Documents
```http
POST /api/v1/claims/{claimId}/documents
GET  /api/v1/claims/{claimId}/documents
```

### Payment
```http
POST /api/v1/claims/{claimId}/payment
GET  /api/v1/claims/{claimId}/payment
```

## 5. Example Create Claim Request

```json
{
  "policyId": "POL-10021",
  "customerId": "CUS-20012",
  "claimType": "ACCIDENT",
  "incidentDate": "2026-08-10",
  "claimedAmount": 125000,
  "description": "Vehicle accident"
}
```

## 6. Response

```json
{
  "claimId": "CLM-90001",
  "status": "DRAFT",
  "policyId": "POL-10021",
  "claimedAmount": 125000,
  "createdAt": "2026-08-10T10:15:30Z"
}
```

## 7. Technology Stack

- Java 17/21 concepts; Java 8 compatibility concepts for interviews
- Spring Boot
- Spring Web / REST
- Spring Data JPA / Hibernate
- PostgreSQL
- Kafka
- Redis
- Maven
- JUnit 5 / Mockito
- Docker
- AWS: ECS/EKS, RDS, S3, MSK, CloudWatch depending on deployment
- Git + CI/CD

## 8. Interview Positioning

Present this as an enterprise microservices project where your contribution is clearly separated from the overall system.

A safe 5-year developer positioning is:

> "I worked primarily on the Claim Service and its integrations with Policy, Assessment, Payment and Notification services. I was involved end-to-end from requirement analysis and API design to implementation, database design, Kafka integration, testing, debugging, code review and production support."

Do not claim ownership of infrastructure or architecture decisions you did not actually make. Use "I implemented/contributed to" for team-level work and "I designed" only for components you can explain deeply.
