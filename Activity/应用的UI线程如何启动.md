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

**再看一个问题，一个异常，是子线程刷新UI时抛出的**

只有create view hierarchy的那个riginal thread才可以touch这个view，

也是在暗示并不是只有主线程才可以touch view

它是怎么判断我们的线程是不是最初创建view hierarchy的线程？

看看信息，View.requstLayout会调到ViewRootImpl的checkThread，所以每次requestLayout都会检查线程，不对就抛出异常

**看看checkThread实现：**
```
void checkThread(){
    //mThread就是创建View hierarchy的thread，什么时候init的？在ViewRootImpl的构造函数init的，
    //所以可以确定UI线程就是ViewRootImpl所在的线程。
    //注意不是View布局加载所在线程，而是ViewRootImpl创建所在线程
    if(mThread != Thread.currentThread()){
        throw new CalledFromWrongThreadException(
            "Only the orignal thrad that catred a view can touch its views"
        )
    }
}
```
所以ViewRootImpl创建是在哪个线程？

调用流程：

ActivityThread.handleResumeActivity => WindowManagerImpl.addView => WindowManagerGlobal.addView => ViewRootImpl的创建

 

handleResumeActivity是在应用进程的主线程调的，

所以

***Activity的DecorView对应的ViewRootImpl是在主线程创建的***

**结论：**

一个Activity，里面有一个PhoneWIndow，PhoneWindow厘米有一个DecorView，

DecorView是整个界面最顶层的View，

DecorView有一个ViewRootImpl，负责DecorView的绘制流程，事件分发，以及与WMS通信，

**VIewRootImpl是在主线程创建的，所以对于Activity的DecorView来说，UI线程就是主线程**

 

 

UI线程就是主线程，这个结论之所以成立，

是因为DecorView的ViewRootImpl恰好在主线程创建而已，如果不再主线程创建会怎么样？那就不在主线程刷新UI了吗？

看看Activity是怎么触发ViewRootImpl创建的，以handleResumeActivity为入口分析一下：

```
final void handleResumeActivity(IBinder token,...){
    ...
    final Activity a = r.activity;
    //得到Activity的WindowManager
    ViewManager wm = a.getWindowManager();
    //把Actvitiy的decorView add到WindowManager
    wm.addView(decor, l);
    ....
}
 
//addView里做了什么：
//new一个ViewRootImpl对象
root = new ViewRootImpl(view.getContext(), display);
//通过serView把View交给ViewRootImpl管理，这步非常重要。
root.setView(view, wparams, panelParentView);
```

模仿一下，做一个实验
```
new Thread(){
    @Override
    public void run(){
        Looper.prepare();
        ...
        //view是通过LayoutInflater加载出来的，通过addView把View加入WindowManager，
        //设置下View的textView，发现成功了，点击View的一个按钮，onCreate函数是跑 在子线程的，
        //如果在主线程去刷新UI，反而会抛出那个异常:WrongThread
        getWindowManager().addView(view, params);
        
        Looper.loop();
    }
}.start();
```
### UI线程的启动
对应用来说，UI线程默认就是主线程，

那么UI线程的启动，就是主线程的启动，这个大家都熟悉，AP启动时主线程就有了，

AP启动流程：

zygote fork进程 =》 启动Binder线程 =》 执行入口函数

入口函数就是ActivityThread.main()
```
public static void main(String[] args){
    //创建Looper，looper我们都熟悉，平时创建子线程的looper都要调用Looper.prepare()
    //这个prepareMainLooper我们没用过，因为我们不能调。
    Looper.prepareMainLooper();
    ...
    Looper.loop();
}
```
