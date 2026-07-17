# JDBC Crash Course: Detailed Notes

## 1. Introduction to JDBC
In software development, almost every application needs to interact with data. While variables (`int`, `String`) hold data in memory temporarily, they are lost once the application closes. For permanent storage, we use a **Database Management System (DBMS)**.

Relational databases (like PostgreSQL, MySQL, Oracle) store data in tabular formats. However, a database understands SQL (Structured Query Language), not Java. 

**JDBC (Java Database Connectivity)** is an API provided by the Java Development Kit (JDK) that acts as a bridge, allowing a Java application to connect to a relational database, execute SQL queries, and process the results.

### How JDBC Works:
- Java provides the JDBC API (interfaces and classes in the `java.sql` package).
- The DBMS provider (e.g., PostgreSQL, MySQL) provides the actual implementation of these interfaces via a **Driver** (a `.jar` file).
- To connect Java to PostgreSQL, you need the PostgreSQL JDBC Driver. If you switch to MySQL later, you just swap the driver and update a single connection string.

---

## 2. Setup and Prerequisites
Before writing Java code, you need:
1. **A Database Engine:** e.g., PostgreSQL.
2. **A Database Client:** e.g., pgAdmin (installed alongside PostgreSQL).
3. **A Database and Table:**
   - Open pgAdmin, create a database named `demo`.
   - Create a table (e.g., `student` with columns: `sid` [integer, Primary Key], `sname` [text], `marks` [integer]).
4. **The JDBC Driver:**
   - Download the driver (`.jar`) from the official website (e.g., `jdbc.postgresql.org`) or Maven Repository.
   - Add this JAR file to your project's External Libraries (in IntelliJ: `File -> Project Structure -> Libraries`).

---

## 3. The 7 Steps of JDBC
Whenever you interact with a database using JDBC, you generally follow these 7 steps (an easy analogy is making a phone call):

1. **Import the Package:** `import java.sql.*;`
2. **Load and Register the Driver:** (Analogy: Getting a working SIM card/network). *Note: Optional since JDBC 4.0 / Java 6, as the driver is automatically loaded if the JAR is present.*
3. **Create a Connection:** (Analogy: Dialing the number and establishing a call).
4. **Create a Statement:** (Analogy: Thinking about what to say).
5. **Execute the Statement:** (Analogy: Actually speaking).
6. **Process the Results:** (Analogy: Listening and processing the response).
7. **Close the Connection:** (Analogy: Hanging up the call to save resources).

---

## 4. Reading Data (SELECT Query)
Here is a basic example of connecting to the database and reading data:

```java
import java.sql.*;

public class DemoJdbc {
    public static void main(String[] args) throws Exception {
        
        // Database credentials & URL
        String url = "jdbc:postgresql://localhost:5432/demo"; // Protocol:DBMS://Host:Port/DBName
        String uname = "postgres";
        String pass = "0000";
        
        // Step 2: Load Driver (Optional)
        Class.forName("org.postgresql.Driver");
        
        // Step 3: Create Connection
        Connection con = DriverManager.getConnection(url, uname, pass);
        System.out.println("Connection Established!");
        
        // Step 4: Create Statement
        Statement st = con.createStatement();
        
        // Step 5: Execute Statement
        String query = "SELECT * FROM student";
        ResultSet rs = st.executeQuery(query); // executeQuery() is used for SELECT because it returns a ResultSet
        
        // Step 6: Process the Results
        // rs.next() moves the pointer to the next row. It returns true if a row exists.
        while (rs.next()) {
            // Fetching by column index (1-based) or column name
            int sid = rs.getInt(1);
            String name = rs.getString("sname"); 
            int marks = rs.getInt(3);
            
            System.out.println(sid + " - " + name + " - " + marks);
        }
        
        // Step 7: Close the Connection
        con.close();
    }
}
```

---

## 5. CRUD Operations (INSERT, UPDATE, DELETE)
To modify data, we use the `execute()` or `executeUpdate()` methods instead of `executeQuery()`.
- `executeQuery()`: Returns a `ResultSet` (used for `SELECT`).
- `executeUpdate()`: Returns an `int` representing the number of rows affected (used for `INSERT`, `UPDATE`, `DELETE`).

```java
// Example: INSERT Query using standard Statement
String insertQuery = "INSERT INTO student VALUES (5, 'Max', 48)";
st.executeUpdate(insertQuery);

// Example: UPDATE Query
String updateQuery = "UPDATE student SET sname='Johnny' WHERE sid=5";
st.executeUpdate(updateQuery);

// Example: DELETE Query
String deleteQuery = "DELETE FROM student WHERE sid=5";
st.executeUpdate(deleteQuery);
```

---

## 6. Statement vs PreparedStatement
If user input is involved, generating SQL queries dynamically via string concatenation using standard `Statement` is highly discouraged due to several reasons:

1. **Complex Syntax:** Handling nested single and double quotes is extremely error-prone (`"INSERT INTO student VALUES (" + sid + ", '" + name + "', " + marks + ")"`).
2. **SQL Injection:** Malicious users can input harmful SQL logic into your text fields to manipulate or destroy the database.
3. **Performance:** The database engine cannot easily cache queries that are constantly changing dynamically.

### The Solution: `PreparedStatement`
A `PreparedStatement` solves all the above problems. It uses placeholders (`?`) for dynamic values, pre-compiles the query for better performance, and escapes user input automatically to prevent SQL Injection.

```java
import java.sql.*;

public class PreparedStatementDemo {
    public static void main(String[] args) throws Exception {
        
        String url = "jdbc:postgresql://localhost:5432/demo";
        String uname = "postgres";
        String pass = "0000";
        Connection con = DriverManager.getConnection(url, uname, pass);
        
        // Dynamic variables (e.g., fetched from a UI form)
        int sid = 102;
        String sname = "Jasmine";
        int marks = 52;
        
        // Use ? as placeholders for dynamic data
        String query = "INSERT INTO student VALUES (?, ?, ?)";
        
        // Step 4: Create PreparedStatement (Query is passed here)
        PreparedStatement st = con.prepareStatement(query);
        
        // Set the values for the placeholders (1-based index)
        st.setInt(1, sid);
        st.setString(2, sname);
        st.setInt(3, marks);
        
        // Step 5: Execute
        st.executeUpdate(); // No query passed here, it's already pre-compiled
        
        con.close();
    }
}
```

**Key Takeaway:** Always use `PreparedStatement` when dealing with dynamic data or user inputs!
