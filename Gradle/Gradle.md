## Gradle常用命令
windows下去掉前面的 `./`

```gradle
./gradlew -v        //版本号
./gradlew assemble  //构建项目输出
./gradlew check     //运行检测和测试任务
./gradlew clean     //清除build文件夹
./gradlew build     //编译打包,包含debug、release
./gradlew build --refresh-dependencies // 强制重新下载依赖
./gradlew assembleDebug     //编译并打debug包
./gradlew assembleRelease   //编译并打release包
./gradlew installRelease    //release打包并安装
./gradlew uninstallRelease  //卸载release包
./gradlew app:dependencies  //查看依赖关系
```
