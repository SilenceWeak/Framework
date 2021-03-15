# FrameWork之一：谈谈对Zygote的理解
[帖子传送门](https://www.jianshu.com/p/53d7e0475791)

### Zygote的作用是什么？
对于Zygote的作用实际上可以概括为以下两点：

* 创建SystemServer
* 孵化应用进程

### Zygote的启动过程？
* Zygote进程在Init进程启动过程中被以service服务的形式启动：
```
  service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
  class main
  socket zygote stream 660 root system 
```
* 调用app_main.cpp的main函数中的AppRuntime的start方法来启动Zygote进程

* 调用startVm函数来创建虚拟机，调用startReg函数为java虚拟机注册JNI方法

* 通过toSlashClassName找到ZygoteInit，通过GetStaticMethedID函数找到main方法然后调用，ZygoteInit的main方法是由Java语言编写的，当前的运行逻辑在Native中，这就需要JNI来调用Java，这样Zygote就从Native层进入了Java框架层。

* ZygoteInit的main方法的源码如下：

```
  不会吧不会吧，不会有人想让我当纯搬运工吧，自己死去帖子里看嗷
```

* 总结下来的流程：  

  * 通过registerServerSocket方法来创建一个Server端的socket，这个name为zygote的socket用于等待ActivityManagerService请求Zygote来创建新的应用程序进程

  * 预加载，预加载项如下：
  ```
  preloadClasses();
  preloadResources();
  preloadOpenGL();
  preloadSharedLibraries();
  preloadTextResources();
  WebViewFactory.prepareWebViewInZygote();
  ...
  ```

  * 以下是Android应用进程共享内存图：
    ![image](https://upload-images.jianshu.io/upload_images/2456775-dfe3e9badfda6977.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

    通过上图可以很容易理解在Zygote进程预加载系统资源后，然后通过它孵化出其他的虚拟机进程，进而共享虚拟机内存和框架层资源，这样大幅度提高应用程序的启动和运行速度。

  * 启动SystemServer进程

  * 执行runSelectLoop()方法等待消息去创建应用进程


### 工作流程图
![image](https://upload-images.jianshu.io/upload_images/2456775-4233abb95c2e3c01.png?imageMogr2/auto-orient/strip|imageView2/2/w/897/format/webp)
 

* **孵化应用进程这种事为什么不交给SystemServer来做，而专门设计一个Zygote？**  
  * 应用在启动的时候，往往伴随着需要加载很多不同的资源，如java虚拟机，加载各类系统资源等，这些操作都是极其耗时的。那么此时，就需要一个进程来提前把这些耗时的事情操作完，那么之后子进程fork之后就能直接拿到需要的资源，显著提高效率。而说回SystemServer，SystemServer里面跑着很多不同的系统服务，而通过systemserver来孵化进程的话，这些系统服务产生的对于应用进程来说不必要的资源也会继承到子进程中，引发安全问题的同时，还带来应用进程过分臃肿的问题。出于综合考虑，将systemserver进程和应用进程都会用到的资源加载操作放进一个专门的进程，也就是Zygote，再通过Zygote来分化出systemserver和应用进程。


* **Zygote的IPC通信机制为什么使用socket而不采用binder？**  
  * 因为Zygote fork子进程的时候需要单线程，而binder通讯是多线程。
  * 关于为什么子线程怕父进程Binder线程有锁，所以子进程的主线程一直在等待父进程中的子线程（从父进程拷贝进来的子进程）的资源，但是父进程中的子进程并没有被拷贝进来，造成死锁，故fork不允许存在多线程。
