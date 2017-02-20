---
title: VelocityTracker和VelocityTrackerCompat
date: 2016-12-22 20:47:06
tags:
- VelocityTracker
- VelocityTrackerCompat
categories:
- Android
---
# VelocityTracker和VelocityTrackerCompat

查看别人代码的时候，突然看到了这个。可以获取触摸事件的速度。

<!-- more -->

## VelocityTracker

### 继承结构
```java

java.lang.Object
   ↳	android.view.VelocityTracker
```

### 简介
Helper for tracking the velocity of touch events, for implementing flinging and other such gestures.
Use obtain() to retrieve a new instance of the class when you are going to begin tracking.
Put the motion events you receive into it with addMovement(MotionEvent).
When you want to determine the velocity call computeCurrentVelocity(int) and then call getXVelocity(int) and getYVelocity(int) to retrieve the velocity for each pointer id.

## 方法
![](/img/VelocityTracker.png)

## VelocityTrackerCompat

### 继承结构

```java
java.lang.Object
   ↳	android.support.v4.view.VelocityTrackerCompat
```

### 简介
Helper for accessing features in VelocityTracker introduced after API level 4 in a backwards compatible fashion.

### 方法
![](/img/VelocityTrackerCompat.png)

## 用法

1. 在 dispatchTouchEvent(MotionEvent ev) 中调用
```java
if (null == mVelocityTracker) {
    mVelocityTracker = VelocityTracker.obtain();
    }
mVelocityTracker.addMovement(event);
```

2. 在获取速度的时候
```java
verTracker.computeCurrentVelocity(1000, mMaxVelocity);//1000代表需要获取的单位，此处为pixels/s.当为1是为pixels/ms
final float velocityX = verTracker.getXVelocity(mPointerId);
```

3. 释放
```java
if (null != mVelocityTracker) {
    mVelocityTracker.clear();
    mVelocityTracker.recycle();
    mVelocityTracker = null;
}
```