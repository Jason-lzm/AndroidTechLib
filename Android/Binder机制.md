# Binder机制
## 什么是Binder
> Binder是Android系统进程间通信(IPC)的一种方式，
> 理解Binder对于理解整个Android系统有着非常重要的作用，如果对Binder不了解，就很难对Android系统机制有更深入的理解。

## 1. Binder 架构
![](https://img-blog.csdn.net/20170411180827946?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
	
> *  Binder采用C/S体系架构，由四个部分组成：** Client、Server、ServerManager、Binder驱动 , **其中 ServiceManager 用于管理系统中的各种服务。
> * Binder框架的实现分为两层 ** Java层和Native层 **，Java层的Binder通过JNI调用Native层Binder，Native层Binder通过ioctl的方式与Binder驱动(** /dev/binder **)进行通信

## 2. Binder机制

![](https://img-blog.csdn.net/20170411180950468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZnJlZWtpdGV5dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> * Server,Client,ServiceManager运行于用户空间，驱动运行于内核空间。Client、Server和Service Manager通过open和ioctl文件操作函数与Binder驱动程序进行通信。
> * Client进程与Server进程在内核空间通过共享内存完成数据的交换

> ###  用户空间与内核空间
>  Linux使用两级保护机制：0级供内核使用，3级供用户程序使用，将虚拟地址空间也分为两个部分，内核空间和用户空间。
>  * 每个进程有各自的私有用户空间（0～3G），这个空间对系统中的其他进程是不可见的。++用户空间的内存是进程隔离的++
>  * 最高的1GB字节虚拟内核空间则为所有进程以及内核所共享 ++内核空间的内存为所有进程及内核共享，用户空间只能通过系统调用访问++

> ### Binder通讯过程
> 1. 服务注册
> 	Server端通过ServiceManager注册服务，向Binder驱动的全局链表**binder_procs**和ServiceManager的**svcinfo**列表中插入服务端信息（服务的引用）
> 2. 获取服务引用
>	客户端通过ServiceManager查询**svcinfo**列表，获取Service代理
> 3. 调用服务
> 	客户端通过Service代理BinderProxy，将调用参数发送给ServiceManager，通过共享内存的方式使用内核方法 copy_from_user() 将我们的参数先拷贝到内核空间，这时我们的客户端进入等待状态，然后 Binder 驱动向服务端的 todo 队列里面插入一条事务，然后调用Service方法执行，执行完之后把执行结果通过 copy_to_user() 将内核的结果拷贝到用户空间（这里只是执行了拷贝命令，并没有拷贝数据，**binder只进行一次拷贝**），唤醒等待的客户端并把结果响应回来，这样就完成了一次通讯。

> ### 为什么Binder只进行了一次数据拷贝？
> Linux内核实际上没有从一个用户空间到另一个用户空间直接拷贝的函数，需要先用copy_from_user()拷贝到内核空间，再用copy_to_user()拷贝到另一个用户空间。为了实现用户空间到用户空间的拷贝，mmap()分配的内存除了映射进了接收方进程里，还映射进了内核空间。所以调用copy_from_user()将数据拷贝进内核空间也相当于拷贝进了接收方的用户空间，这就是Binder只需一次拷贝的‘秘密’。

> ### Binder线程池
> Binder 线程池：每个 Server 进程在启动时创建一个 binder 线程池，并向其中注册一个 Binder 线程；之后 Server 进程也可以向 binder 线程池注册新的线程，或者 Binder 驱动在探测到没有空闲 binder 线程时主动向 Server 进程注册新的的 binder 线程。对于**一个 Server 进程有一个最大 Binder 线程数限制，默认为16个 binder 线程**，例如 Android 的 system_server 进程就存在16个线程。对于所有 Client 端进程的 binder 请求都是交由 Server 端进程的 binder 线程来处理的。

https://blog.csdn.net/freekiteyu/article/details/70082302





