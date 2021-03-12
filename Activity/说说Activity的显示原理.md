# Activity的显示原理
### PhoneWindow是什么，什么时候创建的
phoneWindow是整个手机的Window，它管理整个手机的Window，除了应用的Window还包括很多系统本身的Window，它是在Activity初始化的attach函数中被创建的
### setContentView的原理是什么，DecorView又是什么？
原理是里面有个installDecor方法，该方法会拿到DecorView，也即一个FrameLayout，手机的RootView，在其中加载系统布局后找到其中的部件mContentParent，再向mContentParent中设置布局，建立一套ViewTree数据结构。
### ViewRoot是什么，有什么作用？
ViewRoot是View的数据结构的根，也即DecorView，它创建了ViewRootImpl，与WMS通信进行绘制页面
### View的显示原理是什么，WMS发挥了什么作用？
显示原理是，在ViewRootimpl的setView方法中与WMS通信，注册Window，同时进行onMeasure等绘制流程，在WMS注册后会拿到surface，应用进行绘制后，SurfaceFlinger根据WMS的管理合成View输送到缓冲区
* **setContentView的原理是什么**
```
    setContentView(){
       getWindow().setContentView();
    }
    
    attach(Context context){ //在Activity的attach函数中初始化mWindow
      ...
      mWindow = new PhoneWindow(this);
      ...
    }
    
    setContentView(int layoutResID){
     if(mContentParent == null){
      installDecor(); //初始化mDecor（即系统布局），并且在其中通过findViewbyId初始化mContentParent
     }
     
     mLayoutInflater.inflate(LayoutResID,mContentParent); //在此处给ViewPartent填充布局View
    }
```
**上述工作只是建立了ViewTree的数据结构，要展示出来还有很多的工作**
* Activity在onresume之后才会显示出来的原因是什么
    **在View被onCreate创建好数据结构之后，在onResume回调中将其真正展示出来**
     ```
     handleResumeActivity(){
       ... = performResumeActivity(token)；      //调用onResume回调方法
       
       ...                                      //省略代码，此处做的事主要是拿到DecorView，同时添加到WindowManager之中
       
       r.acticity.makeVisible()                 //最后设置Activity可见,不过这里只是一些重绘而已，真正的管理还是WindowManager来管理的绘制流程
     }
     ```
**在windowManager的addView中，使用ViewRootImpl.setView()方法来管理刚才设置的View**
    ```
     (ViewRootImpl)root.setView(view,wparams,panelParentView);
     
     setView(View view){
      if(mView = null){
       mView = view;
       requestLayout();
       ...
       mWindowSession.addToDisplay(mWindow,...);     //mWindowSession其实就是个应用和WM进行通信的binder对象
       ...
      }
     }
    ```
    * **requestLayout() -> ... -> performTraversals()，这个函数里会真正执行绘制流程**
      ```
       performTraversals(){
        relayoutWindow(params,...);       //向WM拿到surface，也即申请surface，只有拿到surface才能使用surfaceFlinger来绘制
        ...
        performMeasure(...);
        ...
        performLayout(lp,...);
        ...
        performDraw();
        ...
       }
      ```
    * WMS的作用
      * 分配surface
      * 掌管surface显示顺序及位置尺寸等
      * 控制窗C动画
      * 输入事件分发
      ```
      addToDisplay() -> mService.addWindow(...);
      ```
    

* Activity的UI刷新机制


* UI的绘制原理


* Surface原理


# 图解Activity显示原理
![image](https://github.com/SilenceWeak/Framework/blob/main/Pic/Activity%E6%98%BE%E7%A4%BA%E5%8E%9F%E7%90%86.jpg)
* 其中Activity会在attach方法中创建一个phonewindow，phonewindow再创建DecorView，DecorView中的一部分就是ContentView，而DecorView做的最重要的一件事就是初始化了一个ViewRootImpl对象，该对象会和WMS进行Binder双向调用，而在WMS中，会注册一个Window，通过WMS全权进行管理该窗口。
* 第一次绘制时，还会注册一个surface，拿到surface之后应用就可以进行绘制，之后surfaceFlinger会根据WMS提供的window大小位置等信息来合成View输入到缓冲区进行显示
