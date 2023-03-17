#### Looper
- 构造函数创建MessageQueue和Thread，Thread指向当前线程
- Looper的prepare方法内，一个线程只能关联一个Looper，如果已存在Looper，则抛出异常；如果不存在，new一个Looper并绑定到当前线程`ThreadLocal.set(new Looper(boolean))`
- Looper的loop方法内，拿到当前Looper和绑定的MessageQueue，然后死循环，不断从MessageQueue取出Message，然后通过msg.target.despatchMessage(message)开始分发消息，分发完通过msg.recycleUnchecked()进行标记


#### Handler
- handler.enqueunMessage方法
- 设置msg.target=this,绑定一个Handler对象。同时调用MessageQueue的enqueueMessage方法，将Message添加到MessageQueue。
- handler.despatchMessage方法
- 此方法在上面looper.loop里面调用，内部回调handleMessage方法


#### MessageQueue
- MessageQueue内部是通过一个单链表的数据结构来维护消息列表，单链表在插入和删除上比较有优势。
- MessageQueue主要包含两个操作：插入和读取。
- 插入操作：enqueueMessage的作用是往消息队列中插入一条消息
- 读取操作：next的作用是从消息队列中取出一条消息并将其从消息队列中移除。
