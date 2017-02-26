---
title: ContentProvider相关
date: 2017-02-20 21:15:26
tags:
categories:
- Android
- 学习笔记
---

* 相关代码地址为 https://github.com/yangxbfuj/ContentProviderDemo

* ContentProvider 在 Android 中的作用是对外共享数据。
* ContentProvider 统一了数据的访问方式，它实际上是对 SQLiteOpenHelper 的进一步封装，降低了用户使用成本。
<!-- more -->
* ContentProvider Uri结构:
![](/img/contentprovider_uri_ structure.png)
    Authority：授权信息，用以区别不同的ContentProvider；
    Path：表名，用以区分ContentProvider中不同的数据表；
    Id：Id号，用以区别表中的不同数据；
* 如下将字符串转换为 Uri:
```java
Uri uri = Uri.parse("content://contacts/people");
```
* 使用 UriMatcher 管理不同的Uri对应的Type类型

```java
static UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

static{
    uriMatcher.addURI(AUTHORIY,"tablename",INFOS); 
    uriMatcher.addURI(AUTHORIY,"tablename/*",INFO_ITEM); // *匹配任意长度的字符
    uriMatcher.addURI(AUTHORIY,"tablename/#",INFO_ITEM2);// #匹配任意长度的数字
}
```
### ContentProviderResult

包含使用 ContentProviderOperation 操作 ContentProvider 的一项结果结果
 
### ContentProviderClient

通过如下代码获得该对象：
```java
ContentResolver.acquireContentProviderClient(Uri uri)
```
或
```java
ContentResolver.acquireUnstableContentProviderClient(Uri uri)
```
该对象保持一个指向ContentProvider的连接,所以在不需要保持该链接时需要手动调用close()方法.即该对象的方法是不是线程安全的。
PS: relase()方法已经被弃用

附上官方文档原文：

The public interface object used to interact with a specific ContentProvider.

Instances can be obtained by calling acquireContentProviderClient(Uri) or acquireUnstableContentProviderClient(Uri). Instances must be released using close() in order to indicate to the system that the underlying ContentProvider is no longer needed and can be killed to free up resources.

Note that you should generally create a new ContentProviderClient instance for each thread that will be performing operations. Unlike ContentResolver, the methods here such as query(Uri, String[], String, String[], String) and openFile(Uri, String) are not thread safe -- you must not call close() on the ContentProviderClient those calls are made from until you are finished with the data they have returned.

### ContentProviderOperation

用于指代一个对于ContentProvider的操作。applyBatch(String authority,ArrayList<ContentProviderOperation> operations)同时支持事务。
如下为一个简单事例
```java
 private void insertWithClientAndOperation() {
        long timeB = SystemClock.uptimeMillis();
        ContentProviderClient contentProviderClient = getContentResolver().acquireContentProviderClient(MyContentProvider.USER_INTO_URI);
        int index = 0;
        ArrayList<ContentProviderOperation> ops = new ArrayList<>(100); //操作集合
        try {
            while (index++ < 100) {
                ops.add(ContentProviderOperation.newInsert(MyContentProvider.USER_INTO_URI)
                        .withValue(ContentProviderDemoSQLiteHelper.USER_NAME, "A")
                        .withValue(ContentProviderDemoSQLiteHelper.USER_PHONE, "1")
                        .build());
            }
            timeB = SystemClock.uptimeMillis();
            ContentProviderResult[] results = contentProviderClient.applyBatch(ops);
        } catch (OperationApplicationException e) {
            Log.e(getClass().getSimpleName(), "e", e);
            e.printStackTrace();
        } catch (RemoteException e) {
            Log.e(getClass().getSimpleName(), "e", e);
            e.printStackTrace();
        } finally {
            contentProviderClient.close();
        }
        long timeA = SystemClock.uptimeMillis();
        Log.d(getClass().getSimpleName(), String.format("Insert value in insertWithClientAndOperation() %s cost %d ms.", "all info ", timeA - timeB));
    }
```

### ContentProvider.PipeDataWriter

Interface to write a stream of data to a pipe. Use with openPipeHelper(Uri, String, Bundle, T, ContentProvider.PipeDataWriter).

### AsyncQueryHandler
一个用于在使用ContentResolver异步查询时更佳方便的帮助类。
PS:其实也可以用于insert、delete等操作

实例如下：
```java
private void insertWithAsyncQueryHandler() {
        MyAsyncQueryHandler asyncQueryHandler = new MyAsyncQueryHandler(getContentResolver());
        ContentValues contentValues = new ContentValues();
        contentValues.put(ContentProviderDemoSQLiteHelper.USER_NAME, "A");
        contentValues.put(ContentProviderDemoSQLiteHelper.USER_PHONE, "1");
        asyncQueryHandler.startInsert(0, null, MyContentProvider.USER_INTO_URI, contentValues);
    }
```

### ContentObserver
作为一个ContentProvider的数据改变的观察者

类定义
```java
    public static class MyContentObserver extends ContentObserver{
        /**
         * Creates a content observer.
         *
         * @param handler The handler to run {@link #onChange} on, or null if none.
         */
        public MyContentObserver(Handler handler) {
            super(handler);
            Log.d(getClass().getSimpleName(), "MyContentObserver constructed ");
        }

        /**
         * 当观察到数据改变时，回调此方法
         * @param selfChange
         * @param uri
         */
        @Override
        public void onChange(boolean selfChange, Uri uri) {
            Log.d(getClass().getSimpleName(), "ContentObserver onChange(boolean,Uri) start");
            super.onChange(selfChange, uri);
            Log.d(getClass().getSimpleName(), "ContentObserver onChange(boolean,Uri) end");
        }

        /**
         * 当观察到数据改变时，回调此方法
         * @param selfChange
         */
        @Override
        public void onChange(boolean selfChange) {
            Log.d(getClass().getSimpleName(), "ContentObserver onChange(boolean) start");
            super.onChange(selfChange);
            Log.d(getClass().getSimpleName(), "ContentObserver onChange(boolean) start");
        }
    }
```

在ContentProvider中操作数据库后，通知数据库ContentResolver
```java
getContext().getContentResolver().notifyChange(Uri,ContentObserver);
```

在使用ContentResolver的地方注册观察者
```java
private void setContentObserver() {
    Log.d(getClass().getSimpleName(),"registerContentObserver uri = "+ MyContentProvider.USER_INTO_URI.toString());
    getContentResolver().registerContentObserver(MyContentProvider.USER_INTO_URI, true, contentObserver);
}
```

最后在Activity、Services的声明周期结束时取消观察者
```java
@Override
protected void onDestroy() {
    getContentResolver().unregisterContentObserver(contentObserver);
    super.onDestroy();
}
```