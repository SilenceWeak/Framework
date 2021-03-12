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
  * ViewRoot是干什么的，是View Tree的rootView吗

* Activity的UI刷新机制


* UI的绘制原理


* Surface原理

