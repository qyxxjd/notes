## ROM源码编译流程


#### 删除生成的文件

> 它会删除所有设置所生成的所有的 output 与中间文件

```
make clobber
```


#### 设置

> 设定core文件的最大值，单位为区块

```
ulimit -c unlimited
```


#### 初始化

> 执行脚本文件，完成一些命令的初始化

```
source build/envsetup.sh
```


#### 选择ROM类型

> eng：工程版本  
> user：发行版本  
> userdebug：部分调试版本  

| user | userdebug | eng |
| ----- | ----- | ----- |
| 仅安装标签为 `user` 的模块 | 安装标签为 `user、debug` 的模块 | 安装标签为 `user、debug、eng` 的模块 |
| 设定属性 `ro.secure=1`，打开安全检查功能 | 设定属性 `ro.secure=1`，打开安全检查功能 | 设定属性 `ro.secure=0`，关闭安全检查功能 |
| 设定属性 `ro.debuggable=0`，关闭应用调试功能 | 设定属性 `ro.debuggable=1`，启用应用调试功能 | 设定属性 `ro.debuggable=1`，启用应用调试功能 |
|  |  | 设定属性 `ro.kernel.android.checkjni=1`，启用 `JNI` 调用检查 |
| 默认关闭 `adb` 功能 | 默认打开 `adb` 功能 | 默认打开 `adb` 功能 |
| 打开 `Proguard` 混淆器 | 打开 `Proguard` 混淆器 | 关闭 `Proguard` 混淆器 |
| 打开 `DEXPREOPT` 预先编译优化 | 打开 `DEXPREOPT` 预先编译优化 | 关闭 `DEXPREOPT` 预先编译优化 |

```
lunch
```


#### 构建

> -j参数指定并行运行任务的数量，推荐：CPU核心数*2

```
make -j4
```


#### 编译源码，生成Android项目工程

```
mmm development/tools/idegen/
development/tools/idegen/idegen.sh
```
