# TASK A2 – DON'T READ WHAT YOU DON'T NEED

**Bloom's Level:** K4 – Analyse

## A2.1 Predict the Cost

The program reads a 50 MB attendance log using `FileInputStream.read()` one byte at a time.

### Calculation

Given:

- File size = 50 MB
- 1 MB = 1,048,576 bytes

Total bytes:

50 × 1,048,576 = 52,428,800 bytes

Therefore:

- The program makes approximately 52,428,800 read() calls.
- Each call reads only one byte.
- Repeated method calls create additional overhead.
- The program becomes slow because it does not read multiple bytes in one operation.

## A2.2 Redesign Using Buffering

Use `BufferedInputStream` to read multiple bytes efficiently.

### How Buffering Improves Performance

- BufferedInputStream maintains an internal buffer.
- Instead of requesting one byte from the operating system each time, it reads a larger block of data.
- Subsequent reads are served from the buffer.
- This reduces expensive underlying I/O operations.
- The time complexity remains O(n), but the practical execution time is reduced.

## A2.3 When Buffering Does Not Help

- Buffering may provide little benefit when the program reads data only once and the data is already available in memory.
- It may also provide little improvement when processing a very small file.
- Additional buffering is unnecessary if the existing API already performs efficient buffered operations.

### Alternative Concept: Files.readAllLines()

**Complexity Analysis:**

**Buffered Stream:**

- Time complexity: O(n).
- Space complexity: O(b), where b is the buffer size.
- Suitable for large files because only a small buffer is maintained.

**Files.readAllLines():**

- Time complexity: O(n).
- Space complexity: O(n), because the entire file and its lines are stored in memory.
- Suitable for small or moderate-sized files that comfortably fit in memory.

### Conclusion

For a 50 MB attendance file, buffered reading is generally preferable because it reduces memory usage while maintaining efficient sequential reading.

## JAVA PROGRAM 1 – BufferedCounter.java

```java
import java.io.*;

public class BufferedCounter {

    public static void main(String[] args) {

        String file = "attendance.txt";

        int lines = 0;

        try (BufferedInputStream bis =
                new BufferedInputStream(
                    new FileInputStream(file))) {

            int data;

            while ((data = bis.read()) != -1) {

                if (data == '\n') {
                    lines++;
                }
            }

            System.out.println("Total Lines: " + lines);

        } catch (IOException e) {

            System.out.println(e.getMessage());

        }
    }
}
```

### Expected Output

```text
Total Lines: [Number of lines in attendance.txt]
```

## JAVA PROGRAM 2 – ReadAllLinesExample.java

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.List;

public class ReadAllLinesExample {

    public static void main(String[] args) {

        try {

            List<String> lines =
                Files.readAllLines(
                    Paths.get("attendance.txt"));

            System.out.println(
                "Total Lines: " + lines.size());

        } catch (IOException e) {

            System.out.println(e.getMessage());

        }
    }
}
```

### Expected Output

```text
Total Lines: [Number of lines in attendance.txt]
```

**Note:** Both programs require an `attendance.txt` file in the working directory. The actual line count depends on the contents of that file.
