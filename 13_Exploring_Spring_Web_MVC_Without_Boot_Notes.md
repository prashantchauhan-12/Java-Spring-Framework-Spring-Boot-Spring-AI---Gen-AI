# Exploring Spring Web MVC (Without Spring Boot) - Detailed Notes

## 1. Introduction: Why go backwards?
Before Spring Boot came along to auto-configure everything, developers had to manually set up Spring MVC projects. Understanding this process is crucial because it teaches you exactly what Spring Boot is doing behind the scenes. In this section, we build a Spring MVC application purely using **Maven** and **Spring Web MVC** (no Spring Boot starters!).

---

## 2. Setting Up the Maven Web Project

Unlike a Spring Boot project generated via Spring Initializr, we start with a plain Maven web application.

1. **Create the Project:** In Eclipse/IntelliJ, create a New Maven Project.
2. **Select Archetype:** Choose the "Web Application" internal catalog archetype (e.g., `maven-archetype-webapp`).
3. **Add Dependencies:** We need the core Spring MVC dependency in our `pom.xml`.

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Core Spring MVC Framework -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>6.1.0</version> <!-- Version may vary -->
    </dependency>
</dependencies>
```

4. **Add Tomcat Server:** You need to manually download Apache Tomcat and configure it in your IDE to run the web application, as we don't have the "Embedded Tomcat" that Spring Boot provides.

---

## 3. The Front Controller (DispatcherServlet)

### The Problem
When a client sends an HTTP request, Tomcat (the Servlet Container) receives it. But Tomcat has no idea what a Spring `@Controller` is. Tomcat only understands traditional Java Servlets. 

### The Solution: DispatcherServlet
Spring provides a master servlet called the **DispatcherServlet**. It acts as the **Front Controller**. 
Every single request coming into the application must first go to the DispatcherServlet. The DispatcherServlet then inspects the URL and routes it to the appropriate `@Controller`.

### Configuring DispatcherServlet in `web.xml`
To tell Tomcat to send all requests to the DispatcherServlet, we must configure the standard Java web deployment descriptor file: `WEB-INF/web.xml`.

```xml
<!-- src/main/webapp/WEB-INF/web.xml -->
<web-app>
  <display-name>Spring Web MVC Demo</display-name>
  
  <!-- 1. Define the Servlet -->
  <servlet>
    <servlet-name>telusko</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  </servlet>

  <!-- 2. Map the Servlet to handle all requests -->
  <servlet-mapping>
    <servlet-name>telusko</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```
*Note: We mapped the `<servlet-name>` as `telusko`. This specific name is extremely important for the next step.*

---

## 4. The Spring XML Configuration File

The `DispatcherServlet` routes requests, but by default, it doesn't know where your controllers are or how to resolve your views. It expects a configuration file to tell it what to do.

**Naming Convention:** 
By default, the DispatcherServlet looks in the `/WEB-INF/` directory for an XML file named `<servlet-name>-servlet.xml`. Since we named our servlet `telusko`, we must create `telusko-servlet.xml`.

### What goes inside `telusko-servlet.xml`?
1. **Component Scan:** Tell Spring which package to scan to find the `@Controller` classes.
2. **Annotation Driven:** Tell Spring that we are using annotations (like `@RequestMapping`).
3. **View Resolver:** Tell Spring where the JSP files are located.

```xml
<!-- src/main/webapp/WEB-INF/telusko-servlet.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:ctx="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/schema/beans"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 1. Scan for components in this package -->
    <ctx:component-scan base-package="com.telusko" />

    <!-- 2. Enable Annotation-based configuration -->
    <mvc:annotation-driven />

    <!-- 3. Configure the View Resolver (replaces application.properties from Spring Boot) -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/views/" />
        <property name="suffix" value=".jsp" />
    </bean>

</beans>
```

---

## 5. The Controller and The View

Now that the heavy configuration is done, writing the actual application code is identical to how we do it in Spring Boot!

### The Controller Class (`HomeController.java`)
```java
package com.telusko.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class HomeController {

    @RequestMapping("/")
    public String home() {
        System.out.println("Home method called");
        return "index"; // The ViewResolver adds "/views/" + "index" + ".jsp"
    }

    @RequestMapping("/add")
    public String add(@RequestParam("num1") int n1, @RequestParam("num2") int n2, Model model) {
        int result = n1 + n2;
        model.addAttribute("result", result);
        return "result"; // The ViewResolver looks for /views/result.jsp
    }
}
```

### The JSP View (`result.jsp`)
Sometimes, older web application configurations ignore the Expression Language (`${}`) used in JSPs. If your JSP prints `${result}` literally instead of the actual number, you must explicitly enable EL evaluation by adding `isELIgnored="false"`.

```jsp
<!-- src/main/webapp/views/result.jsp -->
<%@ page language="java" isELIgnored="false" %>
<html>
<head>
    <title>Result Page</title>
</head>
<body>
    <h2>The Calculated Result is: ${result}</h2>
</body>
</html>
```

---

## 6. Summary: The Request Flow

1. User goes to `http://localhost:8080/add?num1=5&num2=4`
2. **Tomcat** receives the request.
3. Tomcat checks `web.xml` and forwards the request to the **DispatcherServlet** (our front controller).
4. **DispatcherServlet** checks `telusko-servlet.xml` to see how the app is configured.
5. Because of `<ctx:component-scan>`, it knows `HomeController` exists. It routes the `/add` request to the `add()` method.
6. The `add()` method calculates `9`, adds it to the Model, and returns the String `"result"`.
7. **DispatcherServlet** takes `"result"` and asks the **InternalResourceViewResolver** what to do with it.
8. The ViewResolver translates it to `/views/result.jsp`.
9. The DispatcherServlet processes the JSP, injects the Model data, and returns the final HTML to Tomcat.
10. **Tomcat** sends the HTML response back to the user's browser.

**Conclusion:** Spring Boot simply automates the creation of `web.xml`, the `DispatcherServlet`, and the ViewResolver configurations!
