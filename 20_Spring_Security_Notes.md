# Spring Security — Comprehensive Revision Notes

## 1. Introduction: Why Spring Security?

When building an application, it's not enough that the code simply works and performs well. It must also be secure. Security is critical, especially when handling personal data, medical records, or financial information. A breach can lead to severe penalties and loss of user trust.

### OWASP Top 10
The Open Web Application Security Project (OWASP) publishes a list of the top 10 most critical security risks to web applications (updated roughly every 4 years). Some key vulnerabilities include:
- **Broken Access Control:** Users accessing data/functions they shouldn't.
- **Cryptographic Failures:** Storing or transmitting passwords/data in plain text or using weak encryption (e.g., MD5).
- **Injection Attacks (SQL/NoSQL):** Hackers injecting malicious SQL statements (e.g., `' OR 1=1 --`) to bypass authentication.
- **Security Misconfigurations:** Leaving default passwords or configurations active.

**Spring Security** helps mitigate many of these risks right out of the box.

---

## 2. Default Spring Security Behavior

Simply adding the `spring-boot-starter-security` dependency secures your entire application by default.

### What happens when you add the dependency?
1. **All Endpoints Secured:** You can no longer access endpoints like `/hello` without authenticating.
2. **Default Login Form:** Spring auto-generates a login page (`/login`) and a logout page (`/logout`).
3. **Default Credentials:**
   - **Username:** `user`
   - **Password:** Auto-generated and printed in the console at application startup (e.g., `Using generated security password: <uuid>`).
4. **Filter Chain:** Behind the scenes, Spring intercepts the request *before* it reaches the `DispatcherServlet`. It passes the request through a `SecurityFilterChain` (a series of filters checking for CORS, CSRF, Authentication, etc.).

---

## 3. Session Management and CSRF

### Sessions
When you log in via the browser, Spring Security creates a session and gives the browser a cookie containing a `JSESSIONID`. Subsequent requests (like navigating to `/about`) use this session ID so you don't have to log in repeatedly.

### CSRF (Cross-Site Request Forgery)
**The Threat:** A malicious website steals your active `JSESSIONID` and makes unauthorized POST/PUT/DELETE requests on your behalf to the secure server.

**The Fix:** By default, Spring Security blocks `POST`, `PUT`, and `DELETE` requests unless they contain a valid **CSRF Token**.
- `GET` requests are allowed because they shouldn't modify server state.
- To make a `POST` request (e.g., via Postman), you must first fetch the CSRF token and pass it in the header (`X-CSRF-TOKEN`).

*Fetching the CSRF token manually (for testing):*
```java
@GetMapping("/csrf-token")
public CsrfToken getCsrfToken(HttpServletRequest request) {
    return (CsrfToken) request.getAttribute("_csrf");
}
```

---

## 4. Custom Configuration (Bypassing Defaults)

If you're building a REST API, you don't want a login form. You want your API to be **stateless** and to accept Basic Auth or Tokens. To change default behavior, create a Configuration class.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.config.Customizer;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        
        // 1. Disable CSRF (safe for stateless REST APIs)
        http.csrf(customizer -> customizer.disable());
        
        // 2. Authenticate all requests
        http.authorizeHttpRequests(request -> request.anyRequest().authenticated());
        
        // 3. Enable Basic Authentication (for Postman/REST clients)
        http.httpBasic(Customizer.withDefaults());
        
        // 4. Make session STATELESS (no JSESSIONID, forces re-auth every request)
        http.sessionManagement(session -> 
            session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
}
```

---

## 5. Database-Backed Authentication (The Architecture)

To authenticate users against a database, you must override the default `AuthenticationProvider` and provide a custom `UserDetailsService`.

**The Flow:**
1. User provides credentials.
2. `AuthenticationProvider` talks to `UserDetailsService`.
3. `UserDetailsService` uses a `UserRepository` (Spring Data JPA) to fetch the user from the DB.
4. The DB User is mapped to a `UserDetails` object (a Spring Security interface).
5. The `AuthenticationProvider` compares the stored password (via `PasswordEncoder`) with the input password.

### Step 1: The Model and Repository
Create a `User` entity that matches your database table.

```java
@Data
@Entity
@Table(name = "users") // Mapping to 'users' because 'user' is often a reserved keyword in DBs
public class User {
    @Id
    private int id;
    private String username;
    private String password;
}

@Repository
public interface UserRepo extends JpaRepository<User, Integer> {
    User findByUsername(String username); // Custom query method
}
```

### Step 2: Implementing UserDetails (The Principal)
Spring Security doesn't understand your custom `User` class. It only understands `UserDetails`. We must create an adapter class (often called a Principal).

```java
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collection;
import java.util.Collections;

public class UserPrincipal implements UserDetails {

    private User user;

    public UserPrincipal(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // Hardcoding the role to 'USER' for now
        return Collections.singleton(new SimpleGrantedAuthority("USER"));
    }

    @Override
    public String getPassword() { return user.getPassword(); }

    @Override
    public String getUsername() { return user.getUsername(); }

    // Set the following to true to bypass account locks/expiration checks
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

### Step 3: Implementing UserDetailsService
This service fetches the user from the DB and returns the `UserPrincipal`.

```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepo repo;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = repo.findByUsername(username);
        
        if (user == null) {
            System.out.println("User Not Found");
            throw new UsernameNotFoundException("user not found");
        }
        
        return new UserPrincipal(user); // Wrap DB user in UserDetails interface
    }
}
```

### Step 4: Connecting the Provider in `SecurityConfig`
Finally, tell Spring to use a DAO-based Authentication Provider hooked up to your custom service and a `BCryptPasswordEncoder`.

```java
@Autowired
private UserDetailsService userDetailsService;

@Bean
public AuthenticationProvider authenticationProvider() {
    DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
    
    // Set our custom UserDetailsService to talk to the DB
    provider.setUserDetailsService(userDetailsService);
    
    // Set the password encoder (BCrypt)
    provider.setPasswordEncoder(new BCryptPasswordEncoder(12)); // 12 rounds of hashing
    
    return provider;
}
```

---

## 6. Password Encoding (Bcrypt)

Storing passwords in plain text is a severe security risk. We must use **Hashing** (a one-way encryption that cannot be decrypted).

**BCrypt** is a powerful hashing algorithm used by Spring Security. It applies a salt and hashes the password multiple times (rounds) to make brute-force attacks computationally expensive.
- E.g., `12` rounds means `2^12` iterations of hashing.

### Registering a User with BCrypt
When a user signs up, you must hash their password before saving it to the database.

```java
@Service
public class UserService {

    @Autowired
    private UserRepo repo;

    // Use BCrypt with a strength of 12 rounds
    private BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12);

    public User saveUser(User user) {
        // Overwrite plain text password with hashed password
        user.setPassword(encoder.encode(user.getPassword()));
        return repo.save(user);
    }
}
```
Now, the database stores the hashed BCrypt string (e.g., `$2a$12$...`), and the `AuthenticationProvider` automatically handles verifying the plain-text login attempt against the stored hash.
