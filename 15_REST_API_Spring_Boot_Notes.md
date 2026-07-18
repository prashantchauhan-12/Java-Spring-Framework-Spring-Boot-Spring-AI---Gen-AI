# REST API Using Spring Boot — Comprehensive Revision Notes

## 1. Why REST? The Shift From Full-Stack to API-Based Architecture

### The Old Way (What We Did Before)

In the job portal app, we had **everything in one project**: the Java backend AND the JSP views. The server returned fully rendered **HTML pages**.

```
Browser ──► Spring Boot Server ──► Returns HTML (JSP pages)
```

### The Problem

What if you also need a **mobile app** (Android/iOS)? Mobile apps don't need HTML pages — they need **raw data** (JSON). Now you'd need two separate backends:
- Server 1: Returns HTML for browsers
- Server 2: Returns JSON for mobile apps

That's wasteful and unmaintainable.

### The Solution: REST API

Build **one backend** that returns only **data** (JSON/XML). Then any client can consume it:

```
┌───────────────┐
│  React App    │───┐
│  (Browser)    │   │
└───────────────┘   │     ┌────────────────────────┐     ┌──────────┐
                    ├────►│  Spring Boot REST API  │────►│ Database │
┌───────────────┐   │     │  Returns JSON data     │     └──────────┘
│  Mobile App   │───┘     │  (No HTML/JSP!)        │
│  (Android/iOS)│         └────────────────────────┘
└───────────────┘
```

> **Key insight**: The backend no longer returns views (JSP/Thymeleaf). It returns **JSON data**. The frontend (React, Angular, mobile app) handles the UI separately.

---

## 2. What is REST?

**REST** = **RE**presentational **S**tate **T**ransfer

### Core Concepts

| Concept | Explanation | Example |
|---|---|---|
| **Resource** | Any entity/data on the server | A job posting, an employee, a company |
| **State** | The current value of a resource at a given moment | Job #3 has `profile="Data Scientist"` right now |
| **Representation** | The format used to transfer state | JSON (`{"profile": "Data Scientist"}`) or XML |
| **Transfer** | Sending the state between client and server | HTTP request/response |
| **Stateless** | Server doesn't remember previous requests | Every request must carry all needed info (auth, params) |

### REST URL Convention: Use Nouns, Not Verbs

| ❌ Non-REST (action verbs) | ✅ REST (nouns) |
|---|---|
| `/viewAllJobs` | `/jobPosts` |
| `/addJob` | `/jobPost` (POST method) |
| `/deleteJob?id=3` | `/jobPost/3` (DELETE method) |
| `/getJobById?id=5` | `/jobPost/5` (GET method) |

> The **URL stays the same** — the **HTTP method** determines what action to perform.

---

## 3. HTTP Methods → CRUD Operations

| HTTP Method | CRUD | Purpose | Spring Annotation | Example URL |
|---|---|---|---|---|
| **GET** | **R**ead | Fetch data from server | `@GetMapping` | `GET /jobPosts` |
| **POST** | **C**reate | Send new data to server | `@PostMapping` | `POST /jobPost` |
| **PUT** | **U**pdate | Update existing data on server | `@PutMapping` | `PUT /jobPost` |
| **DELETE** | **D**elete | Remove data from server | `@DeleteMapping` | `DELETE /jobPost/3` |

### Same URL, Different Methods

```
GET    /jobPost/3  → Fetch job with ID 3
DELETE /jobPost/3  → Delete job with ID 3

POST   /jobPost    → Create a new job
PUT    /jobPost    → Update an existing job
GET    /jobPosts   → Fetch ALL jobs
```

> The URL is the same (`/jobPost`), but the HTTP method tells the server **what to do**. This is the REST way.

---

## 4. JSON — The Data Format

**JSON** = **J**ava**S**cript **O**bject **N**otation (but used by ALL languages, not just JavaScript).

```json
{
    "postId": 3,
    "postProfile": "Data Scientist",
    "postDesc": "Strong background in ML and data analysis",
    "reqExperience": 4,
    "postTechStack": ["Python", "Machine Learning", "TensorFlow", "SQL"]
}
```

### JSON vs XML Comparison

