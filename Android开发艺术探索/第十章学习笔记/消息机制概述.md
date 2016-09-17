# Android消息机制 #
> Android消息机制其实就是Handler的运行机制，我们学习它的时候需要整明白**ThreadLocal**、**Looper**、**MessageQueue**和**Handler**

## 1. Handler 并不只是专门用于更新UI的 ##
Handler是Android消息机制的上层接口，在日常的开发中，我们只需要同Handler交互即可，而不用关心一个任务在线程中切换的细节。Handler的使用过程很简单，通过它可以轻松的将一个任务切换到Handler所在的线程中去执行。

很多人认为Handler的作用就是更新UI，其实，**更新UI仅仅是Handler的一个特殊使用场景**。

> 比如，有时候需要在子线程中做一些耗时的I/O操作，可能是文件的读取，也可能是网络的访问等等。当耗时的操作完成以后，需要在UI上做一些改变。这个时候，由于Android的单线程模型限制（即，UI的更新只能在主线程，不能在子线程，否则就会触发程序的异常），这个时候，我们就可以借助Handler将更新UI的操作切换到主线程去执行。

	<!--ANR Exception-->
	“Only the original thread that create a view hierarchy can touch its views”

### 1. Android系统为什么不允许在子线程中访问UI呢？ ###
	Android的UI控件是非线程安全的，在多线程中并发访问可能导致UI控件处于不可预期的状态。
### 2. 为什么系统不对UI控件的访问添加锁机制呢？ ###
 	1. 锁机制会让UI访问的逻辑变得复杂
	2. 锁机制会阻塞某些线程的执行，因为会降低UI访问的效率


## 2. Android的消息机制主要是指Handler的运行机制 ##
Handler的运行需要底层的MessageQueue和Looper作为支撑。

- MessageQueue：即，消息队列。它的内部存储了一组消息，以队列的形式对外提供插入和删除的操作。它虽然叫做消息队列，但是它的内部存储结构并不是真正的队列，而是采单链表的数据结构来存储消息列表。

- Looper：即，消息循环。它以无线循环的方式去查询在MessageQueue中是否有新的消息，如果有，就处理，否则就一直等待。

- ThreadLocal：ThreadLocal不是线程，它的作用是在不同的线程中互不干扰的存储并提供数据。

> Handler在被创建的时候会采用当前线程Looper来构造消息循环系统，由于ThreadLocal可以轻松获取每个线程的Looper，因此在Handler内部获取当前线程的Looper时，用的就是ThreadLocal

需要注意的是，**线程默认都是没有Looper的**，所以，如果需要使用Handler就必须为线程创建Looper。

**ActivityThread（主线程、UI线程）被创建的时候就会初始化Looper，因此可以直接在主线程使用Handler。**

### Handler的send方法工作过程图 ###

![](http://i.imgur.com/fFAiIyt.png)