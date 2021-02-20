# Android系统的启动
## 开机启动init进程
[帖子传送门](https://blog.csdn.net/z240336124/article/details/98472607)
*  /system/core/init/Init.cpp 的 main()：
  ```
  代码去帖子里看，大概的做的事情是
  1.创建目录,挂载分区
  2.析启动脚本，
  3.启动解析的服务，
  4.守护解析的服务。init.rc 文件是 Android 系统的重要配置文件，位于 /system/core/rootdir/init.rc
  ```
  
  * init.rc
  ```
    // 导入其它的一些脚本
  import /init.environ.rc
  import /init.usb.rc
  // 当前硬件版本的脚本
  import /init.${ro.hardware}.rc
  import /init.${ro.zygote}.rc
  import /init.trace.rc

  on early-init
  ...
  on init
  ...
  // 服务 服务名称 执行文件路径 执行参数
  // 有几个重要的服务
  service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
  service servicemanager /system/bin/servicemanager
  service surfaceflinger /system/bin/surfaceflinger
  service media /system/bin/mediaserver
  service installd /system/bin/installd
  ```
