# 一、定义 #
Looper在Android的消息机制中扮演着消息循环的角色。

Looper会不停的从MessageQueue中查看是否有新的消息到来，如果有新消息就会立刻处理，否则就一直阻塞在哪里。
# 二、原理 #




### 1. Looper的构造方法 ###
从Looper的构造方法，我们可以看出，Looper在构造方法内会创建一个MessageQueue，同时将当前线程的对象保存起来。

	private Looper(boolean quitAllowed){
	  mQueue = new MessageQueue(quitAllowed);
	  mThread = Thread.currentThread();
	}

### 2. Looper的创建 ###
我们知道，Handler的工作需要Looper，没有Looper的线程就会报错，那么怎么样才能创建一个Looper呢？很简单，通过`Looper.prepare()`即可为当前线程创建一个Looper，接着通过`Looper.loop()`来开启消息循环即可。

	new Thread("Thread#2"){
	  @override
	  public void run(){
	    Looper.prepare();
	    Handler handler = new Handler();
	    Looper.loop(); 
	  }
	}.start();

> Looper除了prepare方法之外，还提供了prepareMainLooper方法，这个方法主要是给主线程（ActivityThread）创建Looper使用的，它本质上也是通过prepare方法来实现的。但由于主线程Looper的特殊性，所以Looper提供了一个getMainLooper方法，通过它可以在**任何地方**获取到主线程的Looper。

> Looper是可以退出的。Looper提供了quit和quitSafely来退出一个Looper。二者的区别是：quit会直接退出Looper，而quitSafely只是设定一个退出标记，然后把消息队列中的已有消息处理完毕后才安全退出。

> Looper退出后，通过Handler发送的消息会失败，这个时候Handler的send方法会返回false。

> 在子线程中，如果手动为其创建了Looper，那么在所有的操作完成后，应该调用quit方法来终止消息循环。否则这个子线程会一直处于等待（阻塞）的状态，而如果推出Looper以后，这歌线程会立刻终止。因为，建议在不需要的时候要终止Looper。

### 3. Looper的启动（消息循环系统的启动） ###
Looper最终要的一个方法是loop方法，只有调用了loop后，消息循环系统才会真正地起作用。


