# Database — Interview Notes

## 1. Database Fundamentals

### What is a Database?
A database is an organized collection of data that can be stored, managed, queried, and updated efficiently.

### DBMS vs RDBMS
- **DBMS:** General database management system.
- **RDBMS:** Stores data in tables and supports relationships using keys and typically SQL.

**Memory:** RDBMS = Tables + Relationships + SQL.

### Table / Row / Column
- Table = collection of related records.
- Row = one record.
- Column = one attribute.

---

## 2. Keys ⭐⭐⭐

### Primary Key
Uniquely identifies each row.
- Unique
- Not null
- One primary-key constraint per table (can contain multiple columns as a composite key)

### Foreign Key
References a key in another table and enforces referential integrity.

### Candidate Key
A minimal set of columns that can uniquely identify a row.

### Unique Key / Constraint
Ensures uniqueness according to database-specific NULL semantics.

### Composite Key
Key made from multiple columns.

**Memory:**
`Primary = Identity`  
`Foreign = Relationship`  
`Composite = Multiple columns`

---

## 3. Relationships

- **One-to-One:** One record ↔ one record.
- **One-to-Many:** One parent ↔ many children.
- **Many-to-Many:** Many records ↔ many records, usually through a junction table.

Example:
`Department 1 → Many Employees`

---

## 4. Normalization ⭐⭐⭐⭐⭐

Normalization reduces redundancy and update anomalies by organizing data into related tables.

### 1NF
Atomic values; no repeating groups.

### 2NF
1NF + no partial dependency on part of a composite key.

### 3NF
2NF + no transitive dependency of non-key attributes on the key.

### BCNF
Every determinant is a candidate key.

**Memory:**
`1NF = Atomic`  
`2NF = No Partial Dependency`  
`3NF = No Transitive Dependency`

### Denormalization
Intentionally introduces redundancy to improve read performance or simplify queries where appropriate.

**Trade-off:** Faster reads can mean more storage and more complex writes/consistency management.

---

## 5. SQL Command Categories ⭐⭐⭐

### DDL
Defines database objects.
- CREATE
- ALTER
- DROP
- TRUNCATE

### DML
Changes data.
- INSERT
- UPDATE
- DELETE

### DQL
- SELECT

### DCL
- GRANT
- REVOKE

### TCL
- COMMIT
- ROLLBACK
- SAVEPOINT

---

## 6. DELETE vs TRUNCATE vs DROP ⭐⭐⭐⭐

### DELETE
- Removes selected rows.
- Supports `WHERE`.
- Row-level operation semantics.

### TRUNCATE
- Removes all rows from a table.
- Usually faster than deleting all rows row-by-row.
- Transaction/rollback behavior and identity-reset behavior depend on the database.

### DROP
- Removes the table/object itself.

**Memory:**
`DELETE = rows`  
`TRUNCATE = all rows`  
`DROP = table`

---

## 7. Constraints ⭐⭐⭐

Common constraints:
- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- NOT NULL
- CHECK
- DEFAULT

Purpose: enforce data integrity at the database layer.

---

## 8. JOINs ⭐⭐⭐⭐⭐

### INNER JOIN
Returns matching rows from both sides.

### LEFT JOIN
Returns all rows from left table + matching rows from right; unmatched right side becomes NULL.

### RIGHT JOIN
Returns all rows from right table + matching rows from left.

### FULL OUTER JOIN
Returns matched rows plus unmatched rows from both sides where supported.

### CROSS JOIN
Cartesian product.

### SELF JOIN
A table joined to itself.

**Memory:**
`INNER = common`  
`LEFT = all left`  
`RIGHT = all right`  
`FULL = everything`

---

## 9. WHERE vs HAVING ⭐⭐⭐⭐

### WHERE
Filters rows **before** grouping/aggregation.

### HAVING
Filters groups **after** `GROUP BY` aggregation.

Example:
```sql
SELECT department_id, COUNT(*)
FROM employee
WHERE salary > 50000
GROUP BY department_id
HAVING COUNT(*) > 5;
```

**Memory:**
`WHERE = rows`  
`HAVING = groups`

---

## 10. GROUP BY / ORDER BY

### GROUP BY
Groups rows for aggregation.

### ORDER BY
Sorts the result.

Can use:
```sql
ORDER BY salary DESC;
```

---

## 11. Aggregate Functions ⭐⭐⭐

- COUNT()
- SUM()
- AVG()
- MIN()
- MAX()

Example:
```sql
SELECT MAX(salary) FROM employee;
```

---

