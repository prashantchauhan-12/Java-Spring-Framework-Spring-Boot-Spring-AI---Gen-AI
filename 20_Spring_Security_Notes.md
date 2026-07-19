# Spring Security — Complete Revision Notes

## 1. Why Security Matters

When building an application, priority order is usually:
1. **It should work** (functionality)
2. **It should be stable** (exception handling, works for every kind of input)
3. **It should be scalable** (handles many users, cloud deployment)
4. **It should perform well** (fast response times)
5. **It should be secure** — often the most neglected, but critical

Security is not just "firewalls" or "who can log in." It covers:
- **Authentication** — proving *who you are* (username/password, biometrics, tokens)
- **Authorization** — controlling *what you can access/do* once authenticated (e.g., only an `ADMIN` can create/delete a job; a normal `USER` cannot)

**How critical is security?**
- Public data websites → security matters, but less critical.
- Apps handling **personal data, credit cards, medical records, banking** → critical.
- Credit card leak → you can block the card. Bank account hacked → you can freeze it.
- **Personal/medical data leak → cannot be undone.** Regulatory bodies can impose heavy penalties or jail time for mishandling personal data.

---

## 2. OWASP Top 10

**OWASP** = Open Web Application Security Project. It publishes a **Top 10 security risks list roughly every 3–4 years** (2017 → 2021 → next expected update; always check the latest list at [owasp.org](https://owasp.org/www-project-top-ten/)).

| # | Risk | Key Idea |
|---|------|----------|
| 1 | **Broken Access Control** | Users can access/modify resources they shouldn't be authorized for. |
| 2 | **Cryptographic Failures** | Data in transit/at rest not properly encrypted; using deprecated hashing (MD5, SHA-1) instead of modern standards. |
| 3 | **Injection** | SQL/NoSQL/OS command/ORM injection. Classic example: building SQL with string concatenation lets an attacker pass `' OR 1=1 --` and bypass login. **Fix:** use `PreparedStatement` (parameterized queries) instead of concatenation. |
| 4 | **Insecure Design** | Security flaws baked into the architecture/design itself, not just the implementation. |
| 5 | **Security Misconfiguration** | Using default configs of frameworks/servers/libraries that attackers already know about. |
| 6 | **Vulnerable & Outdated Components** | Using deprecated/unsupported libraries or DB versions that no longer receive security patches. |
| 7 | **Identification & Authentication Failures** | Storing plain-text passwords, weak/no rate-limiting, missing Multi-Factor Authentication (MFA). |
| 8 | **Software & Data Integrity Failures** | Trusting unverified updates/plugins/CI pipelines. |
| 9 | **Security Logging & Monitoring Failures** | Not logging (miss attacks) vs. logging everything without monitoring (can't find the attack in the noise). Log *and* actively monitor. |
| 10 | **Server-Side Request Forgery (SSRF)** | Server sends a request to an internal/malicious resource on behalf of an unvalidated input, exposing internal systems. |

**SQL Injection example (concept):**
```sql
-- Vulnerable (string concatenation)
SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'"

-- If username input = "navin' OR '1'='1"
SELECT * FROM users WHERE username = 'navin' OR '1'='1' AND password = '...'
-- '1'='1' always evaluates true → attacker bypasses authentication
```
**Fix:** Always use `PreparedStatement` / parameterized queries — never string-concatenate user input into SQL.

---

## 3. Getting Started with Spring Security

### 3.1 Project Setup (Spring Initializr)
- Group: `com.telusko`
- Dependencies: **Spring Web**, **Spring Security**, **Spring Boot DevTools**
- Java 21, Spring Boot 3.2.x, packaging: Jar

### 3.2 A Simple Controller
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String greet() {
        return "Hello World";
    }

    @GetMapping("/about")
    public String about() {
        return "Telusko";
    }
}
```

### 3.3 What Happens the Moment You Add `spring-boot-starter-security`
Just by adding the **dependency** (no extra code):
- Every endpoint becomes **secured** automatically.
- Hitting `/hello` redirects to an **auto-generated login form** (no HTML file exists in your project — Spring Security generates it).
- **Default username:** `user`
- **Default password:** randomly generated on every application start, printed in the console:
  ```
  Using generated security password: 3f8e2a1c-xxxx-xxxx
  ```
  > ⚠️ This message explicitly says: *"This generated password is for development use only. Your security configuration must be updated before running the application in production."*
- Wrong password → **"Bad credentials"**
- `/logout` → built-in logout confirmation page.
- Login creates a **session** — subsequent requests to other endpoints (`/about`, etc.) don't need to re-authenticate as long as the session cookie exists.

**Behind the scenes (architecture):**
```
Client → [Spring Security Filter Chain] → DispatcherServlet → Controller
```
Before your request ever reaches `DispatcherServlet`, it passes through a **chain of security filters**.

---

## 4. Filters & FilterChain

- Spring Security uses the classic **Servlet Filter** concept: a filter sits *before* the actual servlet and can intercept/reject the request before it's processed.
- Multiple filters are linked together into a **`FilterChain`** — they execute **one after another, in a defined order**, not in parallel.
- Spring Security's default chain is called **`DefaultSecurityFilterChain`**, and includes (among many others):
  - `DisableEncodeUrlFilter`
  - `WebAsyncManagerIntegrationFilter`
  - `SecurityContextHolderFilter`
  - CORS filter
  - CSRF filter
  - `UsernamePasswordAuthenticationFilter`
  - Logout filter, Login filter, and more.
- You **can** reorder/customize filters, but it's rarely needed — default ordering works for most apps.

---

## 5. Sessions & Cookies

- After successful login, Spring Security creates an **HTTP session** and stores a **session ID** in a cookie (`JSESSIONID`).
- As long as the cookie is sent with each request, you stay authenticated — **no need to log in again** for other secured endpoints.
- Deleting the cookie (or logging out) invalidates the session → login required again.
- You can access session info in a controller:
```java
@GetMapping("/hello")
public String greet(HttpServletRequest request) {
    return "Hello World " + request.getSession().getId();
}
```
> ⚠️ Never print session IDs or passwords in production — this is for learning only.

---

## 6. Custom Username & Password (Quick & Dirty — application.properties)

```properties
spring.security.user.name=telusko
spring.security.user.password=1234
```

**Drawbacks:**
- Hardcoded — password stored as **plain text**.
- Only **one** username/password for the entire application — no per-user data, no multi-user support.
- Not suitable beyond local experimentation.

---

## 7. Authentication via Postman (Basic Auth)

- Hitting a secured endpoint from Postman without credentials → **`401 Unauthorized`**.
- In Postman: **Authorization tab → Basic Auth** → enter username/password.
- This sends credentials Base64-encoded in the `Authorization` header (`Authorization: Basic base64(username:password)`).
- ⚠️ Base64 is **not encryption** — it's trivially decodable. Basic Auth should always be paired with **HTTPS** in production.

---

## 8. CSRF (Cross-Site Request Forgery)

### 8.1 The Problem
- If you're logged into a site (e.g., your bank) and a session cookie exists, a **malicious website** you visit afterward could silently trigger a request (e.g., a money transfer POST) to your bank site — the browser will automatically attach your valid session cookie. This is **CSRF**.
- By default, Spring Security **enables CSRF protection** for state-changing requests (`POST`, `PUT`, `DELETE`, etc.) but **not** for `GET` (since GET shouldn't change server state).

### 8.2 Reproducing the Problem
```java
@RestController
public class StudentController {

    List<Student> students = new ArrayList<>(List.of(
        new Student(1, "Navin", "Java"),
        new Student(2, "Kiran", "Blockchain")
    ));

    @GetMapping("/students")
    public List<Student> getStudents() {
        return students;
    }

    @PostMapping("/students")
    public void addStudent(@RequestBody Student student) {
        students.add(student);
    }
}
```
- `GET /students` → works fine with just Basic Auth.
- `POST /students` with Basic Auth only → **fails** (CSRF token missing), even though credentials are correct.

### 8.3 Fetching & Using a CSRF Token
```java
@GetMapping("/csrf-token")
public CsrfToken getCsrfToken(HttpServletRequest request) {
    return (CsrfToken) request.getAttribute("_csrf");
}
```
Flow:
1. `GET /csrf-token` → returns a JSON object containing the token value and header name (`X-CSRF-TOKEN`).
2. Copy the token value.
3. In the `POST` request, add header: `X-CSRF-TOKEN: <copied value>`.
4. Now the POST succeeds (`200 OK`).
5. **A new token is issued with every request** — you must fetch it fresh each time.

### 8.4 Alternate Mitigation: `SameSite` Cookie Attribute
```properties
server.servlet.session.cookie.same-site=strict
```
Options: `lax` (default), `strict`, `none`. `strict` prevents the cookie from being sent on cross-site requests entirely — a second layer of defense against CSRF.

### 8.5 Stateless vs Stateful APIs
- **Stateful** — session maintained; same session ID reused across requests (what we've done so far).
- **Stateless** — no session; every single request must carry credentials (e.g., a JWT or Basic Auth header). Common for REST APIs.
- **If your API is stateless, you don't need CSRF protection at all** (no session cookie to forge), so it's typically **disabled**.

---

## 9. Custom Security Configuration (`SecurityFilterChain`)

To override Spring Security's defaults, define a `@Configuration` class returning a `SecurityFilterChain` bean.

### 9.1 Lambda Style (Recommended / What You'll Actually Use)
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(customizer -> customizer.disable())
            .authorizeHttpRequests(request -> request.anyRequest().authenticated())
            .formLogin(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```
> ⚠️ If you define your own `SecurityFilterChain` bean and configure **nothing** inside it, Spring Security assumes you know what you're doing and **disables all default protections** (no login form, no auth requirement at all) — you must explicitly configure everything.

`SessionCreationPolicy` options: `ALWAYS`, `IF_REQUIRED` (default), `NEVER`, `STATELESS`.

> Note: If you go `STATELESS`, `formLogin()`/session-based login and `/logout` filters become irrelevant — every request must carry auth credentials (Basic Auth header or a token) since no session is stored.

### 9.2 Imperative Style (For Understanding Only — Don't Write This in Real Projects)
This shows what the lambda is doing behind the scenes:
```java
Customizer<CsrfConfigurer<HttpSecurity>> custCsrf = new Customizer<CsrfConfigurer<HttpSecurity>>() {
    @Override
    public void customize(CsrfConfigurer<HttpSecurity> configure) {
        configure.disable();
    }
};
http.csrf(custCsrf);

Customizer<AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry> custHttp =
    new Customizer<>() {
        @Override
        public void customize(AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry registry) {
            registry.anyRequest().authenticated();
        }
    };
http.authorizeHttpRequests(custHttp);
```
> `Customizer<T>` is a functional interface with a single method `customize(T t)`. The lambda syntax (`customizer -> customizer.disable()`) is exactly implementing that interface — that's why lambdas work here.

`HttpSecurity` follows a **builder pattern**, so you can chain calls:
```java
http.csrf(c -> c.disable())
    .authorizeHttpRequests(a -> a.anyRequest().authenticated())
    .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
return http.build();
```

---

## 10. Multiple Users — In-Memory (Step Toward DB-Backed Auth)

Before moving to a real database, hardcode multiple users using `InMemoryUserDetailsManager`:

```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails user1 = User.withDefaultPasswordEncoder()
            .username("navin")
            .password("n@123")
            .roles("USER")
            .build();

    UserDetails admin = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("admin@789")
            .roles("ADMIN")
            .build();

    return new InMemoryUserDetailsManager(user1, admin);
}
```
- `UserDetailsService` is an **interface** with one method: `loadUserByUsername(String username)`.
- `InMemoryUserDetailsManager` implements `UserDetailsManager`, which extends `UserDetailsService` — so returning it satisfies the bean requirement.
- `User.withDefaultPasswordEncoder()` is **deprecated** and only meant for **learning/demo purposes** — never use it in production (it stores/uses an insecure default encoding).
- `.roles(...)` can accept multiple roles for a single user.

---

## 11. Database-Backed Authentication

### 11.1 Database Table
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT,
    password TEXT
);

INSERT INTO users (id, username, password) VALUES (1, 'navin', 'n@123');
INSERT INTO users (id, username, password) VALUES (2, 'kiran', 'n@789');
```
> Design tip mentioned in the notes: `username` should ideally be **unique** (or even the primary key), since login lookups happen by username.

### 11.2 `application.properties` — DataSource Config
```properties
# PostgreSQL
spring.datasource.url=jdbc:postgresql://localhost:5432/telusko
spring.datasource.username=postgres
spring.datasource.password=0000

# MySQL equivalent (change port/driver accordingly)
# spring.datasource.url=jdbc:mysql://localhost:3306/telusko
```

### 11.3 Maven Dependencies
Add via Spring Initializr or manually in `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 11.4 Entity Class
```java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    private int id;
    private String username;
    private String password;
}
```

### 11.5 Repository (JPA)
```java
public interface UserRepo extends JpaRepository<User, Integer> {
    User findByUsername(String username);
}
```

---

## 12. Wiring It All Together — The Full Authentication Chain

To connect Spring Security to your database, you need **4 pieces**:

```
AuthenticationProvider (DaoAuthenticationProvider)
        │  needs
        ▼
UserDetailsService (your implementation — MyUserDetailsService)
        │  needs
        ▼
UserRepo (JPA repository → hits the database)
        │  returns
        ▼
User (entity) ──wrapped into──▶ UserPrincipal (implements UserDetails)
```

### 12.1 `SecurityConfig` — Register the `AuthenticationProvider`
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private UserDetailsService userDetailsService;

    @Bean
    public AuthenticationProvider authProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(NoOpPasswordEncoder.getInstance()); // temporary — replaced with BCrypt later
        return provider;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(c -> c.disable())
            .authorizeHttpRequests(r -> r.anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults());
        return http.build();
    }
}
```
- `DaoAuthenticationProvider` — an `AuthenticationProvider` implementation designed for **DAO (Data Access Object) / database-based** authentication.
- `AuthenticationProvider` is an interface with one method: `authenticate(Authentication authentication)` — throws an exception if authentication fails, returns the `Authentication` object if it succeeds.

### 12.2 `MyUserDetailsService` — Implement `UserDetailsService`
```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepo repo;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = repo.findByUsername(username);
        if (user == null) {
            System.out.println("User 404: user not found");
            throw new UsernameNotFoundException("user not found");
        }
        return new UserPrincipal(user);
    }
}
```

### 12.3 `UserPrincipal` — Implement `UserDetails`
```java
public class UserPrincipal implements UserDetails {

