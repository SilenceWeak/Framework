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
