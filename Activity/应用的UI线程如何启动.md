# 应用的UI线程是如何启动的?

### 什么是UI线程
* UI线程是刷新UI所在的线程。非UI线程不能刷新UI
* UI线程是单线程的，如果多个线程都能刷新UI那就无所谓是不是UI线程了，为什么是单线程，如果是多线程，这个UI框架要到处上锁，很容易出问题

* **那么UI线程到底是哪个线程？UI线程是主线程吗？**
  回想下平时写代码有什么耗时操作，都是丢给子线程处理，再回到UI线程刷新UI，这是怎么做的？：
  * Activity.runOnUiThread(Runnable)
  * View.post(Runnable)
  都带个Runnable，Runnable就会执行到UI线程
 **Activity.runOnUiThread(Runable):**
 ```
  final void runOnUiThread(Runnable action){
      //判断当前的thread是不是UIthread
      if(Thread.currentThread() != mUiThread){
          //不是就post到消息队列里面
          mHandler.post(action);
      }else{
          //是的话就直接run
          action.run();
      }
  }

  //看看mUiThread和mHandle在哪里初始化的，
  //mHandler是Activity里面的全局变量，在Acitivity创建时就一起创建了
  //它创建的时候没有指定looper，所以创建Activity对象时在哪个线程，那handler就用的是哪个线程的looper
  final Handler mHandler = new Handler();

  //mUiThread
  //它在调用Activity的attach函数的时候赋值的
  //attach是在activity启动的时候调用的
  final void attach(Context context,...){
      ...
      mUiThread = Thread.currentThread();
      ...
  }
 ```
* 回顾下Activity的启动流程：

  1. 创建Activity对象

  2. 创建Context

  3. 调用attach 给Activity赋context，

  4. onCreate

这4步都是在AP进程的主线程做的，

所以对于Activity来说，UI线程就是主线程。

**View.post(runable)**
```
public boolean post(Runnable action){
    final AttachInfo attachInfo = mAttachInfo;
    //1,attachInfo不为null，把action post到mHandler里面。
    if(attachInfo != null){
        return attachInfo.mHandler.post(action);
    }
    //2，attachInfo是null，把action post 到 ViewRootImpl的runQueue
    ViewRootImpl.getRunQueue().post(action);
    return true;
}
```
**对于代码的解释**
情况1，

何时设置AttachInfo？

它是ViewTree第一次绘制的时候，会递归的给所有的子View都附上一个attachInfo，

attachInfo从哪里来？它是在ViewRootImpl构造函数里面创建的

里面的mHandler对应的就是ViewRootImpl对象创建时所在的线程，所以关键是ViewRootImpl，

 

情况2，

AttachInfo为什么会是NUll

因为ViewRootImpl还没来得及创建，它是在Activity的onResume之后创建的

所以如果你在Activity的onCreate就用View post了一个Runnable，就会放到RunQueue里面，等ViewRootImpl创建好后，

再丢到VIewRootImpl所在线程处理，

 

所以不管哪种情况，最后都要丢到ViewRootImpl的线程处理。

### 
