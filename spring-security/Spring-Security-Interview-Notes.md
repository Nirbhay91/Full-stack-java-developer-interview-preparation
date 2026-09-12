# Spring Security — Complete Interview Notes

> Interview-focused revision notes for Java / Spring Boot developers.
>
> **Goal:** Be able to explain the security flow clearly, then go one level deeper when the interviewer asks follow-ups.

---

# 1. What is Spring Security?

> Spring Security is a framework used to provide authentication, authorization, and protection against common web/application security threats in Spring applications.

It mainly helps with:

- Authentication
- Authorization
- Password management
- Session management
- CSRF protection
- CORS integration
- Security filters
- Method-level security
- OAuth2 / OpenID Connect support
- Resource server / JWT validation

**Memory:**

`Spring Security = Who are you? + What can you access? + Protect the request`

---

# 2. Authentication vs Authorization ⭐⭐⭐⭐⭐

### Authentication

> Authentication verifies **who the user is**.

Example:

```text
username + password
        ↓
     Authentication
        ↓
       User
```

### Authorization

> Authorization decides **what the authenticated user is allowed to do**.

Example:

```text
USER      → /profile      ✅
USER      → /admin        ❌
ADMIN     → /admin        ✅
```

**Memory:**

- Authentication = **Who are you?**
- Authorization = **What can you do?**

---

# 3. Security Flow — High Level ⭐⭐⭐⭐⭐

Typical request flow:

```text
Client
  ↓
HTTP Request
  ↓
Security Filter Chain
  ↓
Authentication / Token Processing
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
  ↓
Response
```

For a JWT-based API:

```text
Client
  ↓
Authorization: Bearer <JWT>
  ↓
Security Filter Chain
  ↓
JWT validation
  ↓
Authentication created
  ↓
SecurityContext
  ↓
Authorization rules
  ↓
Controller
```

---

# 4. What is SecurityFilterChain? ⭐⭐⭐⭐⭐

> `SecurityFilterChain` defines how incoming HTTP requests are secured and which security filters and rules apply.

Modern Spring Security configuration commonly uses a bean like:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        );

    return http.build();
}
```

**Interview line:**

> "Every incoming request passes through the configured security filter chain before it reaches the controller."

---

# 5. SecurityContext ⭐⭐⭐⭐

> `SecurityContext` holds the current authenticated user's security information.

It contains an `Authentication` object.

Typical flow:

```text
Request
  ↓
Authentication
  ↓
SecurityContext
```

You can access the current authentication from:

```java
Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();
```

---

# 6. Authentication Object ⭐⭐⭐⭐

`Authentication` represents the current authentication information.

It can provide:

- Principal
- Authorities
- Authentication status
- Credentials (depending on implementation / lifecycle)

Example:

```java
Authentication auth =
        SecurityContextHolder.getContext().getAuthentication();

String username = auth.getName();
```

---

# 7. Principal

> Principal represents the currently authenticated identity.

For a user-based application, this can be a username or a custom user object.

---

# 8. GrantedAuthority ⭐⭐⭐⭐

> `GrantedAuthority` represents an authority/permission granted to the authenticated user.

Examples:

```text
ROLE_USER
ROLE_ADMIN
READ_REPORT
WRITE_REPORT
```

Used in authorization checks.

---

# 9. Role vs Authority ⭐⭐⭐⭐⭐

A role is commonly represented as an authority with the `ROLE_` prefix.

Example:

```java
.hasRole("ADMIN")
```

typically checks for:

```text
ROLE_ADMIN
```

While:

```java
.hasAuthority("READ_REPORT")
```

checks that exact authority.

**Memory:**

`Role = coarse-grained access`  
`Authority = permission-level access`

---

# 10. UserDetails ⭐⭐⭐⭐

> `UserDetails` represents user information used by Spring Security during username/password authentication.

Common methods:

```java
getUsername()
getPassword()
getAuthorities()
isAccountNonExpired()
isAccountNonLocked()
isCredentialsNonExpired()
isEnabled()
```

A custom user can implement `UserDetails`.

---

# 11. UserDetailsService ⭐⭐⭐⭐⭐

> `UserDetailsService` loads user information, usually from a database, using a username or similar identifier.

Example:

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) {
        // fetch user from DB
        // convert to UserDetails
        return user;
    }
}
```

**Important:**

`UserDetailsService` loads the user; it does not itself define the complete authentication mechanism.

---

# 12. PasswordEncoder ⭐⭐⭐⭐⭐

Never store plain-text passwords.

Use a password hashing strategy such as `BCryptPasswordEncoder`.

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

When registering:

```java
String encoded = passwordEncoder.encode(rawPassword);
```

During authentication, Spring compares the submitted password with the stored hash using the encoder.

**Memory:**

`Raw password → hash → store hash`

---

# 13. Why BCrypt?

BCrypt is a password hashing algorithm designed to be computationally expensive and includes a salt as part of its password-hashing design.

Interview line:

> "Password hashing should be one-way; we should not decrypt stored passwords."

