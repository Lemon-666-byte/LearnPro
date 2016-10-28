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

关于 OnTouchListener 、onTouchEvent、OnClickListener 的优先级：

当一个View设置了OnTouchListener，那么它的 onTouch 方法就会回调

（1）当 onTouch 方法返回 true：onTouchEvent 方法不会被调用

（2）当 onTouch 方法返回 false：onTouchEvent 方法被调用

所以 OnTouchListener 优先级比 onTouchEvent 要高。

在 onTouchEvent 方法中，如果当前设置的有 OnClickListener，那么它的 onClick 方法会被调用。

所以 OnClickListener 优先级最低。

#### 一些结论
（1）某个 View 一旦开始处理事件，如果它不消耗 ACTION_DOWN 事件（onTouchEvent 返回了 fasle），那么同一个事件序列中的其他时间都不会再交给它来处理，并且事件将重新交由它的父元素去处理，即父元素的 onTouchEvent 会被调用。

意思就是事件一旦交给一个 View 处理，那么它就必须消耗掉，否则同一事件序列中剩下的事件就不再交给它来处理了。

（2）如果 View 不消耗除 ACTION_DOWN 以外的其他事件，那么这个点击事件就会消失，此时父元素的 onTouchEvent 并不会被调用，并且当前 View 可以持续接收到后续的事件，最终这些消失的点击事件会传递给 Activity 处理。

（3）ViewGroup 默认不拦截任何时间，因为源码中 ViewGroup 的 onInterceptTouchEvent 默认返回 fasle。

（4）View 没有 onInterceptTouchEvent 方法，一旦有点击事件传递给它，那么它的 onTouchEvent 方法就会被调用。

（5）View 的 onTouchEvent 默认都会消耗事件（返回 true），除非它是不可点击的（clickable 和 longClickable 同时为 false）。

（6）View 的 longClickable 属性默认都为 false，clickable 属性分情况，比如 Button 默认为 true，TextView 默认为 false。

（7）View 的 enable 属性不影响 onTouchEvent 的默认返回值。哪怕一个 View 是 disable 状态的，只要它的 clickable 或者 longClickable 有一个为 true，那么它的 onTouchEvent 就返回 true。

（8）onClick 会发生的前提是当前 View 是可点击的。并且它收到了 down 和 up 的事件。

（9）事件传递过程是由外到内的，即事件总是先传递给父元素，然后再由父元素分发给子 View。通过 requestDisallowInterceptTouchEvent 方法可以在子元素中干预父元素的事件分发过程，但是 ACTION_DOWN 事件除外。