```json
// JSON — Compact, easy to read
{
    "postProfile": "Java Developer",
    "reqExperience": 2
}
```

```xml
<!-- XML — Verbose, more tags -->
<JobPost>
    <postProfile>Java Developer</postProfile>
    <reqExperience>2</reqExperience>
</JobPost>
```

> JSON is preferred because it's lighter (less bytes) and easier to read. Spring Boot uses JSON by default.

---

## 5. Project Setup for REST API

### What Changes From the Previous JSP Project?

| Aspect | JSP Project | REST API Project |
|---|---|---|
| Dependencies | `spring-boot-starter-web` + Jasper + JSTL | `spring-boot-starter-web` + Lombok (that's it!) |
| Controller annotation | `@Controller` | `@RestController` |
| Return type | `String` (view name) | `Object` / `List<>` (actual data) |
| Views (JSP) | Required in `webapp/views/` | **Removed entirely** |
| `application.properties` | prefix/suffix for ViewResolver | **Empty** (no view resolver needed) |
| Response format | HTML page | JSON data |

### `pom.xml` — Dependencies

```xml
<dependencies>
    <!-- Spring Web (includes embedded Tomcat + Jackson for JSON) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Lombok (reduce boilerplate) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

> No Jasper, no JSTL, no JSP — we're not rendering views anymore!

### What We Reuse From the JSP Project

These three classes are **identical** — no changes needed:
- `JobPost.java` (Model)
- `JobService.java` (Service)
- `JobRepo.java` (Repository)

> Only the **Controller** changes. Everything else stays the same — that's the power of layered architecture.

---

## 6. `@Controller` vs `@RestController`

### Problem: `@Controller` tries to resolve views

```java
@Controller  // ← Treats return value as a VIEW NAME
public class JobRestController {

    @GetMapping("/jobPosts")
    public List<JobPost> getAllJobs() {
        return service.getAllJobs();  // ❌ Spring tries to find "jobPosts.jsp"!
    }
}
```

This fails with a **500 error** because Spring treats the return value as a view name and tries to find `jobPosts.jsp`.

### Solution 1: `@Controller` + `@ResponseBody`

```java
@Controller
public class JobRestController {

    @GetMapping("/jobPosts")
    @ResponseBody  // ← Tells Spring: "return this as DATA, not a view name"
    public List<JobPost> getAllJobs() {
        return service.getAllJobs();  // ✅ Returns JSON data
    }
}
```

### Solution 2: `@RestController` (preferred)

```java
@RestController  // = @Controller + @ResponseBody on every method
public class JobRestController {

    @GetMapping("/jobPosts")
    public List<JobPost> getAllJobs() {
        return service.getAllJobs();  // ✅ Automatically returns JSON data
    }
}
```

> **`@RestController` = `@Controller` + `@ResponseBody`** on every method. Use this when ALL methods in the controller return data (not views).

---

## 7. The Complete REST Controller

```java
package com.telusko.springbootrest.controller;

import com.telusko.springbootrest.model.JobPost;
import com.telusko.springbootrest.service.JobService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@CrossOrigin(origins = "http://localhost:3000")  // Allow React frontend
public class JobRestController {

    @Autowired
    private JobService service;

    // ═══════════════════════════════════════════════════
    // GET all job posts
    // URL: GET http://localhost:8080/jobPosts
    // ═══════════════════════════════════════════════════
    @GetMapping("/jobPosts")
    public List<JobPost> getAllJobs() {
        return service.getAllJobs();
    }

    // ═══════════════════════════════════════════════════
    // GET a single job post by ID
    // URL: GET http://localhost:8080/jobPost/3
    // Uses @PathVariable to extract the ID from the URL
    // ═══════════════════════════════════════════════════
    @GetMapping("/jobPost/{postId}")
    public JobPost getJob(@PathVariable int postId) {
        return service.getJob(postId);
    }

    // ═══════════════════════════════════════════════════
    // POST (create) a new job post
    // URL: POST http://localhost:8080/jobPost
    // Body: JSON data of the new job
    // Uses @RequestBody to deserialize JSON → JobPost object
    // ═══════════════════════════════════════════════════
    @PostMapping("/jobPost")
    public JobPost addJob(@RequestBody JobPost jobPost) {
        service.addJob(jobPost);
        return service.getJob(jobPost.getPostId());  // Confirm from repo
    }

    // ═══════════════════════════════════════════════════
    // PUT (update) an existing job post
    // URL: PUT http://localhost:8080/jobPost
    // Body: JSON with updated fields (must include postId)
    // ═══════════════════════════════════════════════════
    @PutMapping("/jobPost")
    public JobPost updateJob(@RequestBody JobPost jobPost) {
        service.updateJob(jobPost);
        return service.getJob(jobPost.getPostId());
    }

    // ═══════════════════════════════════════════════════
    // DELETE a job post by ID
    // URL: DELETE http://localhost:8080/jobPost/3
    // ═══════════════════════════════════════════════════
    @DeleteMapping("/jobPost/{postId}")
    public String deleteJob(@PathVariable int postId) {
        service.deleteJob(postId);
        return "Deleted";
    }
}
```

---

## 8. Key Annotations Explained

### `@PathVariable` — Extract Value from the URL Path

```java
// URL: GET /jobPost/3
//                   ↑ this value is extracted

@GetMapping("/jobPost/{postId}")
public JobPost getJob(@PathVariable int postId) {
    // postId = 3
}
```

The `{postId}` in the URL is a **template variable**. `@PathVariable` binds it to the method parameter.

| If URL is... | `postId` value is... |
|---|---|
| `/jobPost/1` | `1` |
| `/jobPost/42` | `42` |
| `/jobPost/999` | `999` |

> **`@PathVariable` vs `@RequestParam`**: `@PathVariable` extracts from the URL path (`/jobPost/3`). `@RequestParam` extracts from the query string (`/jobPost?id=3`). REST APIs use `@PathVariable`.

### `@RequestBody` — Deserialize JSON from Request Body

```java
@PostMapping("/jobPost")
public JobPost addJob(@RequestBody JobPost jobPost) {
    // JSON from the POST body is automatically converted to a JobPost object
    // by the Jackson library
}
```

When a client sends this JSON in the request body:
```json
{
    "postId": 6,
    "postProfile": "iOS Developer",
    "postDesc": "Experience in mobile development",
    "reqExperience": 2,
    "postTechStack": ["Swift", "iOS", "Xcode"]
}
```

Spring (via Jackson) automatically creates a `JobPost` object with those values.

### `@ResponseBody` — Serialize Object to JSON

```java
@ResponseBody  // Tells Spring: convert the return value to JSON
public List<JobPost> getAllJobs() { ... }
```

> With `@RestController`, `@ResponseBody` is **implied on every method**, so you don't need to write it.

### `@CrossOrigin` — Allow Frontend Requests

```java
@CrossOrigin(origins = "http://localhost:3000")  // React runs on port 3000
```

Without this, the browser blocks requests from a different origin (port 3000 → port 8080) due to **CORS** (Cross-Origin Resource Sharing) security policy.

---

## 9. Updated Service and Repository (New Methods)

### `JobService.java` — Added `getJob`, `updateJob`, `deleteJob`

```java
@Service
public class JobService {

    @Autowired
    private JobRepo repo;

    public List<JobPost> getAllJobs() {
        return repo.getAllJobs();
    }

    public void addJob(JobPost jobPost) {
        repo.addJob(jobPost);
    }

    public JobPost getJob(int postId) {
        return repo.getJob(postId);
    }

    public void updateJob(JobPost jobPost) {
        repo.updateJob(jobPost);
    }

    public void deleteJob(int postId) {
        repo.deleteJob(postId);
    }
}
```

### `JobRepo.java` — New Methods

```java
@Repository
public class JobRepo {

    List<JobPost> jobs = new ArrayList<>(Arrays.asList(
        // ... existing demo data ...
    ));

    public List<JobPost> getAllJobs() {
        return jobs;
    }

    public void addJob(JobPost job) {
        jobs.add(job);
    }

    // ─── Get a single job by ID ───
    public JobPost getJob(int postId) {
        for (JobPost job : jobs) {
            if (job.getPostId() == postId) {
                return job;
            }
        }
        return null;  // Not found
    }

    // ─── Update an existing job ───
    public void updateJob(JobPost jobPost) {
        for (JobPost existingJob : jobs) {
            if (existingJob.getPostId() == jobPost.getPostId()) {
                existingJob.setPostProfile(jobPost.getPostProfile());
                existingJob.setPostDesc(jobPost.getPostDesc());
                existingJob.setReqExperience(jobPost.getReqExperience());
                existingJob.setPostTechStack(jobPost.getPostTechStack());
                return;
            }
        }
    }

    // ─── Delete a job by ID ───
    public void deleteJob(int postId) {
        for (JobPost job : jobs) {
            if (job.getPostId() == postId) {
                jobs.remove(job);
                return;
            }
        }
    }
}
```

---

## 10. Testing with Postman

**Postman** is a REST client for testing API endpoints without needing a frontend.

### GET All Jobs

```
Method: GET
URL:    http://localhost:8080/jobPosts
Body:   (none)
```

### GET One Job

```
Method: GET
URL:    http://localhost:8080/jobPost/3
Body:   (none)
```

### POST (Create) a New Job

```
Method:       POST
URL:          http://localhost:8080/jobPost
Body Type:    raw → JSON
Body Content:
{
    "postId": 6,
    "postProfile": "iOS Developer",
    "postDesc": "Experience in mobile development for iOS",
    "reqExperience": 2,
    "postTechStack": ["Swift", "iOS", "Xcode"]
}
```

### PUT (Update) an Existing Job

```
Method:       PUT
URL:          http://localhost:8080/jobPost
Body Type:    raw → JSON
Body Content:
{
    "postId": 2,
    "postProfile": "React Developer",
    "postDesc": "Experience in building responsive web pages using React",
    "reqExperience": 2,
    "postTechStack": ["HTML", "CSS", "JavaScript", "React"]
}
```

### DELETE a Job

```
Method: DELETE
URL:    http://localhost:8080/jobPost/3
Body:   (none)
```

---

## 11. Jackson Library — How JSON Conversion Works

**Jackson** is the library that converts Java objects ↔ JSON. It's included automatically with `spring-boot-starter-web`.

```
Java Object (JobPost) ──── Jackson ──── JSON string
       ↕                                    ↕
  @RequestBody (JSON → Object)    @ResponseBody (Object → JSON)
```

| Direction | Annotation | Jackson Process | Term |
|---|---|---|---|
| JSON → Java Object | `@RequestBody` | **Deserialization** | Reading JSON into an object |
| Java Object → JSON | `@ResponseBody` / `@RestController` | **Serialization** | Writing an object to JSON |

> You can find Jackson in your external libraries: `jackson-databind`, `jackson-core`, `jackson-annotations`.

---

## 12. Content Negotiation (JSON vs XML)

By default, Spring Boot returns JSON. But clients can request XML too.

### Requesting XML from Postman

In Postman **Headers**:
```
Key:   Accept
Value: application/xml
```

Without XML support → **406 Not Acceptable** error.

### Adding XML Support

Add the Jackson XML dependency to `pom.xml`:

```xml
<!-- Jackson XML — enables XML serialization/deserialization -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.15.3</version> <!-- Match your Jackson core version -->
</dependency>
```

> After adding this, clients can request either JSON or XML using the `Accept` header.

### Restricting Response Format (produces / consumes)

```java
// This endpoint ONLY returns JSON — XML requests get 406 error
@GetMapping(path = "/jobPosts", produces = {"application/json"})
public List<JobPost> getAllJobs() { ... }

// This endpoint ONLY accepts XML input — JSON posts get 415 error
@PostMapping(path = "/jobPost", consumes = {"application/xml"})
public JobPost addJob(@RequestBody JobPost jobPost) { ... }
```

| Attribute | Purpose | Error if Violated |
|---|---|---|
| `produces` | Specifies what format the server **returns** | 406 Not Acceptable |
| `consumes` | Specifies what format the server **accepts** | 415 Unsupported Media Type |

---

## 13. Connecting a React Frontend (Overview)

The React frontend is a separate project that calls the Spring Boot REST API.

### Key Steps

1. Run the Spring Boot backend on port `8080`
2. In React, use `axios` (or `fetch`) to call `http://localhost:8080/jobPosts`
3. Add `@CrossOrigin(origins = "http://localhost:3000")` on the controller
4. React renders the JSON data in its UI

```javascript
// React: AllPosts component (simplified)
import axios from 'axios';

function AllPosts() {
    const [posts, setPosts] = useState([]);

    useEffect(() => {
        axios.get("http://localhost:8080/jobPosts")
             .then(res => setPosts(res.data));
    }, []);

    return posts.map(post => (
        <div key={post.postId}>
            <h3>{post.postProfile}</h3>
            <p>{post.postDesc}</p>
            <p>Experience: {post.reqExperience} years</p>
        </div>
    ));
}
```

> This is just to show how the backend connects. The React frontend is provided as a separate project — you just need to change the URL to your Spring Boot server.

---

## 14. Complete Project File Structure

```
springbootrest/
├── pom.xml                                          ← Only web + lombok
└── src/main/
    ├── java/com/telusko/springbootrest/
    │   ├── SpringbootrestApplication.java           ← @SpringBootApplication
    │   ├── controller/
    │   │   └── JobRestController.java               ← @RestController (5 endpoints)
    │   ├── model/
    │   │   └── JobPost.java                         ← @Data @Component
    │   ├── service/
    │   │   └── JobService.java                      ← @Service
    │   └── repo/
    │       └── JobRepo.java                         ← @Repository
    └── resources/
        └── application.properties                   ← EMPTY (no view resolver!)
```

> Notice: **No `webapp/` folder, no `views/`, no JSP files, no `prefix`/`suffix` in properties.** This is a pure API server.

---

## 15. Full Annotations Summary

| Annotation | Purpose |
|---|---|
| `@RestController` | `@Controller` + `@ResponseBody` on every method. Returns data, not views. |
| `@ResponseBody` | Tells Spring to serialize the return value to JSON (not treat as view name) |
| `@RequestBody` | Deserializes JSON from the request body into a Java object |
| `@PathVariable` | Extracts a value from the URL path (e.g., `/jobPost/{id}`) |
| `@GetMapping` | Maps HTTP GET requests |
| `@PostMapping` | Maps HTTP POST requests |
| `@PutMapping` | Maps HTTP PUT requests |
| `@DeleteMapping` | Maps HTTP DELETE requests |
| `@CrossOrigin` | Allows requests from a different origin (CORS) |

---

## 16. API Endpoints Summary Table

| Method | URL | Body | Action | Returns |
|---|---|---|---|---|
| `GET` | `/jobPosts` | — | Get all jobs | `List<JobPost>` as JSON |
| `GET` | `/jobPost/{id}` | — | Get one job by ID | `JobPost` as JSON |
| `POST` | `/jobPost` | JSON `JobPost` | Create a new job | Created `JobPost` |
| `PUT` | `/jobPost` | JSON `JobPost` | Update existing job | Updated `JobPost` |
| `DELETE` | `/jobPost/{id}` | — | Delete a job by ID | `"Deleted"` string |

---

## 17. Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| **500: Circular view path** | Using `@Controller` — treats return as view name | Use `@RestController`, or add `@ResponseBody` |
| **404: Not Found** | No mapping for the URL/method combination | Add the correct `@GetMapping`/`@PostMapping` etc. |
| **405: Method Not Allowed** | Client sends POST but server has only GET for that URL | Add the matching HTTP method mapping |
| **406: Not Acceptable** | Client requests XML but no XML library installed | Add `jackson-dataformat-xml` dependency |
| **415: Unsupported Media Type** | Client sends JSON but server expects XML (`consumes` restriction) | Match Content-Type to what the server consumes |
| **Network Error (CORS)** | React on port 3000 calling Spring on port 8080 | Add `@CrossOrigin(origins = "http://localhost:3000")` |
| **ConcurrentModificationException** | Modifying a list while iterating (in delete) | Add `return` after `jobs.remove(job)` to exit the loop |
| **Null response for single job** | Job ID doesn't exist in the list | Return a proper 404 response or check ID before querying |
