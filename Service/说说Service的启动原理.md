# Service的启动原理

![image](https://user-images.githubusercontent.com/32014204/113102717-fa476300-9230-11eb-99ee-34721a5baa8c.png)

* 启动流程与Activity类似，先向AMS发起请求，AMS做出一系列操作后，来到判断其进程是否**启动**以及其进程是否**就绪**的流程
![image](https://user-images.githubusercontent.com/32014204/113103698-316a4400-9232-11eb-8b34-cae0b7582436.png)

## 在应用端的进程被创建后，会发起attachApplication，其中的方法处理会启动PendingService，并开始真正的启动Service
![image](https://user-images.githubusercontent.com/32014204/113104604-56ab8200-9233-11eb-96fc-d02531a2845b.png)  
  
* 其中scheduleCreateService()是AMS端向应用端发起调用，执行Service的启动流程，以及onCreate()等回调方法
* sendServiceArgsLocked是用来启动ServiceStartCommon()的方法，该方法会真正的启动Service的工作  
  
  
## 在应用端接收到AMS的Create消息时，应用端会开始Create Service
![image](https://user-images.githubusercontent.com/32014204/113105324-0c76d080-9234-11eb-8c16-4f372c0ff8e9.png)

* 其中，启动流程和Activity类似，会创建其Context，makeApplication，同时通过attach绑定上下文（还会向AMS报告），再调用Service的onCreate()回调
