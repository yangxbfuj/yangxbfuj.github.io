---
title: Android测试（一）
date: 2016-12-24 23:58:17
tags:
- 测试
categories:
- Android
- 学习笔记
---

此笔记纪录阅读Android官方文档.只纪录一些相对重要的概念和比较难于理解的知识点，有一些没有理解到
位的，将会在之后更新.

# Getting Started with Testing
Android tests are based on [JUnit](http://junit.org/), and you can run them either as **local unit tests** on the JVM or as **instrumented tests** on an Android device.

## Test Types
When using Android Studio to write any of your tests, your test code must go into one of two different code directories (source sets).

<!-- more -->

#### Local unit tests
Located at module-name/src/test/java/

These tests **run on the local JVM** and **do not have access to functional Android framework APIs**.

To get started, see [Building Local Unit Tests](#jump).

#### Instrumented tests
Located at module-name/src/androidTest/java/.

These are all tests that **must run on an Android hardware device or an Android emulator**.

Instrumented tests are built into an APK that runs on the device alongside your app under test. The system runs your test APK and your app under tests in the **same** process, so your tests can invoke methods and modify fields in the app, and **automate user interaction** with your app.

For information about how to create instrumented tests, see the following topics:

* [Building Instrumented Unit Tests](): Build complex unit tests with Android dependencies that cannot be satisfied with mock objects.
* [Automating User Interface Tests](): Create tests to verify that the user interface behaves correctly for user interactions within a single app or for interactions across multiple apps.
* [Testing App Component Integrations](): Verify the behavior of components that users do not directly interact with, such as a Service or a Content Provider.

![](https://developer.android.com/images/testing/test-types_2x.png)

However, the local unit tests and instrumented tests described above are just terms that help distinguish the tests that run on your local JVM from the tests that run on the Android platform (on a hardware device or emulator). The real testing types that you should understand when building a complete test suite are described in the following table.

![](/img/table_test.png)

## Test APIs

### JUnit

The following snippet shows an example **JUnit** 4 integration test that uses the **Espresso** APIs to perform a click action on a UI element, then checks to see if an expected string is displayed.

```java

@RunWith(AndroidJUnit4.class)
@LargeTest
public class MainActivityInstrumentationTest {

    @Rule
    public ActivityTestRule mActivityRule = new ActivityTestRule<>(
            MainActivity.class);

    @Test
    public void sayHello(){
        // 根据字符串找到View后执行一个点击
        onView(withText("Say hello!")).perform(click());
        // 根据id找到View后检查内容是否匹配
        onView(withId(R.id.textView)).check(matches(withText("Hello, World!")));
    }
}
```

In your JUnit 4 test class, you can call out sections in your test code for special processing by using the following annotations:

* **@Before**: Use this annotation to specify a block of code that **contains test setup operations**. The test class invokes this code block before each test. You can have multiple @Before methods but the order in which the test class calls these methods is not guaranteed.
* **@After**: This annotation specifies a block of code that contains test tear-down operations. The test class calls this code block after every test method. You can define multiple @After operations in your test code. Use this annotation to **release any resources from memory**.
* **@Test**: Use this annotation to mark a test method. A single test class can contain multiple test methods, each prefixed with this annotation.
* **@Rule**: Rules allow you to flexibly add or redefine the behavior of each test method in a reusable way. In Android testing, use this annotation together with one of the test rule classes that the Android Testing Support Library provides, such as ActivityTestRule or ServiceTestRule.
* **@BeforeClass**: Use this annotation to specify static methods for each test class to invoke only once. This testing step is useful for expensive operations such as connecting to a database.
* **@AfterClass**: Use this annotation to specify static methods for the test class to invoke only after all tests in the class have run. This testing step is useful for **releasing** any resources allocated in the @BeforeClass block.
* **@Test(timeout=)**: Some annotations support the ability to pass in elements for which you can set values. For example, you can specify a timeout period for the test. If the test starts but does not complete within the given timeout period, it automatically fails. You must specify the timeout period in milliseconds, for example: @Test(timeout=5000).

### Android Testing Support Library

The Android Testing Support Library provides a set of APIs that allow you to quickly build and run test code for your apps, including JUnit 4 and functional UI tests. The library includes the following instrumentation-based APIs that are useful when you want to automate your tests:

* **AndroidJUnitRunner**

A JUnit 4-compatible test runner for Android.

* **Espresso**

A UI testing framework; suitable for functional UI testing within an app.

* **UI Automator**

A UI testing framework suitable for cross-app functional UI testing between both system and installed apps.

### Assertion classes


Because Android Testing Support Library APIs extend JUnit, you can use assertion methods to display the results of tests. An assertion method compares an actual value returned by a test to an expected value, and throws an AssertionException if the comparison test fails. Using assertions is more convenient than logging, and provides better test performance.

To simplify test development, you should use the Hamcrest library, which lets you create more flexible tests using the Hamcrest matcher APIs.

### Monkey and monkeyrunner

* **Monkey**

This is a command-line tool that sends pseudo-random streams of keystrokes, touches, and gestures to a device. You run it with the Android Debug Bridge (adb) tool, and use it to stress-test your app, report back errors any that are encountered, or repeat a stream of events by running the tool multiple times with the same random number seed.

* **monkeyrunner**

This tool is an API and execution environment for test programs written in Python. The API includes functions for connecting to a device, installing and uninstalling packages, taking screenshots, comparing two images, and running a test package against an app. Using the API, you can write a wide range of large, powerful, and complex tests. You run programs that use the API with the monkeyrunner command-line tool.
