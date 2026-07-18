# Building a Spring Boot Project (Job Portal App) — Comprehensive Revision Notes

## 1. What We're Building

A **Job Portal** web application where:
- **Employers** can post new job listings (Add Job)
- **Job seekers** can browse all available jobs (View All Jobs)

This is the first real project that ties together everything learned so far: Spring Boot, MVC, Controllers, Services, Repositories, JSP views, and the N-Tier architecture.

### Pages in the Application

| Page | URL | Purpose |
|---|---|---|
| Home | `/` or `/home` | Landing page with navigation |
| View All Jobs | `/viewAllJobs` | Lists all jobs with details |
| Add Job | `/addJob` | Form to post a new job |
| Success | (redirect from form) | Shows the just-added job as confirmation |

---

## 2. Project Setup (Spring Initializr)

Go to [start.spring.io](https://start.spring.io):

| Setting | Value |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.2+ |
| Group | `com.telusko` |
| Artifact | `jobApp` |
| Packaging | JAR |
| Java | 21 |

### Dependencies to Add

| Dependency | Purpose |
|---|---|
| **Spring Web** | Embedded Tomcat + Spring MVC |
| **Lombok** | Reduce boilerplate (getters, setters, constructors, toString) |

### Additional Dependencies (add manually in `pom.xml`)

Since we use JSP as the view technology, we need three more dependencies:

```xml
<dependencies>
    <!-- ═══ From Spring Initializr ═══ -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- ═══ Added Manually for JSP Support ═══ -->

    <!-- 1. Jasper: Compiles JSP files into Servlets -->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
    </dependency>

    <!-- 2. JSTL API: For forEach loops, conditionals in JSP (Jakarta version for Tomcat 10+) -->
    <dependency>
        <groupId>jakarta.servlet.jsp.jstl</groupId>
        <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
    </dependency>

    <!-- 3. JSTL Implementation: GlassFish provides the actual JSTL tag library -->
    <dependency>
        <groupId>org.glassfish.web</groupId>
        <artifactId>jakarta.servlet.jsp.jstl</artifactId>
    </dependency>
</dependencies>
```

> **Why JSTL?** The View All Jobs page uses `<c:forEach>` to loop through job posts. JSTL (JSP Standard Tag Library) provides these loop and conditional tags. After Tomcat 10 (Jakarta EE), the package changed from `javax.*` to `jakarta.*`, so you need the Jakarta JSTL dependencies.

### `application.properties`

```properties
# ViewResolver: tell Spring where JSP files are and their extension
spring.mvc.view.prefix=/views/
spring.mvc.view.suffix=.jsp
```

---

## 3. Project Architecture (N-Tier / Layered)

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Client   │ ──► │  Controller  │ ──► │   Service    │ ──► │  Repository  │
│ (Browser) │ ◄── │ @Controller  │ ◄── │  @Service    │ ◄── │ @Repository  │
└──────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                  Accepts HTTP         Business logic        Data access
                  requests             (processing)          (DB / list)
                  Returns views
```

| Layer | Class | Annotation | Responsibility |
|---|---|---|---|
| **Controller** | `JobController` | `@Controller` | Accept HTTP requests, call service, return views |
| **Service** | `JobService` | `@Service` | Business logic, coordinate between controller and repo |
| **Repository** | `JobRepo` | `@Repository` | Data access (in-memory list now, database later) |
| **Model** | `JobPost` | `@Component` + `@Data` | POJO holding job data (DTO — Data Transfer Object) |

> **DTO (Data Transfer Object)**: The `JobPost` object gets passed between layers — from Controller to Service to Repo. Objects transferred between layers like this are called DTOs.

---

## 4. The Model: `JobPost.java`

This POJO represents a single job posting. It uses **Lombok** annotations to eliminate boilerplate.

```java
package com.telusko.jobApp.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.stereotype.Component;

import java.util.List;

@Data                // Generates: getters, setters, toString(), hashCode(), equals()
@NoArgsConstructor   // Generates: public JobPost() {}
@AllArgsConstructor  // Generates: public JobPost(int postId, String postProfile, ...) {}
@Component           // Makes it a Spring-managed bean (so it can be injected)
public class JobPost {

    private int postId;
    private String postProfile;
    private String postDesc;
    private int reqExperience;
    private List<String> postTechStack;
}
```

### What Lombok Does Behind the Scenes

Without Lombok, you would need to write all of this manually:

```java
// @Data generates ALL of this automatically:
public int getPostId() { return postId; }
public void setPostId(int postId) { this.postId = postId; }
public String getPostProfile() { return postProfile; }
public void setPostProfile(String postProfile) { this.postProfile = postProfile; }
// ... getters/setters for ALL fields ...

@Override
public String toString() {
    return "JobPost{postId=" + postId + ", postProfile='" + postProfile + "', ...}";
}

@Override
public int hashCode() { /* generated */ }

@Override
public boolean equals(Object o) { /* generated */ }
```

> **Important**: When using Lombok in IntelliJ, you must **enable annotation processing**: `Settings → Build, Execution, Deployment → Compiler → Annotation Processors → Enable annotation processing ✅`

### Key Lombok Annotations Reference

| Annotation | What It Generates |
|---|---|
| `@Data` | Getters, setters, toString, hashCode, equals |
| `@NoArgsConstructor` | Default empty constructor: `JobPost()` |
| `@AllArgsConstructor` | Constructor with all fields: `JobPost(int, String, String, int, List)` |
| `@Getter` / `@Setter` | Only getters / only setters (if you don't want both) |
| `@ToString` | Only toString (if `@Data` is too much) |
| `@Builder` | Builder pattern: `JobPost.builder().postId(1).postProfile("Dev").build()` |

---

## 5. The Repository: `JobRepo.java`

The repo layer is responsible for **data access**. Here we use an in-memory `ArrayList` to simulate a database. Later, this can be swapped with a real database (JPA, JDBC) without changing Service or Controller code.

```java
package com.telusko.jobApp.repo;

import com.telusko.jobApp.model.JobPost;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

@Repository  // Tells Spring: this is the data access layer
public class JobRepo {

    // In-memory "database" — a mutable ArrayList with pre-loaded demo data
    List<JobPost> jobs = new ArrayList<>(Arrays.asList(
        new JobPost(1, "Java Developer",
            "Must have good experience in core Java and advanced Java",
            2, List.of("Core Java", "J2EE", "Spring Boot", "Hibernate")),

        new JobPost(2, "Frontend Developer",
            "Experience in building responsive web pages using React",
            3, List.of("HTML", "CSS", "JavaScript", "React")),

        new JobPost(3, "Data Scientist",
            "Strong background in machine learning and data analysis",
            4, List.of("Python", "Machine Learning", "TensorFlow", "SQL")),

        new JobPost(4, "DevOps Engineer",
            "CI/CD pipeline setup and cloud infrastructure management",
            5, List.of("Docker", "Kubernetes", "Jenkins", "AWS")),

        new JobPost(5, "Full Stack Developer",
            "Proficiency in both frontend and backend development",
            3, List.of("Java", "React", "Node.js", "MongoDB"))
    ));

    // Return all jobs
    public List<JobPost> getAllJobs() {
        return jobs;
    }

    // Add a new job to the list
    public void addJob(JobPost job) {
        jobs.add(job);
    }
}
```

### The `List.of()` vs `new ArrayList<>()` Bug

```java
// ❌ WRONG — creates an IMMUTABLE list. jobs.add() throws UnsupportedOperationException!
List<JobPost> jobs = List.of(
    new JobPost(1, "Java Dev", ...),
    new JobPost(2, "Frontend Dev", ...)
);

// ✅ CORRECT — wraps in a new ArrayList, making it MUTABLE
List<JobPost> jobs = new ArrayList<>(Arrays.asList(
    new JobPost(1, "Java Dev", ...),
    new JobPost(2, "Frontend Dev", ...)
));
```

> This was the actual bug encountered in the transcript. `List.of()` and `Arrays.asList()` return fixed-size or immutable lists. Wrapping with `new ArrayList<>(...)` makes it mutable, so `jobs.add()` works.

---

## 6. The Service: `JobService.java`

The service layer contains **business logic** and acts as a bridge between Controller and Repository. In this simple project, it just delegates calls, but in real apps it would contain validation, transformation, caching, etc.

```java
package com.telusko.jobApp.service;

import com.telusko.jobApp.model.JobPost;
import com.telusko.jobApp.repo.JobRepo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service  // Tells Spring: this is the service/business layer
public class JobService {

    @Autowired
    private JobRepo repo;  // Spring injects the JobRepo bean

    public List<JobPost> getAllJobs() {
        return repo.getAllJobs();
    }

    public void addJob(JobPost jobPost) {
        repo.addJob(jobPost);
    }
}
```

---

## 7. The Controller: `JobController.java`

The controller handles all HTTP requests, maps URLs, accepts form data, calls the service, and returns view names.

```java
package com.telusko.jobApp.controller;

import com.telusko.jobApp.model.JobPost;
import com.telusko.jobApp.service.JobService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.List;

@Controller
public class JobController {

    @Autowired
    private JobService service;

    // ─── Homepage: maps BOTH "/" and "/home" to the same page ───
    @RequestMapping({"/", "/home"})
    public String home() {
        return "home";  // → /views/home.jsp
    }

    // ─── Show the "Add Job" form ───
    @GetMapping("/addJob")
    public String addJob() {
        return "addJob";  // → /views/addJob.jsp
    }

    // ─── Handle form submission (POST request) ───
    @PostMapping("/handleForm")
    public String handleForm(JobPost jobPost) {
        // Spring auto-creates JobPost and binds form fields to it
        // (form input "name" attributes must match JobPost field names)
        service.addJob(jobPost);
        return "success";  // → /views/success.jsp
    }

    // ─── View all jobs ───
    @GetMapping("/viewAllJobs")
    public String viewAllJobs(Model m) {
        List<JobPost> jobs = service.getAllJobs();
        m.addAttribute("jobPosts", jobs);  // "jobPosts" is used in JSP forEach
        return "viewAllJobs";  // → /views/viewAllJobs.jsp
    }
}
```

### Key Concepts in the Controller

#### 1. Mapping Multiple URLs to One Method
```java
@RequestMapping({"/", "/home"})  // Array of URLs — both call the same method
public String home() { ... }
```

#### 2. `@GetMapping` vs `@PostMapping`
```java
@GetMapping("/addJob")          // For fetching the form page (GET request)
@PostMapping("/handleForm")     // For submitting form data (POST request)
```

| Annotation | HTTP Method | When to Use |
|---|---|---|
| `@GetMapping` | GET | Fetching/viewing data (pages, lists) |
| `@PostMapping` | POST | Submitting/sending data (forms, creation) |
| `@RequestMapping` | ALL (default GET) | General mapping (can handle any method) |

> **Why POST for forms?** With GET, form data appears in the URL (`/add?name=Java&exp=2`). POST keeps data in the request body — hidden from the URL. Essential for sensitive data.

#### 3. Auto-Binding Form Data to Object
```java
@PostMapping("/handleForm")
public String handleForm(JobPost jobPost) {
    // Spring creates a new JobPost() and calls:
    //   setPostId(101), setPostProfile("Java Dev"), etc.
    // Form <input name="postId"> → jobPost.setPostId()
    // Form <input name="postProfile"> → jobPost.setPostProfile()
}
```

#### 4. Passing Data to the View
```java
@GetMapping("/viewAllJobs")
public String viewAllJobs(Model m) {
    List<JobPost> jobs = service.getAllJobs();
    m.addAttribute("jobPosts", jobs);  // JSP accesses this via ${jobPosts}
    return "viewAllJobs";
}
```

---

## 8. The Views (JSP Pages)

### `home.jsp` — Landing Page

Contains navigation links:
```html
<a href="/viewAllJobs">View All Jobs</a>
<a href="/addJob">Add Job</a>
<a href="/home">Home</a>
```

### `addJob.jsp` — Job Posting Form

```jsp
<%@ page language="java" %>
<html>
<body>
    <h2>Post a New Job</h2>

    <!-- method="post" sends a POST request to /handleForm -->
    <form action="handleForm" method="post">
        Post ID:          <input type="text" name="postId"><br>
        Post Profile:     <input type="text" name="postProfile"><br>
        Post Description: <input type="text" name="postDesc"><br>
        Experience (yrs): <input type="text" name="reqExperience"><br>

        <!-- Multi-select for tech stack (sends as List<String>) -->
        Tech Stack:
        <select name="postTechStack" multiple>
            <option value="Java">Java</option>
            <option value="Python">Python</option>
            <option value="JavaScript">JavaScript</option>
            <option value="TypeScript">TypeScript</option>
            <option value="React">React</option>
            <!-- ... many more options ... -->
        </select><br>

        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

> **Critical**: The `name` attribute of each `<input>` must **exactly match** the `JobPost` field name. `name="postId"` → `setPostId()`, `name="postProfile"` → `setPostProfile()`, etc.

### `success.jsp` — Confirmation Page

```jsp
<%@ page language="java" isELIgnored="false" %>
<%@ page import="com.telusko.jobApp.model.JobPost" %>
<html>
<body>
    <%
        // Get the jobPost object from the request scope
        JobPost jobPost = (JobPost) request.getAttribute("jobPost");
    %>
    <h2>Job Posted Successfully!</h2>
    <p>Profile: <%= jobPost.getPostProfile() %></p>
    <p>Description: <%= jobPost.getPostDesc() %></p>
    <p>Experience: <%= jobPost.getReqExperience() %> years</p>
    <p>Tech Stack: <%= jobPost.getPostTechStack() %></p>
</body>
</html>
```

### `viewAllJobs.jsp` — List All Jobs (Uses JSTL `forEach`)

```jsp
<%@ page language="java" isELIgnored="false" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<html>
<body>
    <h2>All Available Jobs</h2>

    <!-- Loop through the "jobPosts" list sent from the Controller -->
    <c:forEach var="jobPost" items="${jobPosts}">
        <div class="job-card">
            <h3>${jobPost.postProfile}</h3>
            <p>${jobPost.postDesc}</p>
            <p>Experience: ${jobPost.reqExperience} years</p>
            <p>Tech Stack:
                <!-- Nested loop for the tech stack list -->
                <c:forEach var="tech" items="${jobPost.postTechStack}">
                    <span class="badge">${tech}</span>
                </c:forEach>
            </p>
        </div>
    </c:forEach>
</body>
</html>
```

> **`<c:forEach>`** is a JSTL tag for looping. `items="${jobPosts}"` references the model attribute. `var="jobPost"` is the loop variable. Access fields with `${jobPost.postProfile}`.

---

## 9. Complete Project File Structure

```
jobApp/
├── pom.xml
└── src/main/
    ├── java/com/telusko/jobApp/
    │   ├── JobAppApplication.java           ← @SpringBootApplication (main)
    │   ├── controller/
    │   │   └── JobController.java           ← @Controller
    │   ├── model/
    │   │   └── JobPost.java                 ← @Data @Component (POJO)
    │   ├── service/
    │   │   └── JobService.java              ← @Service
    │   └── repo/
    │       └── JobRepo.java                 ← @Repository (in-memory list)
    ├── resources/
    │   └── application.properties           ← prefix/suffix config
    └── webapp/
        ├── css/
        │   └── style.css                    ← Styling
        └── views/
            ├── home.jsp                     ← Landing page
            ├── addJob.jsp                   ← Job posting form
            ├── success.jsp                  ← Confirmation page
            └── viewAllJobs.jsp              ← Job listing page
```

---

## 10. Full Data Flow: Adding a Job

```
1. User visits http://localhost:8080/addJob
         │
         ▼
2. JobController.addJob() → returns "addJob" → ViewResolver → /views/addJob.jsp
         │
         ▼
3. User fills form: postId=101, postProfile="Java Dev", ...
   Clicks Submit → Browser sends POST /handleForm
         │
         ▼
4. JobController.handleForm(JobPost jobPost)
   ├─ Spring auto-creates JobPost object from form data
   ├─ Calls service.addJob(jobPost)
   │     └─ service calls repo.addJob(jobPost)
   │           └─ repo adds jobPost to ArrayList
   └─ Returns "success" → /views/success.jsp
         │
         ▼
5. success.jsp displays: "Java Dev, 1 year, [Java]"
```

### Full Data Flow: Viewing All Jobs

```
1. User clicks "View All Jobs" → GET /viewAllJobs
         │
         ▼
2. JobController.viewAllJobs(Model m)
   ├─ Calls service.getAllJobs()
   │     └─ service calls repo.getAllJobs()
   │           └─ repo returns ArrayList<JobPost> (6 entries now)
   ├─ m.addAttribute("jobPosts", jobs)
   └─ Returns "viewAllJobs" → /views/viewAllJobs.jsp
         │
         ▼
3. viewAllJobs.jsp loops through ${jobPosts} with <c:forEach>
   └─ Renders a card for each JobPost
```

---

## 11. Wiring Summary (Dependency Injection)

```
JobController                JobService                  JobRepo
┌───────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│ @Controller       │       │ @Service         │       │ @Repository      │
│                   │       │                  │       │                  │
│ @Autowired        │──────►│ @Autowired       │──────►│ List<JobPost>    │
│ JobService service│       │ JobRepo repo     │       │   jobs           │
│                   │       │                  │       │                  │
│ service.addJob()  │       │ repo.addJob()    │       │ jobs.add(job)    │
│ service.getAll()  │       │ repo.getAllJobs() │       │ return jobs      │
└───────────────────┘       └──────────────────┘       └──────────────────┘
```

Each layer only knows about the layer directly below it. The Controller never talks to the Repo directly.

---

## 12. Annotations Summary for This Project

| Annotation | Class | Purpose |
|---|---|---|
| `@Controller` | `JobController` | Marks as Spring MVC controller |
| `@Service` | `JobService` | Marks as service/business layer |
| `@Repository` | `JobRepo` | Marks as data access layer |
| `@Component` | `JobPost` | Makes it a Spring-managed bean |
| `@Data` | `JobPost` | Lombok: generates getters, setters, toString, etc. |
| `@NoArgsConstructor` | `JobPost` | Lombok: default constructor |
| `@AllArgsConstructor` | `JobPost` | Lombok: constructor with all fields |
| `@Autowired` | Controller, Service | Injects the dependency (Spring DI) |
| `@GetMapping` | Controller methods | Maps GET requests |
| `@PostMapping` | Controller methods | Maps POST requests |
| `@RequestMapping` | Controller methods | Maps any HTTP method |

---

## 13. Future Improvements (Mentioned in Transcript)

| Feature | Status | How |
|---|---|---|
| In-memory list storage | ✅ Done | `ArrayList` in `JobRepo` |
| Real database (PostgreSQL) | 🔜 Next | Replace `JobRepo` with Spring Data JPA |
| REST API (JSON instead of JSP) | 🔜 Later | Use `@RestController` instead of `@Controller` |
| React frontend | 🔜 Later | Separate React app calling the REST API |
| Auto-generated Post ID | 🔜 Later | Use `@GeneratedValue` with JPA |

> **Key insight**: When switching to a database, only the `JobRepo` class changes. The Service and Controller remain the same — that's the power of the layered architecture.

---

## 14. Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| JSP downloads instead of rendering | Missing `tomcat-embed-jasper` | Add Jasper dependency to `pom.xml` |
| 500: Cannot resolve `JobPost` type in JSP | Wrong package import in JSP `<%@ page import="..." %>` | Match the import to your actual package: `com.telusko.jobApp.model.JobPost` |
| `UnsupportedOperationException` on `jobs.add()` | Used `List.of()` which creates immutable list | Use `new ArrayList<>(Arrays.asList(...))` |
| 405: Method Not Allowed on form submit | Form sends POST but controller uses `@GetMapping` | Use `@PostMapping` for form handlers |
| `${variable}` prints as literal text | EL ignored in JSP | Add `isELIgnored="false"` in `<%@ page %>` |
| JSTL `<c:forEach>` not recognized | Missing JSTL dependencies | Add `jakarta.servlet.jsp.jstl-api` and `org.glassfish.web` dependencies |
| Lombok getters/setters not found | Annotation processing not enabled | Enable in IDE: Settings → Compiler → Annotation Processors → ✅ |
