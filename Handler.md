### Android消息机制
#### Handler
一个Handler容许你发送和处理与一个线程MessageQueue关联的Message和Runnable对象。
每一个Handler实例关联一个单独的线程和这个线程的MessageQueue.
当你创建一个Handler时，他会绑定创建他的线程和其中的MessageQueue，从这一刻起，他就会发送message和runnables到MessageQueue，并且从MessageQueue中取出他们来执行。
Handler有两个主要的用法：
（1）安排message和runnable在将来的某一点被执行；
（2）让一个行为在一个不同的线程中被执行。
安排message是靠下面几个函数实现的：
post(),postAtTime(),postDelayed(),
sendEmptyMessage(),sendMessage(),sendMessageAtTime(),sendMessageDelayed().
你可以用post系列方法操作Runnable入队列，使用send系列方法操作Message入队列，这个Message是由Handler的handleMessage()方法处理的。

你可以创建一个子线程并且通过handler与你的主线程进行沟通



- 比较有意思的是Handler将成员变量声明在了类的最末尾
final Looper mLooper;
final MessageQueue mQueue;
final Callback mCallback;
final boolean mAsynchronous;
IMessenger mMessenger;//这个成员变量和相对应的方法，怀疑是通过IPC接受系统消息的方法，暂时不讨论

- 构造函数 主要工作就是为成员变量赋值了，
```java 
public Handler(Callback callback, boolean async) {
    //检测是否有内存泄漏危险，用log提示开发者。
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }
    //如果looper为null的话会抛出异常
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
/**
The looper, must not be null.
*/
public Handler(Looper looper, Callback callback, boolean async) {
    //looper为空的话不能正常处理消息
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```
- 获取Message方法,这些方法的实现全是调用Message相应的方法。
```java
 public final Message obtainMessage()
{
    return Message.obtain(this);
}
public final Message obtainMessage(int what)
{
    return Message.obtain(this, what);
}v
public final Message obtainMessage(int what, Object obj)
{
    return Message.obtain(this, what, obj);
}
public final Message obtainMessage(int what, int arg1, int arg2)
{
    return Message.obtain(this, what, arg1, arg2);
}

public final Message obtainMessage(int what, int arg1, int arg2, Object obj)
{
    return Message.obtain(this, what, arg1, arg2, obj);
}

```
- 发送消息方法,post方法全部是调用相应的send方法实现的。最后send方法调用的就是sendMessageAtTime()方法，其中调用的是enqueueMessge()方法。在这个方法中设置了message的target=this,设置了message是否异步：msg.setAsynchronous(true);
最后调用MessageQueue的enqueueMessage（）方法将Message入队列。
```java
public final boolean post(Runnable r)
{
    return  sendMessageDelayed(getPostMessage(r), 0);
}

public final boolean postAtTime(Runnable r, long uptimeMillis)
{
    return sendMessageAtTime(getPostMessage(r), uptimeMillis);
}
public final boolean postAtTime(Runnable r, Object token, long uptimeMillis)
{
    return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
}
public final boolean postDelayed(Runnable r, long delayMillis)
{
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
public final boolean postAtFrontOfQueue(Runnable r)
{
    return sendMessageAtFrontOfQueue(getPostMessage(r));
}


public final boolean sendMessage(Message msg)
{
    return sendMessageDelayed(msg, 0);
}
public final boolean sendEmptyMessage(int what)
{
    return sendEmptyMessageDelayed(what, 0);
}
public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}
public final boolean sendMessageDelayed(Message msg, long delayMillis)
{
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    //其中SystemClock.uptimeMillis()是获取系统开机到现在的毫秒数，没有用System.currentTimeMillis（）是因为这个方法可以通过System.setCurrentTimeMillis修改。
}
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

public final boolean sendMessageAtFrontOfQueue(Message msg) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
            this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, 0);
}

private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```
其中SystemClock.uptimeMillis()是获取系统开机到现在的毫秒数，没有用System.currentTimeMillis（）是因为这个方法可以通过System.setCurrentTimeMillis修改。sendMessageAtFrontOfQueue(Message msg)设置时间为0,达到插入MessageQueue最前边的目的。sendMessage(Message msg) 调用的是sendMessageDelay(msg , 0),又调用森达MessageAtTime(msg,SystemClock.uptimeMillis()+delayMillis),就是说设置时间为当前时间


