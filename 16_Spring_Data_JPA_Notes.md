# Spring Data JPA — Comprehensive Revision Notes

## 1. Introduction: The Evolution of Database Access

In our previous applications, we progressed through different ways to connect to a database:

1.  **JDBC (Java Database Connectivity):** The oldest, most verbose way. Requires manual connection management, writing complex SQL queries as strings, and manual result set mapping.
2.  **Spring JDBC (JdbcTemplate):** Simplified JDBC by removing connection boilerplate, but we still had to write raw SQL queries for every operation (`INSERT`, `SELECT`, etc.).

### The Problem with SQL in Java
Java is an Object-Oriented Language. We work with **Objects** (e.g., `Student`, `JobPost`). Databases are Relational. They work with **Tables** and **Rows**.
Writing raw SQL means we constantly have to manually map between our Java Objects and our Database Rows.

### The Solution: ORM (Object-Relational Mapping)
ORM bridges the gap between the Object world (Java) and the Relational world (Database).
*   **Class** → Maps to a **Table**
*   **Properties (Variables)** → Map to **Columns**
*   **Object** → Maps to a single **Row**

With ORM, you ask the framework: *"Here is my Object. Please save it."* The ORM tool generates the `INSERT` SQL query for you behind the scenes.

**Hibernate** is the most popular ORM tool for Java.

### Where does JPA fit in?
*   **JPA (Jakarta Persistence API, formerly Java Persistence API):** This is just a **specification** (a set of interfaces and rules). It doesn't actually *do* the database work.
*   **Hibernate:** This is the **implementation** of the JPA specification.
*   **Spring Data JPA:** This is a Spring module that sits on top of JPA (which uses Hibernate by default). It drastically reduces the amount of boilerplate code required to implement data access layers. It automatically generates the repository implementation at runtime.

---

## 2. Spring Data JPA Setup and Configuration

To use Spring Data JPA, you don't write SQL. You define your entity and an interface.

### Dependencies (`pom.xml`)

You need the Spring Data JPA starter and the driver for your chosen database (e.g., PostgreSQL or H2).

```xml
<!-- Spring Data JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- PostgreSQL Driver -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### Application Properties (`application.properties`)

You need to configure the database connection and tell Hibernate how to behave.

```properties
# Database Connection Info
spring.datasource.url=jdbc:postgresql://localhost:5432/telusko
spring.datasource.username=postgres
spring.datasource.password=0000
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate Configuration
# 'update' creates the table if it doesn't exist, and updates it if the schema changes.
# Other options: 'create' (drops and recreates every time), 'none', 'validate'
spring.jpa.hibernate.ddl-auto=update

# Show the SQL queries Hibernate generates in the console (helpful for debugging)
spring.jpa.show-sql=true
```

---

## 3. Defining the Entity (The Model)

To tell JPA that a Java class should be mapped to a database table, we use annotations.

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import org.springframework.stereotype.Component;

@Component
@Entity // Tells JPA: "Create a table for this class"
public class Student {

    @Id // Tells JPA: "This field is the Primary Key"
    private int rollNo;
    private String name;
    private int marks;

    // Getters, Setters, Constructors, toString() omitted for brevity
}
```

*   **`@Entity`**: Marks the class as a JPA entity. By default, the table name will be the same as the class name (e.g., `student`).
*   **`@Id`**: Denotes the primary key of the entity. Every entity **must** have an identifier.

---

## 4. Creating the Repository Interface