---

# 14. AuthenticationManager ⭐⭐⭐⭐

> `AuthenticationManager` is the main abstraction responsible for authenticating an `Authentication` request.

Conceptually:

```text
Username + Password
        ↓
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService + PasswordEncoder
        ↓
Authenticated Authentication
```

---

# 15. AuthenticationProvider ⭐⭐⭐⭐⭐

> `AuthenticationProvider` contains the actual authentication logic for a particular type of authentication.

For username/password authentication, `DaoAuthenticationProvider` is a common implementation.

Conceptually:

```text
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService
        ↓
PasswordEncoder
```

---

# 16. Username/Password Authentication Flow ⭐⭐⭐⭐⭐

```text
1. Client sends username + password
2. Authentication request is created
3. AuthenticationManager receives it
4. Provider loads user using UserDetailsService
5. PasswordEncoder verifies password
6. Authentication succeeds/fails
7. SecurityContext gets authenticated user
8. Authorization is applied
9. Request proceeds to controller
```

**Interview answer:**

> "Spring Security delegates authentication to an AuthenticationManager, which uses an AuthenticationProvider. For DAO-based authentication, the provider loads the user through UserDetailsService and verifies the password using PasswordEncoder."

---

# 17. JWT ⭐⭐⭐⭐⭐

JWT = JSON Web Token.

It is commonly used for stateless authentication in REST APIs.

Structure:

```text
Header.Payload.Signature
```

Example conceptually:

```text
xxxxx.yyyyy.zzzzz
```

---

# 18. JWT Components

### Header

Contains metadata such as the signing algorithm and token type.

### Payload

Contains claims.

Examples:

```text
sub
iss
exp
iat
roles
scope
```

### Signature

Used to verify that the token was signed by the expected issuer/key and has not been modified.

**Important:** JWT payload is encoded, not automatically encrypted.

---

# 19. JWT Authentication Flow ⭐⭐⭐⭐⭐

```text
1. User logs in
2. Server authenticates credentials
3. Server issues signed JWT
4. Client stores token appropriately
5. Client sends Bearer token on future requests
6. Security filter extracts token
7. Token signature/claims are validated
8. Authentication is created
9. Authorization rules are checked
10. Request reaches controller
```

Header:

```http
Authorization: Bearer <token>
```

---

# 20. Why JWT?

Advantages:

- Stateless request authentication
- Good fit for REST APIs
- Easy to pass between services
- Self-contained claims can reduce lookup needs

Trade-offs:

- Revocation is harder than server-side sessions
- Token size can be larger
- Stolen tokens can be used until they expire or are otherwise invalidated
- Sensitive information should not be placed in plain JWT claims

---

# 21. JWT vs Session ⭐⭐⭐⭐⭐

| Session | JWT |
|---|---|
| Server maintains session state | Commonly stateless authentication |
| Session ID sent by client | Token sent by client |
| Server looks up session | Token can contain claims |
| Revocation is straightforward server-side | Revocation needs additional strategy |
| Easy for traditional web apps | Common for REST/microservice APIs |

**Interview answer:**

> "JWT is useful when we want stateless authentication, while session-based authentication keeps authentication state on the server. The choice depends on application architecture and operational requirements."

---

# 22. Access Token vs Refresh Token ⭐⭐⭐⭐⭐

### Access Token

Short-lived token used to access protected APIs.

### Refresh Token

Longer-lived credential used to obtain a new access token without asking the user to log in again.

Typical flow:

```text
Login
 ↓
Access Token + Refresh Token
 ↓
Access token expires
 ↓
Refresh token
 ↓
New access token
```

**Security point:** Refresh tokens require stronger protection and lifecycle management.

---

# 23. OAuth 2.0 ⭐⭐⭐⭐⭐

> OAuth 2.0 is an authorization framework that allows a client to obtain limited access to protected resources on behalf of a resource owner.

Important actors:

```text
Resource Owner
Client
Authorization Server
Resource Server
```

Example:

```text
User → Authorization Server
              ↓
         Access Token
              ↓
Client → Resource Server
```

---

# 24. OpenID Connect (OIDC)

> OIDC is an identity layer built on top of OAuth 2.0.

Memory:

`OAuth2 = Authorization`  
`OIDC = Authentication / Identity on top of OAuth2`

---

# 25. Resource Server ⭐⭐⭐⭐

A resource server hosts protected APIs/resources and validates access tokens before serving the resource.

In a JWT-based setup:

```text
Client
 ↓
Bearer Token
 ↓
Resource Server
 ↓
JWT validation
 ↓
Authorization
 ↓
API
```

---

# 26. CSRF ⭐⭐⭐⭐⭐

CSRF = Cross-Site Request Forgery.

It attempts to trick a user's browser into making an unwanted state-changing request to a site where the user is already authenticated.

Traditional cookie/session-based browser applications are especially relevant to CSRF.

For a stateless API using Bearer tokens in the `Authorization` header and not relying on browser cookies for authentication, CSRF protection is often configured differently.

