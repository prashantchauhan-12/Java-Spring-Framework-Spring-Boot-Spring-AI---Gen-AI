# Spring Boot Web & MVC: Comprehensive Revision Notes

## 1. Introduction: How Web Applications Work

A web application has a **client** (browser/mobile) and a **server** (backend). The flow is:

```
┌────────────┐    HTTP Request     ┌────────────────┐     Query      ┌──────────┐
│   Client   │ ──────────────────► │  Server (Java) │ ─────────────► │ Database │
│  (Browser) │ ◄────────────────── │   (Tomcat)     │ ◄───────────── │          │
└────────────┘    HTTP Response    └────────────────┘     Result     └──────────┘
```

- **Static content** (HTML/CSS) — looks the same for everyone.
- **Dynamic content** — data changes per user. This is why you need a backend (Java, Spring).
- **Modern architecture**: Spring Boot (backend, returns JSON) + React/Angular (frontend).
- **Traditional architecture**: Servlet/Spring MVC + JSP/Thymeleaf (server renders full HTML pages).

---

## 2. Servlets — The Foundation Behind Spring MVC

A **Servlet** is a Java class that accepts HTTP requests and generates responses. Servlets run inside a **Servlet Container** (web container), the most popular being **Apache Tomcat**.

> Spring MVC uses Servlets behind the scenes. Understanding Servlets helps you know what Spring Boot automates for you.

### Key Concepts
- **Servlet Container (Tomcat)**: Manages the lifecycle of Servlets, routes requests, handles threads.
- **External Tomcat**: A standalone server you download, deploy `.war` files into its `webapps/` folder.
- **Embedded Tomcat**: A Tomcat instance bundled inside your project (what Spring Boot uses). You just run the JAR.
- **`.war` (Web Archive)**: Package format for deploying to external Tomcat.
- **`.jar` (Java Archive)**: Package format when using embedded Tomcat (Spring Boot default).

### HTTP Methods (Request Types)

| Method | Purpose | Servlet Method | Example |
|--------|---------|----------------|---------|
| GET | Retrieve data | `doGet()` | Loading a page, fetching student list |
| POST | Send/submit data | `doPost()` | Submitting a form, creating a user |
| PUT | Update data | `doPut()` | Editing a profile |
| DELETE | Remove data | `doDelete()` | Deleting an order |

> The generic `service()` method handles ALL request types. Use specific methods (`doGet`, `doPost`) for clarity.

---

## 3. Building a Servlet App Manually (Without Spring)

### Dependencies in `pom.xml`

```xml
<dependencies>
    <!-- Jakarta Servlet API (after 2018, javax → jakarta) -->
    <dependency>
        <groupId>jakarta.servlet</groupId>
        <artifactId>jakarta.servlet-api</artifactId>
        <version>4.0.4</version>
    </dependency>

    <!-- Embedded Tomcat Core -->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-core</artifactId>
        <version>8.5.96</version>
    </dependency>
</dependencies>
```

### Creating a Servlet (`HelloServlet.java`)

```java
package com.telusko;

import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class HelloServlet extends HttpServlet {

    // service() handles ALL request types (GET, POST, etc.)
    // For specific handling, override doGet() or doPost() instead.
    public void service(HttpServletRequest req, HttpServletResponse res) throws IOException {
        System.out.println("In Service Method");

        // Tell the browser that we're sending HTML content
        res.setContentType("text/html");

        // Get a PrintWriter to write data INTO the response (sent to client)
        // Think: response = blank paper, getWriter() = getting a pen
        PrintWriter out = res.getWriter();
        out.println("<h2>Hello World from Servlet!</h2>");
    }
}
```

### Manual Embedded Tomcat Configuration (`App.java`)

```java
package com.telusko;

import org.apache.catalina.Context;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.startup.Tomcat;

public class App {
    public static void main(String[] args) throws LifecycleException {

        System.out.println("Starting application...");

        // 1. Create Tomcat instance
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080); // Default port. Change to 8081 etc. if 8080 is occupied.

        // 2. Create a Context (application context — empty context path, no base dir)
        Context context = tomcat.addContext("", null);

        // 3. Register the Servlet with a name and its object
        Tomcat.addServlet(context, "HelloServlet", new HelloServlet());

        // 4. Map the URL pattern "/hello" to the Servlet named "HelloServlet"
        context.addServletMappingDecoded("/hello", "HelloServlet");

        // 5. Start the server and keep it running (await = don't exit)
        tomcat.start();
        tomcat.getServer().await(); // Without this, Tomcat starts and immediately stops!
    }
}
```

