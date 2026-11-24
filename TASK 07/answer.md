# TASK B2 – ENGINEER AN ADVERSARIAL INPUT

## B2.1 Aim

To test unsafe SQL login, PreparedStatement login, and file validation using adversarial inputs.

## B2.2 SQL Injection Input

The adversarial input used is:

```text
ID: ' OR '1'='1' -- 
Password: wrongpassword
```

In an unsafe SQL query, directly concatenating this input can change the query logic and may allow an invalid login.

### CODE 1: UnsafeLogin.java

```java
import java.sql.*;

public class UnsafeLogin {
    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/campus_events";
        String user = "root";
        String password = "your_password";

        String id = "' OR '1'='1' -- ";
        String pwd = "wrongpassword";

        String sql = "SELECT * FROM users WHERE id='" +
                     id + "' AND pwd='" + pwd + "'";

        try (Connection con =
                 DriverManager.getConnection(url, user, password);
             Statement st = con.createStatement()) {

            ResultSet rs = st.executeQuery(sql);

            if (rs.next()) {
                System.out.println("Login Successful");
            } else {
                System.out.println("Invalid Login");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**Expected output when the injection succeeds:**

```text
Login Successful
```

## B2.3 PreparedStatement Login

PreparedStatement uses placeholders and treats the supplied values as data instead of SQL commands. Therefore, the injection input does not change the query structure.

### CODE 2: PreparedLogin.java

```java
import java.sql.*;

public class PreparedLogin {
    public static void main(String[] args) {

        String url = "jdbc:mysql://localhost:3306/campus_events";
        String user = "root";
        String password = "your_password";

        String sql = "SELECT * FROM users WHERE id=? AND pwd=?";

        try (Connection con =
                 DriverManager.getConnection(url, user, password);
             PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setString(1, "' OR '1'='1' -- ");
            ps.setString(2, "wrongpassword");

            ResultSet rs = ps.executeQuery();

            if (rs.next()) {
                System.out.println("Login Successful");
            } else {
                System.out.println("Invalid Login");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**Expected output when no matching account exists:**

```text
Invalid Login
```

## B2.4 Create Damaged Bus-Pass File

Create a file named `damaged.txt` with the following content:

```text
P1001,192311001,Arun,12,4500.0
P1002,DAMAGED LINE
P1003,192311003,Divya,7,ABC
P1004,192311004,Ravi,5,3200.0

```

### Explanation

- P1001 is a valid record.
- P1002 has an incorrect number of fields.
- P1003 contains a non-numeric amount.
- P1004 is a valid record.
- The final empty line should be ignored.

## B2.5 File Validation Program

### CODE 3: ValidatePasses.java

```java
import java.io.*;

public class ValidatePasses {
    public static void main(String[] args) {

        try (BufferedReader br =
                 new BufferedReader(new FileReader("damaged.txt"))) {

            String line;
            int lineNumber = 0;

            while ((line = br.readLine()) != null) {

                lineNumber++;

                if (line.trim().isEmpty()) {
                    System.out.println("Line " + lineNumber +
                                       ": Empty line skipped");
                    continue;
                }

                String[] fields = line.split(",", -1);

                if (fields.length != 5) {
                    System.out.println("Line " + lineNumber +
                                       ": Invalid field count");
                    continue;
                }

                try {
                    Integer.parseInt(fields[3]);
                    Double.parseDouble(fields[4]);

                    System.out.println("Line " + lineNumber +
                                       ": Valid record");

                } catch (NumberFormatException e) {
                    System.out.println("Line " + lineNumber +
                                       ": Invalid numeric value");
                }
            }

        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

### Expected Output

```text
Line 1: Valid record
Line 2: Invalid field count
Line 3: Invalid numeric value
Line 4: Valid record
Line 5: Empty line skipped
```

## B2.6 What Each Adversarial Input Reveals

1. **SQL Injection:** Directly concatenating user input can change SQL query logic.
2. **PreparedStatement:** Parameterized queries protect the SQL structure.
3. **Incorrect Field Count:** File validation is necessary to identify damaged records.
4. **Non-numeric Amount:** Numeric conversion must be handled safely.
5. **Empty Line:** Blank records should be ignored.
6. **Continue Processing:** The program should continue processing after encountering an invalid record.

## B2.7 Alternative Concept: Escaping or Regular Expression Validation

- Escaping or regular expressions can attempt to reject or modify dangerous input.
- Escaping rules may differ between databases and character encodings.
- Regular expressions cannot reliably replace parameterized queries.
- Different query structures may require different validation rules.

## B2.8 Complexity Analysis

| Technique | Time Complexity |
|---|---|
| Regular expression validation | O(m) |
| Escaping | O(m) |
| PreparedStatement parameter binding | Generally O(m), in addition to query execution |

Here, `m` represents the length of the input.

PreparedStatement is preferred because it separates SQL structure from user data.

## B2.9 Conclusion

PreparedStatement provides a safer and more reliable solution than manually escaping or validating SQL input. File validation also helps identify damaged records, ignore empty lines, and continue processing without stopping the entire program.