This is where the magic of Spring Data JPA happens. Instead of writing a class with SQL queries, you create an **Interface** that extends `JpaRepository`.

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface StudentRepo extends JpaRepository<Student, Integer> {
    // That's it! No methods needed for basic CRUD.
}
```

### Understanding `JpaRepository<T, ID>`
You must specify two generic types:
1.  **`T` (Entity Type):** The class you are managing (e.g., `Student`).
2.  **`ID` (Primary Key Type):** The wrapper class for the type of your `@Id` field (e.g., `Integer` for an `int` primary key).

By simply extending `JpaRepository`, Spring automatically provides implementations for standard CRUD operations:
*   `save(entity)` (Creates or Updates)
*   `findById(id)` (Reads one)
*   `findAll()` (Reads all)
*   `deleteById(id)` (Deletes)
*   `count()` (Counts rows)

---

## 5. Basic CRUD Operations in Action

Here is how you use the injected `StudentRepo` (e.g., inside a Service class or Main application).

### Create (Insert)
Use the `.save()` method.

```java
Student s1 = new Student(101, "Navin", 75);
repo.save(s1); // Generates: INSERT INTO student (marks, name, roll_no) VALUES (?, ?, ?)
```

### Read (Select)
#### Fetch All Records
```java
List<Student> students = repo.findAll();
// Generates: SELECT * FROM student
```

#### Fetch Single Record by Primary Key (`findById`)
Because a record might not exist, `findById` returns an **`Optional<T>`** (introduced in Java 8) to prevent `NullPointerException`.

```java
// Method 1: Using orElse to provide a default if not found
Student s = repo.findById(103).orElse(new Student());

// Method 2: Checking if present
Optional<Student> opt = repo.findById(103);
if(opt.isPresent()){
    Student s2 = opt.get();
}
```

### Update
To update, you fetch the existing record, modify it, and call `.save()` again. **There is no separate `update()` method.**

```java
// 1. Fetch the existing entity
Student s = repo.findById(102).orElse(new Student());

// 2. Modify it
s.setMarks(65);

// 3. Save it back
repo.save(s);
// Generates: UPDATE student SET marks=?, name=? WHERE roll_no=?
```
*Note: Hibernate usually fires a `SELECT` query first before updating to check if the record actually exists.*

### Delete
```java
repo.deleteById(102);
// Generates: SELECT ... then DELETE FROM student WHERE roll_no=?
```

---

## 6. Custom Queries: Query DSL (Domain Specific Language)

What if you need to search by something other than the Primary Key? For example, searching for a student by their `name` or `marks`.

Spring Data JPA allows you to define custom queries simply by **naming your method correctly**. You don't have to write the implementation!

### 6.1 Derived Query Methods (Method Naming Convention)

You declare the method signature in your interface following specific naming rules:
`findBy` + `PropertyName` + `Condition`

```java
@Repository
public interface StudentRepo extends JpaRepository<Student, Integer> {

    // Fetch students by exact name
    // Generates: SELECT * FROM student WHERE name = ?
    List<Student> findByName(String name);

    // Fetch students by exact marks
    // Generates: SELECT * FROM student WHERE marks = ?
    List<Student> findByMarks(int marks);

    // Fetch students with marks greater than a value
    // Generates: SELECT * FROM student WHERE marks > ?
    List<Student> findByMarksGreaterThan(int marks);
}
```

#### Applying this to the Job Portal Example (Searching via React UI)
In the job portal, we wanted a search bar that checked if a keyword was present in *either* the job profile *or* the description.

```java
@Repository
public interface JobRepo extends JpaRepository<JobPost, Integer> {

    // Using 'Containing' acts like a SQL LIKE '%keyword%'
    // Using 'Or' allows searching across multiple columns
    List<JobPost> findByPostProfileContainingOrPostDescriptionContaining(String keyword1, String keyword2);
}
```
*Service Implementation for the above:*
```java
public List<JobPost> search(String keyword) {
    // We pass the same keyword to both parameters
    return repo.findByPostProfileContainingOrPostDescriptionContaining(keyword, keyword);
}
```

### 6.2 Using `@Query` (JPQL)
If the method name becomes too long or the query is too complex for the naming convention, you can use the `@Query` annotation and write **JPQL (Java Persistence Query Language)**.

**JPQL vs SQL:**
*   **SQL:** Operates on Tables and Columns (`SELECT * FROM student WHERE name = 'Navin'`)
*   **JPQL:** Operates on Java Classes and Properties (`SELECT s FROM Student s WHERE s.name = ?1`)

```java
@Repository
public interface StudentRepo extends JpaRepository<Student, Integer> {

