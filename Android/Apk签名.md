#### APK手动签名(v1)

```
// 语法
jarsigner -verbose -keystore 签名文件名称 -signedjar 签名后的APK名称 原始或未签名的APK名称 签名文件的Alias

// 示例。先进入签名文件所在路径，再执行命令
jarsigner -verbose -keystore xxjd.jks -signedjar signed.apk no_sign.apk xxjd
```

#### APK手动签名(v1+v2)

```
// 1. 进入build-tools目录，示例：
cd /Users/classic/Library/Android/sdk/build-tools/28.0.3

// 2. 使用apksigner签名(默认同时使用V1和V2签名)
./apksigner sign --ks /Users/classic/ClassicSource/NewWages/tools/qyxxjd.jks --ks-key-alias xxjd /Users/classic/Desktop/NewWages_2.2_22_20191121155117_legu.apk

// 3. 按照提示输入签名的密码，完成签名
```
