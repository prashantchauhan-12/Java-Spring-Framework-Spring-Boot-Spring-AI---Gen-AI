# Exploring Spring Web MVC (Without Spring Boot) — Comprehensive Revision Notes

## 1. Introduction: Why Build Without Spring Boot?

Spring Boot auto-configures everything — Embedded Tomcat, DispatcherServlet, ViewResolver, component scanning. But **understanding the manual setup teaches you exactly what Spring Boot automates**. In production, you may encounter legacy projects that use plain Spring MVC, and knowing this gives you the ability to debug and configure them.

### What Spring Boot Does For You (That You Must Do Manually Here)

| What | Spring Boot | Plain Spring MVC (This Section) |
|---|---|---|
| Tomcat | Embedded, auto-starts | External, manually downloaded & configured in IDE |
| DispatcherServlet | Auto-configured | Must register in `web.xml` |
| Component Scanning | Auto-scans `@SpringBootApplication` package | Must configure in `<servlet-name>-servlet.xml` |
| ViewResolver | Configured via `application.properties` | Must define `InternalResourceViewResolver` bean in XML |
| Packaging | `.jar` (with embedded server) | `.war` (deployed to external Tomcat) |
| Dependencies | `spring-boot-starter-web` (one starter) | `spring-webmvc` + external Tomcat library |

---

## 2. Setting Up the Maven Web Project

### Step 1: Create a Maven Project in Eclipse

Unlike Spring Boot where you use Spring Initializr, here you create a plain **Maven Web Application**:

1. **File → New → Maven Project**
2. Click **Next**, select **Internal** catalog
3. Choose the **`maven-archetype-webapp`** archetype (NOT quickstart — we need a web app structure)
4. Set **Group ID**: `com.telusko`, **Artifact ID**: `SpringMVCDemo`
5. Click **Finish**

> **Eclipse vs IntelliJ**: This section uses **Eclipse IDE for Enterprise Java Developers** (free, has built-in Tomcat server support). IntelliJ Community Edition doesn't have this.

### Step 2: Fix the Folder Structure

The Maven webapp archetype gives you:
```
SpringMVCDemo/
├── src/main/
│   ├── webapp/
│   │   ├── WEB-INF/
│   │   │   └── web.xml         ← Deployment descriptor
│   │   └── index.jsp           ← Default page (delete this)
│   └── (NO java/ folder!)      ← You must create this manually
```

**Fix**: Right-click `src/main/` → New → Folder → name it `java`. Then create your package inside it:
```
src/main/java/com/telusko/SpringMVCDemo/
```

### Step 3: Add the Spring MVC Dependency

This is the **only** Spring dependency needed (it transitively pulls in Spring Core, Spring Beans, Spring Context, Spring Web, etc.):

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Core Spring MVC Framework (NOT spring-boot-starter-web) -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>6.1.0</version> <!-- Use the latest stable version -->
    </dependency>
