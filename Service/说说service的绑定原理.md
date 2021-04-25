# Service的绑定原理
![image](https://user-images.githubusercontent.com/32014204/115986376-6c1d8d00-a5e2-11eb-9589-ed0d9dac9a10.png)

## Service的常规用法
![image](https://user-images.githubusercontent.com/32014204/115987262-e94b0100-a5e6-11eb-9080-73ae824bef96.png)

## binderService的总体流程
![image](https://user-images.githubusercontent.com/32014204/115987280-0384df00-a5e7-11eb-8ff0-1d72019bcc59.png)  
  
### * 其中一个细节, 如果binderService时, Service还没有启动的话, 那么AMS还要走启动Service的那套流程

## bindService的处理
![image](https://user-images.githubusercontent.com/32014204/115987616-63c85080-a5e8-11eb-9cb6-9330cb2c399f.png)

### * 其中会先调用getServiceDispatcher来将serviceConnection注册
