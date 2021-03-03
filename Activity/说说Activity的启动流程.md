# 说说Activity的启动流程
* 启动Activity要经历哪些生命周期回调
* 冷启动大概流程，涉及哪些组件，通信过程是怎样的
* Activity启动过程中，生命周期回调的原理

### 启动过程中向AMS发起请求的过程
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/startActivity.jpg)
* 其中重要的部分——startActivity的主要实现为:
  ```
  startSpecificActivityLocked(ActivityRecord r,...){
    ProcessRecord app = mService.getProcessRecordLocked(r.processName,...); //获取Activity运行的线程
    
    if(app != null && app.thread != null){ //如果Activity运行的线程是存在的，且向AMS注册过ActivityThread，这时再真正启动Activity的启动流程
      realStartActivityLocked(r,app,...);
      return;
    }
    
    mService.startProcessLocked(r.processName,...); //Activity运行的线程还没被创建，那么先创建线程
  }
  

  private final void startProcessLocked(ProcessRecord app,) {
    if (entryPoint == null) entryPoint = " android.app.ActivityThread"; //应用进程启动后，应用运行入口函数的Java类
    Process.ProcessStartResult startResult = Process.start(entryPoint, );　//向zygote发送启动应用进程的请求

    synchronized (mPidsSelfLocked) {
      mPidsSelfLocked.put(startResult.pid, app);
      Message msg = mHandler.obtainMessage(PROC_START_TIMEOUT_MSG);
      msg.obj = app;
      mHandler.sendMessageDelayed(msg, PROC_START_TIMEOUT); //如果超时没有相应AMS报告的话，那么应用相关的所有信息就会被AMS所清理
    }
  }

  public static final ProcessStartResult start(final String processClass,) {
    return startViaZygote(processClass,); //AMS端处理应用进程启动
  }
  
  ProcessStartResult startViaZygote (final String processClass,) { //AMS端的处理函数
    return zygoteSendArgsAndGetResult( //通过zygote获取参数并返回结果
    openZygoteSocketIfNeeded(abi), argsForZygote); //开启zygote的socket
  }

  boolean runOnce() { //zygote端的处理
    String[] args = readArgumentList(); //读取AMS发送过来的参数列表
    int pid = Zygote.forkAndSpecialize(...);

    if(pid == 0) {
      handleChildProc(parsedArgs, ); //启动binder机制，执行应用程序的入口函数等
      return true;
    }else {
      return handleParentProc(pid, );
    } 
  }
  ```
* 在应用程序中的入口函数
  ```
  public static void main(String[] args) {
    Looper.prepareMainLooper();
    
    ActivityThread thread = new ActivityThread();
    thread.attach(false); //向AMS报告，在startProcessLocked中有实现，如果10s内不向AMS报告就会被清理
    Looper.loop();
    
    throw new RuntimeException("Main thread loop unexpectedly exited");
  }
  
  private void attach (boolean system) {
    IActivityManager mgr = ActivityManagerNative.getDefault(); //拿到AMS的binder对象
    mgr.attachApplication(mAppThread); // mAppThread:应用端的binder对象，类型是ApplicationThread
  }
  ```
* 创建Activity的流程
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/ProcessOfCreateActivity.jpg)
```
boolean attachApplicationLocked (lApplicationThread thread, int pid) {
  ProcessRecord app = mPidsSelfLocked.get(pid);
  thread.bindApplication(...); //thread是应用端注册到AMS的binder对象，类型是ApplicationThread，binderApplication是做一些初始化的工作
  
  mStackSupervisor.attachApplicationLocked(app); //启动挂起的组件，由于应用进程被创建所以需要将其启动
  ...
  mServices.attachApplicationLocked(app, processName);
  ...
  sendPendingBroadcastsLocked(app);
}

boolean attachApplicationLocked(ProcessRecord app) { //启动Activity的组件
  ...
  //stack:mFocusedStack；AMS中有两种Stack，LaunchStack：与桌面相关的stack。还有与桌面不相关的mFocusedStack
  //去stack中找到最近的task去占领activity，即，应用还未启动的时候这些数据结构都准备好了，
  //只要应用程序一启动，那么立马就会把准备好的东西塞进去
  ActivityRecord hr = stack.topRunningActivityLocked(null);
  realStartActivityLockedhr, app, true, true);
  ...
}
```
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/RealStartActivityLocked.jpg)
```
@Override 
public final void scheduleLaunchActivity (Intent intent, IBinder token, ) {
  ActivityClientRecord r = new ActivityClientRecord(); //封装一个消息丢到主线程
  sendMessage(H.L AUNCH ACTIVITY, r); // ---->   mH.sendMessage(msg);
}

//主线程处理逻辑
final ActivityClientRecord r = (ActivityClientRecord) msg.obj;
//生成一个LoadedApk类对象，其中存储着APK的各类信息，之后启动Activity的时候，需要用到classloader来加载Activity类
r.packagelnfo = getPackagelnfoNoCheck...); 
handleLaunchActivity(r, null);

private void handleLaunchActivity (ActivityClientRecord r...){
  Activity a = performLaunchActivity(r, customIntent);
  if(a!= null) l
  handleResumeActivity(r.token, fale,..);
}

private Activity performLaunchActivity (ActivityClientRecord r, ...) {
  Activity activity = mInstrumentation.newAcivity(...); //创建一个新Activity
  Application app = r.packagelnfo.makeApplication(false, mInstrumentation); //获取Application:是之前已经创建好的Application
  Context appContext = createBaseContextForActivity(r, activity); //创建上下文ContextImpl
  activity.attach(appContext, ...); //attach 应用的上下文
  mInstrumentation.callActivityOnCreate(activity, r.state); //Activity的生命周期回调,onCreate()
  activity.performStart(); //Activity.onStart()
  return activity;
}
```
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/HandleOnResume.jpg)

* 应用端Activity启动的流程
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/应用端Activity启动的流程.jpg)

* Activity的启动流程
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/Activity的启动流程.jpg)

###  启动Activity要向AMS发起binder调用
###  Activity所在的进程是怎么被启动的：
   * 先向AMS发起请求，如果进程是存在的那么直接调用realStartActivityLocked方法，如果不是则调起zygote，同时10s后如果没有收到进程向AMS报告，那么就把它清除掉
   * 在zygote中，fork出一个新的应用进程，同时调用AMS中传过来的参数，启动应用进程的入口函数
   * 在进程的入口函数中，调用attach方法，向AMS进行报告，报告完毕后，AMS会拿到进程的binder对象，在attachApplication中对其作初始化处理，同时启动被挂起的各种组件
###  进程端Activity是怎么初始化的，Activity的生命周期是怎么回调的
   * 在上面说到的启动组件中，有一个方法会启动Activity。
   * 该方法会发送一个消息至主线程，主线程接收到消息之后，便开始启动Activity
   * 首先LaunchActivity，先创建一个新Activity，同时attach application 和 context等，同时在此处还会callActivityOnCreate等，也就是回调其onCreate，onStart等生命周期方法
   * 接着会performResume（）
