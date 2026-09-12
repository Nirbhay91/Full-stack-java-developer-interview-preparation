# Spring Boot — Interview Notes

## 1. What is Spring Boot?
Spring Boot is a Spring-based framework that simplifies application development by providing auto-configuration, starter dependencies, embedded servers, and production-ready features.

**Memory:** Spring Boot = Spring + Auto-Configuration + Starters + Embedded Server + Production readiness.

## 2. Why Spring Boot?
- Reduces boilerplate configuration
- Auto-configures common components
- Provides starter dependencies
- Supports embedded servers
- Easy production deployment and monitoring

## 3. Spring vs Spring Boot
| Spring | Spring Boot |
|---|---|
| General application framework | Simplifies Spring application setup |
| More explicit configuration often needed | Convention and auto-configuration |
| Server/dependency setup can be manual | Embedded server and starters simplify setup |

## 4. Spring Boot Starters
Starters provide a convenient dependency bundle for a capability.

Examples:
- `spring-boot-starter-web`
- `spring-boot-starter-data-jpa`
- `spring-boot-starter-security`
- `spring-boot-starter-test`

**Memory:** Starter = ready-made dependency bundle.

## 5. `@SpringBootApplication` ⭐⭐⭐
It is the main Boot application annotation and combines:
- `@Configuration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**Memory:** Configuration + Auto Configuration + Component Scan.

## 6. Auto-Configuration ⭐⭐⭐⭐⭐
Spring Boot attempts to configure beans based on the classpath, existing beans, and configuration properties.

Example: when web dependencies are present, Boot can configure the web application infrastructure automatically.

**Interview line:**
> Spring Boot auto-configuration reduces manual configuration by conditionally creating and configuring beans based on the application's environment.

## 7. Conditional Configuration
Boot uses conditional annotations such as:
- `@ConditionalOnClass`
- `@ConditionalOnMissingBean`
- `@ConditionalOnProperty`
- `@ConditionalOnBean`

These allow configuration to activate only when specific conditions are satisfied.

## 8. Component Scanning
`@ComponentScan` discovers Spring-managed components such as:
- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`

The default scan starts from the package of the main application class and its subpackages.

## 9. Dependency Injection ⭐⭐⭐⭐⭐
Dependency Injection means the container supplies an object's dependencies instead of the object creating them itself.