**Do not say:** "CSRF is never needed in REST."

Correct interview answer:

> "Whether CSRF protection is required depends on how authentication credentials are transported and the type of client. Cookie-based authentication is particularly exposed to CSRF because browsers automatically attach cookies."

---

# 27. CORS ⭐⭐⭐⭐⭐

CORS = Cross-Origin Resource Sharing.

It controls whether a browser allows frontend JavaScript from one origin to access resources from another origin.

Example:

```text
Frontend: https://app.example.com
Backend:  https://api.example.com
```

The backend must return appropriate CORS headers for the browser to allow the cross-origin request.

**Important:**

CORS is primarily a **browser security policy**, not an authentication mechanism.

---

# 28. CORS vs CSRF

| CORS | CSRF |
|---|---|
| Cross-origin browser access | Forged requests from an authenticated browser context |
| Browser policy / server permission | Request-forgery attack |
| Controls which origins may access responses | Protects state-changing requests |
| Not an authentication mechanism | Not solved merely by CORS configuration |

---

# 29. Authentication Entry Point ⭐⭐⭐⭐

When an unauthenticated client tries to access a protected resource, Spring Security can invoke an `AuthenticationEntryPoint`.

Typical REST response:

```text
401 Unauthorized
```

**401 = Authentication is missing/invalid.**

---

# 30. AccessDeniedHandler ⭐⭐⭐⭐

When the user is authenticated but not authorized to access the resource, Spring Security can use an `AccessDeniedHandler`.

Typical response:

```text
403 Forbidden
```

**403 = Authenticated, but insufficient permission.**

### Memory:

`401 → Who are you?`  
`403 → I know you, but you cannot do this.`

---

# 31. ExceptionTranslationFilter — Concept

Spring Security translates security exceptions into the appropriate response behavior.

Conceptually:

```text
AuthenticationException
        ↓
AuthenticationEntryPoint
        ↓
401

AccessDeniedException
        ↓
AccessDeniedHandler
        ↓
403
```

---

# 32. Request Authorization

