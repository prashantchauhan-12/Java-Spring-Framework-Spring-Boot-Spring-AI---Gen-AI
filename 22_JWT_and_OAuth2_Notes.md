# JWT and OAuth2 — Revision Notes

## 1. Cryptography Basics
Before understanding tokens, it is important to understand how data is secured on the internet.

### Encryption vs. Decryption
- **Encryption:** Converting normal text (plaintext) into an unreadable format (ciphertext).
- **Decryption:** Converting ciphertext back to normal text.
- **Symmetric Key Cryptography:** The same key is used for both encryption and decryption. Faster, but sharing the key securely is a major challenge.
- **Asymmetric Key Cryptography:** Uses a pair of keys (Public and Private). 
  - Data encrypted with a Public Key can *only* be decrypted by the corresponding Private Key.
  - Data encrypted with a Private Key can *only* be decrypted by the corresponding Public Key.

### Digital Signatures
A digital signature proves the authenticity of a message.
- Sender (A) encrypts the message using **A's Private Key**.
- Receiver (B) decrypts the message using **A's Public Key**. 
- If decryption works, it proves that the message definitely came from A.
- To ensure privacy as well as authenticity, double encryption is used: A encrypts the message with B's Public Key (privacy) AND A's Private Key (signature).

---

## 2. JSON Web Tokens (JWT)

### The Stateless Problem
In traditional monolithic architectures, after a user logs in, the server creates a **Session ID** and stores it in memory (or a shared database) and sends a cookie to the client. In a scaled microservices environment, managing distributed sessions becomes difficult.

### The JWT Solution
JWT allows the server to issue a cryptographically signed "pass" (token) to the user. The client stores this token and sends it with every subsequent request. The server doesn't need to look up a session in a database; it simply verifies the signature on the token to authenticate the user.

### Structure of a JWT
A JWT consists of three parts separated by dots (`.`):
1. **Header:** Contains the token type (JWT) and the signing algorithm being used (e.g., `HS256` for symmetric HMAC, or `RS256` for asymmetric RSA).
2. **Payload (Claims):** Contains the actual data (Subject/Username, Issued At, Expiration Time). *Never put sensitive data like passwords here, as the payload is merely base64 encoded, NOT encrypted.*
3. **Signature:** Created by hashing the Header, Payload, and a Secret Key known only to the server. This prevents tampering.

---

## 3. Implementing JWT in Spring Boot

### Step 1: Add Dependencies
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
</dependency>
```

### Step 2: Generate the Token (`JwtService`)
Create a service class to handle token creation using a secret key.
```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import java.security.Key;
import java.util.Date;
import java.util.HashMap;

public String generateToken(String username) {
    return Jwts.builder()
            .setClaims(new HashMap<>())
            .setSubject(username)
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 30)) // 30 minutes
            .signWith(getKey(), SignatureAlgorithm.HS256)
            .compact();
}

private Key getKey() {
    String secret = "your-very-long-secure-secret-key-that-should-be-kept-safe";
    return Keys.hmacShaKeyFor(secret.getBytes());
}
```

### Step 3: Validate the Token (`JwtFilter`)
Create a custom filter that extends `OncePerRequestFilter`. This filter intercepts every incoming request to check for the JWT.

```java
import org.springframework.web.filter.OncePerRequestFilter;

public class JwtFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) {
        String authHeader = request.getHeader("Authorization");
        String token = null;
        String username = null;

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            token = authHeader.substring(7);
            username = jwtService.extractUsername(token);
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            if (jwtService.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = 
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

### Step 4: Register the Filter in `SecurityConfig`
Add the filter to the `SecurityFilterChain` so it executes *before* the default `UsernamePasswordAuthenticationFilter`.
```java
http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
```

---

## 4. OAuth2 (Open Authorization)

OAuth2 allows your application to authenticate users via third-party providers (like Google, GitHub, Facebook) without ever handling the user's passwords.

### How to Implement Google/GitHub Login

#### Step 1: Add Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

#### Step 2: Configure `SecurityConfig`
Tell Spring Security to use `oauth2Login` instead of the default `formLogin`.
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .oauth2Login(Customizer.withDefaults()) // Replaces formLogin()
            .build();
}
```

#### Step 3: Register your App with the Provider
- **Google:** Go to Google Cloud Console > APIs & Services > Credentials. Create an "OAuth Client ID". Set the redirect URI to `http://localhost:8080/login/oauth2/code/google`.
- **GitHub:** Go to Developer Settings > OAuth Apps. Create a new App. Set the callback URL to `http://localhost:8080/login/oauth2/code/github`.

#### Step 4: Add Credentials to `application.properties`
Once you obtain the Client ID and Client Secret from Google/GitHub, add them to your properties file:

```properties
# Google OAuth2 Config
spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET

# GitHub OAuth2 Config
spring.security.oauth2.client.registration.github.client-id=YOUR_GITHUB_CLIENT_ID
spring.security.oauth2.client.registration.github.client-secret=YOUR_GITHUB_CLIENT_SECRET
```
*Note: Never commit your actual `client-secret` or JWT secret keys to a public repository.*
