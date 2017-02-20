---
title: Building Effective Unit Tests
date: 2016-12-25 12:44:45
tags:
- 测试
categories:
- Android
---

Unit tests are the fundamental tests in your app testing strategy. By creating and running unit tests against your code, you can easily verify that the logic of individual units is correct. Running unit tests after every build helps you to quickly catch and fix software regressions introduced by code changes to your app.

A unit test generally **exercises the functionality of the smallest possible unit** of code (which could be a method, class, or component) in a repeatable way. You should build unit tests when you need to verify the logic of specific code in your app. For example, if you are unit testing a class, your test might check that the class is in the right state. Typically, the unit of code is tested in isolation; your test affects and monitors changes to that unit only. A mocking framework can be used to isolate your unit from its dependencies.

<!-- more -->

>Note: Unit tests are not suitable for testing complex UI interaction events. Instead, you should use the UI testing frameworks, as described in Automating UI Tests.

For testing Android apps, you typically create these types of automated unit tests:
* **Local tests**: Unit tests that run on your local machine only. These tests are compiled to run locally on the Java Virtual Machine (JVM) to minimize execution time. Use this approach to run unit tests that **have no dependencies on the Android framework** or **have dependencies that can be filled by using mock objects**.
* **Instrumented tests**: Unit tests that run on an Android device or emulator. These tests have access to instrumentation information, such as the Context for the app under test. Use this approach to run unit tests that **have Android dependencies which cannot be easily filled by using mock objects**.

The lessons in this class teach you how to build these types of automated unit tests.

## Lessons

[Building Local Unit Tests]()

Learn how to build unit tests that run on your local machine.

[Building Instrumented Unit Tests]()

Learn how to build unit tests that run on an Android device or emulator.