- 删除队列中还没有执行的runnable,其实现是调用MessageQueue中的相应方法实现的
```java 
 /**
* Remove any pending posts of Runnable r that are in the message queue.
*/
public final void removeCallbacks(Runnable r)
{
    mQueue.removeMessages(this, r, null);
}

/**
* Remove any pending posts of Runnable <var>r</var> with Object
* <var>token</var> that are in the message queue.  If <var>token</var> is null,
* all callbacks will be removed.
*/
public final void removeCallbacks(Runnable r, Object token)
{
    mQueue.removeMessages(this, r, token);
}

public final void removeMessages(int what) {
    mQueue.removeMessages(this, what, null);
}
public final void removeMessages(int what, Object object) {
    mQueue.removeMessages(this, what, object);
}
public final void removeCallbacksAndMessages(Object token) {
     mQueue.removeCallbacksAndMessages(this, token);
}
```
- 处理分发来的message
 ```java
/**
    * Handle system messages here.
    */
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```
   


有待深入理解的方法：
public final boolean runWithScissors(final Runnable r, long timeout) 

#### Message
这个类定义了一个包含描述和任意数据对象的message,这个message可以被发送到Hander.这个类包含两个额外的int型的属性和一个额外的object型的属性，通过他们，在许多情况下，你就不必自行分配了。

- 获取Message对象的方法
```java
 /**
    * Return a new Message instance from the global pool. Allows us to
    * avoid allocating new objects in many cases.
    */
public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // clear in-use flag
            sPoolSize--;
            return m;
        }
    }
    return new Message();
}

/**
    * Same as {@link #obtain()}, but copies the values of an existing
    * message (including its target) into the new one.
    * @param orig Original message to copy.
    * @return A Message object from the global pool.
    */
public static Message obtain(Message orig) {
    Message m = obtain();
    m.what = orig.what;
    m.arg1 = orig.arg1;
    m.arg2 = orig.arg2;
    m.obj = orig.obj;
    m.replyTo = orig.replyTo;
    m.sendingUid = orig.sendingUid;
    if (orig.data != null) {
        m.data = new Bundle(orig.data);
    }
    m.target = orig.target;
    m.callback = orig.callback;

    return m;
}

/**
    * Same as {@link #obtain()}, but sets the value for the <em>target</em> member on the Message returned.
    * @param h  Handler to assign to the returned Message object's <em>target</em> member.
    * @return A Message object from the global pool.
    */
public static Message obtain(Handler h) {
    Message m = obtain();
    m.target = h;

    return m;
}

 /**
    * Same as {@link #obtain(Handler)}, but assigns a callback Runnable on
    * the Message that is returned.
    * @param h  Handler to assign to the returned Message object's <em>target</em> member.
    * @param callback Runnable that will execute when the message is handled.
    * @return A Message object from the global pool.
    */
public static Message obtain(Handler h, Runnable callback) {
    Message m = obtain();
    m.target = h;
    m.callback = callback;

    return m;
}

```
  
#### MessageQueue
入队列
```java
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}

```


出队列
```java

Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }

        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }

            // If first time idle, then get the number of idlers to run.
            // Idle handles only run if the queue is empty or if the first message
            // in the queue (possibly a barrier) is due to be handled in the future.
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // No idle handlers to run.  Loop and wait some more.
                mBlocked = true;
                continue;
            }

            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }

        // Run the idle handlers.
        // We only ever reach this code block during the first iteration.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler

            boolean keep = false;
            try {
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }

            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;

        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}

```
无限循环，先拿当前时间，然后从链表中去取message,判断message.when和当前时间的大小，如果message.when大于当前时间，则说明这个message还没有到执行的时候，所以就是计算出下次唤醒的时间。

#### Looper
```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}

private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}


 /**
    * Run the message queue in this thread. Be sure to call
    * {@link #quit()} to end the loop.
    */
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        try {
            msg.target.dispatchMessage(msg);
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(nevwIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```
Looper的构造函数就是mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
  初始化参数
  prepare()就是将当前的looper存到threadlocal中
  
#### ThreadLocal
内部类：static class ThreadLocalMap {}


Message复用问题
队列中的mQuitting问题

#### ActivityThread



1，自己独立项目
2，技术博客
3，学新技术的途径，自主学习能力
4，11点上班不打卡
5，cto 追求前沿技术
6，平时阅读源码的习惯
7，外国一手资料
8，国内外技术大牛讨论