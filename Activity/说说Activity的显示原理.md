# Activity的显示原理
* Activity的显示原理
  * setContentView的原理是什么
  ```
    setContentView(){
       getWindow().setContentView();
    }
    
    attach(Context context){
      ...
      mWindow = new PhoneWindow(this);
      ...
    }
  ```
  
  * Activity在onresume之后才会显示出来的原因是什么
  * ViewRoot是干什么的，是View Tree的rootView吗

* Activity的UI刷新机制


* UI的绘制原理


* Surface原理

