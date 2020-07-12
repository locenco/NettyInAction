# 1.3 JVM上的阻塞 VS 非阻塞

    Web的持续增长，也增加了对能够处理其规模的网络应用程序的需求。在满足这些需求方面，效率变得很重要。幸运的是，Java自带了创建高效、可伸缩网络程序所需的工具。尽管早期的Java版本已经支持网络编程，但在Java 1.4才引入了NIO API，为写高效的网络程序铺平了道路。

    在Java 7中，引入了新的API（NIO.2\)。它允许我们编写异步网络代码，但尝试提供了一个比之前更高层的API。

    要在Java中完成与网络相关的任务，你可以采用一下两种方法之一：

* 使用IO，即阻塞IO
* 使用NIO，即新/非阻塞IO

> new or non-blocking？
>
> “N"通常意味非阻塞而不是新的。NIO已经存在很长时间了，现在没人再管它叫新IO了。大多数人把它当作非阻塞IO。

    图1.2展示了如何阻塞IO如何使用一个线程处理一个连接，这意味着在连接和线程之间有着一对一的关系，因此，阻塞IO的瓶颈在于你能在JVM中创建的线程数。

![&#x56FE;1.2 &#x963B;&#x585E;IO](../../.gitbook/assets/image%20%282%29.png)

   与此相反，图1.3 展示了阻塞IO怎样让你用一个选择器处理多个连接。

![&#x56FE;1.3 &#x975E;&#x963B;&#x585E;IO](../../.gitbook/assets/image%20%283%29.png)

     在脑海中记着这两张图，让我们更深入地了解阻塞IO和非阻塞IO。我将用一个简单的回声服务来演示IO和NIO的区别。回声服务是指，接收客户端请求并回复从客户端收到的消息。

## 1.3.1. 基于阻塞IO的回声服务

    回声服务的第一个版本基于阻塞IO,这可能是编写网络相关的程序最常见的方法。主要有两个原因：阻塞IO在早期版本的Java中已经存在，并且它相对易用。

除非遇到可伸缩的问题，不然使用阻塞IO是不成问题的。下面列出了EchoServer的实现。

{% code title="Listing 1.4 EchoServer v1: IO" %}
```java
public class PlainEchoServer {
	public void serve(int port) throws IOException {
		final ServerSocket socket = new ServerSocket(port); #1
		try {
			while (true) {
				final Socket clientSocket = socket.accept(); #2
				System.out.println("Accepted connection from " +
						clientSocket);
				new Thread(new Runnable() { #3
					@Override
					public void run() {
						try {
							BufferedReader reader = new BufferedReader(
									new
											InputStreamReader(clientSocket.getInputStream()));
							PrintWriter writer = new PrintWriter(clientSocket
									.getOutputStream(), true);
							while(true) { #4
								writer.println(reader.readLine());
								writer.flush();
							}
						} catch (IOException e) {
							e.printStackTrace();
							try {
								clientSocket.close();
							} catch (IOException ex) {
								// ignore on close 
							}
						}
					}
				}).start(); #5
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
#1 Bind server to port
#2 Block until new client connection is accepted 
#3 Create new thread to handle client connection 
#4 Read data from client and write it back 
#5 Start thread
```
{% endcode %}

如果你之前用Java写过网络程序，那这个表单你应该很熟悉。让我们停下来想一想，这种设计会有什么问题呢？

让我们再重新看下下面的代码：

```java
final Socket clientSocket = socket.accept(); 
new Thread(new Runnable() { 
 @Override 
 public void run() { 
 ... 
 } 
}).start();
```

每一个新的连接都需要一个新的线程。你可能会争辩说，我们可以使用线程池来摆脱创建线程的开销，但那只是暂时的。最基本的问题仍然存在：客户端的并发数受限于你能够同时拥有的线程数。当你的程序需要处理数以万计的客户端时，这将是个大问题。

在下一版本的EchoServer程序的演示中，因为使用了NIO，将不存在这个问题。在此之前，理解NIO概念的几个关键点是很重要的。

## 1.3.2 非阻塞IO基础

Java7引入了一个新的IO API叫NIO.2。你也可以叫它NIO或者NIO.2。尽管新的API也是异步的，但它与原来NIO的API和实现都是不同的。API并不是完全的不同，有一些共有的特征。例如，都实现了名为ByteBuffer的抽象来当作数据容器。

### ByteBuffer

ByteBuffer对NIO和Netty都是至关重要的。ByteBuffer可以在堆上分配，也可以直接分配，这意味着它存储在堆空间之外。通常情况下，将ByteBuffer传递给Channel时，使用直接分配是非常快的，但分配/取消分配的代价更高。在这两种情况下，ByteBuffer的API是一样的，它提供了**统一的方法访问和操作数据**。ByteBuffer**允许在相同的ByteBuffer实例间共享相同的数据，而不需要更多的内存复制**。它还允许切片和其他操作去限制数据的可见性。

> Slicing
>
> ByteBuffer的切片允许创建一个新的ByteBuffer，并与最初的ByteBuffer共享相同的数据，但只暴露了它的子区域。这对减小内存是非常有用的，并且允许访问数据的一部分

ByteBuffer典型的用法：

* 写数据到ByteBuffer
* 调用ByteBuffer.flip\(\) ，从读模式切换到写模式
* 从ByteBuffer读出数据
* 调用ByteBuffer.clear\(\)或者ByteBuffer.compact\(\)

当你写数据到ByteBuffer的时候，它通过更新buffer中的写索引的位置来追踪写入的数据量；它也可以是手动的。

当你准备写数据的时候，你调用ByteBuffer.flip\(\)，从读模式切换到写模式。调用ByteBuffer.flip\(\)之后，首先会设置ByteBuffer的limit为当前位置，然后更新当前的position为0。通过这种方式，你可以读取ByteBuffer中所有的数据。

如果再写数据到ByteBuffer，先切换到写模式，然后调用以下两种方法中的一种：

* ByteBuffer.clear\(\)--清空整个ByteBuffer
* Bytebuffer.compact\(\)--清除已经通过内存复制读取过的数据

ByteBuffer.compact\(\)移动所有未读的数据到ByteBuffer的开头，并且调整标记位置。ByteBuffer通常用法：

{% code title="Listing 1.5 Working with a ByteBuffer" %}
```java
Channel inChannel=....;
ByteBuffer buf=ByteBuffer.allocate(48);
int bytesRead=-1;
do{
	bytesRead=inChannel.read(buf); #1
	if(bytesRead!=-1){
		buf.flip(); #2
		while(buf.hasRemaining()){
			System.out.print((char)buf.get()); #3
		}
		buf.clear(); #4
	}
}while(bytesRead!=-1);
inChannel.close();
```
{% endcode %}

现在你已经理解了如何使用ByteBuffer，让我们继续讨论选择器的概念。

### Working With NIO Selectors

NIO API仍然是两个NIO API中使用最广泛的，它使用基于选择器的方法去处理网络事件和数据。

channe表示能够执行IO操作的实体的连接，像文件和套接字。

选择器是NIO的一个组件，它决定一个或者多个通道是否准备好读或者写。因此，一个选择器能够用来处理多个连接，通过这种方法减轻你在阻塞IO EchoServer示例中看到的线程-连接模型的需要。

使用选择器，一般需要下面这些步骤：

1. 创建一个或多个能够让打开的管道或套接字注册的选择器
2. 当一个管道注册了之后，你需要具体指定哪些事件是你感兴趣监听的。有四个可用的事件：

* OP\_ACCEPT 套接字接受操作的操作单位
* OP\_CONNECT 套接字连接操作的操作单位
* OP\_READ 套接字读操作的操作单位
* OP\_WRITE套接字写操作的操作单位

    3. 当管道注册完成后，调用Selector.select\(\)方法阻塞直到这些事件发生。

    4. 当方法非阻塞时，可以获得所有的SelectionKey的实例（其中包含对注册管道和选择的Ops的引用）并且做一些事情。

        接下来你做什么取决于哪个操作是准备好的，一个SelectionKey可以在任何给定的时间内包含超过一个操作。

这个是如何工作的呢？让我们实现一个非阻塞的EchoServer版本。你有机会更详细地处理这两个NIO实现，你将会发现ByteBuffer是至关重要的。

## 1.3.3 基于非阻塞IO的回声服务

以下的EchoServer使用了异步IO，它允许在一个线程中服务数以万计的客户端并发。

{% code title="Listing 1.6 EchoServer v2: NIO" %}
```java
public class PlainNioEchoServer {
	public void serve(int port) throws IOException {
		System.out.println("Listening for connections on port " + port); 

		ServerSocketChannel serverChannel = ServerSocketChannel.open();
		ServerSocket ss = serverChannel.socket();
		InetSocketAddress address = new InetSocketAddress(port);
		//将服务绑定到端口
		ss.bind(address); 
		serverChannel.configureBlocking(false);
		Selector selector = Selector.open();
		//在选择器注册管道，并监听OP_ACCEPT（接受连接）事件
		serverChannel.register(selector, SelectionKey.OP_ACCEPT); #2
		while (true) {
			try {
				//阻塞直到被选择
				selector.select(); #3
			} catch (IOException ex) {
				ex.printStackTrace();
				// handle in a proper way 
				break;
			}
			//获取所有的SelectedKey实例
			Set readyKeys = selector.selectedKeys(); #4
			Iterator iterator = readyKeys.iterator();
			while (iterator.hasNext()) {
				SelectionKey key = (SelectionKey) iterator.next();
				//从迭代器中移除
				iterator.remove(); #5
				try {
					if (key.isAcceptable()) {
						ServerSocketChannel server = (ServerSocketChannel)
								key.channel();
						//接收客户端连接
						SocketChannel client = server.accept(); #6
						System.out.println("Accepted connection from " +
								client);
						client.configureBlocking(false);
						//注册连接到选择器，并且设置接收的ByteBuffer
						client.register(selector, SelectionKey.OP_WRITE |
								SelectionKey.OP_READ, ByteBuffer.allocate(100)); #7
					}
					//检测SelectedKey是否可读
					if (key.isReadable()) { #8
						SocketChannel client = (SocketChannel) key.channel();
						ByteBuffer output = (ByteBuffer) key.attachment();
						//读数据到ByteBuffer 
						client.read(output); #9
					}
					//检测SelectedKey是否可写
					if (key.isWritable()) { #10
						SocketChannel client = (SocketChannel) key.channel();
						ByteBuffer output = (ByteBuffer) key.attachment();
						output.flip();
						//写数据从ByteBuffer到管道
						client.write(output); #11
						output.compact();
					}
				} catch (IOException ex) {
					key.cancel();
					try {
						key.channel().close();
					} catch (IOException cex) {
					}
				}
			}
		}
	}
}
#1 Bind server to port 
#2 Register the channel with the selector to be interested in new Client connections that get accepted 
#3 Block until something is selected 
#4 Get all SelectedKey instances 
#5 Remove the SelectedKey from the iterator 
#6 Accept the client connection 
#7 Register connection to selector and set ByteBuffer 
#8 Check for SelectedKey for read 
#9 Read data to ByteBuffer 
#10 Check for SelectedKey for write 
#11 Write data from ByteBuffer to channel
```
{% endcode %}

这个例子比之前的版本更复杂。这种复杂性是一种折中，异步代码通常比同步代码复杂的多。

通常来讲, 原始的NIO和新的NIO.2是相似的, 但它的实现是不同的. 我们将看他们的不同，并且实现第三个版本的EchoServer。

## 1.3.4 基于NIO.2的回声服务

不像NIO的实现，NIO.2允许你发出IO操作，并且提供一个完成处理器（CompletionHandler类）。这个完成处理器在操作完成后执行。完成处理器是由系统底层执行的，实现对开发者隐藏。它保障管道中同时只有一个CompletionHandler执行。这种方法有助于简化代码，因为它消除了多线程执行带来的复杂性。

NIO和NIO.2主要区别在于，不用再检测事件是否在管道上发生了，然后触发一些操作。在NIO.2，你可以触发一个IO操作，并且注册一个完成处理器在上面，一旦操作完成，处理器将会被通知。这样就不需要创建程序逻辑来检测是否完成，因为这本身就会造成不必要的处理。

让我们来看看基于NIO.2实现的回声服务。

```java
public class PlainNio2EchoServer {
	public void serve(int port) throws IOException {
		System.out.println("Listening for connections on port " + port);
		final AsynchronousServerSocketChannel serverChannel =
				AsynchronousServerSocketChannel.open();
		InetSocketAddress address = new InetSocketAddress(port);
		serverChannel.bind(address); #1
		final CountDownLatch latch = new CountDownLatch(1);
		serverChannel.accept(null, new
				CompletionHandler<AsynchronousSocketChannel, Object>() { #2
					@Override
					public void completed(final AsynchronousSocketChannel channel,
										  Object attachment) {
						serverChannel.accept(null, this); #3
						ByteBuffer buffer = ByteBuffer.allocate(100);
						channel.read(buffer, buffer,
								new EchoCompletionHandler(channel)); #4
					}
					@Override
					public void failed(Throwable throwable, Object attachment) {
						try {
							serverChannel.close(); #5
						} catch (IOException e) {
							// ingnore on close
						} finally {
							latch.countDown();
						}
					}
				});
		try {
			latch.await();
		} catch (InterruptedException e) {
			Thread.currentThread().interrupt();
		}
	}
	private final class EchoCompletionHandler implements
			CompletionHandler<Integer, ByteBuffer> {
		private final AsynchronousSocketChannel channel;
		EchoCompletionHandler(AsynchronousSocketChannel channel) {
			this.channel = channel;
		}
		@Override
		public void completed(Integer result, ByteBuffer buffer) {
			buffer.flip();
			channel.write(buffer, buffer, new CompletionHandler<Integer,
					ByteBuffer>() { #6
				@Override
				public void completed(Integer result, ByteBuffer buffer) {
					if (buffer.hasRemaining()) {
						channel.write(buffer, buffer, this); #7
					} else {
						buffer.compact();
						channel.read(buffer, buffer,
								EchoCompletionHandler.this); #8
					}
				}
				@Override
				public void failed(Throwable exc, ByteBuffer attachment) {
					try {
						channel.close();
					} catch (IOException e) {
						// ingnore on close
					}
				}
			});
		}
		@Override
		public void failed(Throwable exc, ByteBuffer attachment) {
			try {
				channel.close();
			} catch (IOException e) {
				// ingnore on close
			}
		}
	}
} 
		#1Bind Server to port
		#2 Start to accept new Client connections. Once one is accepted the CompletionHandler will get called.
		#3 Again accept new Client connections
		#4 Trigger a read operation on the Channel, the given CompletionHandler will be notified once
		something was read
		#5 Close the socket on error
		#6 Trigger a write operation on the Channel, the given CompletionHandler will be notified once
		something was written
		#7 Trigger again a write operation if something is left in the ByteBuffer
		#8 Trigger a read operation on the Channel, the given CompletionHandler will be notified once
		something was read
```

乍一看，它比之前用NIO的实现代码量更多。但注意，NIO.2处理了线程并且提供了事件循环。这种方法简化了构建多线程IO程序所需的代码，尽管在这个例子中并不是这样。随着程序复杂性的提高，NIO.2的收益将越来越明显，你将写出干净的代码。

下一节，我们将看JDK的NIO实现中存在的问题。

