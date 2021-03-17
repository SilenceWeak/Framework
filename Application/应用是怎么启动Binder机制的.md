## 什么时候支持binder机制的呢？

AMS向Zygote发起请求，Zygote Fork线程的时候，在子线程会handleChildProc()，而该方法的主要做的是  
1、CommonInit()  
2、NativeInit()  
3、...  
而在NativeInit中，会拿到ProcessState，在ProcessState中，就会完成启动Binder机制的流程
**即，在进程启动之后的初始化中启动的**

**怎么启用Binder机制  
1、打开binder驱动  
2、映射内存，分配缓冲区  
3、注册binder线程  
4、进入binder loop**

为什么不能从Zygote继承Binder机制呢？因为Zygote为了保护自己的安全性，并没有使用binder，而是采用的Socket