    private User user;

    public UserPrincipal(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singleton(new SimpleGrantedAuthority("USER"));
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return true; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return true; }
}
```
- **`UserPrincipal`** = "the currently authenticated user" in Spring Security terminology (a *principal*).
- The four `is...` methods let you build features like account expiry, account locking, credential expiry (e.g., "change your password every 90 days") — hardcoded `true` here since they're not implemented in this demo.
- `getAuthorities()` is hardcoded to one role here; in a real app, add a `role`/`authority` column to the `users` table and read it dynamically.

**Test flow:** wrong password → `401`; correct password from DB → `200`; non-existent username → `User 404` (custom exception message printed and thrown).

---

## 13. Password Encoding — Hashing vs Encryption

| | **Encryption** (2-way) | **Hashing** (1-way) |
|---|---|---|
| Reversible? | Yes, with a key | No — cannot get plain text back |
| Risk | If the key leaks, *all* passwords are compromised | No key to steal |
| Use for passwords? | ❌ No | ✅ Yes |
| Algorithms | AES, DES, etc. | MD5, SHA-256 (deprecated for passwords), **BCrypt** (recommended) |

**Verification with hashing:** you don't decrypt the stored hash — you hash the *user's input* again and compare the two hashes.

### 13.1 Why BCrypt over plain SHA-256
- BCrypt repeats the hashing process over multiple **rounds** (a "work factor"), making brute-force attacks computationally expensive.
- Format: `$2A$12$<salt><hash>`
  - `$2A` → BCrypt version (`2A`, `2B`, `2Y` also exist)
  - `12` → number of rounds — actual work is **2¹² iterations**, not literally 12. Higher rounds = more secure but slower.
- Default strength is 10 rounds; commonly bumped to 12 for production.

---

## 14. Implementing BCrypt in Spring Security

`BCryptPasswordEncoder` ships **inside Spring Security** — no extra dependency needed.

### 14.1 Register a New User with an Encoded Password
```java
@Service
public class UserService {

