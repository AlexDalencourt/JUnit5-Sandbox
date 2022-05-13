# JUNIT 5

## What is it ?

Unlike JUnit4 and less compose of 3 modules :
JUnit5 = JUnit Platform + JUnit Jupiter + Junit Vintage

- JUnit Platform : provide te engine to launch testing frameworks on the JVM
- JUnit Jupiter : provide new programming models and extension models
- JUnit Vintage : provide a `TestEngine` for running JUnit 3 and 4

Could be parameterized globaly with properties file.

## Requirement 

> Java 8 and more

## Writing tests (2)

### Annotations (2.1)

All annotations are located on `org.junit.jupiter.api`

Unlike Junit4, who Override all annotated methods, JUnit Jupiter inherit methods with the new Extension system

> `@Test` : Denotes a test method but Unlike JUnit4 this annotation don't take any attributes

> `@ParameterizedTest`: Denotes a parameterized test method

> `@DisplayName`: Name a test method

> `@BeforeEach` `@AfterEach` `@BeforeAll` `AfterAll`: analogue to Junit4 
> `@Before` `@After` `@BeforeClass` `AfterClass`

> `@ExtendWith`: Use to register extensions declaratively, the new paradigm

#### Custom Test annotation : Meta-Annotations

Could redefine test annotations for shortcut as :
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Tag("fast")
@Test
public @interface FastTest {
}
```
```java
@FastTest
void myFastTest() {
    // ...
}
```

### Test Class and Methods (2.2)

* Test Class : Contains at least one Test Method, non-abstract and with single no args constructor.
* Test Method : annotated or meta-annotated with test annotation.
* Lifecycle Method : annotated or meta-annotated

Class and method visibility : all visibility except private. It is recommended to not use public if you can.

### Display (2.3)

* `@DisplayName("")` : could give a test pretty name
* `@DisplayNameGeneration` : specify a name generator for test. Custom Generators could be implemented.
  * `DisplayNameGenerator.ReplaceUnderscores.class` : replaces underscores on the method name with whitespace.
  * `Indicative sentences` : concat class name and method to a sentence.
* By using property configuration : `junit.jupiter.displayname.generator.default`

_Look on demo junit5.displayNames.*_

### Assertions (2.4)

Retreive all statics methods of JUnit4 and more, other specific to be used with Java 8 lambdas.
Use methods from `org.junit.jupiter.api.Assertions`

_Look on demo junit5.assertions_2_4.AssertionDemo_

There is new other assertions to support Kotlin framework.

Uses of third-party Assertions libraries could make the code more readable : AssertJ, Hamcrest, Truth.

### Assumptions (2.5)

Assumptions is like JUnit4 with more methods, some specifics Java 8

### Disabling tests (2.6)

`@Disable` is a conditional test execution. It enable to disable test execution like `@Ignore` in JUnit4.
The JUnit Team do not recommand to not explain wy disabled tests

### Conditional test execution (2.7)

ExecutionCondition extension provide all you need to programmatically execute tests with conditional. The most common
implementation is DisabledCondition with the `@Disabled` annotation. Look on the package 
`org.junit.jupiter.api.condition` to se a list of other conditions available.

#### Operating System condition

Could specify the OS to execute some tests :
```java
@Test
@EnabledOnOs(MAC)
void onlyOnMacOs() {
    // ...
}

@TestOnMac
void testOnMac() {
    // ...
}

@Test
@EnabledOnOs({ LINUX, MAC })
void onLinuxOrMac() {
    // ...
}

