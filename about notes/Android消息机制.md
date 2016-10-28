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
