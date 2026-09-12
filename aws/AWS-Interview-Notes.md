# AWS — Interview Notes

## 1. AWS Fundamentals

### What is AWS?
AWS (Amazon Web Services) is a cloud platform that provides on-demand infrastructure and managed services such as compute, storage, databases, networking, security, messaging, and monitoring.

### Region vs Availability Zone
- **Region**: Geographic area containing multiple isolated Availability Zones.
- **Availability Zone (AZ)**: Isolated data-center location(s) within a Region.
- For high availability, deploy critical components across multiple AZs.

### Scalability vs Elasticity
- **Scalability**: Ability to handle increasing/decreasing load by adding/removing resources.
- **Elasticity**: Ability to automatically adjust resources with changing demand.

**Memory:** Scale = capacity; Elastic = automatically adjust.

---

# 2. EC2 — Compute

### What is EC2?
EC2 provides virtual servers in AWS.

Key concepts:
- AMI
- Instance type
- EBS
- Security Group
- Key pair
- Elastic IP
- User data
- IAM role

### Security Group vs NACL
| Security Group | NACL |
|---|---|
| Instance/ENI level | Subnet level |
| Stateful | Stateless |
| Allow rules | Allow and deny rules |
| Return traffic automatically allowed | Return traffic must be explicitly allowed |

**Memory:** SG = Stateful, NACL = Stateless.

---

# 3. S3 ⭐⭐⭐⭐⭐

### What is Amazon S3?
S3 is highly durable object storage used to store files/objects such as documents, images, backups, logs, and static assets.

### S3 terminology
- Bucket → container for objects
- Object → stored data + metadata
- Key → object name/path

### Common S3 use cases
- File/document storage
- Backup and archive
- Static website assets
- Data lake
- Application uploads
- Logs

### S3 storage classes
Common classes include:
- S3 Standard
- S3 Intelligent-Tiering
- S3 Standard-IA
- S3 One Zone-IA
- S3 Glacier Instant Retrieval
- S3 Glacier Flexible Retrieval
- S3 Glacier Deep Archive

Choose based on access frequency, retrieval requirements, and cost.

### S3 versioning
Keeps multiple versions of an object and helps recover from accidental overwrite/delete.

### S3 lifecycle
Automatically transitions objects between storage classes or expires old objects based on rules.

### S3 encryption
Common options:
- SSE-S3
- SSE-KMS
- Client-side encryption

### S3 access control
Prefer IAM/resource policies and least privilege. Bucket policies can grant cross-account access. Object Ownership and modern S3 controls should be preferred over legacy ACL-driven designs unless ACLs are specifically required.

### Pre-signed URL
Provides time-limited access to a specific S3 object without making the bucket/object public.

**Interview line:**
> "For application uploads, I prefer private S3 objects and use a pre-signed URL when temporary direct client access is required."

---

# 4. RDS ⭐⭐⭐⭐⭐

### What is Amazon RDS?
Managed relational database service supporting engines such as PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, and Aurora.

AWS manages operational tasks such as provisioning, backups, patching, and monitoring depending on configuration.

### RDS Multi-AZ
Primarily provides high availability/failover by maintaining a standby in another AZ for supported configurations.

**Important:** Multi-AZ is not primarily a read-scaling mechanism.

### Read Replica
Used primarily for read scaling and read-heavy workloads. Replication is asynchronous in common RDS read-replica configurations.

### Multi-AZ vs Read Replica
| Multi-AZ | Read Replica |
|---|---|
| High availability/failover | Read scaling |
| Standby/secondary | Readable replica |
| Primarily DR/HA | Primarily performance scaling |

---

# 5. DynamoDB ⭐⭐⭐⭐⭐

### What is DynamoDB?
A fully managed NoSQL key-value/document database designed for low-latency access at scale.

### Primary Key
Two common choices:
1. Partition key only
2. Partition key + sort key (composite primary key)

### Partition Key
Determines item distribution across partitions and is critical for even traffic distribution.

### Sort Key
Allows related items to be organized and queried within the same partition key.

### GSI — Global Secondary Index
Allows queries using a different key schema from the base table.

**Interview example:**
If frequent queries use `email` but the table primary key is `userId`, create a GSI with `email` as its partition key when the access pattern justifies it.

### LSI — Local Secondary Index
Uses the same partition key as the base table with a different sort key. It must be defined when the table is created.

### DynamoDB capacity
- On-demand
- Provisioned capacity

### Eventual vs strong consistency
DynamoDB supports eventually consistent reads and, for supported operations, strongly consistent reads.

