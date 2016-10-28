Android中扮演线程角色，除了Thread之外，AsyncTask和IntentService也是。  
同时HandlerThread也是一种特殊的线程。

对于AsyncTask，底层用到了线程池和 Handler。  
对于IntentService 和 HandlerThread，底层直接使用线程。

#### HandlerThread
HandlerThread是一种具有消息循环的线程，在它内部可以使用Handler。

#### IntentService
IntentService是一个服务，内部采用HandlerThread来执行任务，当任务执行完毕后IntentService会自动退出，它不容易被系统杀死从而可以尽量保证任务的执行，这是它的优点。

#### AsyncTask
AsyncTask封装了线程池和Handler。  
AsyncTask有两个线程池：SerialExecutor和THREAD_POOL_EXECUTOR。前者是用于任务的排队，默认是串行的线程池，后者用于真正的执行任务。

#### SerialExecutor
```
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

private static class SerialExecutor implements Executor {
   final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
   Runnable mActive;

   public synchronized void execute(final Runnable r) {
       mTasks.offer(new Runnable() {
           public void run() {
               try {
                   r.run();
               } finally {
                   scheduleNext();
               }
           }
       });
       if (mActive == null) {
           scheduleNext();
       }
   }

   protected synchronized void scheduleNext() {
       if ((mActive = mTasks.poll()) != null) {
           THREAD_POOL_EXECUTOR.execute(mActive);
       }
   }
}
```

THREAD_POOL_EXECUTOR：
```
private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
// We want at least 2 threads and at most 4 threads in the core pool,
// preferring to have 1 less than the CPU count to avoid saturating
// the CPU with background work
private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
private static final int KEEP_ALIVE_SECONDS = 30;

private static final ThreadFactory sThreadFactory = new ThreadFactory() {
    private final AtomicInteger mCount = new AtomicInteger(1);

    public Thread newThread(Runnable r) {
        return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
    }
};

private static final BlockingQueue<Runnable> sPoolWorkQueue =
        new LinkedBlockingQueue<Runnable>(128);

/**
 * An {@link Executor} that can be used to execute tasks in parallel.
 */
public static final Executor THREAD_POOL_EXECUTOR;

static {
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
            CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
            sPoolWorkQueue, sThreadFactory);
    threadPoolExecutor.allowCoreThreadTimeOut(true);
    THREAD_POOL_EXECUTOR = threadPoolExecutor;
}
```

#### InternalHandler 
AsyncTask 还有一个Handler，叫InternalHandler，用于将执行环境从线程池切换到主线程。AsyncTask内部就是通过InternalHandler来发送任务执行的进度以及执行结束等消息。

```
private static class InternalHandler extends Handler {
   public InternalHandler() {
       super(Looper.getMainLooper());
   }

   @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
   @Override
   public void handleMessage(Message msg) {
       AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
       switch (msg.what) {
           case MESSAGE_POST_RESULT:
               // There is only one result
               result.mTask.finish(result.mData[0]);
               break;
           case MESSAGE_POST_PROGRESS:
               result.mTask.onProgressUpdate(result.mData);
               break;
       }
   }
}
```

>
AsyncTask排队执行过程：系统先把参数Params封装为FutureTask对象，它相当于Runnable，接着FutureTask交给SerialExcutor的execute方法，它先把FutureTask插入到任务队列tasks中，如果这个时候没有正在活动的AsyncTask任务，那么就会执行下一个AsyncTask任务，同时当一个AsyncTask任务执行完毕之后，AsyncTask会继续执行其他任务直到所有任务都被执行为止。




