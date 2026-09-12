# Hibernate — Interview Notes

## 1. What is Hibernate?
Hibernate is a Java ORM framework that maps Java objects to relational database tables and reduces boilerplate JDBC code.

**Memory:** ORM = Object ↔ Relational Table.

## 2. What is ORM?
Object Relational Mapping maps:
- Class → Table
- Object → Row
- Field → Column

## 3. Hibernate vs JDBC
**JDBC:** SQL, connection management and result-set mapping are mostly handled manually.

**Hibernate:** provides ORM, entity mapping, persistence context, dirty checking, caching and query abstractions.

## 4. JPA vs Hibernate ⭐
JPA is a Java persistence specification/standard. Hibernate is an implementation of JPA and also provides Hibernate-specific features.

**Interview line:**
> JPA defines the contract; Hibernate provides the implementation.

## 5. Entity
An entity is a persistent Java class mapped to a database table.

```java
@Entity
@Table(name = "employee")
public class Employee {
    @Id
    private Long id;
}
```

## 6. @Entity
Marks a class as a JPA entity.

An entity normally needs:
- An `@Id`
- A no-argument constructor (at least protected/package-private is commonly sufficient)
- Persistent fields/properties

## 7. @Table
Specifies table mapping.

```java
@Table(name = "employees")
```

## 8. @Id ⭐
Marks the primary key.

```java
@Id
private Long id;
```

## 9. @GeneratedValue
Defines primary-key generation strategy.

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Common strategies:
- `AUTO`
- `IDENTITY`
- `SEQUENCE`
- `TABLE`

## 10. Persistence Context ⭐⭐⭐
The persistence context is a set of managed entity instances associated with an EntityManager/Session.

It acts like a first-level cache and tracks entity state changes.

**Memory:** Persistence Context = EntityManager/Session managed object space.

## 11. Entity States ⭐⭐⭐
Main states:
- **Transient:** new object, not associated with persistence context.
- **Managed/Persistent:** associated with persistence context.
- **Detached:** previously managed, but no longer associated.
- **Removed:** scheduled for deletion.

**Memory:** T → M → D / R.

## 12. persist() vs merge() ⭐⭐⭐
`persist(entity)` makes a new entity managed.

`merge(entity)` copies state from the supplied detached/transient entity into a managed instance and returns the managed instance.

**Important:** `merge()` does not simply attach the same object instance.

## 13. remove()
Marks a managed entity for deletion.

```java
entityManager.remove(employee);
```

## 14. detach()
Removes an entity from the persistence context.

```java
entityManager.detach(employee);
```

## 15. clear()
Detaches all managed entities from the persistence context.

```java
entityManager.clear();
```

## 16. refresh()
Reloads the entity state from the database, discarding in-memory changes for that entity.

## 17. First-Level Cache ⭐⭐⭐
The first-level cache is associated with the persistence context and is enabled by default.

Within the same persistence context, repeated lookup of the same entity identity can return the managed instance without another SQL query.

**Memory:** L1 = Session/EntityManager scope.

## 18. Second-Level Cache ⭐⭐
Second-level cache is shared across persistence contexts for a SessionFactory/EntityManagerFactory and is optional.

Used to reduce database access for suitable entities/queries.

Examples of providers include Ehcache and Infinispan depending on project setup.

**Memory:** L1 = one persistence context; L2 = shared across contexts.

## 19. Query Cache
Hibernate can cache query result information when query caching is explicitly configured. It is different from entity second-level caching and must be used carefully because result sets can become invalidated.

## 20. Dirty Checking ⭐⭐⭐
Hibernate automatically detects changes made to managed entities and generates SQL during flush when needed.

```java
Employee e = entityManager.find(Employee.class, 1L);
e.setSalary(70000);
// update can be generated during flush/transaction commit
```

**Memory:** Managed entity changed → Hibernate detects → SQL update.

## 21. Flush ⭐⭐⭐
Flush synchronizes pending changes in the persistence context with the database.

Flush is not the same as commit.

```text
flush  → synchronize SQL changes
commit → complete transaction
```

## 22. Flush Modes
Common JPA flush modes:
- `AUTO`
- `COMMIT`

Hibernate also has additional flush options/APIs depending on configuration.

## 23. Lazy vs Eager Loading ⭐⭐⭐⭐⭐
**LAZY:** relationship/data is loaded when accessed, subject to mapping and provider behavior.

