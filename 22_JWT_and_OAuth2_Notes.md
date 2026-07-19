# JWT & OAuth2 — Complete Revision Notes

> Based on the Spring Security series (Student Management App). Covers:
> Cryptography basics → Digital Signatures → JWT (theory + full implementation) → OAuth2 (theory + Google/GitHub login).

---

## 1. Cryptography Basics

### 1.1 The Problem
When data travels over the internet (client ↔ server), anyone sitting in the middle can:
- **Read** the data (passive attack — eavesdropping)
- **Modify** the data (active attack — Man-in-the-Middle / MITM attack)

Example: A tells B "meet at 5 PM". An attacker **C** intercepts and changes it to "6 PM" before B receives it.

**Goal of cryptography:**
1. Others should not be able to **read** the data (even if intercepted).
2. Others should not be able to **modify** the data undetected.

Achieved via **Encryption** (plain text → cipher text) and **Decryption** (cipher text → plain text), using a **key**.

### 1.2 Symmetric Key Cryptography
- Same key used by both sender and receiver for encryption **and** decryption.
- A encrypts with key `K1` → B decrypts with the **same** `K1`.
- **Pros:** Fast, can use large key sizes → more secure.
- **Cons:**
  - Key must be shared **before** communication starts (can't send it over the internet — defeats the purpose).
  - With `n` people in a network, you need a **different key for every pair** (A-B needs K1, A-D needs K2, D-E needs K3...) → key management becomes very hard as network grows.
- **Algorithms:** AES, DES.

### 1.3 Asymmetric Key Cryptography
- Each person has a **key pair**: a **Public Key** (known to everyone) and a **Private Key** (known only to the owner).
- **Rule:** Whatever is encrypted with one key of the pair can only be decrypted with the **other** key of the same pair — never the same key for both.

**Encryption for confidentiality:**
```
A wants to send secret data to B:
  A encrypts using B's PUBLIC key
  B decrypts using B's PRIVATE key (only B has it)

If attacker C intercepts the encrypted packet:
  C does NOT have B's private key → cannot decrypt → data is safe
```

- **Algorithms:** RSA, ECC.
- Everyone's public key can sit in a shared/central repository — no risk in sharing it.

### 1.4 Digital Signature (Proving Identity)
Encryption alone secures data but does **not prove who sent it**. If C intercepts A's message to B, re-encrypts a fake message with B's public key, and sends it — B has no way to know it isn't really from A.

**Solution — sign with your own private key:**
```
A encrypts (signs) the message using A's PRIVATE key
B decrypts using A's PUBLIC key

If decryption succeeds → proof the message could only have come from A
  (only A knows A's private key)
If C tries to fake it, C can only sign with C's private key,
  so B would need C's public key to decrypt → mismatch detected
```

- Problem: this proves identity, but the message is **not secret** anymore — anyone with A's public key (i.e., everyone) can read it.

### 1.5 Combining Both — Security + Identity (Double Encryption)
```
Step 1: A encrypts message with B's PUBLIC key   → confidentiality
Step 2: A encrypts that result with A's PRIVATE key → signature/identity

B receives packet:
Step 1: B decrypts with A's PUBLIC key  → proves it came from A
Step 2: B decrypts with B's PRIVATE key → recovers the original secret message
```
If attacker C intercepts: C can undo A's signature layer (A's public key is public), but the inner layer is still encrypted with B's public key — C doesn't have B's private key, so C is stuck. ✅ Secure and authenticated.

This concept underlies how JWT signatures and OAuth2 tokens establish trust.

---

## 2. JWT (JSON Web Token)

### 2.1 The Analogy
- **Old way (session/ID based):** Cafe keeps a register/book with your ID (e.g., ID 102 = monthly subscriber). You show your ID each visit; staff checks the book.
  - **Problem:** Doesn't scale across branches/cities — the book is local to one location (just like `JSESSIONID` is tied to one server; doesn't work well with horizontal scaling unless you share a session store or use sticky load balancing).
- **New way (token/pass based):** Cafe gives you a **signed pass** that itself contains all the info (name, valid dates, what it covers). Any branch can verify the pass without checking a central book — because it's signed and self-contained.
  - This is exactly how **JWT** works.

### 2.2 What is JWT?
- **JWT = JSON Web Token**, an open industry standard — **RFC 7519**.
- A compact, URL-safe way to securely transfer claims (data) between two parties.
- Alternative to XML (too verbose) or plain JSON (still needs a compact/encoded transport format).

### 2.3 Structure of a JWT
A JWT string looks like: `xxxxx.yyyyy.zzzzz` — three Base64URL-encoded parts separated by dots:

```
HEADER.PAYLOAD.SIGNATURE
```

