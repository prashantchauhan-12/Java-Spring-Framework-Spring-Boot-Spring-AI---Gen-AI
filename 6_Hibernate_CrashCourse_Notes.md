# Hibernate Masterclass: Comprehensive Revision Notes

## 1. Introduction: Why Hibernate?
* **ORM (Object Relational Mapping):** Java is an object-oriented language where data is stored in Objects. Relational databases (like PostgreSQL, MySQL) store data in tabular formats (rows and columns). ORM is the technique that maps Java objects directly to database tables.
* **Problems with JDBC:** In raw JDBC, developers must manually write SQL queries (like `INSERT INTO table VALUES(...)`), map `ResultSet` data back to Java objects, manage connections, and catch boilerplate exceptions.
* **The Hibernate Solution:** Hibernate is an ORM framework that abstracts away all SQL queries. You simply tell Hibernate to "save this object," and it automatically generates the SQL, sets the parameters, and executes the query.
* **JPA (Jakarta Persistence API):** JPA is a standard specification for ORM in Java. Hibernate is one of the most popular implementations of JPA.

---

## 2. Project Setup & Dependencies
To use Hibernate in a Maven project, add two main dependencies in your `pom.xml`:
1. The JDBC Driver for your database (e.g., PostgreSQL).
2. The Hibernate ORM Core library.

```xml
<dependencies>
    <!-- PostgreSQL JDBC Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.3</version> <!-- Example version -->
    </dependency>

    <!-- Hibernate Core -->
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.6.3.Final</version> <!-- Example version -->
    </dependency>
</dependencies>
```

---

## 3. Configuration (`hibernate.cfg.xml`)
By default, Hibernate looks for a configuration file named `hibernate.cfg.xml` in the `src/main/resources` folder.

```xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database Connection Settings -->
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/telusko</property>
        <property name="hibernate.connection.username">postgres</property>
        <property name="hibernate.connection.password">0000</property>
        
        <!-- SQL Dialect: Translates HQL to Database-specific SQL -->
        <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQLDialect</property>
        
        <!-- Console Output Settings -->
        <property name="hibernate.show_sql">true</property> <!-- Prints SQL queries in console -->
        <property name="hibernate.format_sql">true</property> <!-- Formats the printed SQL beautifully -->
        
        <!-- Schema Auto-Generation (hbm2ddl.auto) -->
        <!-- "none" (default), "create" (drops and recreates every time), "create-drop", "update" (alters schema without dropping) -->
        <property name="hibernate.hbm2ddl.auto">update</property>
    </session-factory>
</hibernate-configuration>
```

---

## 4. Entity Mapping Annotations
Hibernate maps Java classes to database tables using JPA annotations (`jakarta.persistence.*`).

### Basic Annotations
* `@Entity`: Marks the class as a database entity.
* `@Id`: Marks a field as the Primary Key.
* `@Table(name = "alien_table")`: Changes the mapped table name (default is the class name).
* `@Entity(name = "alien_table")`: Changes the entity name itself (which affects HQL queries and table names).
* `@Column(name = "alien_name")`: Changes the mapped column name (default is the field name).
* `@Transient`: Tells Hibernate to ignore this field (it will NOT be stored as a column in the database).

```java
package com.telusko;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import jakarta.persistence.Transient;

@Entity
@Table(name = "alien_table")
public class Alien {
    
    @Id
    private int aid;
    
    @Column(name = "alien_name")
    private String aname;
    
    private String tech; // Will be stored as "tech" column
    
    @Transient
    private int tempAge; // Will NOT be stored in the DB
    
    // 1. A Default Constructor is MANDATORY for Hibernate to instantiate the object
    public Alien() {}

    // 2. Parameterized Constructor for our own convenience
    public Alien(int aid, String aname, String tech, int tempAge) {
        this.aid = aid;
        this.aname = aname;
        this.tech = tech;
        this.tempAge = tempAge;
    }

    // 3. Getters and Setters are MANDATORY for Hibernate to map fields properly
    public int getAid() { return aid; }
    public void setAid(int aid) { this.aid = aid; }

    public String getAname() { return aname; }
    public void setAname(String aname) { this.aname = aname; }

    public String getTech() { return tech; }
    public void setTech(String tech) { this.tech = tech; }

    public int getTempAge() { return tempAge; }
    public void setTempAge(int tempAge) { this.tempAge = tempAge; }

    // 4. toString() method to easily print the object
    @Override
    public String toString() {
        return "Alien [aid=" + aid + ", aname=" + aname + ", tech=" + tech + "]";
    }
}
```

