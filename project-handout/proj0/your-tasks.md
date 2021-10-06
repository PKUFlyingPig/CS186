# Your Tasks

For this assignment you will get acquainted with running RookieDB's command line interface and make a small change to one file to get things working properly.

## Task 1: Running the CLI

Most databases provide a command line interface \(CLI\) to send and view the results of queries. To run the CLI in IntelliJ navigate to the file:

`src/main/java/edu/berkeley/cs186/database/cli/CommandLineInterface`

It's okay if you don't understand most of the code here right now, we just want to run it. Locate the arrow next to the class declaration click on it to start the CLI.

![Click the arrow \(circled in red above\) to run the CLI](../../.gitbook/assets/image.png)

This should open a new panel in IntelliJ resembling the following image:

![](../../.gitbook/assets/image%20%2810%29%20%281%29.png)

Click on this panel and try typing in the following query and hitting enter:

`SELECT * FROM Courses LIMIT 5;`

You should get something similar to the following output:

![](../../.gitbook/assets/image%20%283%29.png)

Hmm, that doesn't look quite right! Follow the instructions in the next task to get the proper output. To exit the CLI just type in `exit` and hit enter.

## Task 2: Welcome to CS186!

Open up `src/main/java/edu/berkeley/cs186/database/databox/StringDataBox.java`. It's okay if you do not understand most of the code right now.

The `toString` method currently looks like:

```java
    @Override
    public String toString() {
        // TODO(proj0): replace the following line with `return s;`
        return "FIX ME";
    }
```

Follow the instructions in the `TODO(proj0)` comment to fix the return statement.

Navigate to`src/test/java/edu/berkeley/cs186/database/databox/TestWelcome.java` and try running the test in the file, which should now be passing. Now you can run through Task 1 again to see what the proper output should be.

## You're done!

Follow the instructions in the next section "Submitting the Assignment" to turn in your work.