**1. Header** — algorithm & token type
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**2. Payload** — the actual claims/data
```json
{
  "sub": "harsh",
  "iat": 1738213080,
  "exp": 1738213260
}
```
- `sub` = subject (username)
- `iat` = issued-at time
- `exp` = expiration time
- You can add custom claims, but **keep the payload minimal** — some servers reject large headers, and it's sent on **every** request.
- ⚠️ **Payload is only Base64-encoded, NOT encrypted** — anyone can decode and read it (verify at jwt.io). **Never put secrets** here (passwords, SSNs, card numbers, etc.).

**3. Signature** — ensures the token hasn't been tampered with
```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```
- With **HS256** (HMAC-SHA256) → **symmetric**: one secret key shared between issuer and verifier.
- With **RS256** (RSA) → **asymmetric**: private key signs, public key verifies. Higher numbers (256/384/512) = bigger key = stronger.
- Signing does **not** encrypt the payload — it only guarantees **integrity** (nobody can silently modify the token). JWT *can* optionally be encrypted (JWE) but that's not the default.

### 2.4 JWT Authentication Flow
```
1. Client sends username + password → Server
2. Server verifies credentials → if valid, generates a JWT → sends it back
3. Client stores the JWT (localStorage/cookie/memory)
4. For every subsequent request, client sends the JWT
     (usually in header: Authorization: Bearer <token>)
5. Server verifies the token's signature + expiry → grants access
```
This is **stateless** — the server does not need to store session data; everything needed is inside the (signed) token.

---

## 3. JWT — Full Spring Boot Implementation

### 3.1 Maven Dependencies
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
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```
> `jjwt-api` = the API, `jjwt-impl` = runtime implementation, `jjwt-jackson` = JSON (de)serialization support (added to avoid JSON conversion exceptions). Reload Maven after adding.

### 3.2 SecurityConfig — allow register/login without auth, stateless session
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private JwtFilter jwtFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(customizer -> customizer.disable())
            .authorizeHttpRequests(request -> request
                .requestMatchers("register", "login").permitAll()
                .anyRequest().authenticated())
            .httpBasic(Customizer.withDefaults())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(new BCryptPasswordEncoder(12));
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```
Key points:
- `.permitAll()` on `register` and `login` — these two endpoints don't require prior authentication.
- `SessionCreationPolicy.STATELESS` — no `JSESSIONID`; every request must carry the JWT.
- `addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)` — our custom filter runs **before** Spring's default auth filter, so it can populate the `SecurityContext` from the token.
- `AuthenticationManager` bean is exposed via `AuthenticationConfiguration` so it can be injected/used in the controller.

### 3.3 User Entity, Registration
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String username;
    private String password;
    // getters & setters
}
```

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @Autowired
    private JwtService jwtService;

    @Autowired
    private AuthenticationManager authenticationManager;

    @PostMapping("/register")
    public User register(@RequestBody User user) {
        return userService.register(user);
    }

    @PostMapping("/login")
    public String login(@RequestBody User user) {
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword())
        );

        if (authentication.isAuthenticated()) {
            return jwtService.generateToken(user.getUsername());
        } else {
            return "failure";
        }
    }
}
```
- `UserService.register()` should encode the password with `BCryptPasswordEncoder` before saving.
- `authenticationManager.authenticate(...)` internally goes through `DaoAuthenticationProvider` → `UserDetailsService` → verifies password against the DB.
- On success, we don't return "success" — we generate and return the **actual JWT**.

### 3.4 JwtService — generate token, keys, extract claims, validate

```java
@Service
public class JwtService {

    private String secretKey = "";

    public JwtService() {
        try {
            KeyGenerator keyGen = KeyGenerator.getInstance("HmacSHA256");
            SecretKey sk = keyGen.generateKey();
            secretKey = Base64.getEncoder().encodeToString(sk.getEncoded());
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }

    // 1. GENERATE TOKEN -----------------------------------------
    public String generateToken(String username) {
        Map<String, Object> claims = new HashMap<>();

        return Jwts.builder()
                .setClaims(claims)
                .setSubject(username)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 3)) // 3 minutes
                .signWith(getKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    private SecretKey getKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    // 2. EXTRACT CLAIMS --------------------------------------------
    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimResolver) {
        Claims claims = extractAllClaims(token);
        return claimResolver.apply(claims);
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    // 3. VALIDATE TOKEN --------------------------------------------
    public boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```

**Notes:**
- `expiration = 1000 * 60 * 3` → `1000 ms` (1 sec) `× 60` (1 min) `× 3` = **3 minutes**. Change the multiplier for longer expiry (e.g., `× 30` for 30 min).
- Secret key can either be **hardcoded** (`Keys.hmacShaKeyFor("your-secret".getBytes())`) or **auto-generated** at startup (as shown above using `KeyGenerator`). Auto-generated keys reset on every restart — old tokens become invalid ("JWT signature does not match locally").
- `validateToken` checks two things: username in token matches the DB user, **and** token is not expired.

