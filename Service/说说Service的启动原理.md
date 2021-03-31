# Service的启动原理

![image](https://user-images.githubusercontent.com/32014204/113102717-fa476300-9230-11eb-99ee-34721a5baa8c.png)

* 启动流程与Activity类似，先向AMS发起请求，AMS做出一系列操作后，来到判断其进程是否**启动**以及其进程是否**就绪**的流程
![image](https://user-images.githubusercontent.com/32014204/113103698-316a4400-9232-11eb-8b34-cae0b7582436.png)

#### 一、在应用端的进程被创建后，会发起attachApplication，其中的方法处理会启动PendingService，并开始真正的启动Service
![image](https://user-images.githubusercontent.com/32014204/113104604-56ab8200-9233-11eb-96fc-d02531a2845b.png)  
  
* 其中scheduleCreateService()是AMS端向应用端发起调用，执行Service的启动流程，以及onCreate()等回调方法
* sendServiceArgsLocked是用来启动ServiceStartCommon()的方法，该方法会真正的启动Service的工作  
  这里sendServiceArgsLocked的流程大概是：
  1. 调用应用端的scheduleServiceArgs
  2. 应用端发送消息丢到主线程中去处理:  
  ![image](https://user-images.githubusercontent.com/32014204/113107581-adff2180-9236-11eb-9c09-2761d522b5df.png)
  👆是主线程里做出的对应处理
  
  
  
#### 二、在应用端接收到AMS的Create消息时，应用端会开始Create Service
![image](https://user-images.githubusercontent.com/32014204/113105324-0c76d080-9234-11eb-8c16-4f372c0ff8e9.png)

* 其中，启动流程和Activity类似，会创建其Context，makeApplication，同时通过attach绑定上下文（还会向AMS报告），再调用Service的onCreate()回调  
  
  
#### 三、Service的大致启动流程图
![image](https://user-images.githubusercontent.com/32014204/113107844-f74f7100-9236-11eb-841d-7eabeb64836e.png)  
  
#### 四、换一个角度来看Service的启动
![image](https://user-images.githubusercontent.com/32014204/113108674-d3405f80-9237-11eb-89e7-87522b38ab0c.png)  
  
  
#### bindService和startService的区别
bindService()不会启动onStartCommand()，启动Service进程后不会给加到pendingService中

