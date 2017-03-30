#### 初始化  
```
LeakCanary.install(application);  
```
返回一个 `RefWatcher` 对象，用于跟踪对象是否被回收

#### ActivityRefWatcher  
`RefWatcher` 的代理类。通过注册 `ActivityLifecycleCallbacks` 回调，当 `Activity` 调用 `onDestroy()` 时进行一次内存泄漏检查，执行 `RefWatcher` 的 `watch` 方法，检测该 `Activity` 是否发生内存泄露。
```
public final class ActivityRefWatcher {
    private final ActivityLifecycleCallbacks lifecycleCallbacks = new ActivityLifecycleCallbacks() {
        public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        }

        public void onActivityStarted(Activity activity) {
        }

        public void onActivityResumed(Activity activity) {
        }

        public void onActivityPaused(Activity activity) {
        }

        public void onActivityStopped(Activity activity) {
        }

        public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
        }

        public void onActivityDestroyed(Activity activity) {
            ActivityRefWatcher.this.onActivityDestroyed(activity);
        }
    };
    private final Application application;
    private final RefWatcher refWatcher;
	// ...
    void onActivityDestroyed(Activity activity) {
        this.refWatcher.watch(activity);
    }
}
```

#### RefWatcher  
检测内存泄漏核心类
```
  private final Set<String> retainedKeys;
  public void watch(Object watchedReference, String referenceName) {
    if (this == DISABLED) {
      return;
    }
    checkNotNull(watchedReference, "watchedReference");
    checkNotNull(referenceName, "referenceName");
    final long watchStartNanoTime = System.nanoTime();
	// 生成一个UUID来标识一个弱引用
    String key = UUID.randomUUID().toString();
    // 将key添加到Set集合,判断内存泄漏时需要用到
    retainedKeys.add(key);
	/**
	 * watchedReference：需要检测的Activity对象
	 * key：UUID
	 * referenceName：检测对象的标识符
	 * queue：引用队列(ReferenceQueue)，当被引用的对象被JVM释放时，会把该对象的弱引用放到该队列中，通过检查ReferenceQueue中是否存在我们关心的KeyedWeakReference来判断引用的对象是否被成功释放。
	 **/
    final KeyedWeakReference reference =
        new KeyedWeakReference(watchedReference, key, referenceName, queue);
    ensureGoneAsync(watchStartNanoTime, reference);
  }
  // 分析内存泄漏，等主线程空闲时再进行处理
  private void ensureGoneAsync(final long watchStartNanoTime, final KeyedWeakReference reference) {
    watchExecutor.execute(new Retryable() {
      @Override public Retryable.Result run() {
        return ensureGone(reference, watchStartNanoTime);
      }
    });
  }
```
`watchExecutor` 的实现类为 `AndroidWatchExecutor` , 内部使用`IdleHandler` 在主线程空闲时处理操作。代码如下：
```
public final class AndroidWatchExecutor implements WatchExecutor {
    // ...
    public void execute(Retryable retryable) {
        if(Looper.getMainLooper().getThread() == Thread.currentThread()) {
            this.waitForIdle(retryable, 0);
        } else {
            this.postWaitForIdle(retryable, 0);
        }
    }
    private void postWaitForIdle(final Retryable retryable, final int failedAttempts) {
        this.mainHandler.post(new Runnable() {
            public void run() {
                AndroidWatchExecutor.this.waitForIdle(retryable, failedAttempts);
            }
        });
    }
    void waitForIdle(final Retryable retryable, final int failedAttempts) {
        Looper.myQueue().addIdleHandler(new IdleHandler() {
            public boolean queueIdle() {
                AndroidWatchExecutor.this.postToBackgroundWithDelay(retryable, failedAttempts);
				// false代表执行完毕之后就移除这个idleHandler, 只执行一次
                return false;
            }
        });
    }
    private void postToBackgroundWithDelay(final Retryable retryable, final int failedAttempts) {
        long exponentialBackoffFactor = (long)Math.min(Math.pow(2.0D, (double)failedAttempts), (double)this.maxBackoffFactor);
        long delayMillis = this.initialDelayMillis * exponentialBackoffFactor;
        this.backgroundHandler.postDelayed(new Runnable() {
            public void run() {
                Result result = retryable.run();
                if(result == Result.RETRY) {
                    AndroidWatchExecutor.this.postWaitForIdle(retryable, failedAttempts + 1);
                }
            }
        }, delayMillis);
    }
}
```
检测内存泄漏的主要业务逻辑在 `ensureGone` 方法内。先调用 `removeWeaklyReachableReferences` 方法来将引用队列中存在的弱引用从 `retainedKeys` 中删除掉，`retainedKeys` 中保留的就是还没有被回收对象的弱引用 `key`。之后再用 `gone` 方法来判断我们需要检查的 `Activity` 的弱引用是否在 `retainedKeys` 中，如果没有，说明这个 `Activity` 对象已经被回收。否则主动触发一次 `GC`，再进行以上两个步骤，如果发现这个 `Activity` 还没有被回收，就认为这个 `Activity` 很有可能泄漏了，并 `dump` 出当前的内存文件进行分析。
```
  Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime) {
    long gcStartNanoTime = System.nanoTime();
    long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);

    removeWeaklyReachableReferences();

    if (debuggerControl.isDebuggerAttached()) {
      // The debugger can create false leaks.
      return RETRY;
    }
    if (gone(reference)) {
      return DONE;
    }
	// 主动触发一次gc
    gcTrigger.runGc();
    removeWeaklyReachableReferences();
    if (!gone(reference)) {
      long startDumpHeap = System.nanoTime();
      long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);
	  // 可能存在内存泄漏，导出内存文件
      File heapDumpFile = heapDumper.dumpHeap();
      if (heapDumpFile == RETRY_LATER) {
        // Could not dump the heap.
        return RETRY;
      }
      long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
      heapdumpListener.analyze(
          new HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
              gcDurationMs, heapDumpDurationMs));
    }
    return DONE;
  }
  // 判断我们需要检查的Activity的弱引用是否在retainedKeys中
  private boolean gone(KeyedWeakReference reference) {
    return !retainedKeys.contains(reference.key);
  }
  // 将引用队列中存在的弱引用从retainedKeys中删除掉，这样，retainedKeys中保留的就是还没有被回收对象的弱引用key
  private void removeWeaklyReachableReferences() {
    KeyedWeakReference ref;
    while ((ref = (KeyedWeakReference) queue.poll()) != null) {
      retainedKeys.remove(ref.key);
    }
  }
```
`HeapDumper` 的实现类为 `AndroidHeapDumper`, 内部通过`Debug.dumpHprofData(path)`实现。代码如下：
```
public final class AndroidHeapDumper implements HeapDumper {
    // ...
    public File dumpHeap() {
        File heapDumpFile = this.leakDirectoryProvider.newHeapDumpFile();
        if(heapDumpFile == RETRY_LATER) {
            return RETRY_LATER;
        } else {
            FutureResult waitingForToast = new FutureResult();
            this.showToast(waitingForToast);
            if(!waitingForToast.wait(5L, TimeUnit.SECONDS)) {
                CanaryLog.d("Did not dump heap, too much time waiting for Toast.", new Object[0]);
                return RETRY_LATER;
            } else {
                Toast toast = (Toast)waitingForToast.get();

                try {
                    Debug.dumpHprofData(heapDumpFile.getAbsolutePath());
                    this.cancelToast(toast);
                    return heapDumpFile;
                } catch (Exception var5) {
                    CanaryLog.d(var5, "Could not dump heap", new Object[0]);
                    return RETRY_LATER;
                }
            }
        }
    }
	// ...
}
```
