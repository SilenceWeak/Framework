## ServiceManager的启动
* 启动进程
* 启动Binder机制
* 发布自己的服务
* 等待并响应请求

在init.rc里启动

```
int main(int argc, char **argv) {
  struct binder state *bs;
  bs = binder open(128*1024);
  binder become context_ manager(bs);
  binder_ loop(bs, sycmgr_ handler);
  return 0;
}
```
三个步骤：
  * 启动binder
  * 把binder注册成上下文，作用就是告诉binder驱动这个服务的binder已经就绪了
  * 进入binder的loop循环：第一个阶段主要是注册binder线程，第二个阶段循环读取请求并处理请求


## 如何获取ServiceManager
先看一个例子SurfaceFlinger是怎么拿到ServiceManager的
```
int main(int, char**) {
  sp < ProcessState> ps(ProcessState::self());
  ps->startThreadPool();
  sp <SurfaceFlinger> flinger = new SurfaceFlinger();
  flinger-> init();
  -------------------拿到ServiceManager并注册自己-------------------------------
  sp<lServiceManager> sm(defaultServiceManager));
  sm-> addService(String1 6(SurfaceFlinger::getServiceName(), flinger, false);
  ----------------------------------------------------------------------------
  flinger-> run();
  return 0;
}
```
所以重点在于defaultServiceManger()  
**在defaultServiceManger（)中，如果已经拿到了binder对象，就直接返回，如果没有，那么就循环尝试，直到拿到为止，之所以需要这样是因为surfaceFlinger和ServiceManager都是通过init.rc拉起的**

## 怎么添加Service？
addService(): 先发送请求，参数是自己的名字和binder对象，而ServiceManager接收到了之后会开始进行处理

```
int svcmgr handler(... struct binder transaction data *txn, .. {
  switch(txn-> code) {
    ...
    case SVC_MGR_ADD_SERVICE:
      ...
      do_add_service(bs, s, len, handle, ..);
    break;
  }
  ...
}
```
可以看到ServiceManager根据发送请求的请求码做出了添加服务的操作

## 怎么获取服务
```
public static lBinder getService(String name) {
  IBinder service = sCache.get(name);
  if (service != null) {
    return service;
  } else if{
    return getlServiceManager().getService(name);
  }
  return null;
}
```
发起一个binder调用，发送获取服务的请求

而ServiceManger接受到请求之后，根据**对应的请求码**做出响应操作
```
int svcmgr handler(... struct binder transaction_ data *txn, .... ){
  uint32 t handle;
  switch(txn->code) {
    ... 
    case SVC_MGR_GET_SERVICE:
      s = bio_get_string16(msg, &len);
      handle = do_find_service(bs, S, len, ..);)
      bio_put_ref(reply, handle);
      return O;
    ...
  }
}
```