</dependencies>
```

> After adding this, check **Maven Dependencies** in Eclipse — you'll see all the Spring JARs: `spring-core`, `spring-beans`, `spring-context`, `spring-web`, `spring-webmvc`, `spring-expression`, etc.

### Step 4: Configure External Tomcat in Eclipse

Since there's no embedded Tomcat, you must:

1. **Download Apache Tomcat** (e.g., Tomcat 10.1.x) from [tomcat.apache.org](https://tomcat.apache.org)
2. Unzip to a folder (e.g., `~/Downloads/apache-tomcat-10.1.16/`)
3. In Eclipse: **Servers** tab → "No servers are available, click to create..." → **Apache Tomcat v10.1** → click **Next**
4. **Browse** to your Tomcat installation directory → click **Next**
5. **Add your project** (`SpringMVCDemo`) to the server → click **Finish**
6. Also: Right-click project → **Build Path** → Libraries → Add **Server Runtime** (Tomcat) and set Java to **Java 21**

> **Port conflict**: If you get "port 8080 already in use", stop any other Tomcat/Spring Boot instance first. Or double-click the Tomcat server in Eclipse and change the port number.

### Step 5: Copy Your Application Code

Copy from the Spring Boot project (or create fresh):
- `Alien.java` (POJO) → into `src/main/java/com/telusko/SpringMVCDemo/`
- `HomeController.java` (Controller) → same package
- `views/` folder (containing `index.jsp`, `result.jsp`) → into `src/main/webapp/`

> **Important**: You do NOT need the `@SpringBootApplication` main class. There's no `main()` method — Tomcat launches the app.

---

## 3. The Problem: Why It Doesn't Work Yet

At this point, if you right-click the project → **Run As → Run on Server** → you get **404 Not Found**.

**Why?** Because:
1. **Tomcat only understands Servlets** — it has no idea what `@Controller` means
2. There's no bridge between Tomcat and your Spring controller classes
3. No one told Tomcat to route requests through Spring's framework

The solution is the **DispatcherServlet** — Spring's Front Controller pattern.

---

## 4. The Front Controller: DispatcherServlet

### What Is It?

The **DispatcherServlet** is a Servlet provided by the Spring Framework (`org.springframework.web.servlet.DispatcherServlet`). It acts as the **Front Controller** — a single entry point for ALL incoming HTTP requests.

```
┌─────────┐       ┌──────────────────────┐       ┌──────────────────┐
│  Client  │ ───► │  DispatcherServlet   │ ───► │  HomeController   │
│ (Browser)│       │  (Front Controller)  │       │  @Controller      │
│          │ ◄─── │  Routes to correct   │ ◄─── │  @RequestMapping  │
│          │       │  controller method   │       │  returns view name│
└─────────┘       └──────────────────────┘       └──────────────────┘
                          │
                          ▼
                  ┌──────────────────┐
                  │   ViewResolver    │
                  │  Resolves "index" │
                  │  → /views/index.jsp│
                  └──────────────────┘
```

**Flow**:
1. Client sends request → Tomcat receives it
2. Tomcat checks `web.xml` → finds that ALL requests (`/`) go to DispatcherServlet
3. DispatcherServlet reads its config (`telusko-servlet.xml`) → knows which controllers exist
4. DispatcherServlet routes the request to the correct `@Controller` method
5. Controller returns a view name (`"index"`) → DispatcherServlet asks the ViewResolver
6. ViewResolver resolves `"index"` → `/views/index.jsp` → rendered HTML sent back

### Configuring DispatcherServlet in `web.xml`

The `web.xml` file is the **standard Java EE deployment descriptor**. It's how you "talk to Tomcat". This file lives at `src/main/webapp/WEB-INF/web.xml`.

```xml
<!-- src/main/webapp/WEB-INF/web.xml -->
<web-app>
  <display-name>Spring Web MVC Demo</display-name>

  <!-- ═══════════════════════════════════════════════════════════ -->
  <!-- STEP 1: Define the Servlet                                 -->
  <!-- Tell Tomcat: "There is a servlet called 'telusko',         -->
  <!--   and its Java class is DispatcherServlet"                  -->
  <!-- ═══════════════════════════════════════════════════════════ -->
  <servlet>
    <servlet-name>telusko</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  </servlet>

  <!-- ═══════════════════════════════════════════════════════════ -->
  <!-- STEP 2: Map URL patterns to the Servlet                    -->
  <!-- Tell Tomcat: "For ALL URLs (/), route to 'telusko' servlet" -->
  <!-- The servlet-name MUST match in both tags.                   -->
  <!-- ═══════════════════════════════════════════════════════════ -->
  <servlet-mapping>
    <servlet-name>telusko</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

### Critical: The Servlet Name Matters!

The `<servlet-name>` you choose (here: `telusko`) determines the name of the Spring configuration file. The DispatcherServlet automatically looks for:

```
/WEB-INF/<servlet-name>-servlet.xml
```

| If `<servlet-name>` is... | DispatcherServlet looks for... |
|---|---|
| `telusko` | `/WEB-INF/telusko-servlet.xml` |
| `dispatcher` | `/WEB-INF/dispatcher-servlet.xml` |
| `app` | `/WEB-INF/app-servlet.xml` |

> If this file doesn't exist, you get a **500 Internal Server Error** with an `IOException` saying the XML file could not be found.

---

## 5. The Spring Configuration File (`telusko-servlet.xml`)

This file is how you **"talk to DispatcherServlet"**. It tells the DispatcherServlet:
1. **Where to find controllers** (component scan)
2. **That you're using annotations** (annotation-driven)
3. **How to resolve view names** (ViewResolver)