### DynamoDB Transactions
Useful when an atomic transaction is needed across multiple items/tables within DynamoDB transaction constraints.

### DynamoDB Streams
Captures item-level changes and can trigger downstream processing, commonly with Lambda.

---

# 6. RDS vs DynamoDB ⭐⭐⭐⭐⭐

| RDS | DynamoDB |
|---|---|
| Relational | NoSQL key-value/document |
| SQL queries | Access-pattern/key-based queries |
| Joins supported by engine | No relational joins like RDBMS |
| Schema-oriented | Flexible item structure |
| Transactions supported | Transactions supported with DynamoDB-specific limits |
| Good for relational business data | Good for massive low-latency key-based workloads |

**Interview line:**
> "I choose RDS when relational modeling, joins and SQL are important, and DynamoDB when predictable key-based access and horizontal scale are the primary requirements."

---

# 7. AWS IAM ⭐⭐⭐⭐⭐

### IAM
Controls who can access which AWS resources and what actions they can perform.

Main concepts:
- User
- Group
- Role
- Policy

### IAM Role
Preferred for workloads such as EC2, ECS, EKS, Lambda and other AWS services because applications can obtain temporary credentials instead of hardcoding access keys.

### Least Privilege
Grant only the minimum permissions required.

**Memory:** Who + What + Which Resource.

---

# 8. Secrets Management

### Secrets Manager
Used to store and retrieve secrets such as database passwords, API keys, and credentials, with features for rotation and controlled access.

### Parameter Store
Stores configuration and parameters; SecureString can be used for sensitive values.

**Interview line:**
> "I avoid hardcoding credentials. I use IAM roles for AWS access and a managed secret store such as Secrets Manager for application secrets."

---

# 9. VPC & Networking ⭐⭐⭐⭐

### VPC
Logical isolated network in AWS.

Key components:
- Subnets
- Route tables
- Internet Gateway
- NAT Gateway
- Security Groups
- Network ACLs

### Public vs Private Subnet
A subnet is commonly considered public when its route table has a route to an Internet Gateway. A private subnet does not have a direct route to the Internet Gateway.

### NAT Gateway
Allows resources in private subnets to make outbound connections to the internet without allowing unsolicited inbound internet connections to those resources.

### Internet Gateway
Provides connectivity between a VPC and the internet for resources/routes configured appropriately.

---

# 10. Load Balancer ⭐⭐⭐⭐

AWS Elastic Load Balancing distributes traffic across targets.

Common types:
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- Gateway Load Balancer (GWLB)

### ALB
Layer 7 HTTP/HTTPS routing.

Supports routing based on:
- Host
- Path
- Headers and other request attributes depending on configuration

### NLB
Designed for high-performance Layer 4 TCP/UDP/TLS traffic.

**Memory:**
ALB = Application/HTTP
NLB = Network/TCP/UDP

---

# 11. Auto Scaling ⭐⭐⭐⭐

Auto Scaling adjusts compute capacity based on demand/policies.

Typical flow:
```text
Traffic increases
→ Metrics/Policy trigger
→ New instances/tasks launched
→ Load balancer distributes traffic
```

For EC2, an Auto Scaling Group maintains desired/min/max capacity and replaces unhealthy instances when configured.

---

# 12. CloudWatch ⭐⭐⭐⭐

Used for monitoring and observability.

### Includes
- Metrics
- Logs
- Alarms
- Dashboards
- Events/automation capabilities

Example:
> CPU crosses threshold → CloudWatch alarm → scaling or notification action.

---

# 13. CloudTrail

Records AWS API activity for governance, auditing, and security investigations.

### CloudWatch vs CloudTrail
- **CloudWatch** → operational monitoring/metrics/logs
- **CloudTrail** → API/account activity auditing

**Memory:**
Watch the system = CloudWatch
Track API actions = CloudTrail

---

# 14. Route 53

AWS managed DNS service.

Capabilities:
- DNS records
- Domain routing
- Health checks
- Routing policies

Common routing policies:
- Simple
- Weighted
- Latency-based
- Failover
- Geolocation
- Geoproximity

---

# 15. API Gateway ⭐⭐⭐⭐

Managed API entry point for applications.

Common uses:
- API exposure
- Authentication/authorization integration
- Throttling
- Request/response transformations
- Routing
- Monitoring

In microservices, API Gateway can provide a single client-facing endpoint and route requests to backend services.

---

# 16. Lambda ⭐⭐⭐⭐

Serverless compute that runs code in response to events without managing servers.

