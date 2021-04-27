# Service的绑定原理
![image](https://user-images.githubusercontent.com/32014204/115986376-6c1d8d00-a5e2-11eb-9589-ed0d9dac9a10.png)

## Service的常规用法
![image](https://user-images.githubusercontent.com/32014204/115987262-e94b0100-a5e6-11eb-9080-73ae824bef96.png)

## binderService的总体流程
![image](https://user-images.githubusercontent.com/32014204/115987280-0384df00-a5e7-11eb-8ff0-1d72019bcc59.png)  
  
### * 其中一个细节, 如果binderService时, Service还没有启动的话, 那么AMS还要走启动Service的那套流程

## bindService的处理
![image](https://user-images.githubusercontent.com/32014204/115987616-63c85080-a5e8-11eb-9cb6-9330cb2c399f.png)

### * 其中会先调用getServiceDispatcher来进行各种操作

![image](https://user-images.githubusercontent.com/32014204/115987915-bb1af080-a5e9-11eb-8c00-c1699ffd70f0.png)  
  
  
通过InnerConnection向主线程post了一个runConnection, 随后来到主线程中进行处理

```
//在主线程中的处理
public void doConnected(ComponentName name, lBinder service) {
  old = mActiveConnections.get(name);
  if (old != null && old.binder == service) {
      return; //防止重复调用onServiceConnected方法
  }
  if (service != null) {
      info = new Connectionlnfo(service);
      mActiveConnections.put(name, info);
  } else {
      mActiveConnections.remove(name);
  }
  if (old != null) {
      mConnection.onServiceDisconnected(name);
  }
  if (service != null) {
      mConnection.onServiceConnected(name, service);
  }
}
```

![image](https://user-images.githubusercontent.com/32014204/115988200-1ac5cb80-a5eb-11eb-9fda-2ca4659ca658.png)
一般都不会回调到onServiceDisconnected这个方法, 一般是当这个Service被干掉的时候, 才会回调到

### 整体流程
![image](https://user-images.githubusercontent.com/32014204/115988257-5496d200-a5eb-11eb-966a-7ac0a17c0de3.png)

![image](https://user-images.githubusercontent.com/32014204/115988281-74c69100-a5eb-11eb-98ad-0fa24a0dbdcb.png)
IServiceConnection 和 ServiceConnection两者不是一对一的关系

### AMS端的处理
![image](https://user-images.githubusercontent.com/32014204/115988442-56ad6080-a5ec-11eb-9fd9-627d9ae1becd.png)

### 搞清楚几个数据结构
![image](https://user-images.githubusercontent.com/32014204/116178084-8755dd00-a747-11eb-9ffd-80af67ecb1a9.png)
#### 在这其中, 从上到下都是 **上** 包含一或多 **下** 的关系



