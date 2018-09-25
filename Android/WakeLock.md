#### 查看 `Linux` 的 `WakeLock`

```shell
cat /sys/power/wake_lock
```

#### 查看应用持有的 `WakeLock`

```shell
adb shell dumpsys power
```

#### 查看应用是否持有屏幕常亮的锁

```shell
adb shell dumpsys power | grep -E "FULL_WAKE_LOCK|SCREEN_BRIGHT_WAKE_LOCK"
```