## 12. Subquery vs JOIN

### Subquery
Query nested inside another query.

### JOIN
Combines data from related tables directly.

Use the form that is clearer and performs well for the database/query plan; neither is universally faster.

---

## 13. EXISTS vs IN

### EXISTS
Checks whether at least one matching row exists.

```sql
WHERE EXISTS (
    SELECT 1
    FROM employee e
    WHERE e.department_id = d.id
)
```

### IN
Checks membership in a set of values.

Performance depends on data, indexes, optimizer, and query shape.

---

## 14. Indexing ⭐⭐⭐⭐⭐

An index is a data structure that helps the database locate rows more efficiently for certain queries.

### Benefits
- Faster lookups
- Can help joins
- Can help sorting/grouping depending on the query and index design

### Cost
- Extra storage
- Slower inserts/updates/deletes because indexes may need maintenance

**Memory:**
`Index = Faster reads, extra write/storage cost`

### Clustered vs Non-Clustered
The exact implementation is database-specific. Conceptually:
- **Clustered/primary storage index:** table data is organized around the index key in systems that support this model.
- **Non-clustered/secondary index:** separate index structure points to rows/data.

Do not assume every database implements these terms identically.

---

## 15. Composite Index ⭐⭐⭐⭐

Example:
```sql
CREATE INDEX idx_emp_dept_salary
ON employee(department_id, salary);
```

Column order matters. A composite index is generally most useful when predicates/orderings align with its leading columns.

**Memory:** `Index order matters.`

---

## 16. Covering Index

An index that contains all columns needed by a query, potentially allowing the database to answer it without fetching the base table rows, depending on the optimizer/database.

---

## 17. Query Performance / EXPLAIN ⭐⭐⭐⭐⭐

Use the database's execution-plan tools such as:
```sql
EXPLAIN ...
```

Look for:
- Full table scans
- Index usage
- Join strategy
- Estimated/actual row counts where available
- Sort/hash operations
- Expensive operators

**Interview line:**
> "I first inspect the execution plan, then verify indexes, predicates, joins, cardinality, and the amount of data being scanned."

---

## 18. Transactions ⭐⭐⭐⭐⭐

A transaction is a unit of work that should satisfy the database transaction guarantees.

### ACID
- **Atomicity:** all or nothing.
- **Consistency:** transaction moves data between valid states according to constraints/rules.
- **Isolation:** concurrent transactions are controlled so their intermediate effects don't violate the chosen isolation semantics.
- **Durability:** committed changes survive failures according to the database's durability guarantees.

🧠 **Memory:** `A-C-I-D`

---

## 19. Transaction Isolation Levels ⭐⭐⭐⭐⭐

Common SQL standard levels:

### Read Uncommitted
May allow dirty reads.

### Read Committed
Prevents dirty reads; other anomalies may remain.

### Repeatable Read
Provides stronger repeat-read guarantees; exact behavior for phantoms is DB-specific.

### Serializable
Strongest standard isolation; transactions behave as if serialized, typically with lower concurrency.

### Common anomalies
- Dirty Read
- Non-Repeatable Read
- Phantom Read

**Memory:**
`RU → RC → RR → Serializable`

---

## 20. Deadlock ⭐⭐⭐⭐⭐

A deadlock happens when transactions/threads wait indefinitely for resources held by each other.

Example:
```text
Transaction 1 → locks A → waits for B
Transaction 2 → locks B → waits for A
```

### Prevention/Reduction
- Consistent lock ordering
- Keep transactions short
- Access resources in predictable order
- Retry transactions safely after deadlock detection where appropriate
- Proper indexes to reduce lock scope/duration where relevant

---

## 21. Optimistic vs Pessimistic Locking ⭐⭐⭐⭐

### Optimistic
Assumes conflicts are uncommon.
Often uses a version column.

```text
Read → work → UPDATE ... WHERE version = oldVersion
```

### Pessimistic
Locks data to prevent conflicting concurrent updates.

**Memory:**
`Optimistic = check conflict`  
`Pessimistic = lock first`

---

## 22. MVCC

**Multi-Version Concurrency Control** maintains multiple row versions so readers and writers can often operate with reduced blocking, depending on the database and isolation level.

---

## 23. Stored Procedure

A stored procedure is executable database-side logic stored in the database.

### Advantages
- Centralized DB-side logic
- Can reduce network round trips
- Can encapsulate operations

### Disadvantages
- Database-specific code
- Can make application/database versioning more complex
- Testing/deployment may require extra coordination

---

## 24. View

