#### Apk打包流程

1. 用 aapt 生成 R.java 文件
	> aapt全称是 Android Asset Packaging Tool  
	> 位于：SDK/build-tools/版本号/aapt.exe  
	> 编译res和asert下的资源生成resource.arsc，编译生成R.java文件。  

2. 编译 AIDL 的java文件
	> 位于：SDK/build-tools/版本号/aidl.exe  
	> 如果项目没有AIDL文件，可以忽略此步骤  

3. 把java文件编译成class文件
	> 把R.java、AIDL的java文件编译出来以后，和java源文件一起编译成class文件  

4. 使用 dx.bat 将编译好的class文件转换为dex文件  
	> 位于：SDK/build-tools/版本号/dx.bat  
	> 将字节码转换为Dalvik字节码  

5. 生成 apk 文件
	- 资源文件打包，包括生成resources.arsc、xml和二进制文件，生成apk文件
	- 往apk里面加入dex文件
	- 对apk进行签名
	- 使用 zipalign 工具对 apk 进行对齐处理。
