
#  一、 定义 #
**ThreadLocal** 是一个线程内部的 **数据存储类**，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。

# 二、 使用场景 #
在日常的开发中用到ThreadLocal的地方较少，但是，在某些特殊的场景下，通过ThreadLocal可以轻松地实现一些看起来很复杂的功能（这一点在Android源码中也有所体现，比如Looper、ActivityThread以及AMS中都用到了ThreadLocal）。
## 1.如何判断什么时候用ThreadLocal比较合适？ ##
当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal。

比如对于Handler来说，它要获取当前线程的Looper，很显然，Looper的作用域就是线程并且不同的线程具有不同的Looper，这个时候通过ThreadLocal就可以轻松实现Looper在线程中的存取。

## 2.如果不采用ThreadLocal，又该怎么办？ ##
在Android系统中，如果没有采用ThreadLocal，那么系统就必须提供一个全局的哈希表功Handler查找指定线程的Looper，这样一来，就必须提供一个类似于LooperManager的类了，但是系统并没有这么做，而是选择了ThreadLocal，这一点再一次证实了ThreadLocal所带来的好处。

# 三、 举个栗子 #

定义一个ThreadLocal对象，这里选择Boolean类型的，如下所示。

	private ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<Boolean>();

然后分别在 **主线程**、**子线程1**、**子线程2**中设置和访问它的值，如下所示。

	mBooleanThreadLocal.set(true);
	Log.d(TAG,"[Thread#main]mBooleanThreadLocal="+mBooleanThreadLocal.get());
	new Thread("Thread#1"){
	  @overide
	  public void run(){
	    mBooleanTreadLocal.set(false);
	    Log.d(TAG,"[Thread#1]mBooleanThreadLocal="+mBooleanThreadLocal.get());
	  }
	}.start();
	new Thread("Thread#2"){
	  @overide
	  public void run(){
	    Log.d(TAG,"[Thread#2]mBooleanThreadLocal="+mBooleanThreadLocal.get());
	  }
	}.start();

运行结果如下。

	D/TestActivity:[Thread#main]mBooleanThreadLocal=true
	D/TestActivity:[Thread#1]mBooleanThreadLocal=false
	D/TestActivity:[Thread#2]mBooleanThreadLocal=null

## 分析 ##
在上面的代码中，主线程中设置了mBooleanThreadLocal的值为true，子线程1中设置了mBooleanThreadLocal为false，子线程2中没有设置mBooleanThreadLocal的值。然后分别在3个线程中通过get方法访问mBooleanThreadLocal的值，根据ThreadLocal的特性，主线程中应该是true，子线程1中应该是false，由于子线程2中没有给mBooleanThreadLocal赋值，所以，子线程2中应该是null。
# 四、 内部实现 #

ThreadLocal是一个泛型类，它的定义为 `public class ThreadLocal<T>`。

	public void set(T value){
	  Thread currentThread = Thread.currentThread();
	  Values values = values(currentThread);
	  if(values==null){
	    values = initializeValues(currentThread);
	  }
	  values.put(this, value);
	}
	/** 在table数组中，如果table[index]存储的是ThreadLocal的reference，那么table[index+1]处存储的必定是ThreadLocal的值 **/
	public T get(){
	  Thread currentThread = Thread.currentThread();
	  Values values = values(currentThread);
	  if(values==null){
	    Object[] table = values.table;
	    int index = hash & values.mask;
	    if(this.reference == table[index]){
		  return (T) table[index + 1];    
	    }
	  }else{
	    values = initializeValues(currentThread);
	  }
	  return (T) values.getAfterMiss(this);
	}
从ThreadLocal的set和get方法不难看出，它们所操作的对象都是**当前线程**的LocalValue对象的table数组，**因此在不同线程中访问同一个ThreadLocal的set和get方法，它们对ThreadLocal所做的读/写操作仅限于各自线程的内部**，这就是为什么ThreadLocal可以在多个线程中互不干扰的存储和修改数据。

理解ThreadLocal的实现方式有助于理解Looper的工作原理。