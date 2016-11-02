 Activity生命周期：onCreate onStart onResume onPause onStop onDestory  
 (1) 启动Activity: onCreate onStart onResume  
 (2) Activity退居后台：onPause onStop   
 (3) Activity返回前台：onRestart onStart onResume  
 (4) Activity退居后台，且内存不足：onPause onStop onDestory  
 (5) 再次回到：onCreate onStart onResume  
 (6) 锁屏：onPause onStop  
 (7) 解锁：onRestart onStart onResume  
 (8) Home键：onPause onStop  

 Fragment生命周期：
 onAttach   
 onCreate   onCreateView   onVieCreate   onActivityCreate  
 onStart   onResume   onPause  onStop  onDestoryView   onDestory  
 onDetach  

#### Activity异常退出该如何处理

使用 onSaveInstanceState() 回调方法保存临时数据和状态，这个方法一定会在活动被回收之前调用。  
其onSaveInstanceState方法会在什么时候被执行：  
1、当用户按下HOME键时。  
2、长按HOME键，选择运行其他的程序时。  
3、按下电源按键（关闭屏幕显示）时。  
4、从activity A中启动一个新的activity时。  
5、屏幕方向切换时，例如从竖屏切换到横屏时。

即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用。

onSaveInstanceState() 如果被调用，这个方法会在 onStop() 前被触发，但系统并不保证是否在 onPause() 之前或者之后触发。

onRestoreInstanceState：  
onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用。  
onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了，  

而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用。  

例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的onRestoreInstanceState方法不会被执行

onRestoreInstanceState 在 onstart 之后执行。

例子：
```
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
        savedInstanceState.putBoolean("MyBoolean", true);
        savedInstanceState.putDouble("myDouble", 1.9);
        savedInstanceState.putInt("MyInt", 1);
        savedInstanceState.putString("MyString", "Welcome back to Android");
        // etc.
        super.onSaveInstanceState(savedInstanceState);
}

@Override
public void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);

        boolean myBoolean = savedInstanceState.getBoolean("MyBoolean");
        double myDouble = savedInstanceState.getDouble("myDouble");
        int myInt = savedInstanceState.getInt("MyInt");
        String myString = savedInstanceState.getString("MyString");
}
```

收集自：[Android基础知识](https://github.com/GeniusVJR/LearningNotes/blob/master/Part1/Android/Android基础知识.md)