---

## 5. Core Execution Architecture
To execute database operations, Hibernate requires a specific setup sequence:
`Configuration` -> `SessionFactory` -> `Session` -> `Transaction`

### Creating the Setup (Boilerplate `App.java`)
```java
package com.telusko;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class App {
    public static void main(String[] args) {
        // 1. Load the configuration and add the entity class
        Configuration cfg = new Configuration();
        cfg.addAnnotatedClass(Alien.class); // Explicitly map the class
        cfg.configure(); // Loads hibernate.cfg.xml

        // 2. Build the SessionFactory (Heavyweight object, create only once per application)
        SessionFactory sf = cfg.buildSessionFactory();

        // 3. Open a Session (Lightweight, create for each unit of work)
        Session session = sf.openSession();

        // 4. Begin a Transaction (Required for any INSERT/UPDATE/DELETE operations)
        Transaction tx = session.beginTransaction();

        // Perform Operations here... (e.g., session.persist(alien))

        // 5. Commit and Close
        tx.commit();
        session.close();
        sf.close();
    }
}
```

---

## 6. CRUD Operations (Create, Read, Update, Delete)

### A. Create / Save (`persist`)
* **Old Way:** `session.save(obj)` (Deprecated in Hibernate v6+)
* **New Way:** `session.persist(obj)` (Standard JPA)
```java
Alien a1 = new Alien();
a1.setAid(101);
a1.setAname("Navin");
a1.setTech("Java");

session.persist(a1); // Stores the object into the DB. Requires a transaction commit!
```

### B. Update (`merge`)
* **Old Way:** `session.update(obj)` or `session.saveOrUpdate(obj)` (Deprecated)
* **New Way:** `session.merge(obj)`
```java
Alien a1 = new Alien();
a1.setAid(101); // Existing ID
a1.setAname("Navin");
a1.setTech("Spring"); // Updated data

session.merge(a1); // Updates the record. If the ID doesn't exist, it inserts it (like saveOrUpdate).
```

### C. Delete (`remove`)
* **Old Way:** `session.delete(obj)` (Deprecated)
* **New Way:** `session.remove(obj)`
```java
Alien a1 = session.get(Alien.class, 101); // Fetch it first
if (a1 != null) {
    session.remove(a1); // Deletes from DB
}
```

---

## 7. Reading Data: `get` vs `load` vs `getReference`

### Eager Fetch: `session.get()`
* Immediately fires a `SELECT` SQL query to the database when the method is called.
* Returns `null` if the object does not exist.
```java
Alien a1 = session.get(Alien.class, 101);
```

### Lazy Fetch: `session.load()` or `session.getReference()`
* **`session.load()` is deprecated.** Use **`session.getReference()`** instead.
* It does **NOT** fire the SQL query immediately. It creates a "proxy" (fake) object.
* The actual SQL `SELECT` query is only fired when you actively try to access the data (e.g., calling `a1.getAname()`).
* If you just fetch the reference but never use it, it saves database calls.
* Throws an `EntityNotFoundException` if the ID doesn't exist when you finally try to access it.
```java
Alien a1 = session.getReference(Alien.class, 101); // No SQL fired yet!
System.out.println(a1.getAname()); // SQL SELECT is fired right here!
```

---

## 8. Value Types: `@Embeddable`
What if an object has another object inside it, but you don't want a separate database table? You just want the inner object's fields to be columns in the parent's table.
* **Child Class:** Mark with `@Embeddable` (It has no `@Id` and no `@Entity`).
* **Parent Class:** Simply declare the reference.

**The Child Class (`Laptop.java`):**
```java
package com.telusko;

import jakarta.persistence.Embeddable;

@Embeddable
public class Laptop {
    private String brand;
    private String model;
    private int ram;
    
    public Laptop() {}

    // Getters and Setters
    public String getBrand() { return brand; }
    public void setBrand(String brand) { this.brand = brand; }
    public String getModel() { return model; }
    public void setModel(String model) { this.model = model; }
    public int getRam() { return ram; }
    public void setRam(int ram) { this.ram = ram; }
}
```

