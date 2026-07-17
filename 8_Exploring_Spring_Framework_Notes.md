# Exploring Spring Framework (Core) Crash Course: Detailed Notes

## 1. Introduction: Moving from Spring Boot to Spring Core
Spring Boot handles a lot of configuration automatically (auto-configuration) and uses annotations to simplify development. However, to truly understand how the Spring IoC Container and Dependency Injection work behind the scenes, we must step back and build a project using pure **Spring Framework** (without Spring Boot).

---

## 2. Project Setup: Maven Quick Start
To build a raw Spring Framework project:
1. **Create a Maven Project** (Using `maven-archetype-quickstart`).
2. Add the **Spring Context** dependency manually to the `pom.xml`.
```xml
<!-- In pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.1.1</version> <!-- Use the second-to-last stable version -->
    </dependency>
</dependencies>
```
3. Load the Maven changes to download the Spring libraries.

---

## 3. The IoC Container: ApplicationContext
In a pure Spring application, you must manually initialize the IoC container. 
- **BeanFactory** is the older, basic interface for the container.
- **ApplicationContext** is the modern superset of `BeanFactory` with more features.
- Since we are using an XML configuration, we use `ClassPathXmlApplicationContext`.

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        // Initializes the IoC container by reading the spring.xml file from the resources folder
        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");

        // Request the bean from the container
        // Note: passing Alien.class removes the need to cast from Object
        Alien obj = context.getBean("alien1", Alien.class);
        obj.code();
    }
}
```

---

## 4. XML Configuration (`spring.xml`)
Instead of using `@Component`, we define our beans in an XML file. This file must be placed in the `src/main/resources` folder so it resides on the classpath.

### Defining Beans
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- A simple bean definition -->
    <bean id="laptop1" class="com.telusko.Laptop"/>

</beans>
```

---

## 5. Dependency Injection via XML
There are two main ways to inject values or objects into a bean: **Setter Injection** and **Constructor Injection**.

### A. Setter Injection (`<property>`)
Spring calls the setter methods (e.g., `setAge()`, `setLap()`) of the class to inject the values.
- `value=""` is used for primitive types and Strings.
- `ref=""` is used to inject other beans (reference to objects).

```xml
<bean id="alien1" class="com.telusko.Alien">
    <property name="age" value="21" />      <!-- Calls setAge(21) -->
    <property name="lap" ref="laptop1" />   <!-- Calls setLap(laptop1) -->
</bean>
```

### B. Constructor Injection (`<constructor-arg>`)
Spring passes the values directly to the constructor when instantiating the object.
- **Ambiguity Problem:** If you pass values blindly, Spring relies on sequence, which can break if the constructor expects different types.
- **Solutions to solve ambiguity:**
  1. **By Type:** `<constructor-arg type="int" value="21" />`
  2. **By Index (Recommended):** `<constructor-arg index="0" value="21" />`
  3. **By Name:** `<constructor-arg name="age" value="21" />` (Requires `@ConstructorProperties({"age"})` in Java code).

```xml
<bean id="alien1" class="com.telusko.Alien">
    <constructor-arg index="0" value="21" />
    <constructor-arg index="1" ref="laptop1" />
</bean>
```

---

## 6. Coding to Interfaces (Loose Coupling)
To make your application scalable, classes should depend on **Interfaces** rather than concrete implementations.
- Create a `Computer` interface.
- Create `Laptop` and `Desktop` classes that implement `Computer`.
- The `Alien` class now has a dependency on `Computer` instead of `Laptop`.

```java
public interface Computer {
    void compile();
}

public class Laptop implements Computer {
    public void compile() { System.out.println("Compiling using Laptop..."); }
}

public class Desktop implements Computer {
    public void compile() { System.out.println("Compiling using Desktop..."); }
}

public class Alien {
    private Computer com;
    public void setCom(Computer com) { this.com = com; }
    public void code() { com.compile(); }
}
```

---

## 7. Autowiring (`autowire="..."`)
Instead of manually writing `<property name="com" ref="laptop1" />`, you can tell Spring to automatically wire dependencies.

### 1. `autowire="byName"`
Spring looks at the variable name in your class (e.g., `com`) and searches the XML for a bean with the exact same `id="com"`.
```xml
<bean id="com" class="com.telusko.Laptop" />
<bean id="alien1" class="com.telusko.Alien" autowire="byName" />
```

### 2. `autowire="byType"`
Spring ignores the name and looks for a bean that matches the *Type* of the variable (e.g., any class implementing `Computer`).
```xml
<bean id="laptop1" class="com.telusko.Laptop" />
<bean id="alien1" class="com.telusko.Alien" autowire="byType" />
```

### The "Primary" Attribute Conflict Resolution
If you use `autowire="byType"`, but have **two** beans that implement `Computer` (e.g., a Desktop and a Laptop bean), Spring will throw a `NoUniqueBeanDefinitionException` because it doesn't know which one to inject.
- **Solution:** Add `primary="true"` to the bean you want Spring to favor.
```xml
<bean id="laptop1" class="com.telusko.Laptop" primary="true" />
<bean id="desktop1" class="com.telusko.Desktop" />
<bean id="alien1" class="com.telusko.Alien" autowire="byType" /> <!-- Will pick laptop1 -->
```

---

## 8. Bean Scopes
By default, the scope of a bean in Spring is **Singleton**.
1. **Singleton (Default):** Spring creates exactly ONE instance of the bean. If you call `getBean()` multiple times, you receive the same memory reference. The object is created automatically as soon as the container loads.
2. **Prototype:** Spring creates a NEW instance every time `getBean()` is called. The object is not created until you explicitly request it.

```xml
<bean id="alien1" class="com.telusko.Alien" scope="prototype" />
```

---

## 9. Lazy Initialization (`lazy-init`)
By default, singleton beans are **Eagerly** loaded. This means they are instantiated the moment `new ClassPathXmlApplicationContext()` executes, even if you never use them in your application.
- For a large application with hundreds of beans, eager initialization consumes a lot of memory and slows down startup time.
- **Solution:** Set `lazy-init="true"`. The bean will only be created when `getBean()` is called for the very first time.

```xml
<bean id="desktop1" class="com.telusko.Desktop" lazy-init="true" />
```

**Important Catch:** If an eager bean (like `Alien`) depends on a lazy bean (like `Desktop`), Spring is forced to create the lazy bean during startup anyway to satisfy the eager bean's dependency.
