## Activity

- `AndroidMainfest` 配置 `android:exported="false"`, 其它应用不可以调用
- 检测栈顶 `Activity`, 防止页面被劫持
- `WebView` 加载网页发生证书认证错误时, 会调用 `WebViewClient` 类的 `onReceivedSslError` 方法, 如果该方法实现调用了 `handler.proceed()` 来忽略该证书错误, 则会受到中间人攻击的威胁, 可能导致隐私泄露。当发生证书认证错误时, 采用默认的处理方法 `handler.cancel()`, 停止加载页面
```java
    mWebView.getSettings().setJavaScriptEnabled(true);
    mWebView.addJavascriptInterface(new JsBridge(mContext), JS_OBJECT);
    mWebView.loadUrl("http://www.xxxx.com/");
    mWebView.setWebViewClient(new WebViewClient() {
        @Override
        public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
            handler.proceed(); // 忽略SSL证书错误(错误做法)
            handler.cancel();  // 停止加载页面(正确做法)
        }
    });
```
- `WebView` 检查是否明文保存密码, 使用 `WebView.getSettings().setSavePassword(false)` 来禁止保存密码
- `WebView` 检测是否使用 `addJavascriptInterface` 接口, 如果使用了需要将 `minSdkVersion` 提升至 `17 (Android 4.2)`, 或者使用一些第三方的库来解决注入漏洞


## BroadcastReceiver

- 使用 `LocalBroadcastManager` 处理应用内部的广播
- 应用间使用广播, 通过自定义权限和设置 `android:protectionLevel`, 同时要避免敏感数据的传递
- 不要使用 `sendStickyBroadcast`、`sendStickyXXX` 等 `Android SDK` 文档中明确说明了存在安全问题的 `API`


## Service

- `AndroidMainfest` 配置 `android:exported="false"`, 其它应用不可以调用
- 通过 `Intent.getXXXExtra()` 获取数据时进行以下判断, 以及用 `try catch` 捕获所有异常, 以防止应用出现[拒绝服务漏洞](https://jaq.alibaba.com/blog.htm?id=55)
1)	空指针异常
2)	类型转换异常
3)	数组越界访问异常
4)	类未定义异常
5)	其他异常

## ContentProvider

- 定义了私有权限, 但是没有定义私有权限的级别, 或者定义的权限级别不够, 导致恶意应用只要声明这个权限就能够访问到相应的 `Content Provider` 提供的数据, 造成数据泄露
- 当 `Content Provider` 的数据源是 `SQLite` 数据库时, 如果实现不当, 而 `Provider` 又是暴露的话, 则可能会引发本地 `SQL` 注入漏洞
- 防止目录遍历漏洞, 去除 `Content Provider` 中没有必要的 `openFile()` 接口, 过滤限制跨域访问, 对访问的目标文件的路径进行有效判断
- 正确的定义私有权限
```
<permission android:description="string resource"
            android:icon="drawable resource"
            android:label="string resource"
            android:name="string"
            android:permissionGroup="string"
            android:protectionLevel=["normal" | "dangerous" | "signature" | "signatureOrSystem"] />
```
`android:protectionLevel` 参数说明

| 属性 | 描述 | 
| ----- | ----- |
| `normal` | 默认值, 低风险权限, 在安装的时候, 系统会自动授予权限 |
| `dangerous` | 高风险权限, 如发短信, 打电话, 读写通讯录。使用此 `protectionLevel` 来标识用户可能关注的一些权限。`Android` 将会在安装程序时, 警示用户关于这些权限的需求 |
| `signature` | 签名权限, 当应用程序所用签名与声明引权限的应用程序所用签名相同时, 才能将权限授给它 |
| `signatureOrSystem` | 除了具有相同签名的 `APP` 可以访问外, `Android` 系统中的程序也有权限访问 |

- [参考链接: Android安全开发之Provider组件安全](https://jaq.alibaba.com/community/art/show?articleid=352)