**EAGER:** data is fetched eagerly according to the mapping/provider strategy.

**Interview preference:** prefer LAZY for associations unless there is a clear reason otherwise.

## 24. LazyInitializationException ⭐⭐⭐⭐
Occurs when Hibernate tries to initialize a lazy association after the persistence context/session needed for initialization is no longer available.

Typical fix is to fetch required data within the transaction, use an appropriate fetch query/entity graph, or map to a DTO before the context closes.

**Avoid:** solving everything by making all relationships EAGER.

## 25. N+1 Query Problem ⭐⭐⭐⭐⭐
One query loads parent rows, then an additional query is executed for each parent while accessing an association.

Example pattern:

```text
1 query for departments
+ N queries for employees
= N+1
```

Solutions include:
- Fetch join
- Entity graph
- Batch fetching
- DTO projection
- Carefully designed queries

## 26. JOIN FETCH ⭐⭐⭐⭐⭐
Used in JPQL to fetch an association in the same query.

```java
@Query("select d from Department d join fetch d.employees")
List<Department> findDepartments();
```

Helps avoid N+1 in suitable scenarios.

## 27. EntityGraph
Defines which associations should be fetched for a query without hard-coding all fetch behavior into the mapping.

```java
@EntityGraph(attributePaths = {"employees"})
List<Department> findAll();
```

## 28. @OneToOne
One entity associated with one other entity.

```java
@OneToOne
private Passport passport;
```

## 29. @OneToMany
One entity associated with many entities.

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

## 30. @ManyToOne ⭐⭐⭐
Many entities reference one parent.

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

## 31. @ManyToMany
Many entities relate to many entities, typically using a join table.

```java
@ManyToMany
@JoinTable(name = "employee_project")
private Set<Project> projects;
```

Use with care because many-to-many mappings can become difficult to manage at scale.

## 32. mappedBy ⭐⭐⭐
Indicates the inverse/non-owning side of a bidirectional relationship.

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

The field `department` on `Employee` owns the relationship.

**Memory:** `mappedBy` = "relationship is mapped by this field on the other side".

## 33. Owning Side ⭐⭐⭐
The owning side controls the relationship mapping and typically contains the foreign-key mapping (`@JoinColumn`) in a bidirectional association.

## 34. Cascade ⭐⭐⭐⭐
Cascade propagates certain entity lifecycle operations from parent to related child entities.

Common options:
- `PERSIST`
- `MERGE`
- `REMOVE`
- `REFRESH`
- `DETACH`
- `ALL`

**Important:** Cascade is not the same as database `ON DELETE CASCADE`.

## 35. orphanRemoval ⭐⭐⭐
When enabled on a supported relationship, removing a child from the relationship can cause the child entity to be deleted when the persistence context is synchronized.

```java
@OneToMany(orphanRemoval = true)
private List<OrderItem> items;
```

## 36. Cascade vs orphanRemoval
**Cascade:** propagates lifecycle operations.

**orphanRemoval:** removes child entities that become orphans through the relationship.

## 37. @JoinColumn
Specifies the foreign-key column used for an association.

```java
@JoinColumn(name = "department_id")
```

## 38. FetchType
For associations, JPA defines `LAZY` and `EAGER` fetch modes.

Typical defaults to remember:
- `@ManyToOne` / `@OneToOne` → EAGER by JPA default
- `@OneToMany` / `@ManyToMany` → LAZY by JPA default

**Interview advice:** explicitly choosing LAZY for many associations is often safer for performance, depending on the use case.

## 39. JPQL ⭐⭐⭐⭐
JPQL queries entities and their fields rather than directly querying tables/columns.

```java
@Query("select e from Employee e where e.salary > :salary")
List<Employee> findHighSalary(@Param("salary") double salary);
```

**Memory:** JPQL = Object-oriented query language.

## 40. Native Query
Runs database-specific SQL.

```java
@Query(value = "select * from employees where salary > ?1", nativeQuery = true)
List<Employee> findHighSalary(double salary);
```

Use when database-specific SQL/features are necessary.

## 41. JPQL vs Native SQL
**JPQL:** portable, entity-oriented.

**Native:** database-specific, direct SQL.

## 42. Criteria API
Programmatic API for constructing type-safe/dynamic queries. It can be useful when query predicates are built dynamically, although it can be more verbose than JPQL or repository query methods.