Modern Spring Security commonly uses:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/login", "/public/**").permitAll()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
)
```

Common authorization methods:

```text
permitAll()
authenticated()
denied()
hasRole()
hasAnyRole()
hasAuthority()
hasAnyAuthority()
```

---

# 33. Method-Level Security ⭐⭐⭐⭐⭐

Authorization can also be placed at service/method level.

Enable it using Spring Security's method-security support, then use annotations such as:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) {
}
```

Other commonly discussed annotations:

```text
@PreAuthorize
@PostAuthorize
@PreFilter
@PostFilter
```

**Why method-level security?**

> It protects business operations even if the method is called from different entry points, not just one URL pattern.

---

# 34. `hasRole()` vs `hasAuthority()` ⭐⭐⭐⭐⭐

```java
.hasRole("ADMIN")
```

usually maps to the authority:

```text
ROLE_ADMIN
```

Whereas:

```java
.hasAuthority("ADMIN")
```

checks the exact authority `ADMIN`.

**Common mistake:**

Do not blindly pass `ROLE_ADMIN` to `hasRole("...")` if the configured role prefix would then produce a mismatch.

---

# 35. Stateless Security ⭐⭐⭐⭐⭐

For JWT-based REST APIs, the server commonly does not create an HTTP session to remember authentication between requests.

Typical configuration:

```java
.sessionManagement(session ->
    session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
)
```

Each request carries its authentication credentials, commonly the Bearer token.

**Memory:**

`Stateful = server remembers session`  
`Stateless = each request carries credentials`

---

# 36. SecurityContext and Stateless APIs

Even in stateless authentication, Spring Security still creates an `Authentication` for the current request and places it in the `SecurityContext` for request processing.

The context does not imply that the application is storing a long-lived server-side login session.

---

# 37. Security Filter / JWT Filter ⭐⭐⭐⭐⭐

A custom JWT filter traditionally extends `OncePerRequestFilter` when custom token extraction/validation is needed.

Conceptual steps:

```text
Read Authorization header
        ↓
Check Bearer token
        ↓
Validate token
        ↓
Extract username/claims
        ↓
Build Authentication
        ↓
Set SecurityContext
        ↓
Continue filter chain
```

**Important:**

If using Spring Security's built-in OAuth2 Resource Server support, prefer the framework's JWT support instead of writing a custom JWT filter unnecessarily.

---

# 38. Why `OncePerRequestFilter`?

It provides a convenient base class for a filter that should execute once per request dispatch according to its documented behavior.

Commonly used for custom authentication token processing.

---

# 39. Filter Order ⭐⭐⭐⭐

Security filters run before the controller.

Filter order matters because authentication must be established before authorization decisions that depend on it.

Interview line:

> "When adding a custom filter, I need to place it at the correct position relative to existing security filters, otherwise authentication may not be available when authorization occurs."

---

# 40. Why use API Gateway with Security?

In microservices, an API Gateway can provide cross-cutting concerns such as:

- Authentication/token validation
- Routing
- Rate limiting
- TLS termination
- Logging
- Correlation IDs
- Request policies

But service-level authorization should still be enforced where the business decision belongs.

**Important interview point:**

> "Gateway authentication can reduce repetition, but internal services should not blindly trust every caller."

---

# 41. Service-to-Service Authentication ⭐⭐⭐⭐⭐

Common options:

- OAuth2 client credentials
- mTLS
- Signed service tokens
- Internal identity mechanisms

Example OAuth2 client-credentials flow:

```text
Service A
   ↓
Authorization Server
   ↓
Access Token
   ↓
Service B
```

---

# 42. JWT vs OAuth2

Do not treat them as the same thing.

> JWT is a **token format**.

> OAuth 2.0 is an **authorization framework**.

OAuth2 access tokens can be JWTs or opaque tokens.

---

# 43. Password Grant — Interview Trap

Do not recommend the OAuth 2.0 Resource Owner Password Credentials approach for new systems.

Modern designs generally use authorization code with PKCE for user-facing applications or client credentials for service-to-service scenarios, depending on the use case.

---

# 44. Session Fixation

Session fixation is an attack where an attacker tries to make a victim use a session identifier known to the attacker.

Spring Security provides protections around session management, including session ID change strategies after authentication where applicable.

---

# 45. Session Management

Topics interviewers may ask:

- Session creation policy
- Concurrent sessions
- Session fixation protection
- Session timeout
- Invalid session handling
- Logout

For stateless JWT APIs, server-side HTTP session usage is generally minimized or disabled.

---

# 46. Logout ⭐⭐⭐

For session-based authentication, logout can invalidate the server-side session and clear relevant authentication state.

For JWT-based stateless authentication:

> There is usually no server-side session to simply invalidate. Logout commonly means removing the token client-side and/or using a server-side token revocation/deny-list strategy when immediate invalidation is required.

---

# 47. JWT Revocation Problem ⭐⭐⭐⭐⭐

JWT is often self-contained and stateless.

Problem:

```text
Token valid for 30 min
        ↓
User gets disabled after 5 min
        ↓
Existing token may still be cryptographically valid
```

Possible approaches:

- Short-lived access tokens
- Refresh-token rotation/revocation
- Token deny-list for selected cases
- Token version / user-session version strategy
- Centralized introspection with opaque tokens

Trade-off: more revocation state reduces some benefits of pure statelessness.

---

# 48. Security Headers ⭐⭐⭐⭐

Common security headers/topics include:

```text
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
HSTS
Frame protections
Cache-Control (for sensitive responses where appropriate)
```

Spring Security can help configure common security headers.

---

# 49. HTTPS / TLS ⭐⭐⭐⭐⭐

Authentication tokens and credentials should be protected in transit.

> Use HTTPS/TLS so credentials/tokens are not exposed in plaintext over the network.

In microservices, internal traffic may also use TLS or mTLS depending on security requirements.

---

# 50. mTLS ⭐⭐⭐⭐⭐

mTLS = mutual TLS.

Normal TLS:

```text
Client verifies Server
```

mTLS:

```text
Client verifies Server
Server verifies Client
```

Useful for service-to-service identity.

---

# 51. OAuth2 Authorization Code + PKCE ⭐⭐⭐⭐⭐

Common for user-facing applications.

High-level:

```text
User
 ↓
Authorization Server
 ↓
Authorization Code
 ↓
Client exchanges code + PKCE verifier
 ↓
Access Token
```

PKCE helps protect public clients from authorization-code interception.

---

# 52. Client Credentials Flow ⭐⭐⭐⭐⭐

Common for service-to-service authentication.

```text
Service A
   ↓ client_id + client_secret
Authorization Server
   ↓
Access Token
   ↓
Service B
```

No end-user is directly involved in the authorization grant.

---

# 53. Scope ⭐⭐⭐⭐

Scopes represent delegated permissions in OAuth2-style systems.

Example:

```text
orders.read
orders.write
profile.read
```

Authorization can check scopes/authorities depending on configuration.

---

# 54. JWT Claims

Common claims:

```text
iss → issuer
sub → subject
aud → audience
exp → expiration
nbf → not-before
iat → issued-at
jti → token identifier
```

Important:

> Do not trust a claim until the token's signature and relevant validation checks have succeeded.

---

# 55. JWT Validation ⭐⭐⭐⭐⭐

At minimum think about:

```text
Signature
Issuer
Audience (when required)
Expiration
Not-before
Algorithm policy
```

A resource server should validate claims according to the application's trust configuration and token contract.

---

# 56. Access Token Storage

For browser applications, token storage requires careful consideration.

Important risks:

- XSS
- CSRF
- Token leakage
- Browser storage exposure

Do not give a simplistic interview answer like "always store JWT in localStorage."

A safer discussion is:

> "Storage strategy depends on application type and threat model. If credentials are stored in cookies, CSRF protections become important; JavaScript-accessible storage increases the impact of XSS."

---

# 57. XSS ⭐⭐⭐⭐⭐

XSS = Cross-Site Scripting.

Attacker-controlled script executes in a user's browser in the security context of a trusted site.

Defenses include:

- Output encoding
- Input handling
- Content Security Policy
- Avoiding unsafe HTML injection
- Proper cookie flags where cookies are used

**Memory:**

`XSS = malicious script in browser`

---

# 58. Secure Cookies

Important flags:

```text
HttpOnly
Secure
SameSite
```

### HttpOnly

Helps prevent JavaScript from directly reading the cookie.

### Secure

Cookie should be sent over HTTPS.

### SameSite

Controls cross-site cookie sending behavior and can reduce CSRF risk.

---

# 59. `permitAll()` vs `anonymous()`

```java
.permitAll()
```

Allows everyone, including authenticated users.

Anonymous-specific controls can distinguish unauthenticated users where needed.

---

# 60. `authenticated()`

Requires an authenticated user.

```java
.anyRequest().authenticated()
```

---

# 61. Authentication vs Authorization at Microservice Level

A good architecture separates:

```text
Authentication
→ establish identity

Authorization
→ evaluate permission for this operation/resource
```

A service should enforce business authorization close to the business resource/action.

---

# 62. RBAC ⭐⭐⭐⭐⭐

RBAC = Role-Based Access Control.

```text
USER → ROLE_USER
ADMIN → ROLE_ADMIN
```

Simple and common.

---

# 63. ABAC ⭐⭐⭐⭐

ABAC = Attribute-Based Access Control.

Authorization depends on attributes such as:

- User role
- Department
- Resource owner
- Region
- Request context
- Time

Example:

> A manager may edit employees only in the manager's own department.

---

# 64. RBAC vs ABAC

| RBAC | ABAC |
|---|---|
| Based primarily on roles | Based on attributes and policies |
| Simpler | More flexible |
| Good for coarse permissions | Good for contextual/business rules |
| Can become role-heavy at scale | More complex policy management |

---

# 65. Object-Level Authorization ⭐⭐⭐⭐⭐

Endpoint-level authorization is not always enough.

Bad example:

```text
GET /orders/123
```

If the user has `ROLE_USER`, that alone does not mean they own order `123`.

Object-level decision:

```text
Does this authenticated user have permission to access order 123?
```

This is especially important in business APIs.

---

# 66. CSRF Token — Concept

A server can issue a CSRF token that the client must include in state-changing requests.

If the attacker does not know the token, the forged request should fail.

This is particularly relevant to cookie-based web applications.

---

# 67. Same-Origin Policy

The browser's same-origin policy restricts how scripts from one origin interact with resources from another origin.

CORS provides a controlled mechanism for permitted cross-origin access.

---

# 68. Brute Force Protection ⭐⭐⭐⭐

Spring Security handles authentication mechanisms, but application architecture should also consider:

- Rate limiting
- Account lockout policy where appropriate
- Increasing backoff
- CAPTCHA for suitable user flows
- Monitoring failed logins
- MFA

Do not rely only on a simple username/password endpoint without abuse protection.

---

# 69. MFA ⭐⭐⭐⭐

Multi-Factor Authentication combines multiple factor types, for example:

```text
Something you know → password
Something you have → security key / OTP device
Something you are   → biometric
```

---

# 70. Common API Status Codes

```text
200 → OK
201 → Created
204 → No Content
400 → Bad Request
401 → Unauthorized / authentication required
403 → Forbidden
404 → Not Found
409 → Conflict
422 → Unprocessable Content (depending on API design)
429 → Too Many Requests
500 → Internal Server Error
503 → Service Unavailable
```

**Most important security pair:**

`401 = authentication problem`  
`403 = authorization problem`

---

# 71. Custom Authentication Failure Response

For REST APIs, avoid returning HTML login pages when the client expects JSON.

Typical custom response:

```json
{
  "timestamp": "...",
  "status": 401,
  "error": "Unauthorized",
  "message": "Authentication required"
}
```

Use a consistent API error format across the application.

---

# 72. Custom Access Denied Response

Example:

```json
{
  "status": 403,
  "error": "Forbidden",
  "message": "You do not have permission to access this resource"
}
```

---

# 73. BCrypt vs Encryption

Password storage should use **password hashing**, not reversible encryption.

Conceptually:

```text
Password hashing → one-way verification
Encryption       → reversible with key
```

---

# 74. Authentication vs Authorization Example ⭐⭐⭐⭐⭐

Imagine an employee portal:

```text
Login with username/password
        ↓
Authentication
        ↓
User = Nirbhay
        ↓
Role = EMPLOYEE
        ↓
Can access profile? yes
Can access payroll admin? no
        ↓
Authorization
```

---

# 75. Spring Security 6+ Configuration Style ⭐⭐⭐⭐⭐

Modern configuration commonly uses a `SecurityFilterChain` bean instead of extending the older adapter style.

Example:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {

        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session ->
                session.sessionCreationPolicy(
                    SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

**Interview note:**

Do not say `csrf.disable()` is universally correct. Explain why it may be appropriate for a stateless Bearer-token API and evaluate the actual credential transport and client type.

---

# 76. `WebSecurityConfigurerAdapter` — Interview Trap

Older Spring Security versions commonly used `WebSecurityConfigurerAdapter`.

Modern Spring Security configuration prefers component-based beans such as `SecurityFilterChain` rather than extending that adapter.

If interviewer asks:

> "How do you configure Spring Security in modern Spring Boot?"

Answer:

> "I use a `SecurityFilterChain` bean and configure `HttpSecurity` using the component-based DSL."

---

# 77. `AuthenticationManager` Bean — Concept

When a login API needs programmatic username/password authentication, an `AuthenticationManager` can be obtained from Spring's `AuthenticationConfiguration`.

Example:

```java
@Bean
AuthenticationManager authenticationManager(
        AuthenticationConfiguration config) throws Exception {
    return config.getAuthenticationManager();
}
```

Then:

```java
Authentication authentication =
    authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(
            username, password));
```

---

# 78. JWT Login Endpoint — Typical Architecture

```text
POST /auth/login
        ↓
AuthenticationManager
        ↓
UserDetailsService
        ↓
PasswordEncoder
        ↓
Authentication success
        ↓
Issue Access Token
        ↓
Return token
```

Then every protected request carries the Bearer token.

---

# 79. OAuth2 Login vs Resource Server

### OAuth2 Client / Login

Application acts as a client and lets users authenticate via an external identity provider.

### Resource Server

Application exposes protected APIs and validates access tokens.

These are related but different responsibilities.

---

# 80. Identity Provider (IdP)

An Identity Provider authenticates users and can issue tokens.

Examples of IdP categories include enterprise identity systems and OAuth2/OIDC providers.

Resource servers trust tokens according to configured issuer/keys/policies.

---

# 81. Public Key vs Secret Key in JWT

### Symmetric signing

Same secret is used to sign and verify.

Example family:

```text
HMAC
```

### Asymmetric signing

Private key signs; public key verifies.

Example families:

```text
RSA
EC
```

For distributed systems, asymmetric signing can be attractive because resource servers can verify with a public key without possessing the signing private key.

---

# 82. Why Public-Key JWT Validation in Microservices?

```text
Authorization Server
    ↓ private key
 signs token

Microservices
    ↓ public key
 verify token
```

Only the trusted issuer holds the private signing key.

---

# 83. JWK / JWKS

A JWKS endpoint exposes public keys used by consumers to validate signed tokens.

A resource server can use the issuer metadata/JWK configuration to obtain verification keys and support key rotation.

---

# 84. Key Rotation ⭐⭐⭐⭐

Signing keys should be rotated periodically.

A robust system supports:

- Multiple valid verification keys during transition
- Key identifiers (`kid`)
- Secure key storage
- Rotation procedures
- Revocation/retirement of old keys

---

# 85. Secret Management ⭐⭐⭐⭐⭐

Do not hard-code secrets in source code.

Avoid:

```java
String secret = "my-secret-123";
```

Use:

- Environment/configuration management
- Secret managers
- Kubernetes secrets with proper controls
- Cloud secret management solutions

Also avoid accidentally logging secrets/tokens.

---

# 86. Token Leakage ⭐⭐⭐⭐⭐

Never log:

```text
Authorization: Bearer eyJ...
```

or passwords, client secrets, refresh tokens, session IDs.

Use redaction/masking in logs.

---

# 87. Security for Microservices — Interview Answer ⭐⭐⭐⭐⭐

> "At the edge, I typically use an API Gateway for routing, rate limiting and centralized token handling where appropriate. Each protected service validates the caller's identity and enforces its own authorization policies. We use HTTPS, short-lived access tokens, secure secret management, proper service-to-service authentication such as OAuth2 client credentials or mTLS, and centralized logging and monitoring."

---

# 88. Spring Security + JWT Project-Level Flow ⭐⭐⭐⭐⭐

```text
Frontend
   ↓
POST /auth/login
   ↓
AuthenticationManager
   ↓
UserDetailsService → DB
   ↓
PasswordEncoder verifies password
   ↓
JWT issued
   ↓
Frontend sends Authorization header
   ↓
API Gateway / Service
   ↓
JWT validation
   ↓
SecurityContext
   ↓
Role/Authority check
   ↓
Controller
   ↓
Service
   ↓
DB
```

---

# 89. How to Secure a Spring Boot REST API?

Interview-ready answer:

> "I use HTTPS, authentication such as JWT/OAuth2, authorization based on roles/authorities, secure password hashing with BCrypt, input validation, proper CORS configuration, CSRF protection when applicable, rate limiting, security headers, secret management, safe error handling, and audit/monitoring."

---

# 90. Why Not Put Roles Only in the Frontend?

Frontend checks are for user experience, not security.

Example:

```text
Button hidden in UI
        ≠
API secured
```

An attacker can call the API directly.

> **Authorization must be enforced on the backend.**

---

# 91. Common Security Mistakes ⭐⭐⭐⭐⭐

1. Plain-text passwords
2. Hard-coded secrets
3. Trusting user-supplied roles
4. Logging JWT/passwords
5. Treating JWT as encrypted data
6. Assuming CORS provides authentication
7. Disabling CSRF without understanding credential transport
8. Returning sensitive stack traces
9. No rate limiting on login endpoints
10. No HTTPS
11. No object-level authorization
12. Long-lived access tokens without justification
13. Trusting a token without validating issuer/audience/signature/expiry as required
14. Relying only on API Gateway security

---

# 92. 401 vs 403 — Must Memorize

```text
401
→ Authentication missing/invalid
→ AuthenticationEntryPoint

403
→ Authentication exists, permission denied
→ AccessDeniedHandler
```

---

# 93. Authentication Flow — 30 Second Answer

> "The request enters the Spring Security filter chain. If it contains a credential such as a Bearer token, Spring Security extracts and validates it. After successful authentication, an Authentication object is placed in the SecurityContext. Then authorization rules are evaluated. If the request is allowed, it reaches the controller; otherwise the framework returns the appropriate 401 or 403 response."

---

# 94. JWT Flow — 30 Second Answer

> "At login, the user's credentials are authenticated and the server issues a signed JWT. The client sends the token as a Bearer token on subsequent requests. The resource server validates the token's signature and required claims such as expiration, issuer and audience. It then creates an authenticated principal in the SecurityContext, and authorization rules determine whether the request is allowed."

---

# 95. Spring Security Filter Chain — 30 Second Answer

> "The SecurityFilterChain is a sequence of security filters applied to incoming requests. These filters can handle authentication, token processing, exception translation, session-related concerns and authorization. The request must pass the applicable security checks before reaching the controller."

---

# 96. OAuth2 — 30 Second Answer

> "OAuth 2.0 is an authorization framework. It defines how a client obtains an access token to access a protected resource. In user-facing apps, authorization code with PKCE is a common flow. For service-to-service communication, client credentials is common. OAuth2 is not the same as JWT; a JWT is just one possible token format."

---

# 97. OIDC — 20 Second Answer

> "OpenID Connect builds an identity layer on top of OAuth 2.0. OAuth2 primarily addresses delegated authorization, while OIDC adds standardized user identity and authentication information."

---

# 98. CSRF — 20 Second Answer

> "CSRF is an attack where a victim's browser is tricked into making an unwanted request while already authenticated. It is especially relevant to cookie-based authentication because browsers automatically attach cookies. For stateless APIs using Authorization Bearer tokens, CSRF configuration is different, but I don't disable it blindly."

---

# 99. CORS — 20 Second Answer

> "CORS is a browser mechanism that controls whether a frontend origin is allowed to access a different origin's resources. It is not an authentication mechanism and should not be confused with CSRF protection."

---

# 100. Most Important Interview Questions ⭐⭐⭐⭐⭐

### Fundamentals

1. What is Spring Security?
2. Authentication vs Authorization?
3. What is SecurityFilterChain?
4. What is SecurityContext?
5. What is Authentication?
6. What is Principal?
7. What is GrantedAuthority?
8. Role vs Authority?

### Username / Password

9. What is UserDetails?
10. What is UserDetailsService?
11. What is PasswordEncoder?
12. Why BCrypt?
13. What is AuthenticationManager?
14. What is AuthenticationProvider?
15. Explain username/password authentication flow.

### JWT

16. What is JWT?
17. JWT structure?
18. Explain JWT authentication flow.
19. What are JWT claims?
20. How do you validate JWT?
21. JWT vs Session?
22. Access token vs refresh token?
23. How do you revoke JWT?
24. Symmetric vs asymmetric signing?
25. Why use public/private keys?
26. What is JWKS?
27. What is key rotation?

### OAuth2/OIDC

28. What is OAuth2?
29. OAuth2 vs JWT?
30. What is OIDC?
31. Authorization Code + PKCE?
32. Client Credentials flow?
33. Resource Server?
34. What is an Identity Provider?
35. What are scopes?

### Web Security

36. What is CSRF?
37. When can CSRF protection be disabled?
38. What is CORS?
39. CORS vs CSRF?
40. What is XSS?
41. What is session fixation?
42. What are secure cookie flags?
43. Why HTTPS?
44. What is mTLS?

### Authorization

45. What is `hasRole()`?
46. What is `hasAuthority()`?
47. RBAC vs ABAC?
48. What is method-level security?
49. What is object-level authorization?
50. Why backend authorization is mandatory?

### Production

51. How do you secure microservices?
52. How do services authenticate with each other?
53. How do you store secrets?
54. How do you protect refresh tokens?
55. How do you prevent brute-force login attacks?
56. Why should tokens not be logged?
57. How do you implement consistent 401/403 responses?
58. How do you handle key rotation?
59. How do you secure an API Gateway?
60. What are common Spring Security mistakes?

---

# 101. Scenario-Based Questions ⭐⭐⭐⭐⭐

## Scenario 1: User is logged in but gets 403

Check:

```text
Authentication exists?
     ↓
Yes
     ↓
Required role/authority present?
     ↓
No → 403
```

Likely issue:

- Wrong authority name
- Missing `ROLE_` prefix expectation
- Incorrect endpoint rule
- Method-level authorization failure

---

## Scenario 2: User gets 401 even with JWT

Check:

- Authorization header format
- Token parsing
- Signature
- Expiration
- Issuer
- Audience
- Signing key
- Clock/time issues
- Resource server configuration
- Filter/authentication setup

---

## Scenario 3: Token works in one microservice but not another

Possible causes:

- Different issuer configuration
- Different signing keys/JWK source
- Audience mismatch
- Different clock skew assumptions
- Inconsistent authority mapping
- Incorrect gateway header forwarding

---

## Scenario 4: User changes role in DB but old JWT still says ADMIN

Reason:

> JWT contains a snapshot of claims at issuance time.

Solutions depending on requirements:

- Short-lived access tokens
- Token versioning
- Introspection
- Revocation strategy
- Re-issue token after authorization changes

---

## Scenario 5: Need immediate logout for JWT

Pure stateless JWT has no built-in server-side logout state.

Possible design:

```text
Short-lived access token
+
Refresh token revocation/rotation
+
Optional deny-list for critical immediate invalidation
```

---

## Scenario 6: Only account owner can access `/orders/{id}`

Do not rely only on:

```java
.hasRole("USER")
```

Instead also verify:

```text
authenticated user ID == order.ownerId
```

This is object-level authorization.

---

## Scenario 7: Security behind API Gateway

Good answer:

> "The gateway can authenticate or validate tokens centrally for cross-cutting concerns, but each downstream service should enforce its own authorization and establish trust in the caller rather than assuming every internal request is safe."

---

# 102. Spring Security Quick Memory Map

```text
SPRING SECURITY
│
├── Authentication
│   ├── AuthenticationManager
│   ├── AuthenticationProvider
│   ├── UserDetailsService
│   ├── UserDetails
│   └── PasswordEncoder
│
├── Authorization
│   ├── Role
│   ├── Authority
│   ├── hasRole()
│   ├── hasAuthority()
│   ├── RBAC
│   └── ABAC
│
├── Request Security
│   ├── SecurityFilterChain
│   ├── SecurityContext
│   ├── 401
│   └── 403
│
├── Token Security
│   ├── JWT
│   ├── Access Token
│   ├── Refresh Token
│   ├── OAuth2
│   ├── OIDC
│   ├── JWKS
│   └── Key Rotation
│
└── Web Security
    ├── CSRF
    ├── CORS
    ├── XSS
    ├── HTTPS/TLS
    ├── Secure Cookies
    └── Session Management
```

---

# 103. Final 2-Minute Revision ⭐⭐⭐⭐⭐

Memorize these lines:

> **Authentication = who you are.**
>
> **Authorization = what you can access.**
>
> **SecurityFilterChain = request security pipeline.**
>
> **SecurityContext = current authenticated security information.**
>
> **UserDetailsService = loads the user.**
>
> **PasswordEncoder = verifies password hashes.**
>
> **AuthenticationManager = coordinates authentication.**
>
> **AuthenticationProvider = performs a specific authentication mechanism.**
>
> **JWT = token format; OAuth2 = authorization framework.**
>
> **401 = authentication problem; 403 = authorization problem.**
>
> **CORS = browser cross-origin access control.**
>
> **CSRF = forged request using an authenticated browser context.**
>
> **XSS = malicious script executing in the browser.**
>
> **RBAC = roles; ABAC = attributes/policy.**
>
> **HTTPS protects credentials/tokens in transit.**
>
> **Never store plain-text passwords or hard-code secrets.**
>
> **Backend must enforce authorization; frontend checks are not security.**

---

# 104. Top 15 Must-Know Questions Before Interview

1. Authentication vs Authorization?
2. Explain Spring Security request flow.
3. What is SecurityFilterChain?
4. Explain JWT authentication flow.
5. JWT vs Session?
6. JWT vs OAuth2?
7. Access token vs Refresh token?
8. UserDetailsService vs AuthenticationProvider?
9. AuthenticationManager role?
10. 401 vs 403?
11. CSRF vs CORS?
12. `hasRole()` vs `hasAuthority()`?
13. How do you secure microservices?
14. How do you revoke JWT / handle logout?
15. How do you handle service-to-service authentication?

---

# 105. Interview Answer Formula

For almost every Spring Security question, answer in this order:

```text
Definition
   ↓
Why we use it
   ↓
How it works
   ↓
Project example
   ↓
Security trade-off / common pitfall
```

Example:

> "JWT is a signed token format used commonly for stateless API authentication. We issue it after successful login, the client sends it as a Bearer token, and the resource server validates it before creating an Authentication in the SecurityContext. In a project I would also use short-lived access tokens, secure refresh-token handling, HTTPS and proper issuer/audience validation."
