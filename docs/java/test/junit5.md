# Junit5 教程

[原始链接](https://www.baeldung.com/junit-5)

## 依赖

`pom.xml` :

```xml

<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>

```

junit5 仅支持 java 8 及以上的版本

## Architecture 架构

JUnit 5 comprises several different modules from three different sub-projects.

### JUnit Platform

The platform is responsible for launching testing frameworks on the JVM. It defines a stable and powerful interface between JUnit and its clients, such as build tools.

The platform easily integrates clients with JUnit to discover and execute tests.

It also defines the [TestEngine](https://junit.org/junit5/docs/5.0.1/api/org/junit/platform/engine/TestEngine.html) API for developing a testing framework that runs on the JUnit platform. By implementing a custom TestEngine, we can plug 3rd party testing libraries directly into JUnit.

### JUnit Jupiter

This module includes new programming and extension models for writing tests in JUnit 5. New annotations in comparison to JUnit 4 are:

- `@TestFactory` – denotes a method that's a test factory for dynamic tests
- `@DisplayName` – defines a custom display name for a test class or a test method
- `@Nested` – denotes that the annotated class is a nested, non-static test class
- `@Tag` – declares tags for filtering tests
- `@ExtendWith` – registers custom extensions
- `@BeforeEach` – denotes that the annotated method will be executed before each test method (previously @Before)
- `@AfterEach` – denotes that the annotated method will be executed after each test method (previously @After)
- `@BeforeAll` – denotes that the annotated method will be executed before all test methods in the current class (previously @BeforeClass)
- `@AfterAll` – denotes that the annotated method will be executed after all test methods in the current class (previously @AfterClass)
- `@Disable` – disables a test class or method (previously @Ignore)

### JUnit Vintage

JUnit Vintage supports running tests based on JUnit 3 and JUnit 4 on the JUnit 5 platform.

## 4. Basic Annotations

To discuss the new annotations, we divided this section into the following groups responsible for execution: before the tests, during the tests (optional), and after the tests:

### 4.1. @BeforeAll and @BeforeEach

Below is an example of the simple code to be executed before the main test cases:

```java

@BeforeAll
static void setup() {
    log.info("@BeforeAll - executes once before all test methods in this class");
}

@BeforeEach
void init() {
    log.info("@BeforeEach - executes before each test method in this class");
}

```

It's important to note that the method with the @BeforeAll annotation needs to be static, otherwise the code won't compile.

### 4.2. @DisplayName and @Disabled

Now let's move to new test-optional methods:

```java

@DisplayName("Single test successful")
@Test
void testSingleSuccessTest() {
    log.info("Success");
}

@Test
@Disabled("Not implemented yet")
void testShowSomething() {
}

```

As we can see, we can change the display name or disable the method with a comment, using new annotations.

### 4.3. @AfterEach and @AfterAll

Finally, let's discuss the methods connected to operations after test execution:

```java

@AfterEach
void tearDown() {
    log.info("@AfterEach - executed after each test method.");
}

@AfterAll
static void done() {
    log.info("@AfterAll - executed after all test methods.");
}

```

Please note that the method with @AfterAll also needs to be a static method.

## 5. Assertions and Assumptions

JUnit 5 tries to take full advantage of the new features from Java 8, especially lambda expressions.

### 5.1. Assertions

Assertions have been moved to org.junit.jupiter.api.Assertions, and have been significantly improved. As mentioned earlier, we can now use lambdas in assertions:

```java
@Test
void lambdaExpressions() {
    List numbers = Arrays.asList(1, 2, 3);
    assertTrue(numbers.stream()
      .mapToInt(Integer::intValue)
      .sum() > 5, () -> "Sum should be greater than 5");
}
```

Although the example above is trivial, one advantage of using the lambda expression for the assertion message is that it's lazily evaluated, which can save time and resources if the message construction is expensive.

It's also now possible to group assertions with assertAll(), which will report any failed assertions within the group with a MultipleFailuresError:

```java
 @Test
 void groupAssertions() {
     int[] numbers = {0, 1, 2, 3, 4};
     assertAll("numbers",
         () -> assertEquals(numbers[0], 1),
         () -> assertEquals(numbers[3], 3),
         () -> assertEquals(numbers[4], 1)
     );
 }
```

This means it's now safer to make more complex assertions, as we'll be able to pinpoint the exact location of any failure.

### 5.2. Assumptions

Assumptions are used to run tests only if certain conditions are met. This is typically used for external conditions that are required for the test to run properly, but which aren't directly related to whatever is being tested.

We can declare an assumption with assumeTrue(), assumeFalse(), and assumingThat():

```java
@Test
void trueAssumption() {
    assumeTrue(5 > 1);
    assertEquals(5 + 2, 7);
}

@Test
void falseAssumption() {
    assumeFalse(5 < 1);
    assertEquals(5 + 2, 7);
}

@Test
void assumptionThat() {
    String someString = "Just a string";
    assumingThat(
        someString.equals("Just a string"),
        () -> assertEquals(2 + 2, 4)
    );
}
```

If an assumption fails, a TestAbortedException is thrown and the test is simply skipped.

Assumptions also understand lambda expressions.

## 6. Exception Testing

There are two ways of exception testing in JUnit 5, both of which we can implement using the assertThrows() method:

```java
@Test
void shouldThrowException() {
    Throwable exception = assertThrows(UnsupportedOperationException.class, () -> {
      throw new UnsupportedOperationException("Not supported");
    });
    assertEquals("Not supported", exception.getMessage());
}

@Test
void assertThrowsException() {
    String str = null;
    assertThrows(IllegalArgumentException.class, () -> {
      Integer.valueOf(str);
    });
}
```

The first example verifies the details of the thrown exception, and the second one validates the type of exception.

## 7. Test Suites

To continue with the new features of JUnit 5, we'll explore the concept of aggregating multiple test classes in a test suite, so that we can run those together. JUnit 5 provides two annotations, @SelectPackages and @SelectClasses, to create test suites.

Keep in mind that at this early stage, most IDEs don't support these features.

Let's have a look at the first one:

```java
@Suite
@SelectPackages("com.baeldung")
@ExcludePackages("com.baeldung.suites")
public class AllUnitTest {}
```

@SelectPackage is used to specify the names of packages to be selected when running a test suite. In our example, it will run all tests. The second annotation, @SelectClasses, is used to specify the classes to be selected when running a test suite:

```java
@Suite
@SelectClasses({AssertionTest.class, AssumptionTest.class, ExceptionTest.class})
public class AllUnitTest {}
```

For instance, the above class will create a suite that contains three test classes. Please note that the classes don't have to be in one single package.

## 8. Dynamic Tests

The last topic that we want to introduce is JUnit 5's Dynamic Tests feature, which allows us to declare and run test cases generated at run-time. Contrary to Static Tests, which define a fixed number of test cases at the compile time, Dynamic Tests allow us to define the test cases dynamically in the runtime.

Dynamic tests can be generated by a factory method annotated with @TestFactory. Let's have a look at the code:

Dynamic tests can be generated by a factory method annotated with @TestFactory. Let's have a look at the code:

```java
@TestFactory
Stream<DynamicTest> translateDynamicTestsFromStream() {
    return in.stream()
      .map(word ->
          DynamicTest.dynamicTest("Test translate " + word, () -> {
            int id = in.indexOf(word);
            assertEquals(out.get(id), translate(word));
          })
    );
}
```

This example is very straightforward and easy to understand. We want to translate words using two ArrayList, named in and out, respectively. The factory method must return a Stream, Collection, Iterable, or Iterator. In our case, we chose a Java 8 Stream.

Please note that @TestFactory methods must not be private or static. The number of tests is dynamic, and it depends on the ArrayList size.
