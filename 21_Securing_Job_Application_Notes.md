# Securing the Job Application Portal — Revision Notes

## 1. Introduction
After learning the fundamentals of Spring Security (in a separate demo project), the next logical step is to integrate it into an actual, existing project—in this case, the **Job Portal Application**. The goal is to secure the endpoints (like `/jobPost`) that were previously completely open to the public.

By migrating the security code from the demo project, we can protect the REST APIs and ensure only authenticated users can access the job postings.

---

## 2. Cross-Origin Resource Sharing (CORS)

Before diving into the security configuration, it's crucial to address a common issue when building full-stack applications: **CORS**.

### The Problem
When you build an application with separate frontend and backend architectures:
- **Backend (Spring Boot):** Runs on `localhost:8080`
- **Frontend (React/Angular/Vue):** Runs on a different port, e.g., `localhost:3000`

By default, modern web browsers and Spring Security block requests made from a different origin (domain, protocol, or port) to protect against malicious cross-site requests. This is a built-in defense mechanism against some of the OWASP Top 10 vulnerabilities.

### The Solution: `@CrossOrigin`
To allow your frontend application to communicate with your backend, you must explicitly tell Spring to accept requests from that specific origin.

**Implementation on a Controller:**
```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RestController;

@RestController
@CrossOrigin(origins = "http://localhost:3000") // Allows requests ONLY from this origin
public class JobRestController {
    // ... endpoints ...
}
```
*Note: Using `@CrossOrigin` without specifying an origin (or using `*`) will allow requests from ANY origin. While useful for public APIs, it is not recommended for secure/private data.*

---

## 3. Integrating Spring Security

To secure the Job Portal, we migrate the configuration and classes we built previously. 

### Step 1: Add the Dependency
Add the Spring Security starter to your `pom.xml`. This instantly secures all endpoints with a default configuration (generating a default `user` and auto-generated password).

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Step 2: Migrate Configuration and Classes
Instead of rewriting the security layer, copy the necessary classes from the security demo project into the Job Portal project:

1. **`SecurityConfig`**: The `@Configuration` class that defines the `SecurityFilterChain`. It disables CSRF, forces authentication on all requests, and configures the session to be stateless (ideal for REST APIs).
2. **`User` Entity**: The JPA model representing the `users` table in the database.
3. **`UserRepo`**: The Spring Data JPA repository to fetch users by username.
4. **`UserPrincipal`**: The class implementing `UserDetails`, acting as a bridge between your custom `User` entity and Spring Security.
5. **`UserDetailsService` Implementation**: The service class that uses the `UserRepo` to load the user from the database during authentication.
6. **`UserService`**: (Optional but recommended) The service to handle new user registration and password hashing using `BCryptPasswordEncoder`.

### Step 3: Database Verification
Ensure the `application.properties` file has the correct database connection details. The authentication system will rely on the `users` table containing hashed (BCrypt) passwords.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=0000
```

---

## 4. Testing the Secured Application

Once the security classes are in place and the application is restarted:

1. **Without Authentication:** Sending a GET request to `http://localhost:8080/jobPost` via Postman without headers will return a `401 Unauthorized` status.
2. **With Basic Auth:** In Postman, go to the Authorization tab, select **Basic Auth**, and provide a valid username and BCrypt-hashed password (e.g., `avni` / `a@123`). The request will return a `200 OK` status with the job post data.

---

## 5. Future Enhancements: Role-Based Authorization

While the application is now secure (requiring authentication), it currently lacks **Authorization** rules based on user roles. 

In a real-world Job Portal:
- **Employers (Role: ADMIN/EMPLOYER):** Should be able to add, edit, and delete job posts.
- **Job Seekers (Role: USER):** Should only be able to view and search for job posts.

*Implementation Idea:* You can add an `authority` or `role` column to the `users` table, map it in the `UserPrincipal`, and then use method-level security (e.g., `@PreAuthorize("hasRole('EMPLOYER')")`) on your Controller methods.