> **URL Mapping is critical**: Tomcat needs to know *which URL* triggers *which Servlet*. Without mapping, the servlet never gets called. In older Java apps, this was done in `web.xml`. With annotations, you use `@WebServlet("/hello")` (for external Tomcat). With embedded Tomcat, you do it programmatically as shown above.

**After running**: Visit `http://localhost:8080/hello` → See "Hello World from Servlet!"

---

## 4. The MVC (Model-View-Controller) Pattern

**Problem**: Writing HTML inside Java Servlet code (`out.println("<h2>...")`) is a nightmare for large applications. Mixing logic and presentation is unmaintainable.

**Solution**: Separate concerns into three layers:

```
┌─────────────────────────────────────────────────────────────────┐
│                    MVC Architecture                             │
│                                                                 │
│  Client ──► Controller (Servlet/@Controller)                    │
│                  │                                              │
│                  ├─ Accepts the HTTP request                    │
│                  ├─ Processes logic / calls DB                  │
│                  ├─ Puts data into Model object                │
│                  └─ Forwards to View ──────────────► Client    │
│                                                                 │
│  Model ──► POJO class holding data (e.g., Alien object)        │
│  View  ──► JSP / Thymeleaf / HTML page that renders the data   │
│  Controller ──► Servlet or @Controller class                    │
└─────────────────────────────────────────────────────────────────┘
```

| Component | Java Technology | Role |
|-----------|----------------|------|
| **Model** | POJO (Plain Old Java Object) | Holds the data (e.g., `Alien` object with `aid`, `aname`) |
| **View** | JSP, Thymeleaf, FreeMarker | Renders the data as an HTML page for the client |
| **Controller** | Servlet or Spring `@Controller` | Accepts request, processes logic, sends model to view |

> **JSP behind the scenes**: JSP pages get **converted into Servlets** by the Jasper engine. Tomcat can only run Servlets, so every JSP is compiled into a Servlet class automatically.

---

## 5. Building Web Apps with Spring Boot (The Easy Way)

Spring Boot eliminates all manual Tomcat configuration. It provides:
- **Embedded Tomcat** (auto-configured, no external server needed)
- **DispatcherServlet** (front controller that routes all requests — auto-configured)
- **ViewResolver** (resolves view names to actual files — auto-configured)

### Setup via Spring Initializr