**The Parent Entity (`Alien.java`):**
```java
package com.telusko;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Alien {
    @Id
    private int aid;
    private String aname;
    
    // Laptop fields (brand, model, ram) will be added as columns inside the Alien table!
    private Laptop laptop; 
    
    public Alien() {}

    // Getters and Setters
    public int getAid() { return aid; }
    public void setAid(int aid) { this.aid = aid; }
    public String getAname() { return aname; }
    public void setAname(String aname) { this.aname = aname; }
    public Laptop getLaptop() { return laptop; }
    public void setLaptop(Laptop laptop) { this.laptop = laptop; }
}
```

---

## 9. Mapping Entity Relationships
When dealing with multiple Entities (separate tables), we map their relationships.
Let's assume `Alien` and `Laptop` are both `@Entity` classes with their own `@Id`.

### A. One-to-One
One Alien has One Laptop.

**The Entity (`Alien.java`):**
```java
package com.telusko;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.OneToOne;

@Entity
public class Alien {
    @Id 
    private int aid;
    private String aname;
    
    @OneToOne
    private Laptop laptop; // Creates a Foreign Key column 'laptop_lid' in Alien table

    public Alien() {}

    // Getters and Setters
    public int getAid() { return aid; }
    public void setAid(int aid) { this.aid = aid; }
    public String getAname() { return aname; }
    public void setAname(String aname) { this.aname = aname; }
    public Laptop getLaptop() { return laptop; }
    public void setLaptop(Laptop laptop) { this.laptop = laptop; }
}
```

### B. One-to-Many / Many-to-One
One Alien has Multiple Laptops.
By default, Hibernate creates a **third mapping table** to store the relationships. To stop this redundancy and just use a Foreign Key in the `Laptop` table, use `mappedBy`.

**The One Side (`Alien.java`):**
```java
package com.telusko;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.OneToMany;
import java.util.List;

@Entity
public class Alien {
    @Id 
    private int aid;
    private String aname;
    
    // mappedBy = "alien" tells Hibernate: "Look at the 'alien' variable in the Laptop class for the foreign key. Do not create a third table!"
    @OneToMany(mappedBy = "alien")
    private List<Laptop> laptops;

    public Alien() {}

    // Getters and Setters
    public int getAid() { return aid; }
    public void setAid(int aid) { this.aid = aid; }
    public String getAname() { return aname; }
    public void setAname(String aname) { this.aname = aname; }
    public List<Laptop> getLaptops() { return laptops; }
    public void setLaptops(List<Laptop> laptops) { this.laptops = laptops; }
}
```

**The Many Side (`Laptop.java`):**
```java
package com.telusko;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.ManyToOne;

@Entity
public class Laptop {
    @Id 
    private int lid;
    private String brand;
    
    @ManyToOne
    private Alien alien; // Creates an 'alien_aid' Foreign Key column in the Laptop table

    public Laptop() {}

    // Getters and Setters
    public int getLid() { return lid; }
    public void setLid(int lid) { this.lid = lid; }
    public String getBrand() { return brand; }
    public void setBrand(String brand) { this.brand = brand; }
    public Alien getAlien() { return alien; }
    public void setAlien(Alien alien) { this.alien = alien; }
}
```

### C. Many-to-Many
Many Aliens share Many Laptops (e.g., a lab).
This **REQUIRES** a third mapping table. If you don't use `mappedBy`, Hibernate creates TWO mapping tables!

**In `Alien.java`:**
```java
package com.telusko;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.ManyToMany;
import java.util.List;

@Entity
public class Alien {
    @Id 
    private int aid;
    
    @ManyToMany(mappedBy = "aliens") // Let Laptop handle the mapping table
    private List<Laptop> laptops;

    public Alien() {}

    // Getters and Setters...
}
```

**In `Laptop.java`:**
```java
package com.telusko;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.ManyToMany;
import java.util.List;

@Entity
public class Laptop {
    @Id 
    private int lid;
    
    @ManyToMany
    private List<Alien> aliens; // This side generates the third table: 'laptop_alien'

    public Laptop() {}

    // Getters and Setters...
}
```

---

## 10. Eager vs Lazy Fetching in Relationships
When you fetch an `Alien` who has a `List<Laptop>`, does Hibernate fetch the laptops immediately?
* **By Default (Lazy):** Collections (`@OneToMany`, `@ManyToMany`) are fetched lazily. If you run `session.get(Alien.class, 101)`, it only fires a query for the Alien. The Laptops query is only fired if you do `alien.getLaptops().size()`.
* **Eager Fetch:** If you want the laptops immediately in one go (via an SQL JOIN), change it:
```java
@OneToMany(mappedBy = "alien", fetch = FetchType.EAGER)
private List<Laptop> laptops;
```

