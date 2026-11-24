# TASK B1 – IMPLEMENT AND INSTRUMENT THE FILE STORE

## B1.1 Validate the Implementation

The bus-pass file store is tested using three types of test cases.

### Test 1: Normal Case
- Add a valid bus-pass record.
- Search using an existing pass ID.
- Verify that all five fields are displayed correctly.

### Test 2: Pass ID Does Not Exist
- Search using an ID that is not present.
- The program should return `null`.
- This verifies that the search handles missing records correctly.

### Test 3: Damaged Line
- Insert a line containing fewer than five fields.
- Search the file.
- The program should skip the damaged line and continue searching.

## B1.2 Compare with Your Design

- The implementation follows the Part A design.
- BufferedWriter is used to add records.
- BufferedReader is used to read records line by line.
- Damaged lines are skipped instead of stopping the entire search.
- Try-with-resources automatically closes the file after execution.
- The implementation uses String.split() to separate fields.

## B1.3 Measure the Effect of Buffering

The program should be tested using files of different sizes.

**Test Sizes:**
- 1 MB
- 10 MB
- 50 MB

### Measurement Procedure
1. Create an attendance file containing the required amount of data.
2. Run the unbuffered version.
3. Record the execution time.
4. Run the buffered version.
5. Record the execution time.
6. Repeat the test and compare the results.

### Expected Interpretation
- Unbuffered reading performs many individual read operations.
- Buffered reading reduces the number of underlying I/O operations.
- Execution time is generally lower with buffering.
- Both approaches have O(n) time complexity.

**Note:** Do not enter assumed timings as measured results. Record the actual output from your computer.

## B1.4 Measure Growth

Search performance should be measured with:
- 1,000 records.
- 10,000 records.
- 100,000 records.

### Explanation
- Finding a pass requires scanning the file.
- As the number of records increases, the number of comparisons increases.
- Searching takes O(n) time.
- The memory complexity is O(1), excluding the current line and record fields.

### Why does the damaged line for P1002 not stop the search?

- The program reads each line separately.
- The damaged line contains only two fields.
- The program checks whether the number of fields is exactly five.
- Since the condition fails, continue skips that line.
- The next record is processed normally.
- Therefore, the damaged line does not terminate the search.

## Alternative Concept: HashMap

If all records are loaded into a HashMap at program startup:
- The pass ID becomes the key.
- The complete pass record becomes the value.
- Searching becomes faster because the program does not scan the entire file repeatedly.

### CODE 1: PassMapExample.java

```java
import java.util.*;

public class PassMapExample {

    public static void main(String[] args) {

        HashMap<String, String[]> passes = new HashMap<>();

        passes.put("P1001",
            new String[]{
                "P1001", "192311001",
                "Arun", "12", "4500.0"
            });

        passes.put("P1003",
            new String[]{
                "P1003", "192311003",
                "Divya", "7", "3800.0"
            });

        String[] result = passes.get("P1003");

        System.out.println(Arrays.toString(result));
    }
}
```

### Expected Output

```text
[P1003, 192311003, Divya, 7, 3800.0]
```

## Complexity Analysis

### File Scanning
- One search: O(n).
- q searches: O(q × n).
- Extra space: O(1), excluding the current record.

### HashMap
- Loading n records: O(n) expected.
- q searches: O(q) expected.
- Total: O(n + q).
- Extra space: O(n).

### Conclusion

A HashMap is suitable when many searches are performed. A file scan is more memory-efficient when only a small number of searches are required.

## CODE 2: PassStore.java

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.Arrays;

public class PassStore {

    private final Path file;

    PassStore(Path file) {
        this.file = file;
    }

    public void add(String passId, String regNo,
                    String name, int route, double amount)
                    throws IOException {

        try (BufferedWriter w =
                Files.newBufferedWriter(file,
                    StandardOpenOption.CREATE,
                    StandardOpenOption.APPEND)) {

            w.write(String.join(",",
                passId,
                regNo,
                name,
                String.valueOf(route),
                String.valueOf(amount)));

            w.newLine();
        }
    }

    public String[] find(String passId)
            throws IOException {

        try (BufferedReader br =
                Files.newBufferedReader(file)) {

            String line;

            while ((line = br.readLine()) != null) {

                String[] fields = line.split(",", -1);

                if (fields.length != 5) {
                    continue;
                }

                if (fields[0].equals(passId)) {
                    return fields;
                }
            }
        }

        return null;
    }

    public static void main(String[] args)
            throws IOException {

        Path f = Files.createTempFile("passes", ".csv");

        PassStore store = new PassStore(f);

        store.add("P1001", "192311001",
                  "Arun", 12, 4500);

        Files.writeString(f,
            "P1002,DAMAGED LINE\n",
            StandardOpenOption.APPEND);

        store.add("P1003", "192311003",
                  "Divya", 7, 3800);

        System.out.println(
            Arrays.toString(store.find("P1003")));

        System.out.println(
            Arrays.toString(store.find("P1002")));

        Files.delete(f);
    }
}
```

### Expected Output

```text
[P1003, 192311003, Divya, 7, 3800.0]
null
```

## Overall Conclusion

The bus-pass file store uses BufferedWriter and BufferedReader to manage records efficiently. It skips damaged lines and handles missing pass IDs correctly. Buffering reduces underlying I/O operations and improves file-reading performance. HashMap provides faster searching when many searches are required, while file scanning uses less additional memory.
