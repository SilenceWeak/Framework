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
    if (entryPoint == null) entryPoint = " android.app.ActivityThread"; //获取入口函数的类
    Process.ProcessStartResult startResult = Process.start(entryPoint, );　//

    synchronized (mPidsSelfLocked) {
      mPidsSelfLocked.put(startResult.pid, app);
      Message msg = mHandler.obtainMessage(PROC_START_TIMEOUT_MSG);
      msg.obj = app;
      mHandler.sendMessageDelayed(msg, PROC_START_TIMEOUT);
    }
  }

  public static final ProcessStartResult start(final String processClass,) {
    return startViaZygote(processClass,);
  }
  ```

