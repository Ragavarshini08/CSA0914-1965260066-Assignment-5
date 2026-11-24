# TASK B4 – FINAL ENGINEERING DECISION

## B4.1 Aim

To develop a practical migration strategy for transferring bus-pass records from text files into MySQL and select a suitable storage system for multiple campuses.

## B4.2 Implement a Practical Strategy

The transport office now serves 12 campuses and has 100 times the original number of bus passes.

### Migration Steps

1. Read the existing bus-pass file using BufferedReader.
2. Read each record line by line.
3. Validate the number of fields.
4. Validate numeric values.
5. Store valid records in a collection.
6. Insert records into MySQL using PreparedStatement batches.
7. Use a single transaction for the migration.
8. Record invalid records in a separate error report.
9. Commit if successful.
10. Roll back if database insertion fails.

## B4.3 Why is DatabaseMetaData Useful?

- It provides information about the database product.
- It identifies the JDBC driver.
- It displays the database version.
- It lists available database tables.
- It helps identify compatibility issues when moving the application to another server.
- It makes database applications easier to maintain across different environments.

## B4.4 Production Decision

### 1. Text Files

**Advantages:**
- Simple to create and maintain.
- Human-readable.
- Useful for small amounts of data.

**Disadvantages:**
- Searching requires scanning records.
- Concurrent updates can cause data consistency problems.
- Transaction management is difficult.
- Not ideal for multiple counters accessing records simultaneously.

### 2. Serialization

**Advantages:**
- Saves Java objects directly.
- Supports complex object relationships.
- Convenient for Java-specific applications.

**Disadvantages:**
- Not easily accessible from other programming languages.
- Class-version compatibility must be managed.
- Not suitable for multiple users updating shared records simultaneously.
- Requires additional synchronization and file-management mechanisms.

### 3. Relational Database

**Advantages:**
- Supports multiple users and concurrent transactions.
- Provides data consistency.
- Supports indexing for faster searching.
- Supports PreparedStatement and batch processing.
- Provides transaction management.
- Supports centralized storage for all campuses.

### Final Decision: MySQL Database

**Reasons for Selection:**

- MySQL can centrally store records from all 12 campuses.
- Multiple counters can access the database concurrently.
- Transactions protect data during updates.
- Indexes improve pass-ID searching.
- Batch processing supports large imports.
- Database security and access control provide better protection.

## B4.5 Alternative Concept: Separate Serialized File for Each Campus

If each campus stores its own serialized file:

- Each campus maintains an independent file.
- Searching one campus requires reading its corresponding file.
- Searching all campuses requires searching all 12 files.
- Concurrent access must be controlled using synchronization or file locking.
- Updating data across campuses becomes difficult.
- Data duplication and inconsistent records may occur.

### Complexity Analysis

Assume each campus has `n` records.

| Operation | Complexity |
|---|---|
| Searching one campus | O(n) |
| Searching all 12 campuses sequentially | O(12n) = O(n) |
| Total storage | O(12n) = O(n) |

The number of campuses is treated as a constant.

### Database Approach

- Searching by an indexed pass ID is approximately O(log n) using a typical B-tree index.
- Multiple counters can access records through database transactions.
- The database provides centralized consistency and concurrency control.

## B4.6 Conclusion

A centralized relational database is the best option for the transport office because it provides efficient searching, transaction safety, concurrent access, and scalability across all 12 campuses.

---

# JAVA PROGRAMS

## CODE 1: PassMigration.java

```java
import java.io.*;
import java.sql.*;
import java.util.*;

public class PassMigration {
    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/transport_db";
        String inputFile = "buspasses.txt";

        List<String[]> valid = new ArrayList<>();
        int failed = 0;

        // Read and validate file
        try (BufferedReader br =
                 new BufferedReader(new FileReader(inputFile));
             BufferedWriter error =
                 new BufferedWriter(new FileWriter("failed_records.txt"))) {

            String line;

            while ((line = br.readLine()) != null) {

                if (line.trim().isEmpty()) {
                    failed++;
                    error.write("Empty record: " + line);
                    error.newLine();
                    continue;
                }

                String[] p = line.split(",", -1);

                try {
                    if (p.length != 5) {
                        throw new IllegalArgumentException(
                            "Invalid field count");
                    }

                    Integer.parseInt(p[3]);
                    Double.parseDouble(p[4]);

                    valid.add(p);

                } catch (Exception e) {
                    failed++;
                    error.write(line + " : " + e.getMessage());
                    error.newLine();
                }
            }

        } catch (IOException e) {
            System.out.println(e.getMessage());
            return;
        }

        // Load valid records into database
        try (Connection con =
                 DriverManager.getConnection(
                     url, "root", "your_password")) {

            int loaded = PassLoader.loadAll(con, valid);

            System.out.println("Successful Records: " + loaded);
            System.out.println("Failed Records: " + failed);
            System.out.println("Migration Completed");

        } catch (Exception e) {
            System.out.println("Migration Failed: " + e.getMessage());
        }
    }
}
```

**Purpose:** This program reads bus-pass records from a text file, validates them, stores invalid records in a separate error file, and inserts valid records into MySQL using the `PassLoader` class from Task B3.

## CODE 2: DatabaseInfo.java

```java
import java.sql.*;

public class DatabaseInfo {
    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/transport_db";

        try (Connection con =
                 DriverManager.getConnection(
                     url, "root", "your_password")) {

            DatabaseMetaData md = con.getMetaData();

            System.out.println("Database Product: " +
                               md.getDatabaseProductName());

            System.out.println("Database Version: " +
                               md.getDatabaseProductVersion());

            System.out.println("Driver Name: " +
                               md.getDriverName());

            System.out.println("Driver Version: " +
                               md.getDriverVersion());

            ResultSet rs = md.getTables(
                null,
                "transport_db",
                "%",
                new String[]{"TABLE"}
            );

            System.out.println("Tables:");

            while (rs.next()) {
                System.out.println(rs.getString("TABLE_NAME"));
            }

            rs.close();

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**Purpose:** This program displays database product details, database version, JDBC driver information, and available tables using DatabaseMetaData.

## B4.7 Conclusion

MySQL is the most suitable storage solution for the university transport office. It supports centralized data management, multiple users, transactions, indexing, and efficient batch processing. It provides better scalability and reliability than text files or separate serialized files.