    // Custom JPQL query.
    // 'Student' is the Class name, 's' is an alias, 'name' is the class variable.
    // '?1' refers to the first method parameter.
    @Query("SELECT s FROM Student s WHERE s.name = ?1")
    List<Student> fetchByName(String name);
}
```

---

## 7. Converting the Job Portal to Spring Data JPA

Let's review the steps taken to migrate the `JobPost` application from using a hardcoded `List` to an actual Database using JPA.

**1. Update the Model (`JobPost.java`)**
Add `@Entity` and `@Id`.
```java
@Entity
public class JobPost {
    @Id
    private int postId;
    private String postProfile;
    private String postDesc;
    private int reqExperience;
    private List<String> postTechStack; // Note: Handled automatically as an array type in Postgres
    // ...
}
```

**2. Create the Interface (`JobRepo.java`)**
Delete the old class implementation and replace it with an interface.
```java
@Repository
public interface JobRepo extends JpaRepository<JobPost, Integer> {
    List<JobPost> findByPostProfileContainingOrPostDescriptionContaining(String keyword1, String keyword2);
}
```

**3. Update the Service Layer (`JobService.java`)**
Change the service to call the inherited methods from `JpaRepository`.
```java
@Service
public class JobService {
    @Autowired
    private JobRepo repo;

    public void addJob(JobPost jobPost) {
        repo.save(jobPost); // Was: repo.addJob()
    }
    public List<JobPost> getAllJobs() {
        return repo.findAll(); // Was: repo.getAllJobs()
    }
    public JobPost getJob(int postId) {
        return repo.findById(postId).orElse(new JobPost()); // Was: repo.getJob()
    }
    public void updateJob(JobPost jobPost) {
        repo.save(jobPost); // Was: repo.updateJob()
    }
    public void deleteJob(int postId) {
        repo.deleteById(postId); // Was: repo.deleteJob()
    }
    public List<JobPost> search(String keyword) {
        return repo.findByPostProfileContainingOrPostDescriptionContaining(keyword, keyword);
    }
}
```
*(The Controller layer requires absolutely **zero** changes!)*

### Loading Initial Data (`saveAll`)
To avoid an empty database upon startup, you can use `saveAll()` to insert a list of objects at once.

```java
List<JobPost> jobs = new ArrayList<>(Arrays.asList(...)); // List of JobPost objects
repo.saveAll(jobs); // Inserts all objects into the DB
```

---

## Summary of Key JPA Concepts

| Concept | Description |
| :--- | :--- |
| **ORM** | Object Relational Mapping. Converts Objects to Rows and vice-versa. |
| **JPA** | Java Persistence API. The specification/rules for Java ORM. |
| **Hibernate** | The default implementation of JPA used by Spring Boot. |
| **`@Entity`** | Marks a class to be mapped to a database table. |
| **`@Id`** | Marks a field as the primary key. |
| **`JpaRepository<T, ID>`** | Interface to extend to instantly get CRUD methods (`save`, `findById`, `findAll`, `deleteById`). |
| **`spring.jpa.hibernate.ddl-auto=update`** | Property that tells Hibernate to automatically create/update the database schema based on your Entities. |
| **Query DSL (Method Naming)** | E.g., `findByMarksGreaterThan(int marks)`. Spring generates the query based on the method name. |
| **`@Query` (JPQL)** | Used for complex queries where you write SQL-like syntax using Class and Property names instead of Table and Column names. |
| **`Optional<T>`** | Returned by `findById` to handle cases where a record might not exist in the database, avoiding NullPointerExceptions. |
