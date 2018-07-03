# Android性能优化-UI优化

## 概述
> UI优化: 防止Android应用UI不合理导致的卡顿、掉帧问题

## UI优化
### Android GPU渲染机制
> App界面的卡顿大多数与GPU的渲染有关，Android渲染机制如下
> 
> ![](http://hukai.me/images/draw_per_16ms.png)
> 
> 如上图所示，Android系统每隔16ms（1000ms/60Hz = 16ms）发出VSYNC(**垂直同步**)信号，触发对UI进行渲染，如果每次渲染都成功，这样就能够达到流畅的画面所需要的60fps，为了能够实现60fps，这意味着程序的大多数操作都必须在16ms内完成。
> 
> ![](http://hukai.me/images/vsync_over_draw.png)
> 
> 如上图所示，如果你的某个操作花费时间是24ms，系统在得到VSYNC信号的时候就无法进行正常渲染，这样就发生了丢帧现象。那么用户在32ms内看到的会是同一帧画面。

### 引起渲染超时的原因
#### 1. 过度绘制
> ##### 什么是过度绘制？
> ** 过度绘制就是屏幕上的某个像素在同一帧的时间内被绘制了多次 **。 在多层次的UI结构里面，如果不可见的 UI 也在做绘制的操作，这就会导致某些像素区域被绘制了多次。 这就浪费大量的 CPU 以及 GPU 资源。
> ##### 如何发现过度绘制?
> ![](http://hukai.me/images/overdraw_options_view.png)
> 我们可以通过手机设置里面的 **开发者选项** ，打开 **显示过渡绘制区域（Show GPU Overdraw** 的选项，可以观察 UI 上的 Overdraw 情况。
> - 蓝色： 意味着overdraw 1倍。像素绘制了两次。大片的蓝色还是可以接受的（若整个窗口是蓝色的，可以摆脱一层）。
- 绿色： 意味着overdraw 2倍。像素绘制了三次。中等大小的绿色区域是可以接受的但你应该尝试优化、减少它们。 
- 淡红： 意味着overdraw 3倍。像素绘制了四次，小范围可以接受。
- 深红： 意味着overdraw 4倍。像素绘制了五次或者更多。这是错误的，要修复它们。

> Overdraw 有时候是因为你的UI布局存在大量重叠的部分，还有的时候是因为非必须的重叠背景
>	* 例如：某个 Activity 有一个背景，然后里面的 Layout 又有自己的背景，同时子 View 又分别有自己的背景。仅仅是通过移除非必须的背景图片，这就能够减少大量的红色 Overdraw 区域，增加蓝色区域的占比。这一措施能够显著提升程序性能。

#### 2. 布局冗余

#### 3. 频繁GC造成内存抖动




### 解决渲染问题的方法
> 1. 使用 Hierarchy Viewer 分析 UI 性能
> 2. 使用 GPU 呈现模式考核 UI 性能
> 
> ####  使用 TraceView 从代码层面分析性能问题
> 3.1 生成Trace文件
> 
> 代码很简单，当你调用开始代码的时候，系统会生产 trace 文件，并且产生追踪数据，当你调用结束代码时，会将追踪数据写入到 trace 文件中。
> ```Java
> Debug.startMethodTracing("test_trace");//开始 trace，保存文件到 "/sdcard/test_trace.trace"
> ...
> Debug.stopMethodTracing();//结束
```
>
>  3.2. 使用 Android Studio 生成 trace 文件
> Android Studio 内置的 Android Monitor 可以很方便的生成 trace 文件到电脑
> 
> ![](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/raw/master/resources/images/android_performance/ui_tracing.jpg)
> 
> 在 CPU 监控的那栏会有一个闹钟似的的按钮，未启动应用时是灰色； 启动应用后，这个按钮会变亮，点击后开始追踪，相当于代码调用 startMethodTracing；

> 当要结束追踪时再次点击这个按钮，就会生成 trace 文件了（文件可在 Caputures 中找到）。
> ![](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/raw/master/resources/images/android_performance/ui_tracing4.jpg)
> 
> 生成 trace 后 Android Studio 自动加载的 traceview 图形如下：

> ![](https://github.com/jeanboydev/Android-ReadTheFuckingSourceCode/raw/master/resources/images/android_performance/ui_tracing3.jpg)
> 
> 从这个图可以大概了解一些方法的执行时间、次数以及调用关系，也可以搜索过滤特定的内容。
> 
> 左上角可以切换不同的线程，这其实也是直接用 Android Studio 查看 trace 文件的缺点：无法直观地对比不同线程的执行时间。