### 3.5 JwtFilter — intercept every request, verify token
```java
@Component
public class JwtFilter extends OncePerRequestFilter {

    @Autowired
    private JwtService jwtService;

    @Autowired
    ApplicationContext context;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain filterChain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");
        String token = null;
        String username = null;

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            token = authHeader.substring(7); // skip "Bearer " (7 chars)
            username = jwtService.extractUsername(token);
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails =
                context.getBean(MyUserDetailsService.class).loadUserByUsername(username);

            if (jwtService.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());

                authToken.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request));

                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response); // pass control to the next filter/servlet
    }
}
```

**Why `extends OncePerRequestFilter`?** Guarantees this filter logic runs **exactly once per incoming request**, and applies to **every** URL.

**Step-by-step of `doFilterInternal`:**
1. Read the `Authorization` header.
2. If it starts with `"Bearer "`, strip that prefix (7 characters: `B-e-a-r-e-r-<space>`) to get the raw token.
3. Extract the username from the token (via `JwtService`).
4. If username exists **and** there's no authentication already set in the `SecurityContext`:
   - Load `UserDetails` from the DB (via `MyUserDetailsService`, fetched through `ApplicationContext.getBean(...)` to avoid a circular dependency).
   - Validate the token (signature/username/expiry).
   - If valid, build a `UsernamePasswordAuthenticationToken` (Spring Security's generic auth object) and set it into the `SecurityContextHolder`.
5. Always call `filterChain.doFilter(request, response)` to continue the chain — otherwise the request never reaches the controller.

### 3.6 Testing with Postman
```
1. POST /register   { "username": "harsh", "password": "h@123" }
2. POST /login      { "username": "harsh", "password": "h@123" }
   → returns JWT string
3. GET /hello
   Header: Authorization: Bearer <paste JWT here>
   → "Hello World" (only works within the token's expiry window)
```
Decode/verify any token at **https://jwt.io** — paste the token to see header, payload, and signature validity.

### 3.7 Common Pitfalls (from the transcript)
- Using **GET** instead of **POST** for `/register` and `/login` → 405/errors.
- Forgetting to set Postman auth to **No Auth** for `/register` and `/login`.
- Reusing an **expired token** (default 3 min here) → "JWT signature does not match" / 403 errors after server restart (because the auto-generated secret key changes on every restart).
- Forgetting the **`Bearer `** prefix (with the trailing space) when sending the token.

---

## 4. OAuth2

### 4.1 What is OAuth2?
- **OAuth = Open Authorization**. Current version: **OAuth2**.
- Lets a user log in to a third-party app using an existing account (Google, GitHub, Facebook, etc.) **without sharing their username/password** with that third-party app.
- Roles:
  - **Resource Owner** — the user.
  - **Client** — the third-party application requesting access.
  - **Authorization Server** — Google/GitHub, which authenticates the user and issues tokens.
  - **Resource Server** — the API holding the user's data (e.g., Google's profile/email API).
- The client only gets access to the **specific scopes** the user consents to (e.g., email, profile) — never the raw password, and not necessarily everything in the account.

### 4.2 New Project Setup
Spring Initializr:
- Dependencies: **Spring Web**, **OAuth2 Client**
- `application.properties`:
```properties
server.port=8000
```
- Simple test controller:
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String greet() {
        return "Welcome to Telusko";
    }
}
```

### 4.3 SecurityConfig for OAuth2 Login
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .oauth2Login(Customizer.withDefaults());

        return http.build();
    }
}
```
- `anyRequest().authenticated()` — every endpoint requires login.
- `.oauth2Login(Customizer.withDefaults())` — replaces the default form-login with an OAuth2-provider-based login page (Google/GitHub buttons), using Spring's built-in defaults.
- Running this without provider credentials configured will throw a startup error (missing client-id/secret properties) — that's expected until you complete the next steps.

