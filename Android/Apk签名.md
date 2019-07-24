#### APK重签名

```
// 语法
jarsigner -verbose -keystore 签名文件名称 -signedjar 签名后的APK名称 原始或未签名的APK名称 签名文件的Alias

// 示例。先进入签名文件所在路径，再执行命令
jarsigner -verbose -keystore xxjd.jks -signedjar signed.apk no_sign.apk xxjd
```
