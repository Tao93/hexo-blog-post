---
title: Android Toast 两个 Crash
tags: [Android]
---

Toast 是 Android 系统一种非常简单的提示性小工具，最近我尝试修复 Toast 相关的两种 Crash，所以把相关的原委和过程记录了下来。先来看一下第一种 Crash 的 log:

```java
android.view.WindowManager$BadTokenException: Unable to add window -- token android.os.BinderProxy@e2815e is not valid; is your activity running?
        at android.view.ViewRootImpl.setView(ViewRootImpl.java:679)
        at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:342)
        at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:93)
        at android.widget.Toast$TN.handleShow(Toast.java:459)
        at android.widget.Toast$TN$2.handleMessage(Toast.java:342)
        at android.os.Handler.dispatchMessage(Handler.java:102)
        at android.os.Looper.loop(Looper.java:154)
        at android.app.ActivityThread.main(ActivityThread.java:6119)
        at java.lang.reflect.Method.invoke(Native Method) 
```

上面的 stack trace 中的代码全部是关于 UI 线程中处理一个消息的，这个消息需要做的是把 Toast 需要显示的 view 添加到 WindowManager 中，从而可以显示出来。这样的 crash 是没有任何 app 代码牵涉其中的，所以无法确定是 app 何处的代码导致的这个 crash。我们先来看看 Toast 显示的大致过程。

首先是通过 `Toast#makeText` 方法或者 `Toast` 构造函数创建 Toast 对象，然后就可以调用它的 show 方法了。不过此方法是异步的，它仅仅是将该 toast 添加到一个队列中，等待显示，即此方法不等 toast 真正显示就已经返回了，而 toast 的显示需要用一个新的 UI 线程消息中的代码来显示出来。

`Toast#show` 方法：

```java
    /**
     * Show the view for the specified duration.
     */
    public void show() {
        if (mNextView == null) {
            throw new RuntimeException("setView must have been called");
        }

        INotificationManager service = getService();
        String pkg = mContext.getOpPackageName();
        TN tn = mTN;
        tn.mNextView = mNextView;

        try {
            service.enqueueToast(pkg, tn, mDuration);
        } catch (RemoteException e) {
            // Empty
        }
    }
```

