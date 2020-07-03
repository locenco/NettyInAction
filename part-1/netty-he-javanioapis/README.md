# 第一章 Netty和Java NIO APIs

## 本章主要内容

* Netty架构
* 为什么需要非阻塞IO（NIO）
* 阻塞IO VS 非阻塞IO
* JDK-NIO实现的已知问题和Netty的解决方案

    本章虽然介绍了Netty，但重点是讲Java 阻塞IO（NIO）API。对于JVM网络编程的新手来说，本章是一个很好的开始。对Java开发老手来说，也是一个很好的复习。如果你熟悉NIO和NIO2，随时可跳到第二章，它将带你运行Netty程序并深入探讨。

    Netty是一个NIO客户端-服务端框架，它能够快速并简易地开发出网络应用，像协议服务器和客户端。Netty提供了一种新的方法去开发网络应用，并使开发变得轻量和可伸缩。它通过抽象出所涉及的复杂性、提供易用的API将业务逻辑网络处理代码解耦，来达到开发轻量可伸缩的目的。由于Netty是基于NIO实现的，所以整个Netty API都是异步的。

    通常来说，无论是Netty或者其他NIO API的网络应用，都需要解决可伸缩问题。Netty的关键组成部分是它的异步特性，本章将讨论同步（阻塞）和异步（非阻塞）IO来说明为什么使用异步编程和它是怎么解决可伸缩问题的。

    对于刚接触网络编程的新手，本章将帮助你理解网络编程，以及Netty是如何实现的。它说明了怎样使用基本的Java网络API，探讨了Java网络API的优点和缺点，并阐述了Netty是如何解决Java NIO问题的，像Epoll的BUG和内存泄漏问题。

    在本章的最后，你将理解Netty是什么以及它提供了什么，并且能够对Java的NIO和异步处理有更深入的理解，这将帮你更好地学习后面的章节。
