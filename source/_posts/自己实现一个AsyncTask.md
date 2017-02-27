---
title: 自己实现一个AsyncTask
date: 2017-02-27 14:19:37
tags:
categories:
- Android
- 多线程
---

本文自己实现了一个简单的类似于 AsyncTask 的类，主要阐述基本思想，可能潜在的 BUG 无视。

<!-- more --> 

接口定义:

```java
/**
 * Created by yangxb on 2017/2/27.
 */

public interface IMyAsyncTask<Params, Progress, Result> {

    /**
     * 执行任务
     * @param params
     */
    public void execute(Params... params);

    /**
     * 后台执行方法
     * @param params
     * @return
     */
    public Result doBackground(Params... params);

    /**
     * 执行在的在主线程中调用的方法
     */
    public void preExecute();

    /**
     * 执行在主线程的更新进度的方法
     * @param progress
     */
    public void onProgressUpdate(Progress progress);

    /**
     * 执行在主线程
     * @param result
     */
    public void onPostExecute(Result result);

}
```

抽象类:

```java
import java.util.concurrent.*;

/**
 * Created by yangxb on 2017/2/27.
 */
public class MyAsyncTask<Params, Progress, Result> implements IMyAsyncTask<Params, Progress, Result> {

    private static final int THREAD_SIZE = 1;

    private static final ExecutorService sExecutor = Executors.newFixedThreadPool(THREAD_SIZE);

    /**
     * 用户不能重写
     * @param params
     */
    @Override
    public final void execute(Params... params) {
        this.preExecute();
        Future<Result> future = sExecutor.submit(new Callable<Result>() {
            @Override
            public Result call() throws Exception {
                Result result = doBackground(params);
                /**
                 * 在 Android 中,此处应该调用 Handler,将结果通过 Message 的形式传递至主线程。
                 * 此处为了简化代码，在主线程中阻塞等待 future 返回结果。虽然不够严谨，但希望能
                 * 将 AsyncTask 的原理表达清楚。
                 */
                return result;
            }
        });

        try {
            this.onPostExecute(future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    /**
     * 后台执行耗时的任务,需要用户重写
     * @param params
     * @return
     */
    @Override
    public Result doBackground(Params... params) {
        return null;
    }

    /**
     * 需要用户重写
     */
    @Override
    public void preExecute() {

    }

    @Override
    public void onProgressUpdate(Progress progress) {

    }

    /**
     * 需要用户重写
     * @param result
     */
    @Override
    public void onPostExecute(Result result) {

    }
}
```

调用:

```java
import java.util.Random;

public class Main {

    public static void main(String[] args) {
        IMyAsyncTask realAsyncTask = new RealAsyncTask();
        for (int n = 0;n < 10;n++){
            Random random = new Random(20);
            int i = random.nextInt(20);
            realAsyncTask.execute(i);
        }
    }

    public static class RealAsyncTask extends MyAsyncTask<Integer,Void,Integer>{

        @Override
        public void preExecute() {
            super.preExecute();
            System.out.println("preExecute's thread is  " + Thread.currentThread().getName());
        }

        @Override
        public Integer doBackground(Integer[] params) {
            System.out.println("doBackground's thread is " + Thread.currentThread().getName());
            return fib(params[0]);
        }

        @Override
        public void onPostExecute(Integer integer) {
            super.onPostExecute(integer);
            System.out.println("onPostExecute's thread is " + Thread.currentThread().getName());
            System.out.println("onPostExecute answer is " + integer);
        }
    }

    public static int fib(int i){
        if (i <= 2){
            return 1;
        }else {
            return fib(i-1) + fib(i-2);
        }
    }
}
```

output:

```java
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
preExecute's thread is  main
doBackground's thread is pool-1-thread-1
onPostExecute's thread is main
onPostExecute answer is 233
```