### 4.4 Google OAuth2 Setup
1. Go to **Google Cloud Console** → `console.cloud.google.com`.
2. **APIs & Services → Credentials → Create Credentials → OAuth Client ID**.
3. Complete the **OAuth consent screen** if prompted (app name + required fields).
4. Application type: **Web application**.
5. **Authorized redirect URI:**
   ```
   http://localhost:8000/login/oauth2/code/google
   ```
   (adjust the port to match your app; this is Spring Security's default OAuth2 redirect path pattern: `/login/oauth2/code/{registrationId}`)
6. Click **Create** → copy the **Client ID** and **Client Secret**.

**`application.properties`:**
```properties
spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET
```
Restart the app → visiting a protected URL now redirects to a **"Login with OAuth2"** page showing a Google option → clicking it takes you through Google's real login/consent flow → redirected back, authenticated.

### 4.5 GitHub OAuth2 Setup
1. GitHub → **Settings → Developer settings → OAuth Apps → New OAuth App**.
2. Fill in:
   - **Application name**: any name (e.g., "sample app")
   - **Homepage URL**: `http://localhost:8000`
   - **Authorization callback URL**:
     ```
     http://localhost:8000/login/oauth2/code/github
     ```
3. Click **Register application** → copy the **Client ID**.
4. Click **Generate a new client secret** (may require 2FA) → copy the **Client Secret**.

**`application.properties`:**
```properties
spring.security.oauth2.client.registration.github.client-id=YOUR_GITHUB_CLIENT_ID
spring.security.oauth2.client.registration.github.client-secret=YOUR_GITHUB_CLIENT_SECRET
```
Restart → the login page now shows **both** Google and GitHub buttons.

> ⚠️ **Gotcha from the transcript:** Copy-pasting the Google redirect URI pattern for GitHub (i.e., leaving it as `.../code/google`) causes a **404** when logging in via GitHub — the redirect URI registered on GitHub's OAuth app **must** exactly match `/login/oauth2/code/github`, matching the registration ID used in `application.properties` (`...registration.github...`).

### 4.6 Full property block (both providers together)
```properties
server.port=8000

spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET

spring.security.oauth2.client.registration.github.client-id=YOUR_GITHUB_CLIENT_ID
spring.security.oauth2.client.registration.github.client-secret=YOUR_GITHUB_CLIENT_SECRET
```
Never commit real client secrets to source control — use environment variables or a secrets manager in real projects (e.g., `${GOOGLE_CLIENT_SECRET}` placeholders pulled from env vars).

---

## 5. Quick Comparison — Session vs JWT vs OAuth2

| Aspect | Session (JSESSIONID) | JWT | OAuth2 |
|---|---|---|---|
| State | Stateful (server stores session) | Stateless (token carries all info) | Delegated auth (token issued by external provider) |
| Scaling | Needs shared session store / sticky sessions | Scales easily — any server can verify | Handled by the external provider |
| Where identity lives | Server-side session store | Signed token itself (payload) | Access/ID token from provider |
| Typical use | Traditional monoliths | REST APIs / SPA / mobile backends | "Login with Google/GitHub/Facebook" |

---

## 6. Extra — Things Worth Reviewing From the Web (not fully covered in the video)

These are natural next steps/gaps to fill in from official docs when revising:

1. **JWT best practices** (OWASP JWT Cheat Sheet): use strong signing algorithms (avoid `none`), keep expiry short, use refresh tokens for long-lived sessions, store tokens securely on the client (avoid `localStorage` for high-security apps due to XSS risk — prefer `HttpOnly` cookies), rotate signing keys.
2. **Refresh Tokens**: this video only implements a short-lived access token (3 min) with no refresh mechanism — in production you'd pair a short-lived **access token** with a longer-lived **refresh token** to avoid forcing re-login every few minutes.
3. **`jjwt` library version differences**: the builder API (`Jwts.builder().setClaims()...`) shown here is from `jjwt 0.11.x`. Newer `jjwt 0.12.x` has a slightly different fluent API (`Jwts.builder().claims()...expiration()...`). Check the [jjwt GitHub repo](https://github.com/jwtk/jjwt) for the version you use.
4. **Spring Security 6.x changes**: `.csrf(customizer -> customizer.disable())` and lambda-based DSL configuration (as used above) is the Spring Security 6 style; older tutorials use `.csrf().disable()` chained calls (Spring Security 5 style, now deprecated).
5. **OAuth2 grant types**: this video covers only the **Authorization Code flow** (used for "login with Google/GitHub" in web apps). Other grant types exist (Client Credentials, PKCE for mobile/SPA, Device Code) — worth reading Spring's official [OAuth2 Client documentation](https://docs.spring.io/spring-security/reference/servlet/oauth2/index.html) for a full picture.
6. **ID Token vs Access Token**: in real OAuth2/OIDC flows, Google/GitHub return both an **access token** (to call their APIs) and (for OpenID Connect) an **ID token** (a JWT identifying the user) — the video's flow abstracts this away since Spring Security's `oauth2Login()` handles it internally via `OAuth2User`/`OidcUser`.
7. **Extracting user profile info after OAuth2 login**: to actually use the logged-in user's name/email in your app (not shown in this video), you typically inject `@AuthenticationPrincipal OAuth2User oauth2User` in a controller and call `oauth2User.getAttributes()`.

---

## 7. One-Line Summary
- **Cryptography** = encrypt/decrypt data + sign it to prove identity.
- **JWT** = a signed, self-contained, stateless token (`header.payload.signature`) your server issues after login and verifies on every request via a custom filter.
- **OAuth2** = delegate the "who are you" question to a trusted third party (Google/GitHub) instead of managing your own username/password database.
