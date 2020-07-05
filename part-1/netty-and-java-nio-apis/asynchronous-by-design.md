# 1.2. 异步设计

    整个Netty API都是异步的。异步处理并不是新的概念；这个概念已经出来一段时间了。然而，IO近来常常是瓶颈，并且异常处理变得日益重要。它是如何工作的和有多少不同的模式可用呢？

    异步处理鼓励你更有效地使用你的资源，通过允许你开始一个任务并获得回调，而不是等待任务完成。你可以在任务运行的时候做其他的事情。本节解释使用或实现一个异步API的常用方法，并且讨论这两种技术的不同。

## 1.2.1 回调

    回调是异步处理中经常使用的技术。回调传给方法，并在方法完成后执行。你可以在JavaScript中找到这种模式，其中回调是语言的核心。下面列出了如何用这种技术获取数据。

{% code title="Callback example" %}
```java
public interface Fetcher {
	void fetchData(FetchCallback callback);
}
public interface FetchCallback {
	void onData(Data data);
	void onError(Throwable cause);
}
public class Worker {
	public void doWork() {
		Fetcher fetcher = ...
		fetcher.fetchData(new FetchCallback() {
			@Override
			public void onData(Data data) { 
				System.out.println("Data received: " + data);
			}
			@Override
			public void onError(Throwable cause) { 
				System.err.println("An error accour: " + cause.getMessage());
			}
		}); 
		...
	}
}
```
{% endcode %}

    fetcher.fetchData\(\)方法使用一个FetchCallback的参数类型，当数据被获取或者发生错误时调用。

        对于每一种情况，它提供一种方法：

* FetchCallback.onData\(\)---如果获取数据时没有错误则调用
* FetchCallback.onError\(\)---如果获取操作的期间收到错误，则调用

     因此，你可以将这些方法从调用者线程移动到其他线程执行。这就不能够保证FetchCallback的方法什么时候会被调用。

    回调方法的一个问题是，当你用不同的回调链接许多异步方法时，会造成意大利面式的代码。一些人认为这种方法会造成难以阅读的代码，但我认为更多的是品位和风格的问题。例如，基于JavaScript的Node.js，现在变得越来越流行。它大量使用了回调，然而很多人发现它易于阅读和开发程序。

## 1.2.2 Futures

   第二种技术是使用Futures，一个Future是一个抽象，它代表一个可能在未来某个时刻可用的值。一个Future对象要么持有计算的结果，亦或者持有一个错误计算的异常。

    Java在java.util.concurrent的包中附带了一个Future接口，通过他的执行器在异步处理中使用它。

    例如，在下面的例子中，无论什么时候你将一个线程对象传给ExecutorService.submit\(\)方法，你会获得一个返回的Future，用它来检查执行是否完成。

{% code title="Future example via ExecutorService" %}
```java
	ExecutorService executor = Executors.newCachedThreadPool();
	Runnable task1 = new Runnable() {
		@Override
		public void run() {
			doSomeHeavyWork();
		}
		...
	}
	Callable<Interger> task2 = new Callable() {
		@Override
		public Integer call() {
			return doSomeHeavyWorkWithResul();
		}
        ...
	}
	Future<?> future1 = executor.submit(task1);
	Future<Integer> future2 = executor.submit(task2);
	while(!future1.isDone()  !future2.isDone())
	{
 		...
		// do something else
		...
	}
```
{% endcode %}

   你也可以使用这个技术在你自己的API里。例如，你可以实现一个使用了Future的Fetcher。代码如下。

{% code title="Future usage for Fetcher" %}
```java
public interface Fetcher {
		Future<Data> fetchData();
	}
	public class Worker {
		public void doWork() {
			Fetcher fetcher = ...
			Future<Data> future = fetcher.fetchData();
			try {
				while(!fetcher.isDone()) { 
           ...
					// do something else 
				}
				System.out.println("Data received: " + future.get());
			} catch (Throwable cause) {
				System.err.println("An error accour: " +
						cause.getMessage());
			}
		}
	} 
```
{% endcode %}

    检查Future是否已经做完，当没有的时候，做一些其他的事情。

   有时候，使用Future会感觉很丑陋，因为你需要间隔性地去检测Future的状态，看它是否已经完成。而Callback则是完成的时候直接被通知。

    在了解了异步执行最常见的技术之后，你或许会想，哪一种最好。这是没有答案的，对于Netty来说，事实上，是将两者的优点融合起来提供给大家。

   下一章提供了在JVM上编写网络应用的介绍，首先使用的是阻塞、其次使用了NIO和NIO2的API。这些基础对本书后续章节的学习是至关重要的。如果你对Java网络API非常熟悉，可快速扫描下一章节来复习你的记忆。