A view is a stored query presented as a virtual table in common relational databases.

Benefits:
- Abstraction
- Simplified queries
- Controlled exposure of columns/rows in suitable designs

### Materialized View
Stores query results physically and needs refresh/maintenance. Availability and refresh behavior depend on the DB.

---

## 25. SQL Injection ⭐⭐⭐⭐⭐

SQL injection occurs when untrusted input changes the structure/meaning of a SQL statement.

### Prevention
Use parameterized queries / prepared statements.

```java
PreparedStatement ps =
    connection.prepareStatement(
        "SELECT * FROM users WHERE id = ?"
    );
ps.setInt(1, userId);
```

Avoid constructing SQL by concatenating untrusted input.

**Memory:** `Parameterized Query = Primary defense`

---

## 26. NULL Handling ⭐⭐⭐⭐

`NULL` means absence/unknown value, not zero or empty string.

Use:
```sql
IS NULL
IS NOT NULL
```

Not:
```sql
= NULL
```

For example:
```sql
WHERE manager_id IS NULL
```

---

## 27. COALESCE

Returns the first non-null expression.

```sql
SELECT COALESCE(phone, 'N/A')
FROM employee;
```

---

## 28. UNION vs UNION ALL

### UNION
Combines results and removes duplicates.

### UNION ALL
Combines results without duplicate elimination and is often faster when deduplication is unnecessary.

🧠 `UNION = unique`  
`UNION ALL = all`

---

## 29. DISTINCT

Removes duplicate result rows according to the selected expressions.

```sql
SELECT DISTINCT department_id
FROM employee;
```

---

## 30. Pagination ⭐⭐⭐⭐

Typical SQL pagination can use `LIMIT/OFFSET` where supported:

```sql
SELECT *
FROM employee
ORDER BY id
LIMIT 20 OFFSET 40;
```

For very large datasets, **keyset/seek pagination** can be more efficient:

```sql
WHERE id > :lastSeenId
ORDER BY id
LIMIT 20;
```

**Memory:**
`Offset = simple`  
`Keyset = scalable for deep pages`

---

## 31. N+1 Query Problem ⭐⭐⭐⭐⭐

Application first loads N parent rows, then executes another query for each parent, causing N+1 total queries.

### Fixes
- Fetch join
- Batch fetching
- Entity graphs / appropriate ORM fetch plans
- Carefully designed queries

Avoid blindly making everything eager; fetch only what the use case needs.

---

## 32. Connection Pool ⭐⭐⭐⭐⭐

A connection pool keeps reusable database connections available to the application.

Benefits:
- Avoid connection creation overhead
- Limit concurrent DB connections
- Improve throughput

Common Java/Spring Boot pool:
**HikariCP**

Important settings include:
- Maximum pool size
- Connection timeout
- Idle timeout
- Maximum lifetime

**Memory:** `Pool = Reuse connections`

---

## 33. Database Replication

Replication keeps copies of data on multiple database nodes.

Common pattern:
- Primary/writer handles writes
- Replica/read nodes serve reads

Benefits:
- Read scalability
- High availability depending on architecture

Trade-off:
- Replication lag
- Failover complexity

---

## 34. Sharding ⭐⭐⭐⭐

Sharding distributes data across multiple database partitions/nodes.

Example:
```text
Customer 1-1M  → Shard A
Customer 1M-2M → Shard B
```

Benefits:
- Horizontal scaling
- Larger data capacity

Challenges:
- Cross-shard queries
- Rebalancing
- Distributed transactions
- Hot shards

---

## 35. Partitioning

Partitioning divides a large table into smaller logical/physical partitions within a database system.

Common strategies:
- Range
- List
- Hash

Benefits can include partition pruning and easier maintenance, depending on database/query design.

---

## 36. CAP vs ACID ⭐⭐⭐⭐

### CAP
For a distributed data system, during a network partition you cannot simultaneously guarantee both perfect consistency and availability in the CAP sense.

CAP:
- Consistency
- Availability
- Partition tolerance

### ACID
Transaction properties:
- Atomicity
- Consistency
- Isolation
- Durability

**Do not confuse them:**
`CAP = distributed system trade-off`  
`ACID = transaction properties`

---

## 37. SQL vs NoSQL ⭐⭐⭐⭐⭐

### SQL / Relational
- Structured schema
- Strong relational model
- SQL
- Powerful joins and transactions

Examples:
- PostgreSQL
- MySQL
- Oracle
- SQL Server

### NoSQL
Category includes key-value, document, wide-column and graph databases.