### Who Talks to Whom?

| File | Talks To | Purpose |
|---|---|---|
| `web.xml` | **Tomcat** | "Route all requests to DispatcherServlet" |
| `telusko-servlet.xml` | **DispatcherServlet** | "Scan this package, use annotations, resolve views like this" |

### Full Configuration File

```xml
<!-- src/main/webapp/WEB-INF/telusko-servlet.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:ctx="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- 1. COMPONENT SCAN                                          -->
    <!-- Tell DispatcherServlet: "Scan the com.telusko package       -->
    <!--   and its sub-packages to find @Controller, @Component..." -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    <ctx:component-scan base-package="com.telusko" />

    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- 2. ANNOTATION-DRIVEN                                       -->
    <!-- Tell DispatcherServlet: "We are using annotations like      -->
    <!--   @RequestMapping, @RequestParam, @ModelAttribute, etc."   -->
    <!-- Without this, @RequestMapping won't work!                  -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    <mvc:annotation-driven />

    <!-- ═══════════════════════════════════════════════════════════ -->
    <!-- 3. VIEW RESOLVER                                           -->
    <!-- Tell DispatcherServlet: "When a controller returns 'index', -->
    <!--   look for the file at /views/index.jsp"                   -->
    <!--   prefix + viewName + suffix = /views/ + index + .jsp      -->
    <!--                                                             -->
    <!-- This replaces Spring Boot's application.properties:         -->
    <!--   spring.mvc.view.prefix=/views/                           -->
    <!--   spring.mvc.view.suffix=.jsp                              -->
    <!-- ═══════════════════════════════════════════════════════════ -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/views/" />
        <property name="suffix" value=".jsp" />
    </bean>

</beans>
```

### Understanding the XML Namespaces

The `xmlns` declarations at the top define tag prefixes:

| Prefix | Namespace | Tags It Enables |
|---|---|---|
| (default) | `beans` | `<bean>`, `<property>` — standard Spring bean configuration |
| `ctx:` | `context` | `<ctx:component-scan>` — package scanning |
| `mvc:` | `mvc` | `<mvc:annotation-driven>` — enable MVC annotations |
| `xsi:` | `XMLSchema-instance` | `xsi:schemaLocation` — XML validation |

> You don't need to memorize these. Copy them from the Spring documentation or any working project.

---

## 6. The Controller and View (Unchanged from Spring Boot!)

The beautiful thing about Spring MVC is that **your application code is identical whether you use Spring Boot or plain Spring MVC**. The only difference is the configuration.

### The Controller (`HomeController.java`)

```java
package com.telusko.SpringMVCDemo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class HomeController {

    @RequestMapping("/")
    public String home() {
        System.out.println("Home method called");
        return "index"; // ViewResolver: /views/ + index + .jsp → /views/index.jsp
    }

    @RequestMapping("/add")
    public String add(@RequestParam("num1") int n1, @RequestParam("num2") int n2, Model model) {
        int result = n1 + n2;
        model.addAttribute("result", result);
        return "result"; // ViewResolver: /views/ + result + .jsp → /views/result.jsp
    }

    @RequestMapping("/addAlien")
    public String addAlien(@ModelAttribute("alien1") Alien alien) {
        // Spring auto-binds form fields to Alien object
        return "result";
    }

    @ModelAttribute("course")
    public String courseName() {
        return "Java"; // Available as ${course} in ALL views
    }
}
```

### The POJO (`Alien.java`)

```java
package com.telusko.SpringMVCDemo;

public class Alien {
    private int aid;
    private String aname;

    public Alien() {}

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

### The JSP Views

**`index.jsp`** (form page):
```jsp
<%@ page language="java" %>
<html>
<head><title>Home Page</title></head>
<body>
    <h2>Telusko Spring MVC App</h2>
    <form action="addAlien">
        Enter ID:   <input type="text" name="aid"><br>
        Enter Name: <input type="text" name="aname"><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

**`result.jsp`** (result page):
```jsp
<%@ page language="java" isELIgnored="false" %>
<html>
<head><title>Result Page</title></head>
<body>
    <h2>Welcome to Telusko</h2>
    <p>${alien1}</p>
    <p>Welcome to the ${course} World!</p>
</body>
</html>
```