    @Autowired
    private UserRepo repo;

    private BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12);

    public User saveUser(User user) {
        user.setPassword(encoder.encode(user.getPassword()));
        System.out.println("Encoded password: " + user.getPassword()); // debug only, remove in production
        return repo.save(user);
    }
}
```

```java
@RestController
public class UserController {

    @Autowired
    private UserService service;

    @PostMapping("/register")
    public User register(@RequestBody User user) {
        return service.saveUser(user);
    }
}
```
> Alternative: instead of `new BCryptPasswordEncoder(12)` as a field, define it as a `@Bean` inside `SecurityConfig` and inject it wherever needed — cleaner for larger apps.

### 14.2 Use BCrypt for Verifying Login (Authentication)
Only **one line changes** in `SecurityConfig`:
```java
@Bean
public AuthenticationProvider authProvider() {
    DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
    provider.setUserDetailsService(userDetailsService);
    provider.setPasswordEncoder(new BCryptPasswordEncoder(12)); // was NoOpPasswordEncoder before
    return provider;
}
```

> ⚠️ **Critical:** the strength (rounds) used to **encode** during registration must match the strength used to **verify** during login. `BCryptPasswordEncoder` internally reads the round value embedded in the stored hash (`$2A$12$...`) so verification still works even if you change the encoder's default strength later — but keep it consistent across your app for predictability and to avoid confusion.

### 14.3 End-to-End Flow Recap
```
Registration:
Controller (UserController) → Service (UserService: encode password with BCrypt) → Repo.save() → DB (hashed password stored)

