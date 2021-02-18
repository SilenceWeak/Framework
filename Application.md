# 如何创建一个Appliction  
**注意点:**  
- [ ] 为何不能在Application的构造函数中使用getResource等函数： 因为这个时候Context上下文还没有准备好，Application在构造函数中通过Attach函数实现Context的赋值等操作
- [ ] 不能在其中执行耗时操作，由于此构造方法中调用了创建Application，丢到UI线程中去操作，如果此时添加了耗时操作，那么势必导致UI线程阻塞。
	同时，在该方法中还有很多不同组件的启动流程，在此处添加耗时操作的话，还将导致各种组件的启动流程受阻

会问到的题目：
Application的作用是什么
1. 初始化资源，App在启动时会在Application的oncreate方法中初始化一些全局资源，系统配置和三方SDK等

它的类继承关系以及生命周期

它的初始化原理
