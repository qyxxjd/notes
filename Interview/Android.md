## Android 基础知识

#### 四大组件
- Activity
- Service
- Content Provider
- BroadCast Receiver
    
#### 访问UI线程
- 协程
- Activity.runOnUiThread(Runnable)
- View.post(Runnable)
- Handler
- AsyncTask

#### Context
> App context数量 = Activity数量 + Service数量 + 1  

- ContextImpl
- ContextWrapper
    - Application
    - Service
    - ContextThemeWrapper
        - Activity
    
#### Activity LaunchMode
- standard：默认模式，始终创建新的实例。
- singleTop：栈顶复用。
- singleTask：栈内复用，会清空上面的实例，回到栈顶。
- singleInstance：使用独立的栈存放实例，不允许其它实例进入。
- singleInstancePerTask：Android 12新增。该Activity只能作为Task的root Activity运行。

#### Fragment两种Adapter之间的区别
- FragmentPagerAdapter
    - 适用于固定、少量的Fragment场景。切换过程中不会销毁，只是调用事务中的detach方法。而在detach方法中只会销毁Fragment中的View，而不会销毁Fragment对象。
- FragmentStatePagerAdapter
    - 适用于Fragment数量较多的场景。页面离开后，会释放其资源。

#### BroadcastReceiver
- 静态注册：常驻型广播，需要在Androidmanifest.xml中进行注册。不受页面生命周期的影响，即使退出了页面，也可以收到广播，所以会占用CPU的资源。
- 动态注册：在代码中注册的，非常驻型广播，收到生命周期的影响，退出页面后，就不会收到广播，我们通常运用在更新UI方面。这种注册方式优先级较高。
- 广播是分为有序广播和无序广播。
- LocalBroadcastReceiver：Handler实现。应用内部使用，更安全，效率更高。不能静态注册，只能动态注册。


#### 进程等级
- 前台进程 (Foreground process)
- 可见进程 (Visible process)
- 服务进程 (Service process)
- 后台进程 (Background process)
- 空进程 (Empty process)


#### SharedPreference的commit、apply区别
- commit
    - 存储的过程是原子操作
    - 有返回值。成功为ture，失败为false
    - 效率低
- apply
    - 存储的过程是原子操作
    - 无返回值，操作结果未知
    - 先写入内存，再写入磁盘，效率高于commit

#### Android有哪几种动画
- FrameAnimation（帧动画）：移动、放大、缩小、旋转、透明度...
- TweenAnimation（补间动画）：按顺序播放图片
- PropertyAnimation（属性动画）：一种不断地对值进行操作的机制
    
    
#### Intent传递数据的限制
- 限制1M，是binder底层的限制，binder不适用于传输大量数据
    

#### Service生命周期
> startService： onCreate()→onStartCommand()→onDestroy()  
> bindService：onCreate()→onBind()→unBind()→onDestroy() 
> startService+bindService：onCreate()→onStartCommand()→onBind()→unBind()→onDestroy()  

#### Service保活
- 提升service进程优先级
- 利用系统广播唤醒
- 在service的生命周期进行重新创建、启动
- JobScheduler
    

#### Intent Filter
- action：可声明多个，区分大小写，和过滤规则中的任何一个action相同即可匹配成功。
- data：由scheme、host、port、path、mimeType等组成
- category：所有的category都必须和过滤规则中的category相同才能匹配成功


#### IPC常见通讯方式
- Intent传递数据；
- 文件共享；
- 基于Binder的AIDL、Messenger
- ContentProvider；
- Socket；
- BroadcastReceiver


#### Apk打包过程
- aapt 工具打包资源文件，生成 R.java 文件
- aidl 工具处理 AIDL 文件，生成对应的 .java 文件
- javac 工具编译 Java 文件，生成对应的 .class 文件
- 把 .class 文件转化成 Davik VM 支持的 .dex 文件
- apkbuilder 工具打包生成未签名的 .apk 文件
- jarsigner 对未签名 .apk 文件进行签名
- zipalign 工具对签名后的 .apk 文件进行对齐处理


#### 安卓打包签名v1，v2区别
- v1: Android7.0以前使用的一套签名方案：在 apk 根目录下的 META-INF/ 文件夹下生成签名文件，然后在安装时在系统的 PackageManagerService 里进行签名文件的验证。
- v2: Android7.0之后，Android 提供了新的 V2 签名方案：利用 apk(zip) 压缩文件的格式，在几个原始内容区之外增加了一块用于存放签名信息的数据区，然后同样在安装时在系统的 PackageManagerService 里进行 V2 版本的签名验证，V2 方案会更安全、使校验更快安装更快。V2 签名方案会向后兼容，如果没有使用 V2 签名就会默认走 V1 签名方案的验证过程。
- v3: Android9.0之后，TODO

#### Activity生命周期
- A跳转B：A.onCreate→A.onStart→A.onResume→A.onPause→B.onCreate→B.onStart→B.onResume→A.onStop 
- 按下Back键后： B.onPause→A.onRestart→A.onStart→A.onResume→B.onStop→B.onDestory
- 若B是个窗口Activity，A失去焦点部分可见，故不会调用onStop，此时生命周期顺序： A.onCreate→A.onStart→A.onResume→A.onPause→B.onCreate→B.onStart→B.onResume
- 按下Back键后：B.onPause→A.onResume→B.onStop→B.onDestory

