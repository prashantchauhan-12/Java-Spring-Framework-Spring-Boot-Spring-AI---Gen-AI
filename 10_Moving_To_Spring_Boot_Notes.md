# Moving to Spring Boot: Detailed Notes

## 1. Spring Framework vs. Spring Boot
Throughout the Spring Core modules, we learned how to configure the IoC Container, create beans, and inject dependencies manually using XML or Java-based configurations (`@Configuration`, `@Bean`, `@ComponentScan`).

**Spring Boot** removes the need for almost all of this manual configuration. 
- The `@SpringBootApplication` annotation on the main class does heavy lifting behind the scenes.
- It automatically configures the Application Context.
- It inherently acts as `@ComponentScan`, automatically scanning the current package and sub-packages for any stereotype annotations (`@Component`, `@Service`, etc.).
- It runs the embedded container (like Tomcat for web apps) and handles the application lifecycle.

Any annotations learned in Spring Core (`@Primary`, `@Qualifier`, `@Value`, `@Autowired`) work exactly the same in Spring Boot.

---

## 2. Using `@Value` for Default Properties
You can inject default values into your Spring-managed beans using the `@Value` annotation. This is cleaner than hardcoding assignment in the constructor.

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class Alien {
    
    @Value("25") // Injects 25 as the default value
    private int age;
    
    @Autowired
    private Computer com;
    
    // getters, setters, and other methods...
}
```

---

## 3. N-Tier Architecture & Advanced Stereotype Annotations
While `@Component` tells Spring to manage a class, a typical enterprise web application is divided into specific layers. To make the code's intent clear and to enable specific framework capabilities for each layer, Spring provides specialized stereotype annotations. All of them behave as `@Component` under the hood, but carry semantic meaning.

### The Standard Web Application Flow
1. **Client:** The user interface or consumer sending HTTP requests (e.g., a browser, Postman, or mobile app).
2. **Controller Layer (`@Controller` / `@RestController`):** Receives the request from the client, validates it, and decides what response to send back. It shouldn't contain business logic.
3. **Service Layer (`@Service`):** Contains the core business logic (e.g., calculating taxes, filtering products). It processes data received from the controller.
4. **Repository / DAO Layer (`@Repository`):** The Data Access Object layer. Its sole responsibility is to communicate with the Database (fetch, save, update, delete records).
5. **Database:** The actual database (MySQL, PostgreSQL, MongoDB, etc.).

### Project Structure (Packages)
To keep the code clean, classes should be separated into packages based on their role:
- `com.example.model` (Entities like `Laptop`, `Alien`, `Student`)
- `com.example.controller`
- `com.example.service`
- `com.example.repo`

---

## 4. Implementing the Layers in Spring Boot

### Step 1: The Model
Models represent the data structures of the application. They are typically just POJOs (Plain Old Java Objects). If they are to be stored in a database later, they might be annotated as Entities.
```java
package com.telusko.model;

import org.springframework.stereotype.Component;

@Component
public class Laptop implements Computer {
    public void compile() {
        System.out.println("Compiling in Laptop");
    }
}
```

### Step 2: The Repository Layer (`@Repository`)
The repository is responsible for writing/reading from the DB. We annotate it with `@Repository`.
```java
package com.telusko.repo;

import com.telusko.model.Laptop;
import org.springframework.stereotype.Repository;

@Repository // Tells Spring this is a data access class
public class LaptopRepository {
    
    public void save(Laptop lap) {
        // JDBC or ORM code to save the laptop into the database goes here.
        System.out.println("Saved in database...");
    }
}
```

### Step 3: The Service Layer (`@Service`)
The service layer contains business logic and calls the repository layer to persist data. We annotate it with `@Service`. We use `@Autowired` to inject the `LaptopRepository`.

```java
package com.telusko.service;

import com.telusko.model.Laptop;
import com.telusko.repo.LaptopRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service // Tells Spring this is a business logic class
public class LaptopService {
    
    @Autowired
    private LaptopRepository repo;
    
    public void addLaptop(Laptop lap) {
        System.out.println("Method called...");
        // Additional business logic checks can go here
        
        repo.save(lap); // Pass the data to the repository layer
    }
}
```

### Step 4: Executing the Flow
In a real app, a Controller would call the Service. In our console app example, we can test it from the main method.

```java
package com.telusko;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

import com.telusko.model.Laptop;
import com.telusko.service.LaptopService;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(App.class, args);
        
        LaptopService service = context.getBean(LaptopService.class);
        Laptop lap = context.getBean(Laptop.class);
        
        // This will print "Method called..." then "Saved in database..."
        service.addLaptop(lap); 
    }
}
```

### Summary of Stereotypes
- `@Component`: Generic stereotype for any Spring-managed component.
- `@Service`: Marks a class as a Service layer (Business Logic).
- `@Repository`: Marks a class as a Data Access layer (DB interactions). Translates DB-specific exceptions into Spring's unified DataAccessException hierarchy.
- `@Controller`: Marks a class as a Web Controller (Handles HTTP requests).