Common triggers:
- API Gateway
- S3 events
- SQS
- EventBridge
- DynamoDB Streams

Key concerns:
- Cold starts
- Execution limits
- Stateless execution model
- Concurrency

---

# 17. ECS / EKS

### ECS
AWS container orchestration service.

### EKS
Managed Kubernetes control plane.

**Interview line:**
> "I use containers when consistent packaging and deployment are important. ECS is AWS-native container orchestration, while EKS is managed Kubernetes."

---

# 18. ECR

Amazon Elastic Container Registry stores container images for AWS workloads such as ECS and EKS.

Typical flow:
```text
Docker build
→ Push image to ECR
→ ECS/EKS pulls image
→ Deploy
```

---

# 19. SQS ⭐⭐⭐⭐⭐

Managed message queue used for asynchronous decoupling between components.

Benefits:
- Buffering
- Decoupling
- Retry handling
- Load smoothing

### Standard Queue
High throughput; at-least-once delivery semantics.

### FIFO Queue
Supports ordering and deduplication features within FIFO constraints.

### Visibility Timeout
When a consumer receives a message, it becomes temporarily invisible to other consumers. If the consumer does not delete it before visibility timeout expires, it can become visible again.

### Dead Letter Queue
Stores messages that could not be successfully processed after configured receive attempts.

**Memory:**
SQS = Queue
DLQ = Failed messages
Visibility Timeout = Temporary hiding during processing

---

# 20. SNS ⭐⭐⭐⭐

Pub/sub messaging service.

```text
Publisher
   ↓
 SNS Topic
 ↙  ↓  ↘
SQS Lambda HTTP
```

### SNS vs SQS
- SNS → publish to multiple subscribers
- SQS → queue for consumers

They are often used together.

---

# 21. EventBridge

Event bus service for event-driven architectures.

Useful for routing events from AWS services, SaaS applications and custom applications to targets based on rules.

---

# 22. Kinesis

Designed for streaming data ingestion and real-time processing.

Use cases:
- Logs
- Clickstreams
- IoT events
- Real-time analytics

**SQS = queue-based messaging**
**Kinesis = streaming data platform**

---

# 23. AWS Code / CI-CD

Common services:
- CodeCommit (where supported)
- CodeBuild
- CodeDeploy
- CodePipeline

Typical CI/CD flow:
```text
Commit
→ Build
→ Test
→ Package/Image
→ Deploy
→ Monitor
```

---

# 24. High Availability Design ⭐⭐⭐⭐⭐

Typical production design:
```text
Route 53
   ↓
Load Balancer
   ↓
Multiple EC2/ECS instances across AZs
   ↓
Application
   ↓
RDS Multi-AZ / managed database
   +
S3 for object storage
   +
ElastiCache for caching
   +
SQS/SNS for async processing
   +
CloudWatch/CloudTrail for observability/auditing
```

Principles:
- Multiple AZs
- Health checks
- Auto Scaling
- Load balancing
- Backups
- Least privilege
- Encryption
- Monitoring/alerting

---

# 25. Caching — ElastiCache ⭐⭐⭐⭐

Managed in-memory caching service using technologies such as Redis and Memcached.

Common use cases:
- Frequently requested data
- Session/state where appropriate
- Reducing database load
- Low-latency reads

Typical flow:
```text
Request
 ↓
Cache lookup
 ↓ hit          ↓ miss
Return        DB query
                ↓
              Cache
```

**Memory:** Cache = Fast read + less DB load.

---

# 26. KMS ⭐⭐⭐⭐

AWS Key Management Service manages cryptographic keys used to protect data.

Common uses:
- S3 encryption with SSE-KMS
- RDS encryption
- EBS encryption
- Application encryption workflows

---

# 27. Encryption

### At Rest
Data stored on disk/object/database is encrypted.

Examples:
- S3 SSE
- EBS encryption
- RDS encryption

### In Transit
Data moving over the network is protected using TLS/HTTPS.

**Memory:**
At rest = stored data
In transit = moving data

---

# 28. How does Spring Boot connect to an AWS database?

Typical approach:
1. Configure DB endpoint/port/database name.
2. Configure datasource/driver.
3. Store credentials securely.
4. Give application runtime the required network access.
5. Use connection pooling.
6. Use JPA/Hibernate or JDBC.

Example:
```properties
spring.datasource.url=jdbc:postgresql://<rds-endpoint>:5432/app
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
```

Production considerations:
- Secrets Manager/Parameter Store
- Private subnets
- Security groups
- TLS where appropriate
- Connection pool sizing

