# Spring Framework & Spring Boot: Getting Started Notes

## 1. Introduction to Spring and Spring Boot
- **Spring Framework:** A comprehensive, lightweight framework for building enterprise-level Java applications. It provides everything needed to build complex applications (web, microservices, cloud, reactive, event-driven) in one ecosystem.
- **Spring Boot:** An opinionated framework built *on top of* the Spring Framework. It auto-configures many things for you, making it extremely easy and fast to create production-ready Spring applications without writing boilerplate configuration. Spring Boot 3 works on top of Spring Framework 6.
- **Why Spring?**
  - Works well with POJOs (Plain Old Java Objects).
  - Huge ecosystem (Spring Data, Spring Security, Spring Cloud, Spring Batch, etc.).
  - Great community and excellent official documentation at `spring.io`.

---

## 2. Prerequisites
Before diving into Spring, ensure you have a solid understanding of:
- **Core Java:** Syntax, OOP concepts, Exception Handling, Collections framework.
- **JDBC:** Java Database Connectivity for basic database operations.
- **Build Tools:** Maven or Gradle (this course uses Maven).
- **ORM:** Object-Relational Mapping (e.g., Hibernate).
- **Servlets:** Basic knowledge of servlets and web containers (like Tomcat).

---

## 3. Setup and IDE Configuration
- **Java Version:** Spring 6 (and Spring Boot 3) requires **Java 17 or above** (Java 21 works perfectly).
- **IDEs:** 
  - **Eclipse:** Install the "Spring Tools 4" plugin via the Eclipse Marketplace to get Spring project support.
  - **IntelliJ IDEA:** The Ultimate version has built-in Spring support. For the Community version, you can generate the project externally and open it.
  - **VS Code:** Install the "Spring Boot Extension Pack" (by VMware).

---

## 4. Creating Your First Spring Boot Project
Since IDEs like IntelliJ Community lack direct Spring project generation, you can use the official Spring generator:
1. Go to **[start.spring.io](https://start.spring.io/)** (Spring Initializr).
2. Choose **Project:** Maven.
3. Choose **Language:** Java.
4. Choose **Spring Boot Version:** 3.2.x (Avoid snapshots).
5. Fill in Metadata (Group, Artifact, Package name).
6. **Packaging:** Jar, **Java:** 17 or 21.
7. Click **Generate** to download the `.zip` file.
8. Unzip the file and open the folder in your IDE.

*Note: The generated `pom.xml` will have the `spring-boot-starter` dependency, which pulls in the core Spring Framework behind the scenes.*

---

## 5. Core Concepts: IoC and DI
### Inversion of Control (IoC)
- **Traditional Approach:** The programmer writes `new Laptop()` to create an object, thereby taking control over the object's lifecycle (creation, management, destruction).
- **IoC Principle:** The control of creating and managing objects is transferred from the programmer to a framework (Spring). Spring acts as a container (the **IoC Container**) that holds all the objects.
- **Beans:** Any object that is created and managed by the Spring IoC Container is called a "Bean".

### Dependency Injection (DI)
- DI is a design pattern used to implement the IoC principle. 
- If class `Alien` depends on class `Laptop`, both objects reside in the IoC Container. Spring "injects" the `Laptop` object into the `Alien` object for you.

---

## 6. Implementing Dependency Injection in Spring Boot
When you run a Spring Boot application, it starts the Spring IoC Container. You can access this container using the `ApplicationContext`.

### Step 1: Getting the Application Context
The `SpringApplication.run()` method returns an instance of `ConfigurableApplicationContext`.
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        // Starts the Spring container and returns the context
        ApplicationContext context = SpringApplication.run(App.class, args);
        
        // Requesting the Bean from the container
        Alien alienObj = context.getBean(Alien.class);
        alienObj.code();
    }
}
```

### Step 2: Marking Classes as Beans (`@Component`)
By default, Spring does not create objects for every class in your project. You must explicitly tell Spring which classes it should manage by annotating them with `@Component`.

```java
import org.springframework.stereotype.Component;

@Component // Tells Spring to manage this class (create its object in the container)
public class Laptop {
    public void compile() {
        System.out.println("Compiling code...");
    }
}
```

### Step 3: Injecting Dependencies (`@Autowired`)
If `Alien` needs a `Laptop`, you declare a `Laptop` field inside `Alien` and annotate it with `@Autowired`. This tells Spring to wire the two beans together.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Alien {
    
    @Autowired // Instructs Spring to inject the Laptop bean from the container here
    private Laptop laptop;
    
    public void code() {
        System.out.println("Coding...");
        laptop.compile(); // Uses the injected object!
    }
}
```

### Summary of Execution Flow:
1. Spring Boot starts.
2. It scans for classes marked with `@Component` (`Alien`, `Laptop`).
3. It creates instances of these classes in the IoC container.
4. It sees `@Autowired` on the `laptop` field in `Alien`, so it injects the `Laptop` bean into the `Alien` bean.
5. In `main`, you retrieve the fully assembled `Alien` bean using `context.getBean(Alien.class)` and use it.