上面代码的第 15 行是一个跨进程调用了 NotificationServiceManager 的方法，而作为参数的 TN 对象是实现了 [IInterface](https://developer.android.com/reference/android/os/IInterface) 的，所以可以通过 Binder 传给其他进程。Toast 真正的显示，需要等 NotificationServiceManager 回调回来，这个回调也就是调用 Toast 内部类 TN 的 show 方法。而从 api 25 开始，此方法还会将 NotificationServiceManager 产生的一个 window token 传递过来。

api 25 中的 `TN#show` 方法：

```java
        /**
         * schedule handleShow into the right thread
         */
        @Override
        public void show(IBinder windowToken) {
            if (localLOGV) Log.v(TAG, "SHOW: " + this);
            mHandler.obtainMessage(0, windowToken).sendToTarget();
        }
```

api 24 中的 `TN#show` 方法：

```java
        /**
         * schedule handleShow into the right thread
         */
        @Override
        public void show() {
            if (localLOGV) Log.v(TAG, "SHOW: " + this);
            mHandler.post(mShow);
        }
```

此 `TN#show` 方法是被远程调用的，所以实际会运行在 app 的 Binder 线程池的线程中，所以此方法向主线程发了一个消息，这个消息才是真正让 toast 显示的地方。不同的是 api 25 的代码还会把 window token 也传递到消息中。处理这个消息的代码，会调用 `TN#handleShow` 方法，这个 `handleShow` 是下面这样的：

```java
        public void handleShow() {
            if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
                    + " mNextView=" + mNextView);
            if (mView != mNextView) {
                // remove the old view if necessary
                handleHide();
                mView = mNextView;
                Context context = mView.getContext().getApplicationContext();
                String packageName = mView.getContext().getOpPackageName();
                if (context == null) {
                    context = mView.getContext();
                }
                mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
                // We can resolve the Gravity here by using the Locale for getting
                // the layout direction
                final Configuration config = mView.getContext().getResources().getConfiguration();
                final int gravity = Gravity.getAbsoluteGravity(mGravity, config.getLayoutDirection());
                mParams.gravity = gravity;
                if ((gravity & Gravity.HORIZONTAL_GRAVITY_MASK) == Gravity.FILL_HORIZONTAL) {
                    mParams.horizontalWeight = 1.0f;
                }
                if ((gravity & Gravity.VERTICAL_GRAVITY_MASK) == Gravity.FILL_VERTICAL) {
                    mParams.verticalWeight = 1.0f;
                }
                mParams.x = mX;
                mParams.y = mY;
                mParams.verticalMargin = mVerticalMargin;
                mParams.horizontalMargin = mHorizontalMargin;
                mParams.packageName = packageName;
                mParams.removeTimeoutMilliseconds = mDuration ==
                    Toast.LENGTH_LONG ? LONG_DURATION_TIMEOUT : SHORT_DURATION_TIMEOUT;
                if (mView.getParent() != null) {
                    if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                    mWM.removeView(mView);
                }
                if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
                mWM.addView(mView, mParams);
                trySendAccessibilityEvent();
            }
        }
```

上面代码的第 37 行，才是真正把 toast 的 view 添加到 WindowManager，也就是让 toast 显示出来，至此理了一遍 toast 显示的流程。而最前面的 crash log 表明，crash 是发生在 `ViewRootImpl#setView` 方法中的，并且提示 window token invalid。这其实就是提示 从 NotificationManagerService 传过来给 TN 的 token 对象失效了。而失效的原因，其实得从 NotificationManagerService 中找。

`NotificationManagerService#showNextToastLocked` 方法：

```java
    void showNextToastLocked() {
        ToastRecord record = mToastQueue.get(0);
        while (record != null) {
            if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
            try {
                record.callback.show(record.token);
                scheduleTimeoutLocked(record);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Object died trying to show notification " + record.callback
                        + " in package " + record.pkg);
                // remove it from the list and let the process die
                int index = mToastQueue.indexOf(record);
                if (index >= 0) {
                    mToastQueue.remove(index);
                }
                keepProcessAliveIfNeededLocked(record.pid);
                if (mToastQueue.size() > 0) {
                    record = mToastQueue.get(0);
                } else {
                    record = null;
                }
            }
        }
    }
```

此方法就是 NotificationManagerService 发起显示下一个 toast 的代码，注意到第 6 行调用的 show 方法，其实就是远程调用 TN 对象的 show 方法，而第 6 行的 callback 其实就是 TN 对象所对应的远程代理对象。紧接着第 7 行调用的 scheduleTimeoutLocked 方法，其实设定了一个失效限制，使得第 6 行传递的 token 会在几秒内失效。

`scheduleTimeoutLocked` 方法

```java
    private void scheduleTimeoutLocked(ToastRecord r)
    {
        mHandler.removeCallbacksAndMessages(r);
        Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
        long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
        mHandler.sendMessageDelayed(m, delay);
    }
```

上面代码发送一个 delayed 消息(截止 api 27，此 delay 时长是 2 秒或 3.5 秒），处理上面方法发送的消息的代码：

```java
        @Override
        public void handleMessage(Message msg)
        {
            switch (msg.what)
            {
                case MESSAGE_TIMEOUT:
                    handleTimeout((ToastRecord)msg.obj);
                    break;
                ...
            }
        }
```

上面被调用的 `handleTimeOut` 方法：

```java
    private void handleTimeout(ToastRecord record)
    {
        if (DBG) Slog.d(TAG, "Timeout pkg=" + record.pkg + " callback=" + record.callback);
        synchronized (mToastQueue) {
            int index = indexOfToastLocked(record.pkg, record.callback);
            if (index >= 0) {
                cancelToastLocked(index);
            }
        }
    }
```

上面第 7 行被调用的 `cancelToastLocked` 方法：

```java
        void cancelToastLocked(int index) {
        ToastRecord record = mToastQueue.get(index);
        try {
            record.callback.hide();
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to hide notification " + record.callback
                    + " in package " + record.pkg);
            // don't worry about this, we're about to remove it from
            // the list anyway
        }

        ToastRecord lastToast = mToastQueue.remove(index);
        mWindowManagerInternal.removeWindowToken(lastToast.token, true, DEFAULT_DISPLAY);

        keepProcessAliveIfNeededLocked(record.pid);
        if (mToastQueue.size() > 0) {
            // Show the next one. If the callback fails, this will remove
            // it from the list, so don't assume that the list hasn't changed
            // after this point.
            showNextToastLocked();
        }
    }
```

上面第 13 行就是使 window token 失效的代码。至此可知，NotificationServiceManager 远程调用 `TN#show` 方法后几秒内，此 token 就会失效，在这几秒内如果 toast 没有真正添加到 WindowManager，那么等添加的时候，就会出现 BadTokenException，应用就会 crash。而阻碍 toast 的 view 被添加到 WindowManager，只有 UI 线程的忙碌，也就是如果 UI 线程已经在执行或者马上要执行的其他消息比较耗时，那么 toast 的 view 就无法及时添加。

不过，Google 也意识到这种 UI 线程 block 不到 ANR 时长就 crash 的现象了，所以在 api 26 中，此 BadTokenException 直接被捕获了，也就是下面的第 42 行：

```java
        public void handleShow(IBinder windowToken) {
            ...
            if (mView != mNextView) {
                ...
                if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
                // Since the notification manager service cancels the token right
                // after it notifies us to cancel the toast there is an inherent
                // race and we may attempt to add a window after the token has been
                // invalidated. Let us hedge against that.
                try {
                    mWM.addView(mView, mParams);
                    trySendAccessibilityEvent();
                } catch (WindowManager.BadTokenException e) {
                    /* ignore */
                }
            }
        }
```

所以此 crash，仅仅发生在 api 25 的系统中，要修复这个问题，可以参考 github 上的 [ToastCompat](https://github.com/drakeet/ToastCompat) 中的方法。

再来看一下另一种 Crash log：

```java
java.lang.IllegalStateException: View android.widget.LinearLayout{41a97eb8 V.E..... ......ID 0,0-540,105 #7f0b020d app:id/toast_layout_root} has already been added to the window manager.
   at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:223)
   at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:69)
   at android.widget.Toast$TN.handleShow(Toast.java:402)
   at android.widget.Toast$TN$1.run(Toast.java:310)
   at android.os.Handler.handleCallback(Handler.java:730)
   at android.os.Handler.dispatchMessage(Handler.java:92)
   at android.os.Looper.loop(Looper.java:137)
   at android.app.ActivityThread.main(ActivityThread.java:5136)
   at java.lang.reflect.Method.invokeNative(Method.java)
   at java.lang.reflect.Method.invoke(Method.java:525)
   at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:737)
   at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:553)
   at dalvik.system.NativeStart.main(NativeStart.java)
```

这个 crash 原因是同一个 view 被重复添加到 WindowManager 导致的。抛出异常的地方是 `WindowManagerGlobal#addView` 方法，也就是下面的代码第 15 行：

```java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (display == null) {
            throw new IllegalArgumentException("display must not be null");
        }
        ...
            int index = findViewLocked(view, false);
            if (index >= 0) {
                if (mDyingViews.contains(view)) {
                    // Don't wait for MSG_DIE to make it's way through root's queue.
                    mRoots.get(index).doDie();
                } else {
                    throw new IllegalStateException("View " + view
                            + " has already been added to the window manager.");
                }
                // The previous removeView() had not completed executing. Now it has.
            }
```

从上面代码可知，第 10 行 index 非负，且 mDyingViews 包含需添加的 view，则会抛出此异常。第 10 行 index 非负的原因，是 mViews 包含此 view，如下面代码所示：

```java
    private int findViewLocked(View view, boolean required) {
        final int index = mViews.indexOf(view);
        if (required && index < 0) {
            throw new IllegalArgumentException("View=" + view + " not attached to window manager");
        }
        return index;
    }
```

所以，也就是当需要添加一个 view 时，如果此 view 在 mViews 中却不在 mDyingViews 中，那就会抛出异常。现在我们看一下 `Toast#TN#handleShow` 方法：

```java
        public void handleShow(IBinder windowToken) {
            ...
            if (mView != mNextView) {
                ...
                if (mView.getParent() != null) {
                    if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
                    mWM.removeView(mView);
                }
                if (localLOGV) Log.v(TAG, "ADD! " + mView + " in " + this);
                // Since the notification manager service cancels the token right
                // after it notifies us to cancel the toast there is an inherent
                // race and we may attempt to add a window after the token has been
                // invalidated. Let us hedge against that.
                try {
                    mWM.addView(mView, mParams);
                    trySendAccessibilityEvent();
                } catch (WindowManager.BadTokenException e) {
                    /* ignore */
                }
            }
        }
```

上面代码显示，实际上，Toast 被现实时，其实会先把 view 从 WindowManager 移除（注意一下移除的前提是 view 的 parent 不空），然后再尝试添加。我们看看 WindowManagerGlobal#removeView 方法：

```java
    public void removeView(View view, boolean immediate) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }

        synchronized (mLock) {
            int index = findViewLocked(view, true);
            View curView = mRoots.get(index).getView();
            removeViewLocked(index, immediate);
            if (curView == view) {
                return;
            }

            throw new IllegalStateException("Calling with view " + view
                    + " but the ViewAncestor is attached to " + curView);
        }
    }
```

需要注意上面方法有个 immediate 参数，不过从 `Toast#TN#handleShow` 调用过来时，这个参数会是 false。现在假设 view 包含在 mViews 中，那么上面第 7 行 index 将非负，上面第 9 行调用了 removeViewLocked 方法：

```java
    private void removeViewLocked(int index, boolean immediate) {
        ViewRootImpl root = mRoots.get(index);
        View view = root.getView();

        if (view != null) {
            InputMethodManager imm = InputMethodManager.getInstance();
            if (imm != null) {
                imm.windowDismissed(mViews.get(index).getWindowToken());
            }
        }
        boolean deferred = root.die(immediate);
        if (view != null) {
            view.assignParent(null);
            if (deferred) {
                mDyingViews.add(view);
            }
        }
    }
```

上面代码 第 11 行因为 immediate 为 false，所以返回的 deferred 是 true，那么第 15 行就会把 view 添加到 mDyingViews。

至此总结一下，只要 view 的 parent 不空，那么它就会尝试被移除，如果 mView是中有次 view，则尝试移除的结果就是 mDyingViews 也会包含此 view，则 crash 不会发生。

经过分析系统代码，我发现给 view 设置 parent 是在 ViewRootImpl 中的 setView 方法调用 `view.assignParent(this)` 做到的，而 ViewRootImpl #setView 是在 WindowManagerGlobal#addView 调用的。置空 parent 则是在 WindowManagerGlobal#removeViewLocked 做的，而从 mViews 移除 view 是在 WindowManagerGlobal#doRemoveView 做的：

```java
    void doRemoveView(ViewRootImpl root) {
        synchronized (mLock) {
            final int index = mRoots.indexOf(root);
            if (index >= 0) {
                mRoots.remove(index);
                mParams.remove(index);
                final View view = mViews.remove(index);
                mDyingViews.remove(view);
            }
        }
        if (ThreadedRenderer.sTrimForeground && ThreadedRenderer.isAvailable()) {
            doTrimForeground();
        }
    }
```

由于这些方法都是在主线程调用的，所以可以肯定，在 addView 时，mView 包含 view 时，则此 view 的 parent不空。而 mView 不包含 view 时，它的 parent 为空。看起来似乎无懈可击，系统代码确保了 toast 的显示不会出现重复添加 view 导致的 IllegalStateException。但是明明 crash 就是发生了，分析堆栈也可知就是出现了 view 在 mViews 中但却不在 mDyingViews 中的情况。

经过分析，我可能找到了一种原因。先来看 WindowManagerGlobal#addView 方法：

```java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {
        ...
        ViewRootImpl root;
        View panelParentView = null;

        synchronized (mLock) {
            ...
            int index = findViewLocked(view, false);
            if (index >= 0) {
                if (mDyingViews.contains(view)) {
                    // Don't wait for MSG_DIE to make it's way through root's queue.
                    mRoots.get(index).doDie();
                } else {
                    throw new IllegalStateException("View " + view
                            + " has already been added to the window manager.");
                }
                // The previous removeView() had not completed executing. Now it has.
            }
            ...
            root = new ViewRootImpl(view.getContext(), display);

            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);

            // do this last because it fires off messages to start doing things
            try {
                root.setView(view, wparams, panelParentView);
            } catch (RuntimeException e) {
                // BadTokenException or InvalidDisplayException, clean up.
                if (index >= 0) {
                    removeViewLocked(index, true);
                }
                throw e;
            }
        }
    }
```

首先上面代码可能存在一处漏洞。假设 view 之前从未添加过，那么低 9 行返回 -1，第 25 至 27 行把 view 添加到了 mViews 中，然后假设此 view 添加过程中失败了，即第 31 行抛出了异常，可是此时 index 是 -1，所以第 15 行企图移除此 view 是做不到的，于是此 view 就留在了 mViews 中，这可能是系统的移除漏洞。另一种情况，假设第 9 行返回非负值，那么此 view 在第 13 行会立即移除，第 25 行重新添加到 mView 中时，此 view 新的 index 已经不是第 9 行的值了，然后如果第 31 行添加失败，那么第 35 行将会被执行，可是 index 是错误的，这将会导致错误的 view 被移除！

上面可能的漏洞要发生，需要第 35 行抛出异常，而查看 `ViewRootImpl#setView` 方法可知，如果异常抛出，那么 view 的 parent 将尚未设置:

```java
    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
        synchronized (this) {
            if (mView == null) {
                ...
                if (res < WindowManagerGlobal.ADD_OKAY) {
                    mAttachInfo.mRootView = null;
                    mAdded = false;
                    mFallbackEventHandler.setView(null);
                    unscheduleTraversals();
                    setAccessibilityFocus(null, null);
                    switch (res) {
                        case WindowManagerGlobal.ADD_BAD_APP_TOKEN:
                        case WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN:
                            throw new WindowManager.BadTokenException(
                                    "Unable to add window -- token " + attrs.token
                                    + " is not valid; is your activity running?");
                        case WindowManagerGlobal.ADD_NOT_APP_TOKEN:
                            throw new WindowManager.BadTokenException(
                                    "Unable to add window -- token " + attrs.token
                                    + " is not for an application");
                        case WindowManagerGlobal.ADD_APP_EXITING:
                            throw new WindowManager.BadTokenException(
                                    "Unable to add window -- app for token " + attrs.token
                                    + " is exiting");
                        case WindowManagerGlobal.ADD_DUPLICATE_ADD:
                            throw new WindowManager.BadTokenException(
                                    "Unable to add window -- window " + mWindow
                                    + " has already been added");
                        case WindowManagerGlobal.ADD_STARTING_NOT_NEEDED:
                            // Silently ignore -- we would have just removed it
                            // right away, anyway.
                            return;
                        case WindowManagerGlobal.ADD_MULTIPLE_SINGLETON:
                            throw new WindowManager.BadTokenException("Unable to add window "
                                    + mWindow + " -- another window of type "
                                    + mWindowAttributes.type + " already exists");
                        case WindowManagerGlobal.ADD_PERMISSION_DENIED:
                            throw new WindowManager.BadTokenException("Unable to add window "
                                    + mWindow + " -- permission denied for window type "
                                    + mWindowAttributes.type);
                        case WindowManagerGlobal.ADD_INVALID_DISPLAY:
                            throw new WindowManager.InvalidDisplayException("Unable to add window "
                                    + mWindow + " -- the specified display can not be found");
                        case WindowManagerGlobal.ADD_INVALID_TYPE:
                            throw new WindowManager.InvalidDisplayException("Unable to add window "
                                    + mWindow + " -- the specified window type "
                                    + mWindowAttributes.type + " is not valid");
                    }
                    throw new RuntimeException(
                            "Unable to add window -- unknown error code " + res);
                }
                ...
                view.assignParent(this);
                ...
            }
        }
    }
```

从上面代码可知，如果抛出异常，`view.assignParent(this)` 将未被调用。

至此，可将我的猜测总结为：当使用同一个 view 多次显示 toast 时，可能某一次添加失败，导致 view 留在 mViews 中，可是 view 的 parent 又因为添加失败而为空，所以 Toast#TN#handleShow 方法没有调用从 WindowManager 移除此 view 的代码，所以 WindowManagerGlobal#addView 被调用时，view 不在 mDyingViews 中，所以 crash 发生了。

可是这只是猜测，无法验证猜测是否正确。

\-\-\-\-\- 以下是 2019 年 1 月 11 日的更新 \-\-\-\-\-\-

现已发现复现第二种 crash (IllegalStateException: view has already been added to the window manager) 的方法，也就是如下的代码：

```java
public class MainActivity extends AppCompatActivity {
    private View toastLayout;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        toastLayout = LayoutInflater.from(this).inflate(R.layout.connection_toast, null);
        
        findViewById(R.id.btn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast toast = new Toast(MainActivity.this);
                toast.setView(toastLayout);
                toast.show();

                try {
                    Thread.sleep(1980);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

上面的代码，点击几次按钮来执行 onClick 方法，就可以复现 crash。复现思路是，先让前一个 toast 把 view 添加到 WindowManager，但要让它添加失败，然后第二次另一个 toast 再添加此 view，此次发生 crash，这个思路也是按照前面的猜想来的。首先，上面的代码是将同一个 view 添加到不同的 toast 对象去 show，当添加到第一个 toast 调用 show 方法后，主线程 sleep 1980 毫秒，这个时间是很微妙的，接近 2000 毫秒但是却略少。这个时间可以使得主线程醒来时，toast 的 token 即将失效。不能用更长的 sleep 时间是因为那样的话，主线程还在 sleep 中 token 已失效，token 的失效是在 NotificationManagerService 中产生的，失效后，NotificationManagerService 会处理 MESSAGE_DURATION_REACHED 消息，最终会跨进程调用到 Toast#TN#hide 方法，而这个方法会让我们 app 的主线程消息队列增加一个 HIDE 消息：

![](https://tao93.top/images/2019/01/11/1547217156.png)

如此一来，当主线程 sleep 结束执行 Toast#TN#handleShow 方法时，就会因消息队列已有 HIDE 消息而提前返回:

![](https://tao93.top/images/2019/01/11/1547217419.png)

既然都提前返回了，view 也就不会被第一个 toast 添加到 WindowManager，那么也就不符合我们的思路中「让第一个 toast 把 view 添加到 WindowManager 是发生异常」的想法。

所以，需要 1980 毫秒这样一个时间，这个时间使得主线程醒来执行到第一个 toast 的 Toast#TN#handleShow 时，token 还没失效，所以 handleShow 方法不会提前返回，所以 view 会继续往 WindowManager 添加，但是 20 毫秒不足以让这个添加顺利完成，相反，很可能添加时 token 失效了， 于是添加失败，发生第一种 crash 的 BadTokenException (Anddroid 8 以上此异常会被捕获，前文已描述)，这样就符合我们的思路了。下面截图证明了确实发生了 BadTokenException：

![](https://tao93.top/images/2019/01/11/1547217762.png)

至此，按照前面的思路，此 view 将会无 parent，但是却留在了 WindowManagerGlobal 的 mViews 中，却又不在 mDyingViews 中，于是，当再次按下按钮，执行另一个 toast 添加此 view 的代码时，WindowManagerGlobal#addView 中将发生 IllegalStateException，crash 也就复现了，截图为证：

![](https://tao93.top/images/2019/01/11/1547217909.png)

至此，toast crash 的分析算是有了一个比较完满的结尾。对于本文中 2019 年 1 月 11 日更新的部分，在此感谢刘成同学提供的帮助，他本来用来复现问题的方式是「先调一个 toast 的 show，sleep 三四秒，然后再用同一个 view 调另一个 toast 的 show」，这个方式因为前面讲的原因而无法复现 crash，但却给了我灵感，让我想到了 1980 毫秒这个时间，最终成功复现了 crash。