@Test
@DisabledOnOs(WINDOWS)
void notOnWindows() {
    // ...
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@EnabledOnOs(MAC)
@interface TestOnMac {
}
```

#### Java Runtime Environment Condition

```java
@Test
@EnabledOnJre(JAVA_8)
void onlyOnJava8() {
    // ...
}

@Test
@EnabledOnJre({ JAVA_9, JAVA_10 })
void onJava9Or10() {
    // ...
}

@Test
@EnabledForJreRange(min = JAVA_9, max = JAVA_11)
void fromJava9to11() {
    // ...
}

@Test
@EnabledForJreRange(min = JAVA_9)
void fromJava9toCurrentJavaFeatureNumber() {
    // ...
}

@Test
@EnabledForJreRange(max = JAVA_11)
void fromJava8To11() {
    // ...
}

@Test
@DisabledOnJre(JAVA_9)
void notOnJava9() {
    // ...
}

@Test
@DisabledForJreRange(min = JAVA_9, max = JAVA_11)
void notFromJava9to11() {
    // ...
}

@Test
@DisabledForJreRange(min = JAVA_9)
void notFromJava9toCurrentJavaFeatureNumber() {
    // ...
}

@Test
@DisabledForJreRange(max = JAVA_11)
void notFromJava8to11() {
    // ...
}
```

#### System properties conditions

```java
@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void onlyOn64BitArchitectures() {
    // ...
}

@Test
@DisabledIfSystemProperty(named = "ci-server", matches = "true")
void notOnCiServer() {
    // ...
}
```

#### Environment variables conditions

```java
@Test
@EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
void onlyOnStagingServer() {
    // ...
}

@Test
@DisabledIfEnvironmentVariable(named = "ENV", matches = ".*development.*")
void notOnDeveloperWorkstation() {
    // ...
}
```

#### Custom conditions

There is two other conditional annotation `@EnableIf("condition")` and `@DisabledIf("condition")` to specify yours
personal conditions :
```java
@Test
@EnabledIf("customCondition")
void enabled() {
    // ...
}

@Test
@DisabledIf("customCondition")
void disabled() {
    // ...
}

boolean customCondition() {
    return true;
}
```

It is possible to outside the condition on a class like this example :
```java
package example;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.condition.EnabledIf;

class ExternalCustomConditionDemo {

    @Test
    @EnabledIf("example.ExternalCondition#customCondition")
    void enabled() {
        // ...
    }

}

class ExternalCondition {

    static boolean customCondition() {
        return true;
    }

}
```

### Tagging and Filtering (2.8)

The `@Tag("name")` would be used to filtering test execution results.

### Test execution Order (2.9)

#### Method Order (2.9.1)

```java
import org.junit.jupiter.api.MethodOrderer.OrderAnnotation;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(OrderAnnotation.class)
class OrderedTestsDemo {

    @Test
    @Order(1)
    void nullValues() {
        // perform assertions against null values
    }

    @Test
    @Order(2)
    void emptyValues() {
        // perform assertions against empty values
    }

    @Test
    @Order(3)
    void validValues() {
        // perform assertions against valid values
    }

}
```
Use MethodOrderer implementation
Use properties param : 
```properties
junit.jupiter.testmethod.order.default = org.junit.jupiter.api.MethodOrderer$OrderAnnotation
```

#### Class Order (2.9.2)

Care of parallel execution. Use ClassOrderer implementation.
```properties
junit.jupiter.testclass.order.default = org.junit.jupiter.api.ClassOrderer$OrderAnnotation
```

### Test instance Lifecycle (2.10)

By default, create a new instance of each test class for each test method (PER_METHOD).
If you want just one instance for all tests methods use `@TestInstance(Lifecycle.PER_CLASS)`

Using PER_CLASS mode have some benefit like use `@BeforeAll @AfterAll` as a non static method and could be apply on 
`@Nested` classes.

If it is wanted to change the mode by default and not just with an annotation use this property 
`junit.jupiter.testinstance.lifecycle.default=per_class`

Care to per_class mode, tests results are unpredictable and fragile. The result could change between build tool and IDE
for example.

### Nested Tests (2.11)

`@Nested` is used to 'regrouping tests', facilitate hierarchical test structure. We put this annotation on test 
subclasses
Look the example.
Only non static classes

### Dependency injection for Constructors and Methods (2.12)

On JUnit4 test constructor or methods with parameters to dependencies injection was not allowed. The Junit5 runner 
allow it.
* `TestInfo` -> give you some information about the test class / methods. The `TestInfoParameterResolver` will inject
the instance of TestInfo needed.
* `RepetitionInfo` -> give you information about annotated `@RepeatedTest @BeforeEach @AfterEach`. The 
`RepetitionInfoParameterResolver` will supply the instance of RepetitionInfo. He return information about current and 
total repetitions
* `TestReporter` -> give you some information about constructors and methods. The `TestReporterParameterResolver` will 
provide you the instance of TestReporter. Give you additional reporting data. This information will be consumed by your 
system to display some information. You can decide to push more information to display at the test end. 
Look the example.

Other resolver parameters should be specified with the appropriate extensions. Look on RandomExtension example.
For more extensions look _MockitoExtension_ and _SpringExtension_

### Test Interfaces and Default Methods (2.13)

Many of JUnit5 tests annotations are allowed on interfaces class and default methods interface like :
see code examples

`@ExtendWith @Tag` could be add on interface too. They will be automatically applied to implementation classes.

### Repeated tests (2.14)

`@RepeatedTest(10)` will mark a method as a test but repeated 'n' times (here 10). The lifecycle will be the same as
classic test method. The execution name could be customized.

look example;

### ParameterizedTest (2.15)