---

# 29. Common AWS Scenario Questions

### Q: Application suddenly receives very high traffic. What will you do?
> "I would put the service behind a load balancer, use Auto Scaling, add caching for hot data, protect the database with proper connection-pool limits, and use asynchronous processing with SQS where appropriate. I would also monitor CloudWatch metrics and define alarms."

### Q: Files need to be uploaded by users. Where will you store them?
> "I would store files in private S3 buckets rather than in the application server or database. For temporary client access, I would generate pre-signed URLs."

### Q: Need to query DynamoDB by an attribute that is not the primary key.
> "If the access pattern is important and recurring, I would create a GSI with that attribute as the index key rather than scanning the whole table."

### Q: Need asynchronous communication between microservices.
> "I would use SQS when I need queue-based decoupling and controlled consumption, or SNS/EventBridge when I need event fan-out/routing."

### Q: How do you secure AWS credentials in an application?
> "I avoid hardcoded credentials. I use IAM roles for AWS resources and managed secret storage for application secrets, following least privilege."

---

# 30. AWS Interview Rapid Revision

**S3** → Object storage

**RDS** → Managed relational DB

**DynamoDB** → Managed NoSQL key-value/document DB

**EC2** → Virtual server

**ALB** → Layer 7 HTTP routing

**NLB** → Layer 4 high-performance networking

**Auto Scaling** → Adjust compute capacity

**IAM** → Identity and access control

**VPC** → Isolated network

**NAT Gateway** → Private subnet outbound internet access

**CloudWatch** → Metrics/logs/alarms

**CloudTrail** → AWS API auditing

**Route 53** → DNS/routing

**API Gateway** → Managed API front door

**Lambda** → Serverless compute

**SQS** → Queue

**SNS** → Pub/sub

**EventBridge** → Event bus/routing

**KMS** → Key management

**Secrets Manager** → Secrets

**ECR** → Container image registry

**ECS** → AWS container orchestration

**EKS** → Managed Kubernetes

**ElastiCache** → In-memory cache

---

# 31. Top AWS Interview Questions

1. Which AWS services are you using in your project?
2. What is S3?
3. What are S3 use cases?
4. How do you secure an S3 bucket?
5. What is a pre-signed URL?
6. What is RDS?
7. RDS Multi-AZ vs Read Replica?
8. RDS vs DynamoDB?
9. What is a DynamoDB partition key?
10. GSI vs LSI?
11. How do you query DynamoDB using a non-key attribute?
12. How does Spring Boot connect to an AWS-hosted database?
13. What is IAM role and why is it preferred over hardcoded credentials?
14. Security Group vs NACL?
15. Public vs private subnet?
16. What is NAT Gateway?
17. ALB vs NLB?
18. How does Auto Scaling work?
19. What is CloudWatch?
20. CloudWatch vs CloudTrail?
21. SQS vs SNS?
22. What is a DLQ?
23. What is visibility timeout?
24. When would you use Lambda?
25. What is API Gateway?
26. How would you design a highly available AWS application?
27. How would you handle a sudden traffic spike?
28. How do you manage secrets?
29. Encryption at rest vs in transit?
30. How do you monitor a production application on AWS?

---

# 32. 2-Minute AWS Interview Answer

> "In a typical Spring Boot microservices environment, I would use services based on the requirement. S3 is used for object/file storage, RDS for relational data and DynamoDB for key-based NoSQL workloads. Applications can run on EC2 or container platforms such as ECS/EKS, with a load balancer and Auto Scaling for availability and traffic management. IAM roles provide AWS access without hardcoding credentials. For asynchronous communication, SQS/SNS or EventBridge can be used. CloudWatch handles operational monitoring, while CloudTrail provides API-level auditing. For security, I use private networking where possible, least-privilege IAM, encryption at rest/in transit, and managed secret storage."

# 33. Memory Map

```text
AWS
│
├── Compute
│   ├── EC2
│   ├── ECS
│   ├── EKS
│   └── Lambda
│
├── Storage
│   ├── S3
│   └── EBS
│
├── Database
│   ├── RDS
│   └── DynamoDB
│
├── Network
│   ├── VPC
│   ├── ALB/NLB
│   ├── Route 53
│   └── NAT Gateway
│
├── Messaging
│   ├── SQS
│   ├── SNS
│   └── EventBridge
│
├── Security
│   ├── IAM
│   ├── KMS
│   └── Secrets Manager
│
└── Observability
    ├── CloudWatch
    └── CloudTrail
```