Go to [start.spring.io](https://start.spring.io):
- **Project**: Maven | **Language**: Java | **Spring Boot**: 3.2+
- **Group**: `com.telusko` | **Artifact**: `SpringBootWeb1`
- **Packaging**: JAR (because embedded Tomcat is included)
- **Java**: 17 or 21
- **Dependency**: `Spring Web`

> Spring Web includes: Embedded Tomcat + Spring MVC + DispatcherServlet

### `pom.xml` Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

## 6. Creating a Spring Controller

Instead of extending `HttpServlet`, you create a plain Java class and annotate it with `@Controller`.

```java
package com.telusko.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller  // Tells Spring: this is a Controller. Behind the scenes, it becomes a Servlet.
public class HomeController {

    @RequestMapping("/")  // Maps the root URL "/" to this method
    public String home() {
        System.out.println("Home method called");

        // Return the VIEW NAME (not the full file path).
        // The ViewResolver adds prefix (folder) and suffix (.jsp/.html) automatically.
        return "index";
    }
}
```

> **How it works behind the scenes**: The **DispatcherServlet** intercepts ALL incoming requests. It reads `@RequestMapping` annotations to determine which controller method to call. The controller returns a view name. The **ViewResolver** locates the actual file and sends it to the client.

---

## 7. Working with JSP in Spring Boot

By default, Spring Boot **does NOT support JSP**. JSP pages need to be compiled into Servlets, which requires the **Jasper engine**.

### Step 1: Add the Jasper Dependency

```xml
<!-- Required to compile JSP into Servlets at runtime -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

> Without this dependency, returning a JSP view name causes the browser to **download** the file instead of rendering it!

### Step 2: Configure ViewResolver in `application.properties`

When you return just the view name (e.g., `"index"` without `.jsp`), the ViewResolver needs to know:
1. **Where** the view files are located (**prefix**)
2. **What extension** they have (**suffix**)

```properties
# src/main/resources/application.properties

# ViewResolver will look in /webapp/views/ folder
spring.mvc.view.prefix=/views/

# ViewResolver will append .jsp extension
spring.mvc.view.suffix=.jsp
```

So when a controller returns `"result"`, the ViewResolver resolves it to: `/views/result.jsp`

### Step 3: Create the Folder Structure

```
src/main/
├── java/com/telusko/...           ← Java code
├── resources/
│   ├── static/                    ← CSS, JS, images (public static files)
│   │   └── style.css
│   └── application.properties     ← Config
└── webapp/
    └── views/                     ← JSP files
        ├── index.jsp
        └── result.jsp
```

> **Important**: CSS/JS/images go in `src/main/resources/static/` (or `src/main/webapp/`). JSP files go in `src/main/webapp/views/`.

### `index.jsp` — The Calculator Form

```jsp
<%@ page language="java" %>
<html>
<head>
    <!-- Static CSS loaded from src/main/resources/static/ -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h2>Telusko Calculator</h2>

    <!-- When submitted, sends GET request to /add with num1 & num2 as query params -->
    <!-- URL becomes: /add?num1=6&num2=7 -->
    <form action="add">
        Enter num1: <input type="text" name="num1"><br>
        Enter num2: <input type="text" name="num2"><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

### `result.jsp` — Displaying the Result

```jsp
<html>
<body>
    <!-- ${result} uses JSTL Expression Language to fetch data -->
    <!-- It searches Session, Request, and Model objects automatically -->
    <h2>The Result is: ${result}</h2>
</body>
</html>
```

---

## 8. Passing Data: Client → Server → View

When the user submits the form, the URL becomes: `/add?num1=6&num2=7`

### Evolution A → B → C: Three Ways to Handle This

---

### A. The Servlet Way (`HttpServletRequest` + `HttpSession`)

```java
@RequestMapping("/add")
public String add(HttpServletRequest req, HttpSession session) {

    // 1. Get data FROM the client (query params)
    int num1 = Integer.parseInt(req.getParameter("num1"));
    int num2 = Integer.parseInt(req.getParameter("num2"));

    int result = num1 + num2;

    // 2. Put data INTO the session (so the view can access it)
    session.setAttribute("result", result);

    return "result"; // → ViewResolver → /views/result.jsp
}
```

> `HttpServletRequest` — contains all data sent by the client.
> `HttpSession` — persists data across pages/requests during a user's session.
> Spring auto-injects both objects when you declare them as method parameters.

---

### B. The Spring Way (`@RequestParam` + `Model`)

Spring can automatically extract query parameters and map them to method arguments.

```java
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestParam;

@RequestMapping("/add")
public String add(@RequestParam("num1") int n1, @RequestParam("num2") int n2, Model model) {

    int result = n1 + n2;

    // model.addAttribute() adds data to the REQUEST scope
    // The View can access it via ${result}
    model.addAttribute("result", result);

    return "result";
}
```

> **`@RequestParam("num1") int n1`** — Takes the value of `num1` from the URL and assigns it to `n1`. If the method parameter name MATCHES the query parameter name, `@RequestParam` is optional:
> ```java
> public String add(int num1, int num2, Model model) { ... }  // also works!
> ```
> But if names differ, you MUST use `@RequestParam` to specify the mapping.

> **`Model`** — A Spring interface (introduced in Spring 2.5) used to pass data from Controller to View. It adds data to the Request scope (lighter than Session, which persists across requests).

---

### C. The `ModelAndView` Way (Data + View in One Object)

Instead of returning a `String` (view name) and separately passing `Model`, you can combine both into a single `ModelAndView` object.

```java
import org.springframework.web.servlet.ModelAndView;

@RequestMapping("/add")
public ModelAndView add(@RequestParam("num1") int n1, @RequestParam("num2") int n2) {

    int result = n1 + n2;

    ModelAndView mv = new ModelAndView();
    mv.addObject("result", result);  // Add data (like model.addAttribute)
    mv.setViewName("result");        // Set view name (like "return result")

    return mv;  // Returns BOTH data and view name in one object
}
```

> **When to use `ModelAndView`?** When your controller needs to set the view name dynamically based on logic, or when you prefer to return a single unified object.

---

## 9. Summary: Comparison of Data-Passing Approaches

| Approach | How to Get Client Data | How to Pass Data to View | What to Return |
|----------|------------------------|--------------------------|----------------|
| Servlet Way | `req.getParameter("num1")` | `session.setAttribute(...)` | `String` (view name) |
| Spring Way | `@RequestParam("num1") int n1` | `model.addAttribute(...)` | `String` (view name) |
| ModelAndView | `@RequestParam("num1") int n1` | `mv.addObject(...)` + `mv.setViewName(...)` | `ModelAndView` |

---

## 10. Accepting Entire Objects with `@ModelAttribute`

**Problem**: If your form has 10 fields, writing 10 `@RequestParam` annotations is tedious.

**Solution**: `@ModelAttribute` lets Spring automatically bind ALL form fields to a Java POJO, as long as the form input `name` attributes match the POJO variable names.

### The POJO (`Alien.java`)

```java
package com.telusko.model;

public class Alien {
    private int aid;
    private String aname;

    public Alien() {}  // Default constructor required for Spring to instantiate

    public int getAid() { return aid; }
    public void setAid(int aid) { this.aid = aid; }

    public String getAname() { return aname; }
    public void setAname(String aname) { this.aname = aname; }

    @Override
    public String toString() {
        return "Alien [aid=" + aid + ", aname=" + aname + "]";
    }
}
```

### The Form (`index.jsp`)

```jsp
<form action="addAlien">
    Enter ID:   <input type="text" name="aid"><br>    <!-- matches Alien.aid -->
    Enter Name: <input type="text" name="aname"><br>  <!-- matches Alien.aname -->
    <input type="submit" value="Submit">
</form>
```

> **Critical**: The `name` attribute in HTML (`name="aid"`) MUST exactly match the Java field name (`private int aid`). Spring uses the setter methods (`setAid()`, `setAname()`) to populate the object.

### The Controller

```java
@RequestMapping("/addAlien")
public String addAlien(@ModelAttribute("alien1") Alien alien) {
    // Spring auto-created the Alien object and called setAid() and setAname()
    // with values from the form!

    // @ModelAttribute("alien1") means: pass this object to the view with key "alien1"
    // In JSP: ${alien1} will print the Alien object

    return "result";
}
```

### `@ModelAttribute` Rules

| Scenario | Usage | Example |
|----------|-------|---------|
| Default name (lowercase class name) | `@ModelAttribute` is **optional** — Spring does it implicitly | `addAlien(Alien alien)` → view gets `${alien}` |
| Custom name for the view | Use `@ModelAttribute("customName")` | `addAlien(@ModelAttribute("alien1") Alien alien)` → view gets `${alien1}` |
| Method-level (common data for ALL views) | Annotate a separate method | See below |

### `@ModelAttribute` on Method Level (Common Data)

If you need a value available on **every** page rendered by this controller (e.g., a header title, course name), create a method annotated with `@ModelAttribute`:

```java
@ModelAttribute("course")
public String courseName() {
    // This value will be available as ${course} in EVERY view rendered by this controller
    return "Java";  // Could be dynamic: fetch from DB, config, etc.
}
```

In JSP: `Welcome to the ${course} World!` → renders as "Welcome to the Java World!"

---

## 11. How DispatcherServlet & ViewResolver Work (Behind the Scenes)

```
Browser Request: GET /add?num1=6&num2=7
         │
         ▼
┌─────────────────────────┐
│    DispatcherServlet     │  ← Spring's Front Controller (auto-configured)
│  (intercepts ALL reqs)  │     Reads @RequestMapping annotations
└────────┬────────────────┘
         │ Routes to matching controller method
         ▼
┌─────────────────────────┐
│   HomeController.add()  │  ← Your @Controller class
│   - processes logic     │
│   - adds data to model  │
│   - returns "result"    │
└────────┬────────────────┘
         │ Returns view name "result"
         ▼
┌─────────────────────────┐
│     ViewResolver        │  ← Resolves "result" → /views/result.jsp
│  prefix: /views/        │     (using prefix + name + suffix)
│  suffix: .jsp           │
└────────┬────────────────┘
         │ Renders JSP with model data
         ▼
   Response sent to browser
```

---

## 12. Transitioning to Thymeleaf (Modern Alternative to JSP)

JSP is outdated. **Thymeleaf** is a modern server-side HTML template engine recommended by Spring.

**Key advantage**: Thymeleaf files are plain `.html` files — they can be opened in a browser directly (without a server) and still look correct. JSP files can't do this.

### Step 1: Update `pom.xml`

Remove `tomcat-embed-jasper` (JSP), add Thymeleaf:

```xml
<!-- Remove this JSP dependency -->
<!--
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
-->

<!-- Add Thymeleaf -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### Step 2: Remove ViewResolver Config & Move Files

- **Delete** `spring.mvc.view.prefix` and `spring.mvc.view.suffix` from `application.properties` (Thymeleaf auto-configures itself).
- **Move** all view files to `src/main/resources/templates/` (Thymeleaf's default location).
- **Rename** `.jsp` → `.html`.
- **Remove** `<%@ page language="java" %>` directive (not needed in HTML).

```
src/main/resources/
├── static/
│   └── style.css
├── templates/          ← Thymeleaf looks here by default
│   ├── index.html
│   └── result.html
└── application.properties  ← No prefix/suffix needed!
```

### Step 3: Thymeleaf Syntax in HTML

Add the Thymeleaf namespace to `<html>`, then use `th:text` for dynamic data:

```html
<!-- result.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">  <!-- 1. Add Thymeleaf namespace -->
<head>
    <title>Result Page</title>
</head>
<body>
    <h2>Welcome to Telusko</h2>

    <!-- 2. th:text replaces the tag's content with the expression value.
         "Fallback Greeting" is shown only if the server sends NO data
         (useful for previewing the HTML in a browser without a server). -->
    <p th:text="${alien}">Fallback Greeting</p>

    <!-- 3. Inline dynamic values within a string -->
    <p th:text="'Welcome to the ' + ${course} + ' World!'">Welcome to the Java World!</p>
</body>
</html>
```

### Thymeleaf vs JSP — Quick Comparison

| Feature | JSP | Thymeleaf |
|---------|-----|-----------|
| File extension | `.jsp` | `.html` |
| Can preview in browser? | ❌ No | ✅ Yes (Natural Templates) |
| Dynamic syntax | `${result}` (JSTL), `<%= ... %>` | `th:text="${result}"` |
| Required dependency | `tomcat-embed-jasper` | `spring-boot-starter-thymeleaf` |
| View location | `src/main/webapp/views/` | `src/main/resources/templates/` |
| Needs `prefix`/`suffix` config? | ✅ Yes | ❌ No (auto-configured) |
| Java directive needed? | `<%@ page language="java" %>` | Not needed |

> **Controller code does NOT change** when switching from JSP to Thymeleaf. The same `@Controller`, `@RequestMapping`, `Model`, `ModelAndView`, `@ModelAttribute` all work identically.

---

## 13. Complete Project File Structure (JSP Version)

```
src/
├── main/
│   ├── java/
│   │   └── com/telusko/
│   │       ├── SpringBootWeb1Application.java    ← @SpringBootApplication, main()
│   │       ├── controller/
│   │       │   └── HomeController.java            ← @Controller, @RequestMapping
│   │       └── model/
│   │           └── Alien.java                     ← POJO (aid, aname)
│   ├── resources/
│   │   ├── application.properties                 ← prefix/suffix config
│   │   └── static/
│   │       └── style.css                          ← Static assets
│   └── webapp/
│       └── views/
│           ├── index.jsp                          ← Form page
│           └── result.jsp                         ← Result page
```

### Complete Project File Structure (Thymeleaf Version)

```
src/
├── main/
│   ├── java/
│   │   └── com/telusko/
│   │       ├── SpringBootWeb1Application.java
│   │       ├── controller/
│   │       │   └── HomeController.java
│   │       └── model/
│   │           └── Alien.java
│   └── resources/
│       ├── application.properties                 ← Empty or minimal
│       ├── static/
│       │   └── style.css
│       └── templates/                             ← Thymeleaf default location
│           ├── index.html
│           └── result.html
```

---

## 14. Key Annotations Summary

| Annotation | Purpose |
|---|---|
| `@Controller` | Marks a class as a Spring MVC Controller (stereotype of `@Component`). Behind the scenes, it works like a Servlet. |
| `@RequestMapping("/path")` | Maps a URL path to a controller method. Works for all HTTP methods (GET, POST, etc.). |
| `@GetMapping("/path")` | Shorthand for `@RequestMapping(value="/path", method=GET)`. Use when you only accept GET requests. |
| `@PostMapping("/path")` | Shorthand for `@RequestMapping(value="/path", method=POST)`. Use when you only accept POST requests. |
| `@RequestParam("name")` | Binds a query parameter from the URL to a method argument. Optional if names match. |
| `@ModelAttribute("name")` | On a parameter: binds form data to a POJO. On a method: adds common data to all views. |
| `@SpringBootApplication` | Marks the main class. Enables auto-configuration, component scanning, and Spring Boot features. |

---

## 15. Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| 404 on homepage | No `@RequestMapping("/")` on any controller method | Add `@RequestMapping("/")` to the home method |
| JSP downloads instead of rendering | Missing `tomcat-embed-jasper` dependency | Add the Jasper dependency to `pom.xml` |
| 404 after moving JSP to a folder | ViewResolver doesn't know the folder or extension | Set `spring.mvc.view.prefix` and `spring.mvc.view.suffix` in `application.properties` |
| 500: `Optional int parameter 'num' is present but cannot be null` | Method param name doesn't match query param name | Use `@RequestParam("num1")` to map explicitly |
| `${result}` prints as literal text in JSP | Missing JSTL support or incorrect EL expression | Ensure `<%@ page language="java" %>` is at the top of the JSP |
| Thymeleaf: `${alien}` shows as literal text | Missing `xmlns:th` namespace in `<html>` tag | Add `xmlns:th="http://www.thymeleaf.org"` |
| CSS not loading | CSS file not in `static/` folder, or wrong path in HTML | Move CSS to `src/main/resources/static/` and reference as `href="style.css"` |

---

## 16. Full Controller Code — All Approaches in One Class

```java
package com.telusko.controller;

import com.telusko.model.Alien;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class HomeController {

    // ─── Homepage ───────────────────────────────────────────
    @RequestMapping("/")
    public String home() {
        return "index";
    }

    // ─── Approach A: Servlet Way ────────────────────────────
    @RequestMapping("/add-servlet")
    public String addServletWay(HttpServletRequest req, HttpSession session) {
        int num1 = Integer.parseInt(req.getParameter("num1"));
        int num2 = Integer.parseInt(req.getParameter("num2"));
        session.setAttribute("result", num1 + num2);
        return "result";
    }

    // ─── Approach B: Spring Way (preferred) ─────────────────
    @RequestMapping("/add")
    public String add(@RequestParam("num1") int n1, @RequestParam("num2") int n2, Model model) {
        model.addAttribute("result", n1 + n2);
        return "result";
    }

    // ─── Approach C: ModelAndView ────────────────────────────
    @RequestMapping("/add-mv")
    public ModelAndView addMV(@RequestParam("num1") int n1, @RequestParam("num2") int n2) {
        ModelAndView mv = new ModelAndView();
        mv.addObject("result", n1 + n2);
        mv.setViewName("result");
        return mv;
    }

    // ─── Object Binding: @ModelAttribute ────────────────────
    @RequestMapping("/addAlien")
    public String addAlien(@ModelAttribute("alien1") Alien alien) {
        // Spring auto-creates Alien, calls setAid() & setAname() from form data
        return "result";
    }

    // ─── Method-Level @ModelAttribute (available in ALL views) ──
    @ModelAttribute("course")
    public String courseName() {
        return "Java";
    }
}
```

---

## 17. Conceptual Recap — The Full Request-Response Flow

```
1. User opens browser → http://localhost:8080/

2. DispatcherServlet intercepts the request
   └─ Checks @RequestMapping annotations
   └─ Finds HomeController.home() mapped to "/"

3. HomeController.home() executes
   └─ Returns "index"

4. ViewResolver resolves "index"
   └─ prefix="/views/" + "index" + suffix=".jsp" → /views/index.jsp

5. index.jsp renders → form displayed to user

6. User enters 6, 7 → clicks Submit
   └─ Browser sends: GET /add?num1=6&num2=7

7. DispatcherServlet routes to HomeController.add()
   └─ @RequestParam extracts num1=6, num2=7
   └─ Controller calculates result=13
   └─ model.addAttribute("result", 13)
   └─ Returns "result"

8. ViewResolver → /views/result.jsp
   └─ ${result} replaced with 13
   └─ HTML sent to browser

9. User sees: "The Result is: 13"
```