## 43. Hibernate Query Language (HQL)
Hibernate's object-oriented query language. HQL is closely related to JPQL; HQL includes Hibernate-specific extensions beyond the JPA standard.

## 44. Pagination ⭐⭐⭐
Spring Data commonly uses `Pageable` / `Page<T>`.

```java
Page<Employee> page = repository.findAll(PageRequest.of(0, 20));
```

Database-level pagination avoids loading the full dataset into memory.

## 45. Sorting
```java
Pageable pageable = PageRequest.of(
    0, 20, Sort.by("salary").descending());
```

## 46. Projection
Projection fetches only required fields rather than the complete entity.

Useful for read-heavy use cases and DTO-oriented APIs.

## 47. DTO vs Entity in REST APIs ⭐⭐⭐⭐
Avoid exposing JPA entities directly as your API contract in many production systems.

DTOs help:
- Control exposed fields
- Avoid lazy-loading surprises
- Prevent accidental persistence coupling
- Shape responses for specific use cases

## 48. @Version / Optimistic Locking ⭐⭐⭐⭐⭐
Optimistic locking uses a version field to detect concurrent updates.

```java
@Version
private Long version;
```

If another transaction changes the row first, an optimistic locking conflict can occur instead of silently overwriting the change.

**Memory:** Optimistic = check version before update.

## 49. Pessimistic Locking
Database locking is used to prevent concurrent modifications/access according to the selected lock mode.

Examples:
- `PESSIMISTIC_READ`
- `PESSIMISTIC_WRITE`

Use when conflicts are expected and database-level locking is appropriate.

## 50. Optimistic vs Pessimistic Locking
**Optimistic:** assume conflicts are rare; detect conflict using version/checks.

**Pessimistic:** acquire database lock and restrict concurrent access.

## 51. Transaction ⭐⭐⭐⭐⭐
A transaction groups operations into a unit of work with atomicity and consistency guarantees as provided by the underlying transaction system/database.

In Spring:

```java
@Transactional
public void transfer() {
    // debit
    // credit
}
```

## 52. Transaction Propagation
Common Spring propagation modes:
- `REQUIRED`
- `REQUIRES_NEW`
- `SUPPORTS`
- `MANDATORY`
- `NOT_SUPPORTED`
- `NEVER`
- `NESTED`

**Most important:**
`REQUIRED` joins an existing transaction or creates one if none exists.

`REQUIRES_NEW` suspends the current transaction and starts a new one.

## 53. Transaction Isolation
Common levels:
- READ_UNCOMMITTED
- READ_COMMITTED
- REPEATABLE_READ
- SERIALIZABLE

Isolation affects visibility/concurrency anomalies and depends on database support.

## 54. OptimisticLockException
Can occur when an optimistic locking check detects that the entity was changed concurrently.

## 55. EntityManager vs Hibernate Session
`EntityManager` is the standard JPA API.

`Session` is Hibernate's native API extending beyond the JPA abstraction.

**Interview line:**
> Use EntityManager when you want the standard JPA abstraction; use Session when you specifically need Hibernate APIs/features.

## 56. SessionFactory
Hibernate's heavyweight, thread-safe factory for creating Sessions. Usually created once per application/database configuration.

## 57. Session
Represents a unit of interaction with the database and contains a persistence context.

A Hibernate `Session` is not intended to be shared across threads.

## 58. EntityManagerFactory
JPA factory for EntityManagers. It is heavyweight and typically application-scoped.

## 59. First-Level Cache vs Second-Level Cache
**L1:** per persistence context; mandatory in JPA/Hibernate behavior.

**L2:** shared across persistence contexts; optional.

## 60. Save vs Persist vs SaveOrUpdate
Hibernate native APIs historically expose `save()` / `saveOrUpdate()`; JPA uses `persist()` / `merge()` with different semantics.

For modern Spring Data JPA code, understand JPA semantics first.

## 61. getReferenceById / getReference
Can return a lazy reference/proxy instead of immediately loading the full entity, subject to provider behavior and transaction/context requirements.

Useful when only an entity reference is needed, for example to set a foreign-key association.

## 62. find() vs getReference()
`find()` obtains the entity state and returns the managed entity if found.

`getReference()` may return a proxy/reference and delay database access until needed; behavior around missing rows is observed when the reference is accessed.

## 63. Proxy ⭐⭐⭐
Hibernate can use runtime-generated proxy objects for lazy loading and change interception.

A lazy association/entity may be represented by a proxy until its state is accessed.

