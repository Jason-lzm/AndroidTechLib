# Handler机制

## 定义
> Handler是Android线程通信的一种机制，最为常用的是其他线程通过Handler向主线程发送消息，更新主线程UI。

## Handler机制
> Android的Handler机制由四部分组成：Handler、Looper、Message、MessageQueue
> ### Handler
> * 功能：发送消息和处理消息，发消息可在任意线程，处理消息在Looper线程
> ++默认构造函数会与new此Handler的线程的Looper进行关联++
_ _ _
> ```Java
> public Handler(Callback callback, boolean async) {
        //打印内存泄露提醒log
        //获取与创建Handler线程绑定的Looper
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        //获取与Looper绑定的MessageQueue
        //因为一个Looper就只有一个MessageQueue，也就是与当前线程绑定的MessageQueue
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
> ```
> ### Looper
> * 功能：1. 关联当前线程， 2. 开启消息死循环，从消息队列中取出Handler发出的消息，在Looper线程回调Handler的handleMessage(Message msg)方法处理消息
_ _ _
> ##### 构造函数
> ```Java
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);//传入的参数代表这个Queue是否能够被退出
        mThread = Thread.currentThread();
    }
```
> ##### Looper.prepare()（在当前线程关联一个Looper对象）
```Java
 private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        //在当前线程绑定一个Looper
        //ThreadLocal线程局部变量，Looper对象是线程隔离的
        sThreadLocal.set(new Looper(quitAllowed));
    }
```
> ##### looper.loop()

```Java
public static void loop(){
        ...

        //进入死循环，不断地去取对象，分发对象到Handler中消费
        for (;;) {
            Message msg = queue.next(); // 不断的取下一个Message对象，在这里可能会造成堵塞。

            ...

            //分发Message，target即为msg对应的Handler
            msg.target.dispatchMessage(msg);

            ...
        }
    }
```
>### MessageQueue
>功能：消息队列，由Looper所持有，但是消息的添加是通过Handler进行；

```Java
　　final boolean enqueueMessage (Message msg, long when) {

              Message p = mMessages;

			synchronized (this) {
                  if (p == null || when == 0 || when < p.when) {
                         msg.next = p;
                         mMessages = msg;
                  }
                  else {
                         Message prev = null;
                         while (p != null && p.when <= when) {
                                prev = p;
                                p = p.next;
                         }

                         msg.next = prev.next;
                         prev.next = msg;
                  }
              }
              ……
　　}
```language
```
> ### Message
> 功能：消息实体，包含消息内容和target Handler（发给哪一个Handler处理，可以是自己）

## Handler注意
> 1. Looper与Thread是一体的，一个Thread只能有一个Looper
> 2. Handler可以有多个，默认与当前new的线程Looper关联，当然也可以传入其他Looper构造
> 3. Looper中用了ThreadLocal来保存当前的Looper对象，
> 	ThreadLocal：线程局部变量，存储的变量为每个线程单独持有，其他线程不能访问，
> 	他用一个ThreadLocalMap来保存对象，而这个map会赋值给当前Thread独有的一个成员threadlocals
> 4. Android里的主线程为什么不会因为Looper.loop()里的死循环卡死？
> 	因为MessageQueue为empty时，线程会被挂起，当有消息来时，就会唤醒线程处理消息