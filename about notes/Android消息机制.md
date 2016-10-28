Handler 的作用是将一个任务切换到Handler所在的线程去执行。

#### ThreadLocal：
ThreadLocal并不是线程，它的作用是在每个线程中存储并提供数据，并Handler内部可以通过它来获得当前线程的Looper。
ThreadLocal是一个线程内部的数据存储类，可以在指定线程中存储数据。数据存储后，只有在指定线程中可以获取到存储的数据，其他线程则无法获取。
例子：
```
private ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<Boolean>();
	mBooleanThreadLocal .set(true);
	Log.d(TAG,"[Thread#Main]mBooleanThreadLocal = "+mBooleanThreadLocal.get());

new Thread("Thread#1"){
	public void run(){
		mBooleanThreadLocal .set(fasle);
		Log.d(TAG,"[Thread#1]mBooleanThreadLocal = "+mBooleanThreadLocal.get());
	}
}

new Thread("Thread#2"){
	public void run(){
		Log.d(TAG,"[Thread#2]mBooleanThreadLocal = "+mBooleanThreadLocal.get());
	}
}
```
结果：
```
[Thread#Main]mBooleanThreadLocal = true
[Thread#1]mBooleanThreadLocal = fasle
[Thread#2]mBooleanThreadLocal = null
```
#### Looper

对于looper，需要关注这几个方法：构造方法，prepare()，loop()。
构造方法：
```
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```
在构造方法里面，做了两件事，
一是创建一个消息队列，二是将线程对象指向了创建Looper的线程。

prepare()：
```
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```
首先判断当前线程有没有存在Looper，如果存在，则抛出异常，不存在则新建一个Looper并存储在ThreadLocal中。（这就解析了为什么一个线程只有一个Looper）

loop()：

```
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
        if (traceTag != 0) {
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
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```
loop() 方法搜寻获取当前现场绑定的 Looper 和与 Looper 绑定的消息队列，然后开启一个for的无限循环，在循环里面，通过 queue.next(); 不断指向消息队列下一个节点（也就是不断遍历消息队列），当找到消息后，调用 msg.target.dispatchMessage(msg); 来分发消息。最后通过   msg.recycleUnchecked(); 来将消息标记为正在使用。而 msg.target 其实就是 Handler 。
