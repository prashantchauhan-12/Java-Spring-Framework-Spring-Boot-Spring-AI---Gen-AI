# Spring Boot Web & MVC: Comprehensive Revision Notes

## 1. Introduction to Web Applications
A web application typically consists of a frontend (client) and a backend (server). 
* **Client:** The browser or mobile app requesting data.
* **Server:** The backend accepting the request, processing it, and responding with either a complete HTML page or raw JSON data.
* **Java's Role:** Java is used to build the server side. While modern architectures use Spring Boot to build REST APIs (returning JSON to React/Angular frontends), understanding how Java historically handles web requests (via Servlets and JSPs) is crucial because Spring still relies on these core technologies under the hood.

---

## 2. Servlets and Embedded Tomcat (Without Spring)
A **Servlet** is a Java component that accepts HTTP requests and generates responses. Servlets run inside a **Web Container** (like Apache Tomcat). 

Before Spring Boot simplified everything, developers had to manually set up a Tomcat server, write Servlets, and map them. We can do this manually in Java using an **Embedded Tomcat**.

### Dependencies needed in `pom.xml`:
```xml
<dependencies>
    <!-- Jakarta Servlet API -->
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

### 1. Creating a Servlet (`HelloServlet.java`)
```java
package com.telusko;

import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class HelloServlet extends HttpServlet {
    
    // The service method handles all incoming HTTP requests (GET, POST, etc.)
    // You can also override doGet() or doPost() for specific HTTP methods.
    public void service(HttpServletRequest req, HttpServletResponse res) throws IOException {
        System.out.println("In Service Method");
        
        // Setting the response type to HTML
        res.setContentType("text/html");
        
        // Getting the PrintWriter to write data to the client's browser
        PrintWriter out = res.getWriter();
        out.println("<h2>Hello World from Servlet!</h2>");
    }
}
```

### 2. Manual Embedded Tomcat Configuration (`App.java`)
```java
package com.telusko;

import org.apache.catalina.Context;
import org.apache.catalina.LifecycleException;
import org.apache.catalina.startup.Tomcat;

public class App {
    public static void main(String[] args) throws LifecycleException {
        
        System.out.println("Starting application...");
        
        // 1. Create Tomcat Object
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080); // Default port is 8080
        
        // 2. Create Context (Application Context)
        Context context = tomcat.addContext("", null);
        
        // 3. Register the Servlet with Tomcat
        Tomcat.addServlet(context, "HelloServlet", new HelloServlet());
        
        // 4. Map the URL pattern to the Servlet
        context.addServletMappingDecoded("/hello", "HelloServlet");
        
        // 5. Start Server and Await Connections
        tomcat.start();
        tomcat.getServer().await(); // Keeps the server running indefinitely
    }
}
```

---

## 3. The MVC (Model-View-Controller) Pattern
Writing HTML tags inside a Java Servlet using `out.println("<h2>...</h2>")` becomes a nightmare for large applications. To solve this, the industry adopted **MVC**:
* **Controller (Servlet/Spring `@Controller`):** Accepts the HTTP request, processes logic, talks to the DB.
* **Model (POJO):** Holds the data that needs to be transferred to the frontend (e.g., an `Alien` object).
* **View (JSP/Thymeleaf):** The HTML template that displays the data to the user.

---

## 4. Web Apps with Spring Boot (The Easy Way)
Spring Boot completely abstracts away manual Tomcat configuration and Servlet mapping. 

### Setup in Spring Initializr
Add the **Spring Web** dependency. This automatically includes the Embedded Tomcat and Spring MVC libraries.

### 1. Creating a Spring Controller (`HomeController.java`)
Instead of extending `HttpServlet`, you simply use the `@Controller` annotation.
```java
package com.telusko.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HomeController {

    // Maps the root URL "/" to this method
    @RequestMapping("/")
    public String home() {
        System.out.println("Home method called");
        
        // This String is the name of the View (e.g., index.jsp or index.html)
        // The ViewResolver will search for this file.
        return "index"; 
    }
}
```

---

## 5. Working with JSP in Spring Boot
By default, Spring Boot **does not** support JSP. You must add the Jasper engine to compile JSPs into Servlets.

### Dependency required in `pom.xml`
```xml
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

### application.properties Configuration
Spring Boot needs to know where your JSP files are located and what their extension is.
```properties
# src/main/resources/application.properties

# Tell the ViewResolver to look in the /webapp/views/ folder
spring.mvc.view.prefix=/views/

# Tell the ViewResolver that all views have a .jsp extension
spring.mvc.view.suffix=.jsp
```

### Folder Structure
Create a `webapp` directory inside `src/main/` and a `views` folder inside it.
`src/main/webapp/views/index.jsp`

