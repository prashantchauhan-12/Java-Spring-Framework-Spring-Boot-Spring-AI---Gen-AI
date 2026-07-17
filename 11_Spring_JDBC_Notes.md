# Spring JDBC: Comprehensive Notes

## 1. Introduction: Why Spring JDBC?
In traditional Java Database Connectivity (JDBC), developers had to manually write a lot of boilerplate code:
- Loading the database driver.
- Opening and closing connections.
- Creating statements and prepared statements.
- Handling complex exception catching for SQL errors.

**Spring JDBC** solves these problems by providing the **`JdbcTemplate`** class, which handles the boilerplate code, connection pooling, and resource management behind the scenes. Spring Boot auto-configures these beans (like `DataSource` and `JdbcTemplate`) so you just inject them and use them.

---

## 2. Setting Up the Project (Dependencies)
To use Spring JDBC with an embedded in-memory database like **H2** (perfect for testing because it is lightweight and auto-configures itself), you need the following dependencies in your `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot JDBC Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    
    <!-- H2 Database (In-Memory) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```
*Note: H2 is a 2MB footprint database. It stores data in memory, meaning data is lost when the application shuts down.*

---

## 3. Creating the Application Structure
Following the N-Tier architecture, we create the Model, Service, and Repository layers.

### A. The Model (`Student.java`)
The class properties should ideally map exactly to the table columns in your database.
```java
package com.telusko.model;

import org.springframework.stereotype.Component;

@Component
public class Student {
    private int rollNo;
    private String name;
    private int marks;
    
    // Getters, Setters, and toString() generated here...
}
```

### B. The Service Layer (`StudentService.java`)
```java
package com.telusko.service;

import com.telusko.model.Student;
import com.telusko.repo.StudentRepo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class StudentService {
    
    @Autowired
    private StudentRepo repo;

    public void addStudent(Student s) {
        repo.save(s);
    }
    
    public List<Student> getStudents() {
        return repo.findAll();
    }
}
```

---

## 4. Initializing the Database (H2 Auto-Configuration)
When using H2, you don't need to manually create tables in a console. Spring Boot automatically executes two specific files if they are placed in the `src/main/resources` folder:

1. **`schema.sql`** (To define the table structure)
```sql
CREATE TABLE student (
    rollno INT PRIMARY KEY,
    name VARCHAR(50),
    marks INT
);
```

2. **`data.sql`** (To pre-load initial data)
```sql
INSERT INTO student (rollno, name, marks) VALUES (101, 'Kiran', 79);
INSERT INTO student (rollno, name, marks) VALUES (102, 'Harsh', 68);
INSERT INTO student (rollno, name, marks) VALUES (103, 'Sushil', 82);
```

---

## 5. Working with `JdbcTemplate` in the Repository

The repository is where the actual SQL interactions occur. We inject the auto-configured `JdbcTemplate`.

### A. Inserting Data (`execute update`)
For `INSERT`, `UPDATE`, and `DELETE` queries, we use `jdbc.update()`.

```java
package com.telusko.repo;

import com.telusko.model.Student;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class StudentRepo {

    @Autowired
    private JdbcTemplate jdbc;

    public void save(Student s) {
        String sql = "INSERT INTO student (rollno, name, marks) VALUES (?, ?, ?)";
        
        // The update method takes the query and the values for the '?' placeholders
        int rows = jdbc.update(sql, s.getRollNo(), s.getName(), s.getMarks());
        System.out.println(rows + " row(s) affected.");
    }
}
```

### B. Fetching Data (`execute query` & `RowMapper`)
For `SELECT` queries, we use `jdbc.query()`. This method requires a **`RowMapper`**, which tells Spring how to convert a database row (`ResultSet`) into a Java Object (`Student`).

#### Using Traditional Anonymous Inner Class:
```java
import org.springframework.jdbc.core.RowMapper;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

public List<Student> findAll() {
    String sql = "SELECT * FROM student";
    
    RowMapper<Student> mapper = new RowMapper<Student>() {
        @Override
        public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
            Student s = new Student();
            s.setRollNo(rs.getInt("rollno"));
            s.setName(rs.getString("name"));
            s.setMarks(rs.getInt("marks"));
            return s;
        }
    };
    
    return jdbc.query(sql, mapper);
}
```

#### Using Modern Lambda Expressions:
Because `RowMapper` is a Functional Interface, we can significantly reduce the boilerplate using a lambda.
```java
public List<Student> findAll() {
    String sql = "SELECT * FROM student";
    
    return jdbc.query(sql, (rs, rowNum) -> {
        Student s = new Student();
        s.setRollNo(rs.getInt("rollno"));
        s.setName(rs.getString("name"));
        s.setMarks(rs.getInt("marks"));
        return s;
    });
}
```

---

## 6. Switching from In-Memory (H2) to an External Database (PostgreSQL)

The true beauty of Spring JDBC is that **your Java code (Service and Repository layers) does NOT change** when switching databases.

To migrate to PostgreSQL:

### Step 1: Update `pom.xml`
Remove or comment out the H2 dependency, and add the PostgreSQL JDBC driver.
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### Step 2: Configure `application.properties`
Since it's no longer an embedded database, you must tell Spring Boot where your external database lives by updating `src/main/resources/application.properties`.

```properties
# Database URL (Protocol, IP, Port, Database Name)
spring.datasource.url=jdbc:postgresql://localhost:5432/telusko

# Database Credentials
spring.datasource.username=postgres
spring.datasource.password=0000

# Driver Class Name (Changes based on DBMS: MySQL, Oracle, Postgres, etc.)
spring.datasource.driver-class-name=org.postgresql.Driver
```

*(Note: You must also ensure that the database `telusko` and the `student` table exist inside your PostgreSQL server before running the application, as external DBs will not auto-run `schema.sql` by default in the exact same way without further configuration).*
