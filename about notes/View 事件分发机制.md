事件分发过程主要由三个方法完成：
```
public boolean dispatchTouchEvent(MotionEvent event)
public boolean onInterceptTouchEvent(MotionEvent ev)
public boolean onTouchEvent(MotionEvent event)
```
onInterceptTouchEvent 和 onTouchEvent 方法在 dispatchTouchEvent 方法内部调用。

#### dispatchTouchEvent ：
dispatchTouchEvent 方法用于事件分发，只要事件能传给View，那么这个方法  一定会调用。表示是否消耗当前事件。

dispatchTouchEvent 方法的返回结果受当前 View 的 onTouchEvent 方法和下级 View 的 dispatchTouchEvent  方法影响。

#### onInterceptTouchEvent（ViewGroup）
onInterceptTouchEvent 方法表示是否拦截当前事件。

1、如果当前 View 拦截了事件（返回true），那么在同一个事件序列中，此方法不会再调用，接着会调用onTouchEvent 方法处理事件。
2、如果不拦截，则当前事件会继续传给它的子元素，接着子元素的 dispatchTouchEvent 方法会调用，如此反复知道事件被最终处理。


同一个事件序列是指从手指触摸屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列以 down 事件开始，中间含有数量不定的 move 事件，最终以 up 事件结束。

#### onTouchEvent 
onTouchEvent 方法表示是否消耗当前事件。如果不消耗（返回false），则在同一个事件序列中，当前 View 无法再次接收到事件。

三者关系：
```
public boolean dispatchTouchEvent(MotionEvent event) {
     boolean consume = false; //是否拦截
     if (onInterceptTouchEvent(event)){
         consume = onTouchEvent(event);
     }else {
         consume = child.onTouchEvent(event);
     }
     return  consume;
 }
```