## 64. Fetch Join vs EntityGraph
Both can solve specific N+1/read-optimization problems.

**Fetch join:** query explicitly controls the fetch.

**EntityGraph:** fetch plan can be specified separately and reused with repository methods.

## 65. Batch Fetching
Hibernate can fetch multiple lazy associations/entities together, reducing round trips in suitable scenarios.

Relevant settings/annotations include batch size configuration such as `@BatchSize`.

## 66. @BatchSize
Can instruct Hibernate to batch-load collections or entities.

```java
@BatchSize(size = 50)
```

## 67. JDBC Batching ⭐⭐⭐
Hibernate can batch multiple INSERT/UPDATE/DELETE statements to reduce database round trips.

Configuration commonly involves JDBC batch size and appropriate identifier strategy.

**Important:** batching effectiveness can depend on database/driver and ID generation strategy.

## 68. open-in-view (OSIV)
Open Session in View can keep the persistence context available through web request processing, which can mask lazy-loading problems but can also cause unexpected database access during response rendering.

**Interview line:**
> OSIV can reduce LazyInitializationException symptoms, but it should not be used as a substitute for designing transactional data fetching correctly.

## 69. equals() and hashCode() for Entities ⭐⭐⭐⭐
Be careful when implementing equality for JPA entities because generated IDs may not exist for transient objects and proxies may be involved.

The strategy should be consistent and safe for the entity lifecycle.

## 70. Embeddable ⭐⭐⭐
`@Embeddable` defines a value object whose fields are stored as part of an owning entity's table.

```java
@Embeddable
class Address {
    private String city;
}
```

Use with:

```java
@Embedded
private Address address;
```

## 71. @MappedSuperclass
Provides mapped fields to entity subclasses but is not itself an entity/table in the same sense as `@Entity`.

Useful for common audit fields.

```java
@MappedSuperclass
class BaseEntity {
    private LocalDateTime createdAt;
}
```

## 72. Inheritance Mapping ⭐⭐⭐
JPA inheritance strategies:
- `SINGLE_TABLE`
- `JOINED`
- `TABLE_PER_CLASS`

## 73. Entity Lifecycle Callbacks
Common callbacks:
- `@PrePersist`
- `@PostPersist`
- `@PreUpdate`
- `@PostUpdate`
- `@PreRemove`
- `@PostRemove`
- `@PostLoad`

## 74. Auditing
Spring Data JPA can populate fields such as created-by, created-date, modified-by and modified-date using auditing support.

Typical annotations:
```java
@CreatedDate
@LastModifiedDate
@CreatedBy
@LastModifiedBy
```

## 75. Named Queries
Queries defined with names and associated with entities/repository configuration.

Useful when you want reusable query definitions, though many modern Spring Data applications favor repository methods and `@Query`.

## 76. Specification
Spring Data JPA Specifications support dynamic, composable predicates using the Criteria API.

Useful for search screens with optional filters.

## 77. Common Hibernate Performance Problems ⭐⭐⭐⭐⭐
Remember:
- N+1 queries
- Loading too much data
- EAGER relationships everywhere
- Missing indexes
- No pagination
- Excessive entity fetching for read-only APIs
- Unnecessary flushes
- Large persistence contexts
- Inefficient joins
- Missing batching

## 78. Read-only Transactions
For read operations, Spring transactions can be marked `readOnly = true` as a performance/intent hint depending on the stack.

```java
@Transactional(readOnly = true)
public List<Employee> getEmployees() {
    return repository.findAll();
}
```

## 79. Hibernate in Spring Boot
Spring Boot typically auto-configures the JPA/Hibernate stack based on dependencies and datasource configuration.

Typical setup:
```properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
spring.jpa.hibernate.ddl-auto=validate
```

## 80. ddl-auto
Common values:
- `none`
- `validate`
- `update`
- `create`
- `create-drop`

**Production advice:** schema migration tools such as Flyway/Liquibase are generally preferred over relying on automatic schema mutation.

## 81. `save()` in Spring Data JPA
`save()` delegates to JPA persistence semantics based on whether the entity is considered new, commonly using `persist()` for new entities and `merge()` otherwise.

Do not assume every `save()` is always an SQL INSERT or always an UPDATE.

## 82. Delete Operations
Spring Data offers methods such as:
```java
delete(entity);
deleteById(id);
deleteAll();
```

