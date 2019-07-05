---
title: 一个不一样的 ANR
tags: [Android]
---

最近碰到一个 ANR 问题，拿到 traces 文件后，显示主线程的堆栈是下面这样的：

```
  at android.os.MessageQueue.nativePollOnce(Native method)
  at android.os.MessageQueue.next(MessageQueue.java:323)
  at android.os.Looper.loop(Looper.java:135)
  at android.app.ActivityThread.main(ActivityThread.java:5417)
  at java.lang.reflect.Method.invoke!(Native method)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:726)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:616)
```

这样一段堆栈，在很多情况下表明主线程是正常的，即处在等待区下一条主线程消息的过程中，而不是陷在某个耗时特别长的消息中。

经过反复测试，最后确认这个 ANR 和设置了 Default UncaughtExceptionHandler 有关，问题代码可以简化成下面的样子：

```java
public class MainActivity extends AppCompatActivity {

    static {
        Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(final Thread t, Throwable e) {
                Log.i("uncaught", Thread.currentThread().toString());
                Log.i("uncaught", t.toString());
            }
        });
    }

    private Thread thread;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.btn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        throw new RuntimeException("hello world");
                    }
                }).start();
                //throw new RuntimeException("hello world!!");
            }
        });
    }
}
```

上面的代码的表现是，如果只让第 26 行抛出异常，那么一切看起来都正常，但是如果让第 29 行抛出异常，那么应用直接就无响应了。

经过一番查找与验证，我发现原因大致是，当一个 Java 线程抛出了未捕获的异常时，JVM 先会调用到 UncaughtExceptionHandler，然后再会把此线程停止掉。所以这段代码中，如果主线程抛出异常，那么第 6 行的方法结束后，主线程就会被 JVM 给停止掉，既然主线程都停止掉了，那自然就无响应了，也就会发生 ANR 了。

事实上，一般我们设置自定义 UncaughtExceptionHandler 时，都会在自定义的 uncaughtException 方法最后再调用一遍被我们顶替掉的系统默认的 UncaughtExceptionHandler，以便把应用 kill 掉，而这个例子，充分的显示了，当未捕获异常发生后，就算赖着不 kill 掉应用也是不行的，因为可能主线程都已经被停掉了。

关于 JVM 先调用 UncaughtExceptionHandler 然后把发生未捕获异常的线程停止掉的说法，见于 [Java Language Specification 11.3](https://docs.oracle.com/javase/specs/jls/se8/html/jls-11.html#jls-11.3)，如下所示的片段：

![](http://tao93.top/images/2019/01/18/1547816082.png)

那么主线程被停止掉，是个什么样的状态呢，我用下面的代码，把主线程的状态给输出来：

```java
public class MainActivity extends AppCompatActivity {

    static {
        Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(final Thread t, Throwable e) {
                Log.i("uncaught", Thread.currentThread().toString());
                Log.i("uncaught", t.toString());
            }
        });
    }

    private Thread thread;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.btn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                final Thread mainThread = Thread.currentThread();
                Log.i("<<<", mainThread.getState().toString());
                if (thread == null) {
                    thread = new Thread(new Runnable() {
                        @Override
                        public void run() {
                            for (int i = 0; i < 3; ++i) {
                                try {
                                    Thread.sleep(1000);
                                } catch (InterruptedException e) {
                                    e.printStackTrace();
                                }
                                Log.i("<<<", mainThread.getState().toString());
                            }
                        }
                    });
                    thread.start();
                }
                throw new RuntimeException("hello world!!");
            }
        });
    }
}
```

上面代码的第 24 行和第 35 行一共会 4 此输出主线程的状态，结果如下面所示：

```
01-18 21:00:24.308 14533 14533 I <<<     : RUNNABLE
01-18 21:00:25.309 14533 14570 I <<<     : NEW
01-18 21:00:26.310 14533 14570 I <<<     : NEW
01-18 21:00:27.311 14533 14570 I <<<     : NEW
```

从上面代码可知，主线程从 RUNNABLE 状态变成了 NEW 状态，为什么是 NEW 状态，我也不清楚，也许将来对 JVM 了解更多了，会清除吧。

回到这个 ANR 来，这个例子说明，我们如果使用自定义的 UncaughtExceptionHandler，记得要把应用 kill 掉，还有 default 的 UncaughtExceptionHandler 是全局公用的，很容易会出现被顶替 (覆盖)，所以切记别随意调用，免得出现意料之外的问题。

修复这样的 ANR，分成两部分，第一部分是先移除或者修改不恰当的设置 UncaughtExceptionHandler 的代码，先消除 ANR，然后还需要把引发 ANR 的另一个原因，也就是未捕获的异常给修复掉，不然应用不会 ANR 了但是会 Crash。