> **`isELIgnored="false"`** — Critical! Some older web app configurations ignore JSTL Expression Language (`${}`) by default. Without this attribute, `${result}` prints as literal text instead of the actual value. Spring Boot projects don't have this issue because they use a newer servlet specification.

---

## 7. Complete Project File Structure

```
SpringMVCDemo/
├── pom.xml                                 ← spring-webmvc dependency
└── src/
    └── main/
        ├── java/
        │   └── com/telusko/SpringMVCDemo/
        │       ├── HomeController.java     ← @Controller
        │       └── Alien.java              ← POJO
        └── webapp/
            ├── WEB-INF/
            │   ├── web.xml                 ← Talks to Tomcat (register DispatcherServlet)
            │   └── telusko-servlet.xml     ← Talks to DispatcherServlet (scan, resolve)
            └── views/
                ├── index.jsp               ← Form page
                └── result.jsp              ← Result page
```

---

## 8. The Full Request-Response Flow (Step by Step)

```
1. User visits http://localhost:8080/SpringMVCDemo/

2. ┌─────────┐
   │  TOMCAT  │  Receives the HTTP request
   │          │  Reads web.xml → "Route ALL requests to 'telusko' servlet"
   │          │  'telusko' servlet = DispatcherServlet
   └────┬─────┘
        │
        ▼
3. ┌──────────────────────────┐
   │   DispatcherServlet      │  Spring's Front Controller
   │                          │  Reads telusko-servlet.xml:
   │   - component-scan:      │    "Scan com.telusko for @Controller classes"
   │     com.telusko           │    → Finds HomeController
   │                          │
   │   - annotation-driven:   │    "Respect @RequestMapping annotations"
   │     enabled              │    → "/" maps to home() method
   │                          │
   │   Routes "/" to          │
   │   HomeController.home()  │
   └────┬─────────────────────┘
        │
        ▼
4. ┌──────────────────────────┐
   │   HomeController.home()  │  Executes the method
   │   - prints "Home called" │
   │   - returns "index"      │  (just a String, not a file path)
   └────┬─────────────────────┘
        │ Returns "index"
        ▼
5. ┌──────────────────────────┐
   │   InternalResource       │  Configured in telusko-servlet.xml
   │   ViewResolver           │
   │                          │  prefix="/views/" + "index" + suffix=".jsp"
   │   Resolves to:           │  → /views/index.jsp
   │   /views/index.jsp       │
   └────┬─────────────────────┘
        │
        ▼
6. ┌──────────────────────────┐
   │   /views/index.jsp       │  JSP rendered with model data
   │   HTML sent to client    │
   └──────────────────────────┘

7. User sees the form. Enters ID=101, Name=Navin, clicks Submit.
   → Browser sends: GET /addAlien?aid=101&aname=Navin

8. DispatcherServlet → routes to addAlien(@ModelAttribute Alien)
   → Spring creates Alien object, calls setAid(101), setAname("Navin")
   → Returns "result" → ViewResolver → /views/result.jsp
   → ${alien1} prints "Alien [aid=101, aname=Navin]"
   → ${course} prints "Java" (from @ModelAttribute method)

9. User sees: "Welcome to Telusko" + alien data + "Welcome to the Java World!"
```

---

## 9. Spring Boot vs. Plain Spring MVC — Side-by-Side Comparison

### Configuration Files

| Concern | Spring Boot | Plain Spring MVC |
|---|---|---|
| Tomcat | None (embedded, auto-starts) | Download Tomcat, configure in IDE, add to Build Path |
| Servlet Registration | None (auto-configured) | `web.xml` → register `DispatcherServlet` |
| Component Scanning | Auto (scans `@SpringBootApplication` package) | `telusko-servlet.xml` → `<ctx:component-scan>` |
| Annotation Support | Auto | `telusko-servlet.xml` → `<mvc:annotation-driven />` |
| ViewResolver | `application.properties` (2 lines) | `telusko-servlet.xml` → `<bean>` with `InternalResourceViewResolver` |
| Total config files | 1 (`application.properties`) | 2 (`web.xml` + `telusko-servlet.xml`) |

### Dependency Comparison

