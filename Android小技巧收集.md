
#### 当前线程判断
```java
public static boolean isOnMainThread() {
    return Looper.myLooper() == Looper.getMainLooper();
}
public static boolean isOnBackgroundThread() {
    return !isOnMainThread();
}
```

#### `EditText` 不自动获取焦点

在父Layout中添加：
```java
android:focusable="true"
android:focusableInTouchMode="true"
```

#### 字符串转 `Html`
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    Html.fromHtml(string, flag);
} else {
    Html.fromHtml(string);
}
```

#### 获取错误信息
```java
Log.getStackTraceString(throwable);
```

#### 将文件大小转换为 `PB、TB、GB、MB、KB、B` 字符串
```java
Formatter.formatFileSize(mAppContext, sizeBytes);
```

#### 获取当前应用程序内存阈值
```java
ActivityManager.getMemoryClass();
```

#### 监听应用 `Activity` 的生命周期
```java
mApplication.registerActivityLifecycleCallbacks(callback);
// mApplication.unregisterActivityLifecycleCallbacks(callback);
```

#### `Sqlite` 事务处理
```java
db.beginTransaction();
try{
    for ( ... ) {
        db.insert(...);
        // 这个方法表示执行当前事务时，如果碰到其它的数据库操作，先让别的操作完再继续执行。
        db.yieldIfContendedSafely();
    }
    db.setTransactionSuccessful();
}finally{
    db.endTransaction();
}
```

#### 处理 `OnTrimMemory`

`OnTrimMemory` 方法来自 `ComponentCallbacks2` 接口，`Android 4.0` 之后提供的 `API`，如果需要兼容 `Android 4.0` 以下的版本使用 `onLowMemory`。
实现此接口的类都可以处理此方法，比如 `Application、Activity、Fragment、Service、ContentProvider`。
```java
// Application、Activity、Fragment、Service、ContentProvider
@Override public void onTrimMemory(int level) {
    switch (level) {
        case Activity.TRIM_MEMORY_UI_HIDDEN:
            // 应用的所有UI界面被隐藏了(点击了Home 或 Back键)，这时候应当释放一些资源。
            break;
        case Activity.TRIM_MEMORY_RUNNING_MODERATE:
            // 应用正常运行，并且不会被杀掉。手机内存已经有点低了，系统开始根据LRU缓存规则杀死进程。
            break;
        case Activity.TRIM_MEMORY_RUNNING_LOW:
            // 应用正常运行，并且不会被杀掉。手机内存已经非常低了，这时候应当释放一些资源。
            break;
        case Activity.TRIM_MEMORY_RUNNING_CRITICAL:
            // 应用正常运行，系统已经根据LRU缓存规则杀掉了大部分缓存的进程，此时应当释放非必须的所有资源。
            break;
        case Activity.TRIM_MEMORY_BACKGROUND:
            // 当前内存已经很低了，系统准备开始根据LRU缓存来清理进程。
            break;
        case Activity.TRIM_MEMORY_MODERATE:
            // 当前内存已经很低了，当前应用处于LRU缓存列表的中间位置，这时候应当释放一些资源。
            break;
        case Activity.TRIM_MEMORY_COMPLETE:
            // 当前内存已经很低了，系统会最优先杀掉当前应用，此时应当释放非必须的所有资源。
            break;
    }
}
```
