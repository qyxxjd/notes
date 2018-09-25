## RxEvent 使用示例

#### 订阅之后开始接收数据

```java
RxEvent mEvent = RxEvent.create();
// 发送数据
mEvent.sendEvent(1);
mEvent.sendEvent(2);
mEvent.sendEvent(3);
// 开始订阅
mEvent.subscribe(new Consumer<Object>() {
            @Override
            public void accept(Object object) throws Exception {
                Log.d("PublishEvent", object.toString());
            }
        });
mEvent.sendEvent(4);
mEvent.sendEvent(5);
mEvent.sendEvent(6);

// Logcat 输出: 4、5、6
```

#### 从订阅之前的最后一条数据开始接收

```java
RxEvent mEvent = RxEvent.createBehavior();
mEvent.sendEvent(1);
mEvent.sendEvent(2);
mEvent.sendEvent(3);
mEvent.subscribe(new Consumer<Object>() {
            @Override
            public void accept(Object object) throws Exception {
                Log.d("BehaviorEvent", object.toString());
            }
        });
mEvent.sendEvent(4);
mEvent.sendEvent(5);
mEvent.sendEvent(6);

// Logcat 输出: 3、4、5、6
```

#### 订阅所有数据

```java
RxEvent mEvent = RxEvent.createReplay();
mEvent.sendEvent(1);
mEvent.sendEvent(2);
mEvent.sendEvent(3);
mEvent.subscribe(new Consumer<Object>() {
            @Override
            public void accept(Object object) throws Exception {
                Log.d("ReplayEvent", object.toString());
            }
        });
mEvent.sendEvent(4);
mEvent.sendEvent(5);
mEvent.sendEvent(6);

// Logcat 输出: 1、2、3、4、5、6
```

#### 背压策略

|值|说明|
|-----|-----|
|`BackpressureStrategy.MISSING`|不缓存，也不丢弃数据|
|`BackpressureStrategy.ERROR`|抛出异常|
|`BackpressureStrategy.BUFFER`|缓存所有数据，直到内存溢出|
|`BackpressureStrategy.DROP`|丢弃最新的数据|
|`BackpressureStrategy.LATEST`|保留最新的数据|

`Subscription`接口有两个方法:  
`cancel()`用来切断订阅者与生产者之间的联系。  
`request()`用来向生产者申请可以消费的事件数量, 这样我们便可以根据本身的消费能力进行消费事件。如果不调用`request()`方法，则不会接收到任何数据。  
`Flowable` 在设计的时候采用了一种新的思路也就是：**响应式拉取** 的方式, 你请求多少, 我便发送多少数据给你。  

```java
private Subscription mSubscription;
private final class TestSubscriber implements Subscriber<Object> {
    private final String tag = "LogSubscriber";
    @Override
    public void onSubscribe(@NonNull Subscription s) {
        mSubscription = s;
        Log.d(tag, "onSubscribe");
        // 订阅关系建立时，拉取一条数据
        mSubscription.request(1);
    }

    @Override
    public void onNext(Object o) {
        // 处理完一条数据后，拉取下一条数据
        mSubscription.request(1);
        Log.d(tag, "onNext:" + o.toString());
    }

    @Override
    public void onError(Throwable t) {
        Log.e(tag, "onError:" + t.getMessage());
    }

    @Override
    public void onComplete() { }
}

// 开始订阅，指定背压策略为：事件消费不过来时，保留最新的一条数据
mEvent.subscribe(new TestSubscriber(), BackpressureStrategy.LATEST);
```
