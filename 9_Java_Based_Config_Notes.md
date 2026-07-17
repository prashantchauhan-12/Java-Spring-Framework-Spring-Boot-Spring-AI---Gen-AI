# Spring Framework: Java-Based & Annotation Configuration

## 1. Introduction to Java-Based Configuration
While XML configuration (`spring.xml`) was traditionally used to configure Spring beans, modern Spring applications heavily favor **Java-based Configuration**. It is type-safe, easier to read, and refactor-friendly.

To use Java-based configuration, we replace the `spring.xml` file with a Java class annotated with `@Configuration`. We also replace `ClassPathXmlApplicationContext` with `AnnotationConfigApplicationContext`.

### Setting up the Application Context
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {
    public static void main(String[] args) {
        // Load the Spring context using the Java configuration class
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        Desktop dt = context.getBean(Desktop.class);
        dt.compile();
    }
}
```

---

## 2. Defining Beans with `@Bean`
In the configuration class, you define methods that return the objects you want Spring to manage, and annotate them with `@Bean`.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public Desktop desktop() {
        // Spring calls this method to create the object and stores it in the IoC container.
        return new Desktop();
    }
}
```

### Naming Beans
- **Default Name:** The default bean ID is the exact name of the method (e.g., `"desktop"`).
- **Custom Name:** You can assign a custom name or multiple aliases using the `name` attribute.
```java
@Bean(name = {"com2", "desktop1", "Beast"})
public Desktop desktop() {
    return new Desktop();
}
```

### Defining Scopes (`@Scope`)
Just like XML, beans are **Singleton** by default. To create a new object every time `getBean()` is called, use `@Scope("prototype")`.
```java
import org.springframework.context.annotation.Scope;

@Bean
@Scope("prototype")
public Desktop desktop() {
    return new Desktop();
}
```

---

## 3. Wiring Dependencies in Java Config
If a bean depends on another bean, you can manually set it up inside the config class.

```java
@Bean
public Alien alien(Computer com) { 
    Alien obj = new Alien();
    obj.setAge(25);
    // Spring automatically injects the 'Computer' bean into the method parameter 'com'
    obj.setCom(com); 
    return obj;
}

@Bean
public Desktop desktop() {
    return new Desktop();
}
```

### Resolving Ambiguity in Java Config
If you have multiple beans implementing `Computer` (e.g., `Laptop` and `Desktop`), Spring will throw an error when trying to inject `Computer`.
- Use `@Primary` on the preferred bean method.
- OR use `@Qualifier("desktop")` as a parameter annotation to specify exactly which bean you want.

---

## 4. Stereotype Annotations (`@Component`)
Writing `@Bean` methods for every single class in a large application is tedious. Spring provides **Stereotype Annotations** to auto-detect and manage beans directly from the class definitions.

By adding `@Component` to a class, you tell Spring: *"Hey, manage this class for me."*

```java
import org.springframework.stereotype.Component;

@Component
public class Desktop implements Computer {
    public void compile() {
        System.out.println("Compiling using Desktop...");
    }
}
```

### Enabling Component Scan
For Spring to find the `@Component` classes, you must enable **Component Scanning** in your configuration class by providing the base package where your classes reside.

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.telusko") // Spring will scan this package and its sub-packages
public class AppConfig {
    // You no longer need @Bean methods here!
}
```

---

## 5. Autowiring Dependencies (`@Autowired`)
When you let Spring create beans via `@Component`, you also need to tell it how to connect (inject) them together. You do this using `@Autowired`.

### Types of Injection:
1. **Field Injection (Common but least recommended):**
```java
@Component
public class Alien {
    @Autowired
    private Computer com;
}
```
2. **Setter Injection:**
```java
@Component
public class Alien {
    private Computer com;

    @Autowired
    public void setCom(Computer com) {
        this.com = com;
    }
}
```
3. **Constructor Injection (Most recommended):**
```java
@Component
public class Alien {
    private Computer com;

    @Autowired
    public Alien(Computer com) {
        this.com = com;
    }
}
```
*Note: In modern Spring versions, if a class only has one constructor, the `@Autowired` annotation is optional for constructor injection.*

---

## 6. Resolving Autowiring Ambiguity
If you have both `@Component` on `Laptop` and `@Component` on `Desktop`, Spring won't know which one to inject into the `Computer` reference inside `Alien`.

**Solution 1: By Name matching**
Spring will fallback to matching the variable name with the bean name (which defaults to the lowercase class name).
```java
@Autowired
private Computer desktop; // Will inject the Desktop bean
```

**Solution 2: `@Qualifier` (Highest Priority)**
Specify exactly which bean name to use.
```java
@Autowired
@Qualifier("laptop")
private Computer com;
```

**Solution 3: Custom Bean Name + `@Qualifier`**
You can give a custom name to a component and use it in the qualifier.
```java
@Component("com2")
public class Desktop implements Computer { ... }

// In Alien.java:
@Autowired
@Qualifier("com2")
private Computer com;
```

**Solution 4: `@Primary`**
Add `@Primary` to one of the component classes. If no qualifier is provided, Spring will always choose the primary bean.
```java
@Component
@Primary
public class Laptop implements Computer { ... }
```

---

## 7. Injecting Values (`@Value`)
To assign primitive values or strings (rather than objects), you can use the `@Value` annotation. This is vastly superior to hardcoding values because `@Value` allows you to inject data dynamically from external property files (like `application.properties`).

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class Alien {
    
    @Value("21") // Injects the value 21 into age
    private int age;
    
    // You can also use SpEL (Spring Expression Language) or properties: @Value("${alien.age}")
}
```
