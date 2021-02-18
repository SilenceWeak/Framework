# 如何创建一个Appliction  
**注意点:**  
- [ ] 为何不能在Application的构造函数中使用getResource等函数： 因为这个时候Context上下文还没有准备好，Application在构造函数中通过Attach函数实现Context的赋值等操作
- [ ] 不能在其中执行耗时操作，由于此构造方法中调用了创建Application，丢到UI线程中去操作，如果此时添加了耗时操作，那么势必导致UI线程阻塞。
	同时，在该方法中还有很多不同组件的启动流程，在此处添加耗时操作的话，还将导致各种组件的启动流程受阻

**会问到的题目：**
* Application的作用是什么  
	* 初始化资源，App在启动时会在Application的oncreate方法中初始化一些全局资源，系统配置和三方SDK等  
	* 数据共享，由于Application对象全局唯一，所以可以用来缓存一些全局变量，这些变量可以在任意地方使用，达到共享的目的
	* 监听App状态，锁屏开屏，锁屏开屏，退到后台回到前台，手机内存状态，横竖屏切换，Activity的生命周期，退出应用（不稳定），这些都可以通过Application监听。

* Application重要的方法  
	* onCreate():在应用启动时调用，可以在里面初始化系统资源和一些三方SDK
	* attachBaseContext(Context base)：该方法传入一个base参数，系统调用该方法，把ContextImpl对象传到该方法中，之后ContextWrapper中所有的方法都委托给该ComtextImpl对象去实现。
```
	//在调用attachBaseContext方法之前，Application中的很多方法都不能用，例如getPackageName()，不然会报空指针。  
	protected void attachBaseContext(Context base) {
	     if (mBase != null) {
		 throw new IllegalStateException("Base context already set");
	     }
	     mBase = base;
	 }
```
```
  //Application构造方法 -> attachBaseContext() -> onCreate()，如果想把初始化时机提前到极致，可以如下操作：
	public class MyApplication extends Application {  

	    @Override  
	    protected void attachBaseContext(Context base) {  
		// 在这里调用Context的方法会崩溃  
		super.attachBaseContext(base);  
		// 在这里可以正常调用Context的方法  
	    }  
	}
```

* 它的类继承关系以及生命周期

* 它的初始化原理