---

## 11. HQL (Hibernate Query Language)
HQL is an object-oriented version of SQL. 
**Crucial Rule:** In HQL, you write queries using **Class Names and Property Names**, NOT Database Table Names and Column Names.

```java
// SQL: SELECT * FROM alien_table WHERE alien_name = 'Navin'
// HQL:
String hql = "FROM Alien WHERE aname = :nameParam"; 

// Create Query
Query<Alien> query = session.createQuery(hql, Alien.class);
query.setParameter("nameParam", "Navin"); // Safe parameter binding

List<Alien> aliens = query.getResultList(); // Execute
```

### Fetching specific columns
If you don't select the whole object, HQL returns an `Object[]` array per row.
```java
// Fetch just the name and tech
Query<Object[]> q = session.createQuery("SELECT aname, tech FROM Alien", Object[].class);
List<Object[]> data = q.getResultList();

for (Object[] row : data) {
    System.out.println(row[0] + " knows " + row[1]);
}
```

---

## 12. Caching in Hibernate
Caching prevents unnecessary round-trips to the database.

### A. Level 1 Cache (First-Level)
* **Default:** Enabled by default.
* **Scope:** Bound to a single `Session`.
* **Behavior:** If you run `session.get(Alien.class, 101)` twice in the *same* session, Hibernate only fires *one* SQL query. The second fetch is grabbed from the L1 cache.

### B. Level 2 Cache (Second-Level)
* **Default:** Disabled. Requires 3rd party providers (like EHCache).
* **Scope:** Bound to the `SessionFactory` (Shared across ALL sessions).
* **Setup:**
  1. Add dependencies (`hibernate-jcache` + a provider like `ehcache`).
  2. In `hibernate.cfg.xml`, enable it:
     ```xml
     <property name="hibernate.cache.use_second_level_cache">true</property>
     <property name="hibernate.cache.region.factory_class">org.hibernate.cache.jcache.JCacheRegionFactory</property>
     ```
  3. Annotate your Entity class:
     ```java
     package com.telusko;
     
     import jakarta.persistence.Cacheable;
     import jakarta.persistence.Entity;
     
     @Entity
     @Cacheable // Enables caching for this entity
     public class Alien { ... }
     ```
* **Behavior:** If `Session1` fetches Alien 101, and then `Session2` tries to fetch Alien 101, it pulls from the L2 cache instead of hitting the DB again!

---

## 13. Complete Code Example

For a complete reference, here is how a simple standalone Hibernate application is structured.

### 1. The Entity Class (`Alien.java`)
```java
package com.telusko;

import jakarta.persistence.*;

@Entity
@Table(name = "alien_table")
public class Alien {
    
    @Id
    private int aid;
    
    private String aname;
    private String tech;
    
    // Default Constructor (required by Hibernate)
    public Alien() {}

    public Alien(int aid, String aname, String tech) {
        this.aid = aid;
        this.aname = aname;
        this.tech = tech;
    }

    // Getters and Setters
    public int getAid() { return aid; }
    public void setAid(int aid) { this.aid = aid; }

    public String getAname() { return aname; }
    public void setAname(String aname) { this.aname = aname; }

    public String getTech() { return tech; }
    public void setTech(String tech) { this.tech = tech; }

    @Override
    public String toString() {
        return "Alien [aid=" + aid + ", aname=" + aname + ", tech=" + tech + "]";
    }
}
```

### 2. The Main Execution Class (`App.java`)
```java
package com.telusko;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class App {
    public static void main(String[] args) {
        
        // 1. Create Data Object
        Alien telusko = new Alien(101, "Navin", "Java");

        // 2. Configure Hibernate
        Configuration cfg = new Configuration()
                .configure("hibernate.cfg.xml") // Loads configuration file
                .addAnnotatedClass(Alien.class); // Explicitly specify the entity

        // 3. Build Session Factory & Open Session
        // Using try-with-resources to automatically close session and factory
        try (SessionFactory sf = cfg.buildSessionFactory();
             Session session = sf.openSession()) {
             
            // 4. Begin Transaction
            Transaction tx = session.beginTransaction();

            // 5. Save the Object
            session.persist(telusko);
            
            // Example of fetching instead:
            // Alien fetchedAlien = session.get(Alien.class, 101);
            // System.out.println(fetchedAlien);

            // 6. Commit the Transaction
            tx.commit();
            System.out.println("Data saved successfully!");
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
