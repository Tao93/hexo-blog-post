---
title: 从 Android 中 Message 的 obj 无法跨进程传送自定义 Parcelable 说起
tags: [Android]
---

今天温习《Android 开发艺术探索》一书时，看到类似下面这么一句话：「使用 Messenger 将 Message 对象跨进程传输时，obj 属性无法传输自定义的 Parcelable，而只能传输 framework 已有的 Parcelable，比如 Bitmap 等」。然后也翻了源码注释，注释也是这么说的：Froyo 之后，才能用 obj 传输 framework 的 Parcelable，且 obj 不能为 null。下面是注释：

```java
/**
 * An arbitrary object to send to the recipient.  When using
 * {@link Messenger} to send the message across processes this can only
 * be non-null if it contains a Parcelable of a framework class (not one
 * implemented by the application).   For other data transfer use
 * {@link #setData}.
 *
 * <p>Note that Parcelable objects here are not supported prior to
 * the {@link android.os.Build.VERSION_CODES#FROYO} release.
 */
public Object obj;
```

当然，Message 还有 setData(Bundle b) 方法可用，而这个方法中是可以放入自定义的 Parcelable，不过此处需要埋下伏笔。

下面我们继续说 obj 为什么不能传自定义 Parcelable。理论上说，只有能加载到自定义类，那么就应该能反序列化出自定义类的对象。带着这个想法，我浏览了 Message 的 writeToParcel 和 readFromParcel 两个方法。发现了玄机就是下面的 readFromParcel 方法：

```java
private void readFromParcel(Parcel source) {
    what = source.readInt();
    arg1 = source.readInt();
    arg2 = source.readInt();
    if (source.readInt() != 0) {
        obj = source.readParcelable(getClass().getClassLoader());
    }
    when = source.readLong();
    data = source.readBundle();
    replyTo = Messenger.readMessengerOrNullFromParcel(source);
    sendingUid = source.readInt();
}
```
上面代码使用 getClass().getClassLoader 来反序列化 obj 所要引用的对象，如果这个 ClassLoader 无法找到自定义类，那么问题肯定就是出在这里了。

下面我就开始一步一步验证。首先建立一个项目，项目中添加一个自定义的 Parcelable 类：

```java
public class Book implements Parcelable {
    private int mId;
    private String mName;

    private Book(Parcel in) {
        mId = in.readInt();
        mName = in.readString();
    }

    public Book(int id, String name) {
        mId = id;
        mName = name;
    }
}
```

然后项目中开启两个进程，分别称为 client 进程和 server 进程好了。然后 client 进程的 activity 中绑定 server 进程中的 server，绑定后，client 进程使用 Messenger 向 server 进程发送一个 Message 对象，此 Message 对象的 obj 引用一个 Book 对象：

```java
private ServiceConnection mMsgConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mMessenger = new Messenger(service);
        try {
            Book book = new Book(1, "name");
            mMessenger.send(Message.obtain(null, 101, book));
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}
```

然后，我在 Message 类的 readFromParcel 方法中打上断点，当代码执行到断点处时，先验证 getClass().getClassLoader() 无法加载到自定义的 Book 类：

![](http://tao93.top/images/2018/09/01/1535789126.png)

然后验证主线程的 Thread.getContextClassLoader 可以加载到 Book 类：

![](http://tao93.top/images/2018/09/01/1535789156.png)

最后验证使用可以加载到 Book 类的 ClassLoader 的话，是可以成功反序列化得到 Book 对象的：

![](http://tao93.top/images/2018/09/01/1535789190.png)

插一句，这个 readFromParcel 不是运行在主线程，而是运行在 server 进程的 Binder 线程池中的。线程池中的线程的 getContextClassLoader() 结果是 null。

OK，上面的测试准确的验证了我的想法，现在回到前面说的 Message 的 setData(Bundle b) 方法的伏笔。显然这里的 Bundle 如果要跨进程传输自定义 Parcelable，我们也需要确定 Bundle 在反序列化时不会重蹈 Message.obj 的覆辙。实际上，Bundle 自己做不到这点，还需要我们帮它一把忙，那就是在 server 进程中，从 Message 拿到 Bundle 后再给 Bundle 设置一个 ClassLoader 即可：

```java
private static class MessengerHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case 101:
                Bundle b = msg.getData();
                b.setClassLoader(Thread.currentThread().getContextClassLoader());
                Book book = b.getParcelable("book");
        }
    }
}
```

注意，上面de  handleMessage 方法是在主线程，所以我直接使用 Thread.currentThread().getContextClassLoader() 就可以。

到此，我得出一个结论，那就是我的对于 Java 的 ClassLoader 一无所知，惭愧惭愧，还有很长的路要走。所以，后续我会加强这方面的学习，然后可能会继续加长这篇文章，毕竟这篇的标题是「从 XXX 说起」。