### Constructor injection — preferred
```java
@Service
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Why preferred?**
- Dependencies are explicit
- Supports immutability with `final`
- Easier testing
- Prevents partially initialized objects

## 10. `@Autowired`
Used by Spring to resolve and inject dependencies.

Constructor injection is generally preferred, and with a single constructor, explicit `@Autowired` is not required.

## 11. Bean vs Component
A **bean** is an object managed by the Spring container.

A **component** is a class detected by component scanning and registered as a bean.

## 12. `@Component`, `@Service`, `@Repository`, `@Controller`
All are stereotype annotations used to register classes as Spring beans, with semantic roles:
- `@Component` → generic component
- `@Service` → service/business layer
- `@Repository` → persistence/data-access layer
- `@Controller` → MVC controller

`@Repository` also participates in Spring's persistence exception translation mechanism.

## 13. `@RestController`
Equivalent to:
```java
@Controller
@ResponseBody
```
It is commonly used for REST APIs where return values are written directly to the HTTP response body.

## 14. `@RequestMapping`
Maps HTTP requests to controller classes or methods.

Specialized shortcuts:
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

## 15. `@PathVariable` vs `@RequestParam`
### PathVariable
```text
/users/101
```
```java
@GetMapping("/users/{id}")
public User get(@PathVariable Long id) { ... }
```

### RequestParam
```text
/users?id=101
```
```java
@GetMapping("/users")
public User get(@RequestParam Long id) { ... }
```

**Memory:** PathVariable = part of path; RequestParam = query parameter.

## 16. `@RequestBody`
Converts the HTTP request body, commonly JSON, into a Java object through HTTP message converters.

```java
@PostMapping("/users")
public User create(@RequestBody User user) {
    return service.create(user);
}
```

## 17. `ResponseEntity`
Provides control over:
- HTTP status
- Response headers
- Response body

```java
return ResponseEntity.status(HttpStatus.CREATED).body(user);
```

## 18. Exception Handling ⭐⭐⭐⭐⭐
Use `@ControllerAdvice` / `@RestControllerAdvice` for centralized exception handling.

```java
@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    ResponseEntity<String> handle(UserNotFoundException ex) {
        return ResponseEntity.status(404).body(ex.getMessage());
    }
}
```

**Memory:** ControllerAdvice = centralized REST exception handling.

## 19. Validation ⭐⭐⭐⭐
Common annotations:
- `@Valid`
- `@NotNull`
- `@NotBlank`
- `@Size`
- `@Min`
- `@Max`
- `@Email`

Example:
```java
@PostMapping("/users")
public User create(@Valid @RequestBody User user) {
    return service.create(user);
}
```

## 20. Configuration Management ⭐⭐⭐⭐
Common sources:
- `application.properties`
- `application.yml`
- environment variables
- command-line arguments
- external configuration sources

Use `@Value` for simple property injection and `@ConfigurationProperties` for grouped, type-safe configuration.

## 21. `@Value` vs `@ConfigurationProperties`
`@Value` is convenient for individual properties.

`@ConfigurationProperties` is better for structured configuration groups because it is type-safe and easier to maintain.

## 22. Profiles ⭐⭐⭐⭐
Profiles separate environment-specific configuration.

Examples:
```text
application-dev.yml
application-test.yml
application-prod.yml
```

Activate with:
```properties
spring.profiles.active=dev
```

## 23. Bean Scopes ⭐⭐⭐
Common scopes:
- `singleton` → one bean instance per Spring container
- `prototype` → new instance when requested by the container
- `request` → one per HTTP request in a web application
- `session` → one per HTTP session
- `application` → one per ServletContext

Default scope is `singleton`.

## 24. Bean Lifecycle ⭐⭐⭐⭐
Typical lifecycle:
```text
Instantiate
→ Dependency Injection
→ Initialization callbacks
→ Bean ready
→ Destruction callbacks
```

Useful annotations:
- `@PostConstruct`
- `@PreDestroy`

## 25. `@Bean` vs `@Component` ⭐⭐⭐⭐
`@Component` is placed on the class and discovered by component scanning.

`@Bean` is placed on a method inside a configuration class and explicitly registers the method's return value as a bean.

Use `@Bean` especially when configuring third-party classes that you cannot annotate yourself.

## 26. `@Configuration`
Marks a class as a source of bean definitions.

```java
@Configuration
class AppConfig {
    @Bean
    MyService myService() {
        return new MyService();
    }
}
```

## 27. Spring IoC Container
IoC means **Inversion of Control**: object creation and dependency management are handled by the framework/container instead of application classes doing it manually.

**Memory:** IoC = Container controls object lifecycle/dependencies.

## 28. Spring AOP ⭐⭐⭐⭐
Aspect-Oriented Programming separates cross-cutting concerns such as:
- Logging
- Security
- Transactions
- Auditing

Core concepts:
- Aspect
- Advice
- Pointcut
- Join point
- Proxy

## 29. Advice Types
Common advice types:
- `@Before`
- `@After`
- `@AfterReturning`
- `@AfterThrowing`
- `@Around`

`@Around` can control whether/when the target method proceeds.

## 30. Spring Transactions ⭐⭐⭐⭐⭐
Use `@Transactional` to define transaction boundaries.

```java
@Transactional
public void createOrder() {
    // multiple DB operations
}
```

Important concepts:
- Atomicity
- Consistency
- Isolation
- Durability (ACID)
- Propagation
- Isolation level
- Rollback rules

## 31. Transaction Propagation
Common propagation modes:
- `REQUIRED` → join existing transaction or create one
- `REQUIRES_NEW` → suspend existing transaction and create a new one
- `SUPPORTS`
- `MANDATORY`
- `NOT_SUPPORTED`
- `NEVER`
- `NESTED`

**Memory:** REQUIRED = join if present, otherwise create.

## 32. Isolation Levels
Common levels:
- `READ_UNCOMMITTED`
- `READ_COMMITTED`
- `REPEATABLE_READ`
- `SERIALIZABLE`

Higher isolation generally gives stronger consistency but can reduce concurrency.

## 33. Rollback
By default, Spring's declarative transaction handling commonly rolls back for unchecked exceptions (`RuntimeException` and `Error`) rather than checked exceptions.

Customize with:
```java
@Transactional(rollbackFor = Exception.class)
```

## 34. Spring Data JPA ⭐⭐⭐⭐⭐
Spring Data JPA reduces repository boilerplate.

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

Provides CRUD and query support.

## 35. `CrudRepository` vs `JpaRepository`
`CrudRepository` provides basic CRUD operations.

`JpaRepository` extends Spring Data repository capabilities and provides JPA-specific methods and paging/sorting support through its hierarchy.

## 36. Entity and Primary Key
```java
@Entity
class User {
    @Id
    @GeneratedValue
    private Long id;
}
```

`@Entity` maps the class to a database table, and `@Id` identifies the primary key.

## 37. JPA Relationships
Common mappings:
- `@OneToOne`
- `@OneToMany`
- `@ManyToOne`
- `@ManyToMany`

Consider ownership, cascading, fetch strategy and join-table behavior carefully.

## 38. Lazy vs Eager Fetching ⭐⭐⭐⭐
**LAZY:** related data is loaded when accessed.

**EAGER:** related data is loaded immediately according to the mapping/provider behavior.

For large object graphs, uncontrolled eager fetching can cause performance and memory issues.

## 39. N+1 Query Problem ⭐⭐⭐⭐⭐
Occurs when one query loads parent records and then an additional query runs for each parent to fetch related data.

Common remedies:
- Fetch join
- Entity graphs
- Batch fetching
- Carefully designed queries

## 40. Pagination ⭐⭐⭐⭐
Use Spring Data's `Pageable` and `Page`.

```java
Page<User> page = repository.findAll(PageRequest.of(0, 20));
```

Useful for large datasets to avoid returning everything at once.

## 41. REST API Design ⭐⭐⭐⭐⭐
Good REST practices:
- Resource-oriented URLs
- Correct HTTP methods/status codes
- Stateless requests
- Validation
- Consistent error responses
- Pagination/filtering where needed
- Idempotency where relevant

## 42. GET vs POST vs PUT vs PATCH
- `GET` → retrieve
- `POST` → create/process non-idempotent operations commonly
- `PUT` → replace/update a resource; designed to be idempotent
- `PATCH` → partial update
- `DELETE` → delete

## 43. Actuator ⭐⭐⭐⭐⭐
Spring Boot Actuator provides production-oriented endpoints and application metrics/health information.

Common endpoints include:
- `/actuator/health`
- `/actuator/info`
- `/actuator/metrics`
- `/actuator/loggers`

Expose only endpoints appropriate for the environment and secure sensitive endpoints.

## 44. Health Checks
Actuator health can expose application and dependency health indicators.

In distributed systems, health endpoints are commonly used by infrastructure/orchestrators for monitoring and traffic management.

## 45. Embedded Server
Spring Boot commonly runs web applications with an embedded servlet container, so the application can be packaged and run without deploying a WAR to an external server.

## 46. Spring Boot Application Startup Flow
High-level:
```text
main()
→ SpringApplication.run()
→ Create ApplicationContext
→ Component Scan
→ Auto-Configuration
→ Bean Creation / Dependency Injection
→ Application Ready
```

## 47. `ApplicationContext`
`ApplicationContext` is Spring's central container abstraction for bean creation, configuration, lifecycle management, events and more.

## 48. `BeanFactory` vs `ApplicationContext`
`ApplicationContext` is a richer container abstraction that builds on the core bean-factory functionality and adds enterprise-oriented capabilities such as event publication, resource handling and message resolution.

## 49. Filter vs Interceptor ⭐⭐⭐⭐
**Filter:** Servlet-level, works before/after servlet processing and can apply broadly to requests.

**Interceptor:** Spring MVC abstraction around controller request handling.

Typical use:
- Filter → low-level request/response concerns
- Interceptor → MVC/controller concerns

## 50. `@CrossOrigin`
Controls CORS configuration for controller endpoints.

CORS is a browser security mechanism controlling which origins can make cross-origin requests.

## 51. CORS
Cross-Origin Resource Sharing allows a browser-based frontend from one origin to access resources on another origin when the server permits it.

Important concepts:
- Origin
- Allowed methods
- Allowed headers
- Credentials
- Preflight `OPTIONS` requests

## 52. Spring Security ⭐⭐⭐⭐⭐
Spring Security provides authentication and authorization capabilities.

Common concepts:
- Authentication → who are you?
- Authorization → what are you allowed to do?
- Security filter chain
- Password encoding
- CSRF
- CORS
- Session management
- OAuth2/JWT integration

## 53. Authentication vs Authorization
**Authentication:** verifies identity.

**Authorization:** determines access permissions.

**Memory:** AuthN = Who? | AuthZ = What can you do?

## 54. JWT Flow ⭐⭐⭐⭐⭐
Typical flow:
```text
Login
→ authenticate credentials
→ issue access token
→ client sends token with requests
→ server validates token
→ authorize request
→ response
```

Keep access tokens appropriately scoped and avoid putting sensitive secrets into token claims.

## 55. Spring Profiles + Configuration in Production
Do not hard-code secrets in source code.

Use environment/configuration management or secret-management systems appropriate to the deployment environment.

## 56. Caching ⭐⭐⭐⭐
Spring Cache abstraction provides annotations such as:
- `@EnableCaching`
- `@Cacheable`
- `@CachePut`
- `@CacheEvict`

Example:
```java
@Cacheable("users")
public User getUser(Long id) {
    return repository.findById(id).orElseThrow();
}
```

## 57. `@Cacheable` vs `@CachePut` vs `@CacheEvict`
- `@Cacheable` → use cached value when available; method may be skipped on a hit
- `@CachePut` → execute method and update cache with result
- `@CacheEvict` → remove cache entries

## 58. Scheduling
Spring supports scheduled tasks with `@Scheduled`.

```java
@Scheduled(cron = "0 0 * * * *")
public void runJob() {
}
```

Enable scheduling with `@EnableScheduling`.

For distributed systems, consider how multiple application instances may execute the same scheduled task and use an appropriate coordination approach.

## 59. Async Processing
`@Async` can execute methods asynchronously when async processing is enabled.

```java
@Async
public CompletableFuture<String> process() {
    return CompletableFuture.completedFuture("done");
}
```

Use an appropriately configured executor rather than relying blindly on defaults in production workloads.

## 60. Messaging / Event-driven Architecture
Spring Boot commonly integrates with brokers such as Kafka or RabbitMQ through ecosystem projects.

Typical pattern:
```text
Producer → Broker → Consumer
```

Useful for decoupling, asynchronous processing and event-driven workflows.

## 61. Spring Boot Microservices ⭐⭐⭐⭐⭐
Common building blocks:
- REST APIs
- API Gateway
- Service Discovery
- Config Management
- Load Balancing
- Resilience/Fault tolerance
- Distributed tracing/observability
- Messaging
- Centralized security

These directly align with common microservices interview topics such as service discovery, API gateways, security, scalability and fault tolerance.

## 62. API Gateway
Acts as an entry point for clients and can provide:
- Routing
- Authentication/authorization integration
- Rate limiting
- Request transformation
- Observability
- Aggregation in some designs

**Memory:** Gateway = one entry point.

## 63. Service Discovery
Allows services to find available instances dynamically rather than hard-coding instance IP/ports.

**Memory:** Service Discovery = Find service instances dynamically.

## 64. Load Balancing
Distributes requests among healthy service instances.

Can be implemented at infrastructure or application/platform layers depending on architecture.

## 65. Fault Tolerance
Common patterns:
- Timeout
- Retry
- Circuit breaker
- Bulkhead
- Fallback
- Rate limiting

Avoid blind retries, especially for non-idempotent operations.

## 66. Circuit Breaker
State model:
```text
CLOSED
  ↓ failures exceed threshold
