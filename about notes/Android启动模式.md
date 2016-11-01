四种启动模式：standard、 singleTop、 singleTask、 singleInstance  

#### standard：  
standard 是默认的启动模式，是一种多实例模式，注意一点，如果用ApplicationContext去启动standard模式的Activity会报错，原因是非Activity类型的Context并没有所谓的任务栈。

#### singleTop：  
singleTop 是栈顶复用模式，如果Activity位于任务栈的栈顶，那么它不会被重新创建，同时它的 onNewIntent() 方法会被调用。

并且它的 onCreate() 和 onStart() 方法不会被调用，因为它并没有发生改变。

假设 ABCD ，A在栈的底部，如果这时候再次启动 D，那么情况仍然是 ABCD。

#### singleTask：
singleTask 是栈内复用模式，这是一种单例模式，只要 Activity 在一个栈中存在，就不重新创建实例，与 singleTop 一起可以理解为一个是栈顶，一个是整个栈，也会调用 onNewIntent() 方法。

具体来说，比如 Activity A，系统会先查找是否存在A 所需要的任务栈，如果没有，要先创建一个任务栈，并把 A 实例化后放进去，如果存在，那要看任务栈中是否存在 A 的实例，存在则把 A 调到栈顶并调用 onNewIntent() 方法，不存在则创建 A 的实例并压入栈。    

栈 S1 的情况是 ABC，如果 D 所需的栈是 S2，这时需要创建 S2 并把 D 实例化后压入 S2。  
栈 S1 的情况是 ABC，如果 D 所需的栈是 S1，这是系统会直接实例化 D 后压入S1。  
栈 S1 的情况是 ADBC，且 D 所需的栈是 S1，这时根据复用原则，D 不会被实例化，而是会把 D 调到栈顶，调用 onNewIntent() 方法，并把 BC 清除掉，这是的栈内情况变成 AD。  

#### singleInstance：
singleInstance 可以理解成是一种加强的 singleTask 模式。它有 singleTask 模式的所以特征。

那么加强的一点是，当 A 启动后，系统会为它创建一个任务栈，并把A 单独在这个栈中，后续的请求均不会创建新的 A，除非系统把这个栈给销毁了。