Often chosen for:
- Flexible data models
- Very high scale or specific access patterns
- Distributed workloads where relational joins are not central

**Interview answer:**
> "I choose based on access patterns, consistency requirements, transaction boundaries, scale, relationship complexity, and operational needs—not simply because one technology is newer."

---

## 38. Relational vs Document Database

Relational:
```text
Employee
Department
```
with relationships.

Document:
```json
{
  "employeeId": 1,
  "name": "John",
  "department": "IT"
}
```

Document models can embed data when it matches access patterns.

---

## 39. ORM / JPA / Hibernate ⭐⭐⭐⭐⭐

### ORM
Maps objects in application code to relational database data.

### JPA
Java specification/API for persistence and ORM concepts.

### Hibernate
A popular JPA implementation and ORM framework.

**Memory:**
`JPA = Specification`  
`Hibernate = Implementation`

---

## 40. Entity Lifecycle (JPA)

Common states:
- New / Transient
- Managed / Persistent
- Detached
- Removed

A managed entity is tracked by the persistence context.

---

## 41. Persistence Context ⭐⭐⭐⭐

Persistence context is a set of managed entity instances associated with an EntityManager/session context.

Responsibilities include:
- First-level identity map/cache semantics
- Dirty checking
- Managing entity state transitions

---

## 42. Dirty Checking ⭐⭐⭐⭐

ORM detects changes made to managed entities and can generate SQL during flush/transaction commit.

```java
employee.setSalary(70000);
```

No explicit update call is necessarily required for a managed entity within an active persistence context.

---

## 43. Lazy vs Eager Loading ⭐⭐⭐⭐⭐

### Lazy
Association/data is loaded when accessed according to the ORM mapping/fetch plan.

### Eager
Association/data is fetched more immediately according to mapping/fetch strategy.

**Interview point:**
Don't blindly use EAGER to solve lazy-loading issues; design fetches around use cases.

---

## 44. First-Level vs Second-Level Cache

### First-Level Cache
Associated with a persistence context/session.
Usually mandatory in JPA implementations.

### Second-Level Cache
Shared across sessions within the persistence unit/application integration when enabled and supported.

**Memory:**
`L1 = Persistence Context`  
`L2 = Shared cache`

---

## 45. JPA `save()` / persist / merge — Interview Caution

Exact behavior depends on the framework implementation and entity state.

Conceptually:
- `persist()` makes a new entity managed.
- `merge()` copies state into a managed instance and returns that managed instance.

A repository `save()` method can choose persist/merge semantics based on entity state/ID strategy and framework implementation.

---

## 46. Common Spring Database Annotations

```java
@Entity
@Table
@Id
@GeneratedValue
@Column
@OneToMany
@ManyToOne
@OneToOne
@ManyToMany
@JoinColumn
@Transactional
```

---

## 47. `@Transactional` ⭐⭐⭐⭐⭐

Defines a transaction boundary in Spring-managed transactional code.

Important interview topics:
- Propagation
- Isolation
- Rollback rules
- Read-only transactions
- Proxy-based behavior / self-invocation limitation in common Spring proxy setups

---

## 48. Propagation

Common propagation modes:

- `REQUIRED` → use current transaction or create one.
- `REQUIRES_NEW` → suspend current transaction and create a new one.
- `SUPPORTS` → use current if one exists.
- `MANDATORY` → require existing transaction.
- `NOT_SUPPORTED` → run non-transactionally, suspending an existing transaction.
- `NEVER` → fail if transaction exists.
- `NESTED` → nested savepoint-style behavior where supported/configured.

🧠 Most important: **REQUIRED vs REQUIRES_NEW**

---

## 49. Read-Only Transaction

```java
@Transactional(readOnly = true)
```

Signals that the transaction is intended for reads. ORM/database optimizations may be possible, but it is not a universal guarantee that writes are impossible.

---

## 50. Database Connection Issues in Production ⭐⭐⭐⭐⭐

Symptoms:
- Connection pool exhausted
- Slow queries
- Timeouts
- Lock contention
- Too many open connections

Debug flow:
```text
Check application logs
→ Check connection pool metrics
→ Check slow queries
→ Check DB CPU/memory/connections
→ Inspect execution plans
→ Inspect locks/deadlocks
```

---

# 🔥 Frequently Asked SQL Interview Queries

## 51. Second Highest Salary

