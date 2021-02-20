# Android系统的启动
## 开机启动init进程
[帖子传送门](https://blog.csdn.net/z240336124/article/details/98472607)
*  /system/core/init/Init.cpp 的 main()：
  ```
  代码去帖子里看，大概的做的事情是
  1.创建目录,挂载分区
  2.析启动脚本，
  3.启动解析的服务，
  4.守护解析的服务。init.rc 文件是 Android 系统的重要配置文件，位于 /system/core/rootdir/init.rc
  ```
  
  * init.rc
  ```
  // 导入其它的一些脚本
  import /init.environ.rc
  import /init.usb.rc
  // 当前硬件版本的脚本
  import /init.${ro.hardware}.rc
  import /init.${ro.zygote}.rc
  import /init.trace.rc

  on early-init
  ...
  on init
  ...
  // 服务 服务名称 执行文件路径 执行参数
  // 有几个重要的服务
  service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
  service servicemanager /system/bin/servicemanager
  service surfaceflinger /system/bin/surfaceflinger
  service media /system/bin/mediaserver
  service installd /system/bin/installd
  ```
* Zygote是怎么启动的
  * init进程fork出zygote进程
  * 启动虚拟机，注册jni函数：为了接下来切换到Java作准备
  * 预加载系统资源：系统主题和一些常用的类
  * 启动SystemServer
  * 进入socket loop：进入Socket Loop中不断接收处理消息

* Zygote的工作流程
  * 在Socket Loop中接收到需要处理的消息时调用RunOnce（）函数
  ```
  runonce(){
    args = readargumentList();
    int pid  = Zygote.forAndSpecialize(...);
    if(pid == 0){
      handleChildProc(ParsedArgs,...);
      return true;
    }else{
      return handleParentProc(pid,...);
    }
  }
  ```

* SystemServer是怎么启动的
  ```
  startSystemServer(...){
    args[] = {
      ...
      "com.android.server.SystemServer",
    };
    
    int pid = Zygote.forkSystemServer(...);
    
    if(pid == 0){
      handleSystemServerProcess(parsedArgs);
    }
    
    return true;
  }
  
  handleSystemServerProcess(parsedArgs){
    RuntimeInit.zygoteInit(...);
  }
  
  zygoteInit(...){
    commonInit();
    nativeZygoteInit();
    applicationInit(argv,...);
  }
  
  //在nativeZygoteInit中主要用来启动Binder机制
  onZygoteInit(){
  
  }
  ```

* 系统服务是怎么启动的