```jsp
<!-- index.jsp -->
<%@ page language="java" %>
<html>
<head>
    <!-- Static files (like CSS) go into src/main/resources/static/ -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h2>Telusko Calculator</h2>
    
    <!-- This form submits a GET request to the /add URL -->
    <form action="add">
        Enter num1: <input type="text" name="num1"><br>
        Enter num2: <input type="text" name="num2"><br>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

---

## 6. Passing Data: Client to Server & Server to View

When the user submits the form above, the URL becomes `/add?num1=6&num2=7`. Here is how the Controller processes it.

### A. The Old Servlet Way (`HttpServletRequest` and `HttpSession`)
```java
@RequestMapping("/add")
public String add(HttpServletRequest req, HttpSession session) {
    // 1. Fetching data from the client
    int num1 = Integer.parseInt(req.getParameter("num1"));
    int num2 = Integer.parseInt(req.getParameter("num2"));
    
    int result = num1 + num2;
    
    // 2. Passing data to the View using Session
    session.setAttribute("result", result);
    
    return "result"; // Calls result.jsp
}
```

### B. The Spring Way (`@RequestParam` and `Model`)
Spring can automatically extract query parameters and pass data using the `Model` object instead of a heavy `HttpSession`.
```java
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestParam;

@RequestMapping("/add")
public String add(@RequestParam("num1") int n1, @RequestParam("num2") int n2, Model model) {
    int result = n1 + n2;
    
    // Adds data to the Request scope, automatically available in the View
    model.addAttribute("result", result);
    
    return "result";
}
```

### C. The Pro Spring Way (`ModelAndView`)
Instead of passing the `Model` as an argument and returning a `String` view name, you can return a unified `ModelAndView` object.
```java
import org.springframework.web.servlet.ModelAndView;

@RequestMapping("/add")
public ModelAndView add(@RequestParam("num1") int n1, @RequestParam("num2") int n2) {
    int result = n1 + n2;
    
    ModelAndView mv = new ModelAndView();
    mv.addObject("result", result); // Add Data
    mv.setViewName("result");       // Set View Name
    
    return mv;
}
```

### Getting Data in JSP (JSTL)
In `result.jsp`, you can access the variable using the `$` JSTL syntax.
```jsp
<html>
<body>
    <h2>The Result is: ${result}</h2>
</body>
</html>
```

---

## 7. Accepting Complete Objects (`@ModelAttribute`)
If your HTML form has 10 fields (e.g., ID, Name, Tech, Age...), manually capturing them with 10 `@RequestParam`s is exhausting.
Spring allows you to directly bind incoming form data to a Java POJO.

### The POJO (`Alien.java`)
```java
package com.telusko.model;

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

### The Controller
The `name` attributes in the HTML form inputs (`name="aid"`, `name="aname"`) MUST exactly match the variable names in the POJO. Spring will create the object, call the setters, and inject it.
```java
import com.telusko.model.Alien;
import org.springframework.web.bind.annotation.ModelAttribute;

@RequestMapping("/addAlien")
public String addAlien(@ModelAttribute("alien1") Alien alien) {
    // Spring mapped form fields 'aid' and 'aname' directly to the Alien object!
    
    // The @ModelAttribute("alien1") explicitly names the object as "alien1" for the View.
    // If you omit @ModelAttribute, the object is passed to the view with its default class name in lowercase ("alien").
    return "result";
}
```
*Note: You can also use `@ModelAttribute` at the method level to pass common variables (like a Header title) to ALL views rendered by that controller.*

---

## 8. Transitioning to Thymeleaf (Modern Alternative to JSP)
JSP is outdated. Modern Spring Boot applications use **Thymeleaf**, a modern Server-Side HTML template engine.
Unlike JSP, Thymeleaf files are purely `.html` files that can be rendered normally in a browser even without a server.

### 1. Update Dependencies
Remove `tomcat-embed-jasper` and add Thymeleaf.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 2. Move Files and Remove Configurations
* **Delete** `spring.mvc.view.prefix` and `spring.mvc.view.suffix` from `application.properties`.
* **Move** all your views to the default Thymeleaf directory: `src/main/resources/templates/`.
* Rename `index.jsp` to `index.html`.

### 3. Thymeleaf Syntax in HTML
In your `index.html` or `result.html`, add the Thymeleaf XML namespace to the `<html>` tag, and use `th:text` for dynamic injection.

```html
<!-- result.html -->
<!DOCTYPE html>
<!-- 1. Add the xmlns:th namespace -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Result Page</title>
</head>
<body>
    <h2>Welcome to Telusko</h2>
    
    <!-- 2. Use th:text to inject data. 
         If the server sends data, it replaces the content inside the tag. 
         If no data is sent, it displays the fallback text "Fallback Greeting". -->
    <p th:text="${alien}">Fallback Greeting</p>
    
    <p>Welcome to the <span th:text="${course}">Java</span> World!</p>
</body>
</html>
```
