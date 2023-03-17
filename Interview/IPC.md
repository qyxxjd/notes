## IPC进程间通信

#### Binder机制
- Client进程：使用服务的进程（Android客户端）
- Server进程：提供服务的进程（服务端）
- ServiceManager进程：管理Service的注册、查询
- Binder驱动：连接Client、Server、ServiceManager的桥梁，通过内存映射传递进程间的数据

#### 进程间通信的方式
- Intent传递数据；
- 文件共享；
- 基于Binder的AIDL、Messenger
- ContentProvider；
- Socket；
- BroadcastReceiver。

#### 优点
- 分担主进程的内存压力
- 不占用主进程的内存空间了
- 防止主进程被杀，和主进程之间相互监视，如果被杀就重新启动它。

#### 缺点
- 静态成员和单例模式完全失效
- SharedPreference的可靠性下降
- 线程同步机制完全失效
- Application会多次创建
- 进程之间通信麻烦

#### 使用多进程
- 在清单文件中使用 android:process 属性指定所处的进程，默认主进程。
- android:process=":XXX"，以:开头的名字，则表示这是一个应用程序的私有进程，否则它是一个全局进程。
- 私有进程的进程名称是会在冒号前自动加上包名，而全局进程则不会。
