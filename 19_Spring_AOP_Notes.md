# Spring AOP — Comprehensive Revision Notes

## 1. Introduction: The Need for AOP

### What is AOP?
AOP stands for **Aspect-Oriented Programming**. It is not a replacement for Object-Oriented Programming (OOP) but rather compliments it. While OOP focuses on creating objects and business logic, AOP focuses on separating "cross-cutting concerns."

### What are Cross-Cutting Concerns?
In a typical application, your methods contain core **business logic** (what the client actually asked for). However, you often need to include additional logic that spans across multiple layers and methods, such as:
- **Logging** (recording method calls, parameters, etc.)
- **Security** (checking if a user is authorized before executing a method)
- **Performance Monitoring** (calculating how long a method takes to execute)
- **Validation** (checking inputs)
- **Exception Handling**

**The Problem:** If you write logging, security, and validation code inside every business method, your code becomes:
1. Cluttered and hard to read.
2. Difficult to maintain (a change to logging requires modifying hundreds of methods).
3. Highly coupled.

**The AOP Solution:** Remove this non-core code from your business methods and place it in a separate, dedicated class. AOP then automatically "weaves" this code into your business methods at runtime based on rules you define.

---

## 2. AOP Terminology (The "Movie" Analogy)

Understanding AOP requires understanding its core vocabulary. 

| Term | Definition | Movie Analogy |
| :--- | :--- | :--- |
| **Aspect** | The class containing the cross-cutting concern logic (e.g., `LoggingAspect.java`). | The script/collection of secondary scenes (e.g., behind-the-scenes actions). |
| **Advice** | The actual action/method to be executed (e.g., `logMethodCall()`). What you want to happen. | An action that happens in a particular scene. |
| **Join Point** | Any point in the application where you *could* apply an advice (e.g., method execution, exception thrown). | A specific scene in the movie where action *could* occur. |
| **Pointcut** | The expression that selects one or more Join Points where the advice *should* execute. | The director specifying exactly *which* scenes the action happens in. |
| **Target Object** | The object whose methods are being intercepted (e.g., `JobService`). | The main character/hero of the movie. |
| **Proxy** | A wrapper created by Spring around the Target Object to apply the advices. | The stunt double taking the hit for the main actor. |
| **Weaving** | The process of linking Aspects with Target Objects to create an Advised Proxy. Spring does this at **runtime**. | The editing process that combines the stunt double's action with the main movie. |

---

## 3. Types of Advice

Advice defines **when** the cross-cutting code should run relative to the matched method.

1.  **`@Before`**: Executes before the target method is called.
2.  **`@AfterReturning`**: Executes only if the target method completes successfully (no exceptions).
3.  **`@AfterThrowing`**: Executes only if the target method throws an exception.
4.  **`@After` (Finally)**: Executes after the target method finishes, regardless of success or failure.
5.  **`@Around`**: Executes *before* and *after* the target method. It has total control over the method execution, allowing it to modify inputs, modify outputs, or completely bypass the method.

---

## 4. Pointcut Expressions

A Pointcut expression tells Spring *exactly which methods* to intercept.

**Basic Syntax:**
```
execution(return_type fully_qualified_class_name.method_name(arguments))
```

**Examples:**
- Intercept *all* methods in `JobService`:
  ```java
  @Before("execution(* com.telusko.springbootrest.service.JobService.*(..))")
  ```
  *( `*` means any return type/method name. `(..)` means zero or more arguments of any type. )*

- Intercept a specific method (`getJob`):
  ```java
  @Before("execution(* com.telusko.springbootrest.service.JobService.getJob(..))")
  ```

- Intercept multiple specific methods using `||` (OR):
  ```java
  @Before("execution(* com.telusko.springbootrest.service.JobService.getJob(..)) || " +
          "execution(* com.telusko.springbootrest.service.JobService.updateJob(..))")
  ```

---

## 5. Implementation Examples

### Setup
Ensure your aspect class is annotated with both `@Component` (so Spring manages it) and `@Aspect`. You also need `slf4j` for logging (included in Spring Boot).

### Example 1: Basic Logging (`@Before`, `@After`, `JoinPoint`)

The `JoinPoint` object allows you to inspect the method being intercepted (e.g., to get its name).

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class LoggingAspect {
    private static final Logger LOGGER = LoggerFactory.getLogger(LoggingAspect.class);

    // Executes before getJob or updateJob
    @Before("execution(* com.telusko.springbootrest.service.JobService.getJob(..)) || " +
            "execution(* com.telusko.springbootrest.service.JobService.updateJob(..))")
    public void logMethodCall(JoinPoint jp) {
        LOGGER.info("Method Called: " + jp.getSignature().getName());
    }

    // Executes after successful execution
    @AfterReturning("execution(* com.telusko.springbootrest.service.JobService.getJob(..))")
    public void logMethodSuccess(JoinPoint jp) {
        LOGGER.info("Method Executed Successfully: " + jp.getSignature().getName());
    }

    // Executes if an exception is thrown
    @AfterThrowing("execution(* com.telusko.springbootrest.service.JobService.getJob(..))")
    public void logMethodCrash(JoinPoint jp) {
        LOGGER.info("Method Crashed: " + jp.getSignature().getName());
    }
}
```

### Example 2: Performance Monitoring (`@Around`)

`@Around` advice requires a `ProceedingJoinPoint`. You **must** call `.proceed()` to execute the actual target method, and you **must** return the object returned by `.proceed()`.

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class PerformanceMonitorAspect {
    private static final Logger LOGGER = LoggerFactory.getLogger(PerformanceMonitorAspect.class);

    @Around("execution(* com.telusko.springbootrest.service.JobService.*(..))")
    public Object monitorTime(ProceedingJoinPoint jp) throws Throwable {
        long start = System.currentTimeMillis();

        // Execute the target method
        Object obj = jp.proceed(); 

        long end = System.currentTimeMillis();

        LOGGER.info("Time taken by " + jp.getSignature().getName() + " : " + (end - start) + " ms");
        
        // You MUST return the object back to the caller
        return obj; 
    }
}
```

### Example 3: Input Validation / Modification (`@Around` with Args)

You can use AOP to intercept method arguments, validate them, and even modify them before the actual method sees them.

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class ValidationAspect {
    private static final Logger LOGGER = LoggerFactory.getLogger(ValidationAspect.class);

    // Using args(postId) binds the intercepted method's argument to this advice's parameter
    @Around("execution(* com.telusko.springbootrest.service.JobService.getJob(..)) && args(postId)")
    public Object validateAndUpdate(ProceedingJoinPoint jp, int postId) throws Throwable {
        
        // Validation Logic: If ID is negative, make it positive
        if (postId < 0) {
            LOGGER.info("Post ID is negative. Updating it.");
            postId = -postId;
            LOGGER.info("New Value: " + postId);
        }

        // Pass the modified argument array to proceed()
        Object obj = jp.proceed(new Object[]{postId});
        
        return obj;
    }
}
```

> **Key Takeaway for Validation:** By using `&& args(variableName)`, we capture the input. By passing `new Object[]{modifiedVariable}` to `jp.proceed()`, we alter what the target method receives.

---

## 6. Summary

- **AOP separates cross-cutting concerns** from business logic, keeping code clean and maintainable.
- **Spring AOP uses Proxies** at runtime to wrap target objects.
- **Pointcut expressions** define the "where" and "when".
- **JoinPoint** allows inspection of the method (`getName()`).
- **ProceedingJoinPoint** is used in `@Around` advice to explicitly call the target method and modify inputs/outputs.
