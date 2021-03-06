# 如何创建一个Appliction  
[帖子传送门](https://blog.csdn.net/menghaocheng/article/details/104459365)

**注意点:**  
- [ ] 为何不能在Application的构造函数中使用getResource等函数： 因为这个时候Context上下文还没有准备好，Application在构造函数中通过Attach函数实现Context的赋值等操作
- [ ] 不能在其中执行耗时操作，由于此构造方法中调用了创建Application，丢到UI线程中去操作，如果此时添加了耗时操作，那么势必导致UI线程阻塞。
	同时，在该方法中还有很多不同组件的启动流程，在此处添加耗时操作的话，还将导致各种组件的启动流程受阻
	
	- [ ] 解释二： bindApplication后处理pending的组件，包括Activity、Service、Broadcast等，本来在启动这些组件的时候它的应用是没用启动的，现在应用进程启动好了，而且Application也初始化好了，就可以去处理这些待启动的组件了，这些组件的启动操作最终是要在应用进程执行的，比如Activity的生命周期是要在应用的UI线程中调用的。如果Application耗时太久，那么将会耽误应用的组件启动。
- [ ] 存在的一个问题：  
	1.Application使用静态变量时产生的一个bug
	![image](https://img-blog.csdnimg.cn/20200223144740870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21lbmdoYW9jaGVuZw==,size_16,color_FFFFFF,t_70)
	* Application中有一个静态变量name，MainActivity会去设置这个name，设置完之后马上跳转到TestActivity，Test读出name，正常情况是没什么问题的。但是如果这里把应用切到后台， 过了一段时间系统因为内存不足把应用杀掉了，再把应用切回来时，系统会重建这个应用，包括重建Application、恢复TestActivity，但这时候Application中的name是没有初始化的，TestActivity拿到的name就是null，这样就可能发生异常。



**会问到的题目：**
* Application的作用是什么  
	* 初始化资源，App在启动时会在Application的oncreate方法中初始化一些全局资源，系统配置和三方SDK等  
	* 数据共享，由于Application对象全局唯一，所以可以用来缓存一些全局变量，这些变量可以在任意地方使用，达到共享的目的
	* 监听App状态，锁屏开屏，锁屏开屏，退到后台回到前台，手机内存状态，横竖屏切换，Activity的生命周期，退出应用（不稳定），这些都可以通过Application监听。
	* 提供应用上下文

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
```
	class Application extend Context implement XXX{
	    final void attach(Context context) {
		attachBaseContext(context);
		mLoadedApk = ContextImpl.getImpl(context).mPackageInfo;
	    }
	}

	public class Context Wrapper extends Context{
	  Context mBase;
	  public ContextWrapper(Context base){
	      mBase = base;
	  }
	  protected void attachBaseContext(Context base){
	      if (mBase != null) {
		throw new IllegalStateException("Base context already set");
	      }
	      mBase = base;
	    }
	  }
```

* 它的初始化原理
```
	public static void main(String[] args){
	  Looper.prepareMainLooper();
	  ActivityThread thread = new ActivityThread;
	  //主要用于应用端向AMS打报告
	  thread.attach(false);
	  Looper.loop();
	  throw new RuntimeExceptiom("Main thread loop unexpectedly exited");
	}

	private void attach(){
	  //获取ActivityManager的的Binder对象
	  final IActivityManager mgr = ActivityManagerNative.getDefault();
	  try{
	    //
	    mgr.attachApplication(mAppThread);
	  }catch(RemoteException ex){
	    //Ignore
	  }
	}

	//跑在AMS端:
	public final void attachApplication(IApplicationThread thread){
	  synchronized(this){
	    attachApplicationLocked(thread, callingPid);
	  }
	
	//跑在AMS端:
	boolean attachApplicationLocked(IApplicationThread thread, ...){
	  ......
	  thread.bindApplication(...);
	  ......
	}
	
	//应用端：(binder线程）
	public final void bindApplication(...){
	  AppBindData data = new AppBindData();
	  ......
	  sendMessage(H.BIND_APPLICATION, data);
	}
	
	//应用端在主线程处理bindApplication的函数：
	private void handleBindApplication(AppBindData data){
	  //获取描述应用安装包的信息
	  data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);
	  //创建Application对象
	  Application app = data.info.makeApplication(...);
	  //调用app.onCreate
	  mInstrumentation.callApplicationOnCreate(app);
	}

	public Application makeApplication(...){
	  if(mApplication != null){
	    return mApplication;
	  }
	  ContextImpl appContext = ContextImpl.createAppContext(...);
	  app = mActivityThread.mInstrumentation.newApplication(...);
	  return app;
	}

	Application newApplicatoin(ClassLoader cl, String className, Context context){
	  //加载一个类
	  return newApplication(cl.loadClass(className), context);
	}

	Static Application newApplication(Class<?> clazz, Context context){
	  //然后调用这个类的构造函数对创建一个Application对象
	  Application app = (Application)clazz.newInstance();
	  //最后把context附给app
	  app.attach(context);
	  return app;
	}

	//虽然Application是一个Context但它只是一个空壳，真正干活的是一个名为mBase Context成员
	final void attach(Context context){
	  attachBaseContext(context);  //给mBase赋值 必须在attachBaseContext后才能使用context中的东西
	}
```  
  * 其中的关键方法：
  	* *newApplication()*
  	* *application.attachBaseContext()*
  	* *application.onCreate()*