OPEN
  ↓ wait period
HALF_OPEN
  ↓ test requests
CLOSED / OPEN
```

Purpose: stop repeatedly calling an unhealthy dependency and help prevent cascading failures.

## 67. Observability
Three common pillars:
- Logs
- Metrics
- Traces

Use correlation/trace IDs to follow a request across distributed services.

## 68. Testing ⭐⭐⭐⭐⭐
Common tools/approaches:
- JUnit
- Mockito
- `@WebMvcTest`
- `@DataJpaTest`
- `@SpringBootTest`

### Unit test
Tests a small unit in isolation.

### Integration test
Verifies interaction among real application components and/or infrastructure.

## 69. `@SpringBootTest` vs `@WebMvcTest`
`@SpringBootTest` loads a broad application context for integration-style testing.

`@WebMvcTest` focuses on the MVC/web layer and usually uses mocks for dependencies.

## 70. Mockito
Used to create test doubles such as mocks.

Typical methods:
```java
when(mock.method()).thenReturn(value);
verify(mock).method();
```

## 71. Externalized Configuration
Prefer external configuration for environment-specific values.

Typical categories:
- application config
- secrets
- database URLs
- feature flags
- environment variables

## 72. Common Production Questions
### How do you handle high traffic?
> Load balancing + horizontal scaling + caching + database optimization + asynchronous processing + rate limiting + observability.

### How do you prevent cascading failures?
> Timeouts + circuit breakers + bulkheads + controlled retries + fallbacks + monitoring.

### How do services communicate?
> Synchronous REST/gRPC or asynchronous messaging depending on latency, coupling and reliability requirements.

## 73. Rapid Revision — 20 Lines
1. Spring Boot simplifies Spring application development.
2. `@SpringBootApplication` = Configuration + Auto Configuration + Component Scan.
3. IoC means the container manages objects/dependencies.
4. Prefer constructor injection.
5. Bean = object managed by Spring.
6. `@Component` is generic; stereotypes express layer intent.
7. `@RestController` returns response bodies directly.
8. `@RequestBody` maps request body data to an object.
9. `ResponseEntity` controls body, status and headers.
10. `@RestControllerAdvice` centralizes REST exception handling.
11. `@Transactional` defines transaction boundaries.
12. `REQUIRED` joins or creates a transaction.
13. Lazy loading defers related data retrieval.
14. N+1 means one parent query plus many child queries.
15. Actuator provides health/metrics/operational endpoints.
16. Authentication = identity; Authorization = permissions.
17. Gateway = entry point; Service Discovery = locate instances.
18. Circuit breaker helps stop calls to failing dependencies.
19. Cache reduces repeated expensive reads.
20. Logs + Metrics + Traces = core observability.

## 74. Interview Memory Tricks
- `@SpringBootApplication` → **C-A-S** = Configuration, Auto-configuration, Scan
- IoC → **Container controls objects**
- DI → **Give dependency from outside**
- REST Controller → **HTTP in, object/response out**
- Transaction → **All or nothing boundary**
- Lazy → **Load later**
- Cache → **Avoid repeated expensive work**
- Gateway → **Single entry point**
- Discovery → **Find service instances**
- Circuit breaker → **Stop calling a failing dependency**
- Actuator → **Application health/operations**
