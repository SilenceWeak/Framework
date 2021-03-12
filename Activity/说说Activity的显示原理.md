# Activity的显示原理
* Activity的显示原理
  * setContentView的原理是什么
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
    
  
  * ViewRoot是干什么的，是View Tree的rootView吗

* Activity的UI刷新机制


* UI的绘制原理


* Surface原理

