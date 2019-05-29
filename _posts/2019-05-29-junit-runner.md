---
title: "JUnit runner"
date: 2019-05-29
tags: java
---

## Class to test

```java
import org.junit.Test;
import static org.junit.Assert.assertEquals;

public class MyClassToTest {
    @Test
    public void testDemo() {
        assertEquals(1, 1);
    }
}
```

## Test runner

```java
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class TestRunner {
    public static final String RED = "\033[31;1m";
    public static final String GREEN = "\033[32;1m";
    public static final String RESET = "\033[0m";

    public static void main(String[] args) {
        Result result = JUnitCore.runClasses(MyClassToTest.class);
        System.out.print("Ran " + result.getRunCount() + ", ");
        System.out.print("ignored " + result.getIgnoreCount() + ", ");
        System.out.print("in " + result.getRunTime() + " ms.");
        int fails = result.getFailureCount();
        String color = (fails == 0 ? GREEN : RED);
        System.out.println(" " + color + result.getFailureCount() + " failed." + 
               RESET);
        if (result.wasSuccessful()) {
            System.out.println(GREEN + "✓ All tests passed." + RESET);
        } else {
            for (Failure failure : result.getFailures()) {
                System.out.println(RED + "𝘹 " + failure.toString() + "\033[0m");
            }
        }

    }
}
```

## Makefile

```make
test:
    @javac -cp .:/lib/junit-4.12.jar MyClassToTest.java TestRunner.java
    @java -cp .:./lib/junit-4.12.jar:./lib/hamcrest-core-1.3.jar \
        TestRunner

testwatch:
    @fswatch MyClassToTest.java | xargs -I % bash -c 'clear; make test'
```

## Output

Success:

```bash
$ make test
Ran 1, ignored 0, in 9 ms. 0 failed.
✓ All tests passed.
```

Failure:

```bash
$ make test
Ran 1, ignored 0, in 88 ms. 1 failed.
𝘹 testDemo(MyClassToTest): expected:<1> but was:<0>
```
