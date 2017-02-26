---
title: Building Local Unit Tests
date: 2016-12-25 13:02:36
tags:
- 测试
categories:
- Android
---
If your unit test has no dependencies or only has simple dependencies on Android, you should run your test on a local development machine. This testing approach is efficient because it helps you avoid the overhead of loading the target app and unit test code onto a physical device or emulator every time your test is run. Consequently, the execution time for running your unit test is greatly reduced. With this approach, you normally use a mocking framework, like Mockito, to fulfill any dependency relationships.

<!-- more -->

## Set Up Your Testing Environment

In your Android Studio project, you must store the source files for local unit tests at **module-name/src/test/java/**. This directory already exists when you create a new project.

You also need to configure the testing dependencies for your project to use the standard APIs provided by the JUnit 4 framework. If your test needs to interact with Android dependencies, include the Mockito library to simplify your local unit tests. To learn more about using mock objects in your local unit tests, see Mocking Android dependencies.

In your app's top-level build.gradle file, you need to specify these libraries as dependencies:

```gradle
dependencies {
    // Required -- JUnit 4 framework
    testCompile 'junit:junit:4.12'
    // Optional -- Mockito framework
    testCompile 'org.mockito:mockito-core:1.10.19'
}
```

## Create a Local Unit Test Class

Your local unit test class should be written as a JUnit 4 test class. JUnit is the most popular and widely-used unit testing framework for Java. The latest version of this framework, JUnit 4, allows you to write tests in a cleaner and more flexible way than its predecessor versions. Unlike the previous approach to Android unit testing based on JUnit 3, with JUnit 4, you do not need to extend the junit.framework.TestCase class. You also do not need to prefix your test method name with the ‘test’ keyword, or use any classes in the junit.framework or junit.extensions package.

To create a basic JUnit 4 test class, create a Java class that contains one or more test methods. A test method begins with the @Test annotation and contains the code to exercise and verify a single functionality in the component that you want to test.

The following example shows how you might implement a local unit test class. The test method **emailValidator_CorrectEmailSimple_ReturnsTrue** verifies that the isValidEmail() method in the app under test returns the correct result.

```java
import org.junit.Test;
import java.util.regex.Pattern;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;

public class EmailValidatorTest {

    @Test
    public void emailValidator_CorrectEmailSimple_ReturnsTrue() {
        assertThat(EmailValidator.isValidEmail("name@email.com"), is(true));
    }
    ...
}
```
To test that components in your app return the expected results, use the junit.Assert methods to perform validation checks (or assertions) to compare the state of the component under test against some expected value. To make tests more readable, you can use Hamcrest matchers (such as the is() and equalTo() methods) to match the returned result against the expected result.

## Mock Android dependencies

By default, the Android Plug-in for Gradle executes your local unit tests against a modified version of the android.jar library, which does not contain any actual code. Instead, method calls to Android classes from your unit test throw an exception. This is to make sure you test only your code and do not depend on any particular behavior of the Android platform (that you have not explicitly mocked).

You can use a mocking framework to stub out external dependencies in your code, to easily test that your component interacts with a dependency in an expected way. By substituting Android dependencies with mock objects, you can isolate your unit test from the rest of the Android system while verifying that the correct methods in those dependencies are called. The Mockito mocking framework for Java (version 1.9.5 and higher) offers compatibility with Android unit testing. With Mockito, you can configure mock objects to return some specific value when invoked.