Login:
Client sends username/password → DaoAuthenticationProvider
→ UserDetailsService.loadUserByUsername() → UserRepo.findByUsername()
→ wraps User into UserPrincipal → AuthenticationProvider compares
  BCryptPasswordEncoder.matches(rawPassword, storedHash) → success/failure
```
- Users registered with a **plain-text** password (before BCrypt was added) will **fail login** once `BCryptPasswordEncoder` is set as the provider's encoder — because it's no longer comparing plain text, it's comparing hashes. Only newly-registered (BCrypt-encoded) users can log in correctly after the switch.

---

## 15. Quick Reference — Key Classes & Interfaces

| Type | Kind | Purpose |
|---|---|---|
| `SecurityFilterChain` | Bean/Object | Defines the chain of security filters for all requests |
| `HttpSecurity` | Object (builder) | Used to configure `SecurityFilterChain` (`.csrf()`, `.authorizeHttpRequests()`, `.formLogin()`, `.httpBasic()`, `.sessionManagement()`) |
| `UserDetailsService` | Interface | One method: `loadUserByUsername(String)` — how Spring Security fetches user data |
| `InMemoryUserDetailsManager` | Class | Built-in `UserDetailsService` impl backed by hardcoded users |
| `UserDetails` | Interface | Represents a user's security-relevant data (username, password, authorities, account flags) |
| `User` (Spring's, `org.springframework.security.core.userdetails.User`) | Class | Built-in `UserDetails` implementation with a fluent builder (`User.builder()...build()`) |
| `AuthenticationProvider` | Interface | One method: `authenticate(Authentication)` — performs the actual auth check |
| `DaoAuthenticationProvider` | Class | `AuthenticationProvider` impl for DB/DAO-based authentication |
| `PasswordEncoder` | Interface | `.encode()` / `.matches()` |
| `BCryptPasswordEncoder` | Class | BCrypt implementation of `PasswordEncoder`, built into Spring Security |
| `NoOpPasswordEncoder` | Class | No encoding at all — **deprecated, demo-only** |
| `CsrfToken` | Class | Represents the CSRF token tied to a session |
| `Customizer<T>` | Functional interface | Single method `customize(T t)` — what powers the lambda-style config syntax |

---

## 16. Things to Explore Beyond This Module (Mentioned as "Coming Later")
- Role-based / method-level authorization (`@PreAuthorize`, `hasRole()`, etc.)
- JWT-based stateless authentication (mentioned as an option in Postman's Authorization tab)
- Adding a `roles`/`authorities` column to the `users` table instead of hardcoding `"USER"`
- Multi-Factor Authentication (MFA)
- Account expiry / credential expiry / account locking (the other `UserDetails` boolean methods)
- Always re-check the **current OWASP Top 10** list periodically — it's updated every few years.

---

## 17. Useful External Resources
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- Spring Security Reference Docs: https://docs.spring.io/spring-security/reference/index.html
- BCrypt explanation: https://en.wikipedia.org/wiki/Bcrypt
- Spring Initializr: https://start.spring.io/
