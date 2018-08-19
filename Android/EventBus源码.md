# EventBus源码解析

## 概述
 EventBus使用的是观察者模式（订阅发布模式），它由EventBus、Publisher、Subscriber三要素组成，它的整体工作流程如下:
 
![](https://upload-images.jianshu.io/upload_images/1074740-b464084e5950ebed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
> #### 三要素
> * Event 事件： EventBus中传输数据的类，可以是任意类型
> * Subscriber 事件订阅者： 在EventBus3.0之前我们必须定义以onEvent开头的那几个方法，分别是onEvent、onEventMainThread、onEventBackgroundThread和onEventAsync，而在3.0之后事件处理的方法名可以随意取，不过需要加上注解@subscribe()，并且指定线程模型，默认是POSTING。
> * Publisher 事件的发布者： 我们可以在任意线程里发布事件，一般情况下，使用EventBus.getDefault()就可以得到一个EventBus对象，然后再调用post(Object)方法即可。

 
##  EventBus的基本用法
### 自定义一个Event事件类
```Java
public class MessageEvent{
    private String message;
    public  MessageEvent(String message){
        this.message=message;
    }
    public String getMessage() {
        return message;
    }
 
    public void setMessage(String message) {
        this.message = message;
    }
}
```
### 事件发布者Publisher
EventBus发布事件很简单，只需要执行以下代码即可
```Java
EventBus.getDefault().post(messageEvent);
```

### 事件订阅者Subscriber
当我们需要在某个类如XXXActivity中处理此事件时，只需要做如下操作，此类便成为Subscriber
##### 1. 注册事件
```Java
@Override
protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main)；
     EventBus.getDefault().register(this)；
}
```
##### 2. 解除注册
```Java
@Override
protected void onDestroy() {
    super.onDestroy();
    EventBus.getDefault().unregister(this);
}
```
##### 3. 处理事件
```Java
@Subscribe(threadMode = ThreadMode.MAIN)
public void XXX(MessageEvent messageEvent) {
    ...
}
```
> EventBus3.0以后，使用注解即可方便的定义事件处理函数


## 实现原理
Event的原理整体结构如下图

![](https://upload-images.jianshu.io/upload_images/1074740-7aa7ebc49e6f8264.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
### 1. 构造方法
```Java
public static EventBus getDefault() {
    if (defaultInstance == null) {
        synchronized (EventBus.class) {
            if (defaultInstance == null) {
                defaultInstance = new EventBus();
            }
        }
    }
    return defaultInstance;
}

 public EventBus() {
        this(DEFAULT_BUILDER);
    }

public EventBus(EventBusBuilder builder) {
        subscriptionsByEventType = new HashMap<>();  //以事件类的class对象为键值，记录注册方法信息，值为一个Subscription的列表
        typesBySubscriber = new HashMap<>();   //以注册的类为键值，记录该类所注册的所有事件类型，值为一个Event的class对象的列表
        stickyEvents = new ConcurrentHashMap<>();  //记录sticky事件
        //三个Poster, 负责在不同的线程中调用订阅者的方法
        mainThreadPoster = new HandlerPoster(this, Looper.getMainLooper(), 10);
        backgroundPoster = new BackgroundPoster(this);
        asyncPoster = new AsyncPoster(this);
        ...
        //方法的查找类，用于查找某个类中有哪些注册的方法
        subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
                builder.strictMethodVerification, builder.ignoreGeneratedIndex);
        ...
        //后面是一些表示开关信息的boolean值以及一个线程池
    }

```
> getDefault()是一个典型的单例模式，
> 在构造方法中有几个重要的成员：
> 1. 两个Map
> * subscriptionsByEventType:  **记录每个Event事件有哪些订阅者**，Key = Event事件类的class对象，value = Subscription的列表； **post(messageEvent)时从此Map中查找到订阅的类，将Event事件分发给对应订阅者**
> * typesBySubscriber:  **记录一个订阅者注册了哪些事件**，
> 2. 三个Poster
> * mainThreadPoster、backgroundPoster、asyncPoster
> * 分别对应subscribe时的三种threadMode，表示将事件分发到不同线程中去处理
> * mainThreadPoster 继承了Handler，使用了Handler机制，backgroundPoster和asyncPoster则是继承自Runnable，同时使用了线程池

### 2. 注册
EventBus的注册分为两步，查找注册方法和注册（即保存注册信息）
1. 查找注册方法findSubscriberMethods()
	> 有两种查找方式
    > * findUsingReflection
    > 使用反射的方式进行查找，也就是遍历注册类，找出注册类的方法中@subscribe注解标记的方法
    > **使用反射的方法会影响运行时的性能**
    > * findUsingInfo
    > index方式，在编译期处理源码，生成有效的查找方法代码，避免了运行期的反射

2. 
    

