# ANR
## 定义
* Application Not Responding 即应用程序无响应

## ANR分类
 1. KeyDispatchTimeout(**5 seconds**) --**主要类型**
	按键或触摸事件在特定时间内无响应
 2. BroadcastTimeout(**10 seconds**)
	BroadcastReceiver在特定时间内无法处理完成
 3. ServiceTimeout(**20 seconds**) --**小概率类型**
	Service在特定的时间内无法处理完成

## 避免ANR的方法
  不在UI线程中做耗时操作

## ANR问题定位及分析方法
  1. 看log
  2. 从trace文件中查看调用栈 adbpull data/anr/traces.txt ./mytraces.txt
  3. 看代码
  4.  仔细查看ANR的成因（iowait?block?memoryleak?）

	### 看log步骤
    1.1 ANR问题，搜索“**ANR ”(加空格)**关键字，快速定位到关键事件log位置 
    1.2 ForceClose或其他异常退出问题，搜索**“Fatal”**关键字，定位到关键log位置
    1.3  定位到关键事件信息后 ， 如果信息不够明确的，再去搜索应用程序包的虚拟机信息(**“Dalvik Thread”**关键字) ，查看具体的进程和线程跟踪的日志，来定位到代码 。

http://www.cnblogs.com/wanqieddy/archive/2013/12/26/3492373.html