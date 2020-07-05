# 1.1. 为什么使用Netty？

大卫·惠勒曾经说过：“计算机科学的所有问题，都能够在另一层面间接地解决”。作为一个NIO 客户-服务端框架，Netty提供了这样一种间接的解决方案。Netty简化了TCP/UDP服务的网络编程，但我们仍然可以使用低层次的TCP api，因为Netty提供了高层次的抽象。

## 1.1.1 不是所有的网络框架都是一样的

 在Netty中，“快和简单”并不意味着最终的应用程序会有可维护性和性能问题。从FTP、SMTP、HTTP、WebSocket、SPDY协议以及各种二进制和传统的文本协议中获得的经验，使得Netty的创始人非常注意Netty的设计。最终，Netty成功地提供了**易于开发、高性能、高稳定性、和高扩展**的优点。

  一些知名度高的公司和开源项目包括RedHat, Twitter, Infinispan, HornetQ, Vert.x, Finagle, Akka, Apache Cassandra, Elasticsearch,以及其他人的使用都有助于Netty，也可以说，Netty的一些特性也正是由于这些项目的需求。经过这些年的发展，Netty已经变得非常的知名并且是最常用的JVM网络框架之一。这在数个流行的开闭源项目中可体现的。事实上，Netty在2011年获得了[杜克选择奖](https://blogs.oracle.com/java/and-the-winners-arethe-dukes-choice-award)。

 并且在2011年，Netty的作者[Trustin Lee](https://twitter.com/trustin)离开了RedHat去了Twitter。至此，Netty项目变得独立于任何公司，并且在努力简化对它的贡献。RedHat和Twitter都使用Netty，所有在撰写本文的时候，这两家公司是Netty最大的贡献者一点也不足为奇。采用Netty的项目在增加，与此同时，个人贡献者也在增加。Netty的[用户社区](https://netty.io/community.html)很活跃，项目仍充满活力。

## 1.1.2 Netty有丰富的功能集

   当你阅读本书的章节时，你将学习使用到很多Netty的特性。图1.1 突出显示了一些支持的传输和协议，以简要概述Netty的架构。

![](../../.gitbook/assets/image%20%281%29.png)

除了其广泛支持的传输和协议外，Netty还在各个发展领域提供了好处：

<table>
  <thead>
    <tr>
      <th style="text-align:center">&#x53D1;&#x5C55;&#x9886;&#x57DF;</th>
      <th style="text-align:left">Netty&#x7279;&#x6027;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">&#x8BBE;&#x8BA1;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x7528;&#x4E8E;&#x5404;&#x79CD;&#x4F20;&#x8F93;&#x7C7B;&#x578B;&#x7684;&#x963B;&#x585E;&#x548C;&#x975E;&#x963B;&#x585E;&#x5957;&#x63A5;&#x5B57;&#x7684;&#x7EDF;&#x4E00;API</li>
          <li>&#x7075;&#x6D3B;&#x8FD0;&#x7528;</li>
          <li>&#x7B80;&#x5355;&#x4F46;&#x6709;&#x7528;&#x7684;&#x7EBF;&#x7A0B;&#x6A21;&#x578B;</li>
          <li>&#x771F;&#x6B63;&#x7684;&#x65E0;&#x8FDE;&#x63A5;&#x6570;&#x636E;&#x62A5;&#x5957;&#x63A5;&#x5B57;&#x652F;&#x6301;</li>
          <li>&#x903B;&#x8F91;&#x94FE;&#x4EE5;&#x4F7F;&#x91CD;&#x7528;&#x53D8;&#x5F97;&#x7B80;&#x5355;(&#x9644;:
            <a
            href="https://sq.163yun.com/blog/article/200313202701238272">&#x8D23;&#x4EFB;&#x94FE;&#x6A21;&#x5F0F;&#x7684;&#x4F7F;&#x7528;</a>)</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">&#x6613;&#x4E8E;&#x4F7F;&#x7528;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x5B8C;&#x5584;&#x7684;javadoc&#x548C;&#x5927;&#x91CF;&#x7684;&#x4F7F;&#x7528;&#x7528;&#x4F8B;</li>
          <li>
            <p>&#x9664;&#x4E86;JDK1.6&#xFF08;&#x53CA;&#x4EE5;&#x4E0A;&#xFF09;&#xFF0C;&#x4E0D;&#x518D;&#x9700;&#x8981;&#x5176;&#x4ED6;&#x7684;&#x4F9D;&#x8D56;&#x3002;&#x6709;&#x4E9B;&#x7279;&#x6027;&#x53EF;&#x80FD;&#x4F9D;&#x8D56;JDK1.7&#xFF08;&#x53CA;&#x4EE5;&#x4E0A;&#xFF09;</p>
            <p>&#x3001;&#x5176;&#x4ED6;&#x7279;&#x6027;&#x53EF;&#x80FD;&#x9700;&#x8981;&#x5176;&#x4ED6;&#x7684;&#x4F9D;&#x8D56;&#xFF0C;&#x4F46;&#x90FD;&#x662F;&#x53EF;&#x9009;&#x7684;&#x3002;</p>
          </li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">&#x6027;&#x80FD;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x66F4;&#x9AD8;&#x7684;&#x541E;&#x5410;&#xFF0C;&#x6BD4;Java API&#x66F4;&#x4F4E;&#x7684;&#x5EF6;&#x8FDF;</li>
          <li>&#x7531;&#x4E8E;&#x6C60;&#x548C;&#x91CD;&#x7528;&#xFF0C;&#x8D44;&#x6E90;&#x6D88;&#x8017;&#x8F83;&#x4F4E;</li>
          <li>&#x6700;&#x5C0F;&#x5316;&#x4E0D;&#x5FC5;&#x8981;&#x7684;&#x5185;&#x5B58;&#x590D;&#x5236;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">&#x5065;&#x58EE;&#x6027;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x4E0D;&#x4F1A;&#x7531;&#x4E8E;&#x8FC7;&#x5FEB;&#x3001;&#x8FC7;&#x6162;&#x3001;&#x91CD;&#x8F7D;&#x8FDE;&#x63A5;&#x800C;&#x9020;&#x6210;OutOfMemoryError</li>
          <li>&#x6D88;&#x9664;&#x5728;&#x9AD8;&#x901F;&#x7F51;&#x7EDC;&#x4E2D;NIO&#x5E94;&#x7528;&#x5E38;&#x89C1;&#x7684;&#x4E0D;&#x516C;&#x5E73;&#x8BFB;&#x5199;&#x6BD4;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">&#x5B89;&#x5168;&#x6027;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x5B8C;&#x6574;&#x7684;SSL/TLS&#x548C;StartTLS&#x652F;&#x6301;</li>
          <li>&#x53EF;&#x8FD0;&#x884C;&#x5728;&#x53D7;&#x9650;&#x73AF;&#x5883;&#x4E2D;&#xFF0C;&#x6BD4;&#x5982;Applet&#x548C;OSGI</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">&#x793E;&#x533A;</td>
      <td style="text-align:left">
        <ul>
          <li>&#x53D1;&#x5E03;&#x5FEB;&#x901F;&#x4E14;&#x53CA;&#x65F6;</li>
          <li>&#x6D3B;&#x8DC3;</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

    除了列出的特性 ，Netty还为JavaNIO中已知的bug和限制提出了解决方案，因此你不再需要处理这些问题。

    回顾一下Netty特性的概览，现在是时候仔细研究一下异步处理和背后的思想。NIO和Netty都使用了大量的异步代码，如果不了解与此相关的选择的影响，就会不能很好地使用它。下一节，我们将展示为什么我们需要异步API。

