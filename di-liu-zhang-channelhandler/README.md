# 第六章 ChannelHandler

本章涵盖：

* ChannelPipeline 
* ChannelHandlerContext
* ChannelHandler 
* Inbound versus outbound

接受连接或创建连接只是应用程序的一部分。尽管这些任务确实很重要，但另一方面通常更复杂，需要编写更多代码。这是传入数据和传出数据的处理。

Netty为您提供了一种强大的方法来对此进行存档。它允许用户接入（**Hook**）处理数据的ChannelHandler实现。使ChannelHandler更加强大的原因在于，您可以将它们链接起来，以便每个ChannelHandler实现都可以完成小任务。这可以帮助您编写干净且可重用的实现。

但是处理数据只是使用ChannelHandler可以做的一件事。您也可以禁止I / O操作，例如写请求（您将在本章后面看到更多示例）。所有这些都可以即时完成，这使其功能更加强大。