Bulk delete operations may behave differently from deleting managed entities one by one, especially regarding callbacks and persistence-context synchronization.

## 83. Bulk Update/Delete ⭐⭐⭐
JPQL bulk operations operate directly on database rows and do not perform normal per-entity dirty checking lifecycle behavior.

After a bulk update/delete, the persistence context may contain stale entities; clearing/refreshing may be necessary.

## 84. Native SQL vs JPQL in Interview
Use JPQL when working with entities and needing portability.

Use native SQL when database-specific functionality, complex SQL, or vendor-specific capabilities are genuinely required.

## 85. Common Interview Scenario: N+1
**Question:** "Your API is slow because one endpoint fires 101 queries. What will you check?"

**Answer:**
> "I would first identify the N+1 pattern using SQL logs/APM. Then I would inspect the relationship fetch strategy and use fetch join, EntityGraph, batch fetching or DTO projection based on the use case. I would verify the generated SQL and measure the result rather than blindly switching everything to EAGER."

## 86. Common Interview Scenario: LazyInitializationException
> "I would keep the required data access inside an appropriate transaction and fetch the required associations using a fetch join/entity graph or map to a DTO. I would avoid simply changing the association to EAGER."

## 87. Common Interview Scenario: Concurrent Update
> "If concurrent updates are rare, I would usually consider optimistic locking with `@Version`. If the business case requires database-level locking, I would consider an appropriate pessimistic lock."

## 88. Rapid Revision — Hibernate
- **JPA = specification; Hibernate = implementation.**
- **Entity = Java class mapped to DB.**
- **Persistence Context = managed entities.**
- **L1 cache = persistence-context scoped.**
- **L2 cache = shared across persistence contexts, optional.**
- **Dirty checking = detects changes to managed entities.**
- **Flush ≠ commit.**
- **LAZY = load when needed; EAGER = eager fetch according to mapping/provider.**
- **N+1 = 1 + N queries.**
- **Fetch join / EntityGraph / batch fetch = common N+1 solutions.**
- **mappedBy = inverse side.**
- **Cascade = lifecycle propagation.**
- **orphanRemoval = delete orphaned children under relationship semantics.**
- **@Version = optimistic locking.**
- **JPQL = entity-oriented; Native SQL = database-specific.**
- **EntityManager = JPA; Session = Hibernate-specific.**

## 89. Top Interview Questions to Practice
1. What is Hibernate and why do we use it?
2. JPA vs Hibernate?
3. Explain persistence context.
4. Explain entity lifecycle states.
5. What is dirty checking?
6. What is first-level cache?
7. What is second-level cache?
8. Lazy vs eager loading?
9. What is LazyInitializationException?
10. Explain N+1 problem and solutions.
11. What is `mappedBy`?
12. Owning side vs inverse side?
13. Cascade vs orphanRemoval?
14. `persist()` vs `merge()`?
15. `find()` vs `getReference()`?
16. JPQL vs native SQL?
17. What is fetch join?
18. What is EntityGraph?
19. What is optimistic locking?
20. Optimistic vs pessimistic locking?
21. What is `@Version`?
22. What is persistence context flush?
23. What is OSIV?
24. How do you optimize slow Hibernate queries?
25. How does Spring Data `save()` work?
26. What problems can bulk update/delete create?
27. How does Hibernate batching work?
28. What are entity states after `detach()` / `clear()`?
29. How do you avoid exposing entities directly from REST APIs?
30. How do you design Hibernate mappings for a high-traffic application?

## 🧠 Ultimate Memory Map
```text
HIBERNATE
│
├── ORM
│   ├── Entity
│   ├── Table mapping
│   └── Relationships
│
├── Persistence Context
│   ├── Managed entities
│   ├── Dirty checking
│   └── L1 Cache
│
├── Fetching
│   ├── LAZY
│   ├── EAGER
│   ├── JOIN FETCH
│   └── EntityGraph
│
├── Relationships
│   ├── OneToOne
│   ├── OneToMany
│   ├── ManyToOne
│   └── ManyToMany
│
├── Performance
│   ├── N+1
│   ├── Pagination
│   ├── Projection
│   ├── Batch Fetch
│   └── JDBC Batching
│
├── Concurrency
│   ├── @Version
│   ├── Optimistic Lock
│   └── Pessimistic Lock
│
└── Query
    ├── JPQL
    ├── HQL
    ├── Native SQL
    └── Criteria / Specification
```
