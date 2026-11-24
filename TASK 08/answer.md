# TASK B3 – STATEMENT VS PREPAREDSTATEMENT VS BATCH

## B3.1 Aim

To transfer bus-pass records into a MySQL database using JDBC and compare Statement, PreparedStatement, and batch processing.

## B3.2 Create Database

Execute the following SQL commands in MySQL Workbench.

```sql
CREATE DATABASE transport_db;

USE transport_db;

CREATE TABLE bus_pass (
    pass_id VARCHAR(20) PRIMARY KEY,
    reg_no VARCHAR(30) NOT NULL,
    name VARCHAR(100) NOT NULL,
    route_no INT NOT NULL,
    amount DOUBLE NOT NULL
);
```

## B3.3 Insert 10 Records

Perform the following experiments:

1. Insert 10 records using Statement.
2. Insert the same 10 records using PreparedStatement.
3. Insert 10 records using PreparedStatement with batching.
4. Clear the table before repeating the experiment to avoid duplicate primary keys.

Verify the records using:

```sql
SELECT * FROM bus_pass;

DESCRIBE bus_pass;
```

## B3.4 Increase the Data Size

The program must insert:

- 1,000 records
- 10,000 records
- 100,000 records

### Three Approaches

**1. Statement**

- Builds SQL commands using string concatenation.
- Executes each insert separately.

**2. PreparedStatement**

- Uses parameterized SQL.
- Executes each insert separately.
- Allows the SQL statement to be reused.

**3. PreparedStatement with Batch Processing**

- Uses `addBatch()` to collect multiple insert operations.
- Uses `executeBatch()` to execute them together.
- Reduces database communication round trips.

### Expected Performance Interpretation

- Statement generally has more SQL parsing overhead.
- PreparedStatement can reuse the prepared SQL structure.
- Batch processing generally improves performance for large datasets.
- Actual timings depend on hardware, MySQL configuration, network, and JDBC driver.

## B3.5 Auto-commit vs Transaction

### Auto-commit ON

- Each successful insert is committed separately.
- If a failure occurs halfway, earlier inserted rows remain in the database.
- Later rows are not inserted.

### Single Transaction

- All insert operations are performed within one transaction.
- If all operations succeed, `commit()` saves them.
- If any operation fails, `rollback()` reverses the transaction.
- No partial set of inserted records remains.

## B3.6 Engineering Trade-Off

**For a nightly import of 100,000 passes:**

- Use PreparedStatement with batch processing.
- Divide the records into batches of 1,000.
- Use a transaction to ensure correctness.
- Validate the input before insertion.

**For a single pass issued at the counter:**

- Use PreparedStatement.
- Insert one record at a time.
- Commit the transaction after successful insertion.

## B3.7 Test Transaction Failure

To test rollback:

1. Create a list of valid records.
2. Insert an invalid amount such as `"INVALID"` halfway through the list.
3. The `Double.parseDouble()` method throws `NumberFormatException`.
4. The catch block executes `rollback()`.
5. The transaction is cancelled.

**Expected output:**

```text
Insertion Failed: For input string: "INVALID"
```

Verify the database:

```sql
SELECT COUNT(*) FROM bus_pass;
```

If the table was empty before the test and all inserts belonged to the failed transaction:

```text
COUNT(*)
0
```

## B3.8 Alternative Concept: Multi-row INSERT and LOAD DATA

### Multi-row INSERT

- Multiple records are included in one SQL INSERT command.
- It reduces database communication round trips.
- The SQL statement becomes larger as the number of records increases.

### LOAD DATA

- MySQL provides LOAD DATA for importing records from files.
- It is useful for large-scale data imports.
- It can load structured data efficiently.

## B3.9 Complexity Analysis

| Technique | Time Complexity | Memory Complexity |
|---|---|---|
| PreparedStatement batch | O(n), excluding database-specific overhead | O(b) |
| Multi-row INSERT | O(n), plus SQL parsing and database overhead | Approximately O(n) for one complete statement |
| LOAD DATA | O(n) for reading and loading records | Depends on implementation and buffering |

Here, `n` is the number of records and `b` is the batch size.

**Conclusion:** PreparedStatement batching provides a good balance between performance, memory usage, and transaction safety. LOAD DATA can be considered for very large MySQL imports.

---

# JAVA PROGRAMS

## CODE 1: MetadataCheck.java

```java
import java.sql.*;

public class MetadataCheck {
    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/transport_db";

        try (Connection con =
                 DriverManager.getConnection(url, "root", "your_password");
             Statement st = con.createStatement();
             ResultSet rs = st.executeQuery("SELECT * FROM bus_pass")) {

            ResultSetMetaData md = rs.getMetaData();

            int count = md.getColumnCount();

            for (int i = 1; i <= count; i++) {

                System.out.println("Column: " +
                                   md.getColumnName(i));

                System.out.println("Type: " +
                                   md.getColumnTypeName(i));

                System.out.println("----------------");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**Purpose:** This program displays the column names and data types of the `bus_pass` table using ResultSetMetaData.

## CODE 2: PassLoader.java

```java
import java.sql.*;
import java.util.List;

public class PassLoader {

    public static int loadAll(Connection con, List<String[]> passes)
            throws SQLException {

        String sql = "INSERT INTO bus_pass " +
                     "(pass_id, reg_no, name, route_no, amount) " +
                     "VALUES (?, ?, ?, ?, ?)";

        boolean oldAutoCommit = con.getAutoCommit();

        con.setAutoCommit(false);

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            int n = 0;

            for (String[] p : passes) {

                ps.setString(1, p[0]);
                ps.setString(2, p[1]);
                ps.setString(3, p[2]);
                ps.setInt(4, Integer.parseInt(p[3]));
                ps.setDouble(5, Double.parseDouble(p[4]));

                ps.addBatch();

                n++;

                if (n % 1000 == 0) {
                    ps.executeBatch();
                    ps.clearBatch();
                }
            }

            ps.executeBatch();
            ps.clearBatch();

            con.commit();

            return n;

        } catch (SQLException | RuntimeException e) {

            try {
                con.rollback();
            } catch (SQLException rollbackError) {
                e.addSuppressed(rollbackError);
            }

            throw e;

        } finally {
            con.setAutoCommit(oldAutoCommit);
        }
    }
}
```

**Purpose:** This program inserts bus-pass records in batches of 1,000 using PreparedStatement and a transaction. If an error occurs, rollback is performed.

## CODE 3: PassLoaderTest.java

```java
import java.sql.*;
import java.util.*;

public class PassLoaderTest {
    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/transport_db";

        try (Connection con =
                 DriverManager.getConnection(url, "root", "your_password")) {

            List<String[]> passes = new ArrayList<>();

            for (int i = 1; i <= 10000; i++) {

                passes.add(new String[]{
                    "P" + i,
                    "REG" + i,
                    "Student" + i,
                    "12",
                    "4500.0"
                });
            }

            int count = PassLoader.loadAll(con, passes);

            System.out.println("Total Records Loaded: " + count);

            System.out.println("Batch Insertion Completed");

        } catch (Exception e) {
            System.out.println("Insertion Failed: " + e.getMessage());
        }
    }
}
```

**Expected output:**

```text
Total Records Loaded: 10000
Batch Insertion Completed
```

The output assumes that the table is empty and the database connection is successful.

## B3.10 Conclusion

PreparedStatement is safer and more efficient than Statement because it supports parameterized queries and statement reuse. Batch processing reduces database communication round trips and improves performance for large datasets. Transactions ensure that all records are inserted successfully or rolled back if an error occurs.