```xml
<!-- Spring Boot: ONE dependency pulls everything -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Plain Spring MVC: ONE dependency + external Tomcat server -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>6.1.0</version>
</dependency>
<!-- + manually downloaded Apache Tomcat added to IDE/Build Path -->
```

### Application Code Comparison

```java
// ✅ IDENTICAL in both Spring Boot and Plain Spring MVC!
@Controller
public class HomeController {
    @RequestMapping("/")
    public String home() {
        return "index";
    }
}
```

> **Key takeaway**: The application code (`@Controller`, `@RequestMapping`, `Model`, `@ModelAttribute`, `@RequestParam`, `ModelAndView`) is **100% identical**. The ONLY difference is the configuration boilerplate.

---

## 10. Common Errors and Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| **404: Resource not available** | No DispatcherServlet registered | Add `<servlet>` and `<servlet-mapping>` to `web.xml` |
| **500: IOException — cannot find `telusko-servlet.xml`** | DispatcherServlet config file is missing | Create `WEB-INF/telusko-servlet.xml` (name must match `<servlet-name>`) |
| **404: No endpoint GET /index** | ViewResolver not configured | Add `InternalResourceViewResolver` bean in `telusko-servlet.xml` |
| **500: `<property>` syntax error** | Used `<property name="prefix">/views/</property>` instead of `value=""` | Use `<property name="prefix" value="/views/" />` |
| **`${result}` prints as literal text** | Expression Language ignored in JSP | Add `isELIgnored="false"` in `<%@ page %>` directive |
| **Port 8080 already in use** | Another Tomcat/Spring Boot app is running | Stop the other app, or change port in Tomcat config |
| **ClassNotFoundException for Spring classes** | `spring-webmvc` dependency not added | Add the dependency to `pom.xml` and reload Maven |
| **No `java/` folder in project** | Maven webapp archetype doesn't create it | Manually create `src/main/java/` folder |
| **Jakarta vs javax import errors** | Tomcat 10+ uses `jakarta.*`, Tomcat 9 uses `javax.*` | Match imports to your Tomcat version |

---

## 11. Key Annotations Reference (Same in Both Spring Boot and Plain MVC)

| Annotation | Purpose |
|---|---|
| `@Controller` | Marks a class as a Spring MVC controller (processed by DispatcherServlet) |
| `@RequestMapping("/path")` | Maps a URL to a controller method |
| `@RequestParam("name")` | Extracts query parameters from the URL |
| `@ModelAttribute("name")` | Binds form data to a POJO, or adds common data to all views |
| `@GetMapping`, `@PostMapping` | HTTP-method-specific shortcuts for `@RequestMapping` |

---

## 12. Conceptual Recap: The Three Layers of Configuration

Think of it as a chain of communication:

```
┌────────────────────────────────────────────────────────────────────────┐
│                                                                        │
│   Layer 1: TOMCAT (web.xml)                                           │
│   "Hey Tomcat, send ALL requests to DispatcherServlet"                │
│       ↓                                                               │
│   Layer 2: DISPATCHERSERVLET (telusko-servlet.xml)                    │
│   "Hey DispatcherServlet, here's how to find controllers & views"     │
│       ↓                                                               │
│   Layer 3: APPLICATION CODE (@Controller, @RequestMapping)            │
│   "Here's the actual business logic and view names"                   │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

**Spring Boot collapses Layers 1 and 2 into auto-configuration**, so you only write Layer 3.

---

## 13. When to Use Plain Spring MVC vs. Spring Boot?

| Use Plain Spring MVC When... | Use Spring Boot When... |
|---|---|
| You need **maximum control** over configuration | You want to **save time** and reduce boilerplate |
| Working on a **legacy project** that doesn't use Boot | Starting a **new project** (99% of cases) |
| You need a **custom Tomcat** setup (clustering, advanced security) | Rapid prototyping and microservices |
| Deploying to a **shared external Tomcat** server (enterprise requirement) | You want embedded server + `java -jar` deployment |
| Learning how Spring MVC works **behind the scenes** | Production-ready apps with sensible defaults |

> "Is it worth building projects in plain Spring MVC? Yes, you get more control. Is it worth building in Spring Boot? Yes, it saves your time. Most modern projects use Spring Boot." — From the transcript.
