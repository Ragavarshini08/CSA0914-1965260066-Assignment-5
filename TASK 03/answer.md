# TASK A3 – DISCOVER THE DATA-ACCESS DESIGN PATTERN

## A3.1 Identify the Common Pattern

The project contains three classes: `StudentDB`, `CourseDB` and `MarksDB`.

**Repeated operations:**
1. Establishing a database connection.
2. Creating SQL queries.
3. Executing SQL statements.
4. Reading records from `ResultSet`.
5. Converting database records into Java objects.
6. Closing database connections.

**Parts that differ:**
1. Table names.
2. Column names.
3. SQL queries.
4. Java object types.
5. Primary key values.
6. Mapping between database columns and object fields.

**Problem:**
- The same database operations are repeated in multiple classes.
- This increases code duplication.
- Maintaining and modifying the application becomes difficult.

## A3.2 Design a Generic Data-Access Interface

A generic DAO (Data Access Object) interface provides common database operations.

**Explanation:**
- `Dao<T,K>` defines common database operations.
- `T` represents the Java object type.
- `K` represents the primary key type.
- `StudentDao` implements the interface for Student objects.
- The same interface can be reused for Course and Marks entities.

### CODE 1: Dao.java

```java
import java.util.List;

public interface Dao<T, K> {

    void insert(T object);

    T findById(K id);

    List<T> findAll();

    void update(T object);

    void delete(K id);
}
```

## A3.3 Map Objects to Tables

**Student object fields:**
- `studentId` maps to `student_id`.
- `name` maps to `name`.
- `email` maps to `email`.

**SQL Table:**

```sql
CREATE TABLE student (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
```

**Primary Key:**
- `student_id` uniquely identifies each student.
- It prevents duplicate student IDs.
- It is used by `findById()`, `update()` and `delete()`.

### CODE 2: Student.java

```java
public class Student {

    private int studentId;
    private String name;
    private String email;

    public Student(int studentId, String name, String email) {
        this.studentId = studentId;
        this.name = name;
        this.email = email;
    }

    public int getStudentId() {
        return studentId;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }
}
```

### CODE 3: StudentDao.java

```java
import java.sql.*;
import java.util.*;

public class StudentDao implements Dao<Student, Integer> {

    private Connection con;

    public StudentDao(Connection con) {
        this.con = con;
    }

    @Override
    public void insert(Student s) {

        String sql = "INSERT INTO student VALUES (?, ?, ?)";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, s.getStudentId());
            ps.setString(2, s.getName());
            ps.setString(3, s.getEmail());

            ps.executeUpdate();

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Student findById(Integer id) {

        String sql = "SELECT * FROM student WHERE student_id=?";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, id);

            ResultSet rs = ps.executeQuery();

            if (rs.next()) {
                return new Student(
                    rs.getInt("student_id"),
                    rs.getString("name"),
                    rs.getString("email")
                );
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }

        return null;
    }

    @Override
    public List<Student> findAll() {

        List<Student> students = new ArrayList<>();

        String sql = "SELECT * FROM student";

        try (Statement st = con.createStatement();
             ResultSet rs = st.executeQuery(sql)) {

            while (rs.next()) {

                students.add(new Student(
                    rs.getInt("student_id"),
                    rs.getString("name"),
                    rs.getString("email")
                ));
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }

        return students;
    }

    @Override
    public void update(Student s) {

        String sql = "UPDATE student SET name=?, email=? WHERE student_id=?";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setString(1, s.getName());
            ps.setString(2, s.getEmail());
            ps.setInt(3, s.getStudentId());

            ps.executeUpdate();

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void delete(Integer id) {

        String sql = "DELETE FROM student WHERE student_id=?";

        try (PreparedStatement ps = con.prepareStatement(sql)) {

            ps.setInt(1, id);
            ps.executeUpdate();

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## Alternative Concept: SQL Directly Inside Service or GUI Classes

If SQL queries are directly written inside service or GUI classes:
- Every class must contain its own database queries.
- When a database column is renamed, all affected SQL queries must be modified.
- The same SQL logic may be repeated in several places.

**Comparison:**

**Generic DAO:**
- SQL operations are centralized.
- Fewer places need modification.
- Easier maintenance and testing.

**Direct SQL:**
- SQL is scattered throughout the application.
- More places must be changed.
- Greater possibility of inconsistent updates.

## Complexity Analysis

- Both approaches generally have the same database operation time complexity.
- Searching by a primary key with an index is approximately O(log n) for a typical B-tree index.
- Without an index, a search may require O(n).
- Space complexity per operation is generally O(1) for a single-record operation, excluding returned collections.
- DAO improves maintainability, not necessarily execution speed.

## Conclusion

The Generic DAO design pattern reduces code duplication by centralizing database operations. It improves code reusability, maintenance and testing. The same interface can be reused for different entities such as Student, Course and Marks.