```sql
SELECT MAX(salary)
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

Alternative with ranking:
```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employee
) t
WHERE rnk = 2;
```

If duplicate salaries exist, clarify whether you want the second **distinct** salary or second row.

---

## 52. Find Duplicate Records

```sql
SELECT email, COUNT(*)
FROM employee
GROUP BY email
HAVING COUNT(*) > 1;
```

🧠 `GROUP BY + HAVING COUNT(*) > 1`

---

## 53. Employees With Highest Salary Per Department

```sql
SELECT *
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (
               PARTITION BY department_id
               ORDER BY salary DESC
           ) AS rnk
    FROM employee e
) t
WHERE rnk = 1;
```

---

## 54. Top N Salaries

```sql
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
FETCH FIRST 3 ROWS ONLY;
```

Syntax varies by database. Use `LIMIT`, `FETCH FIRST`, or equivalent supported syntax.

---

## 55. Count Employees Per Department

```sql
SELECT department_id, COUNT(*)
FROM employee
GROUP BY department_id;
```

---

## 56. Find Employees Without Department

```sql
SELECT e.*
FROM employee e
LEFT JOIN department d
    ON e.department_id = d.id
WHERE d.id IS NULL;
```

---

## 57. Delete Duplicate Rows

Approach depends on the database. With a window function where supported:

```sql
DELETE FROM employee
WHERE id IN (
    SELECT id
    FROM (
        SELECT id,
               ROW_NUMBER() OVER (
                   PARTITION BY email
                   ORDER BY id
               ) AS rn
        FROM employee
    ) t
    WHERE rn > 1
);
```

Always test carefully and retain the row-selection rule that you intend to keep.

---

# 🎯 Database Rapid-Fire Interview Answers

**Primary key?**  
> Uniquely identifies a row and cannot be null.

**Foreign key?**  
> Maintains a relationship/reference between tables and helps enforce referential integrity.

**Normalization?**  
> Organizing data to reduce redundancy and update anomalies.

**Index?**  
> A data structure that can speed up reads for suitable queries at the cost of storage and write overhead.

**ACID?**  
> Atomicity, Consistency, Isolation and Durability.

**Deadlock?**  
> Two or more transactions wait indefinitely for resources held by each other.

**WHERE vs HAVING?**  
> WHERE filters rows before grouping; HAVING filters groups after aggregation.

**DELETE vs TRUNCATE vs DROP?**  
> DELETE removes rows, TRUNCATE removes all rows, DROP removes the object itself.

**INNER vs LEFT JOIN?**  
> INNER returns matches; LEFT returns every left-side row plus matches.

**Optimistic vs Pessimistic locking?**  
> Optimistic detects conflicts; pessimistic locks to prevent conflicting access.

**SQL injection prevention?**  
> Use parameterized queries/prepared statements and never concatenate untrusted input into SQL.

**N+1 problem?**  
> One query loads parents and then one query per parent loads related data, causing excessive queries.

**CAP vs ACID?**  
> CAP describes distributed-system trade-offs under partition; ACID describes transaction properties.

**SQL vs NoSQL?**  
> Choose based on relationships, consistency, access patterns, scale, and operational requirements.

---

# 🧠 2-Minute Revision Map

```text
DATABASE
│
├── Basics
│   ├── Table / Row / Column
│   └── DBMS / RDBMS
│
├── Keys
│   ├── Primary
│   ├── Foreign
│   ├── Candidate
│   └── Composite
│
├── Design
│   ├── Normalization
│   ├── Denormalization
│   └── Relationships
│
├── SQL
│   ├── DDL / DML / DQL
│   ├── JOIN
│   ├── GROUP BY
│   ├── WHERE / HAVING
│   └── Subquery
│
├── Performance
│   ├── Index
│   ├── Composite Index
│   ├── EXPLAIN
│   ├── Pagination
│   └── Connection Pool
│
├── Transactions
│   ├── ACID
│   ├── Isolation
│   ├── Deadlock
│   ├── Optimistic Lock
│   └── Pessimistic Lock
│
├── Distributed DB
│   ├── Replication
│   ├── Sharding
│   ├── Partitioning
│   └── CAP
│
└── Java/Spring
    ├── JPA
    ├── Hibernate
    ├── Persistence Context
    ├── Dirty Checking
    ├── N+1
    └── @Transactional
```

## ⭐ Highest Priority Before Interview

1. Joins
2. Indexing + execution plans
3. Normalization
4. Transactions + ACID
5. Isolation levels + deadlocks
6. SQL query problems
7. N+1 + Hibernate/JPA basics
8. Connection pooling
9. Pagination
10. SQL injection
11. Optimistic vs pessimistic locking
12. SQL vs NoSQL + CAP
