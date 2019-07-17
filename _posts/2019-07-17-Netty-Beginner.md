---
layout:     post
title:      Netty 入门教程
subtitle:    "\"Netty Beginner\""
date:       2019-07-17
author:     Lin
header-img: img/post-bg-netty0717.jpeg
catalog: true
tags:
    - Netty
    - Java
---

## 前言

Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。

Netty4的官方网站是：<http://netty.io/>

Netty 是一个广泛使用的 Java 网络编程框架（Netty 在 2011 年获得了Duke's Choice Award，见https://www.java.net/dukeschoice/2011）。它活跃和成长于用户社区，像大型公司 Facebook 和 Instagram 以及流行 开源项目如 Infinispan, HornetQ, Vert.x, Apache Cassandra 和 Elasticsearch 等，都利用其强大的对于网络抽象的核心代码。

* 设计
  - 针对多种传输类型的统一接口 - 阻塞和非阻塞
  - 简单但更强大的线程模型
  - 真正的无连接的数据报套接字支持
  - 链接逻辑支持复用
* 易用性
  - 完善的Javadoc
  - 全面的代码示例
* 性能
  - 比核心的 Java API 更好的吞吐量，较低的延时
  - 资源消耗更少，这个得益于共享池和重用
  - 减少内存拷贝
* 健壮性
  - 消除由于慢、快、或重载连接产生的OutOfMemoryError
  - 消除经常发现在 NIO 在高速网络中的应用中的不公平读/写比
* 安全
  - 完整的 SSL/ TLS 和 StartTLS 的支持
  - 运行在受限的环境例如 Applet 或 OSGI
* 社区
  - 社区完善、更新/发布频繁

### 背景1 - Reactor模型

wiki:

> The reactor design pattern is an event handling pattern for handling service requests delivered concurrently to a service handler by one or more inputs. The service handler then demultiplexes the incoming requests and dispatches them synchronously to the associated request handlers.

几个关键点:

* 事件驱动(event handling)
* 可以处理一个或多个输入源(one or more inputs)
* 通过Service Handler同步的将输入事件(Event)采用多路复用分发给相应的Request Handler(多个)处理

更多参考: <https://my.oschina.net/u/1859679/blog/1844109>

### 背景2 - Java网络编程(BIO)

经典的BIO服务端:

* 一个主线程监听某个port，等待客户端连接
* 当接收到客户端发起的连接时，创建一个新的线程去处理客户端请求
* 主线程重新回到监听port，等待下一个客户端连接

缺点:

* 每个新的客户端Socket连接，都需要创建一个Thread处理，将会创建大量的线程
* 线程开销较大，连接多时，内存耗费大，CPU上下文切换开销也大

### 背景3 - Java NIO

Java NIO 由以下几个核心部分组成：

* Channels
* Buffers
* Selectors

传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

NIO和传统IO（一下简称IO）之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。

更多「NIO相关基础篇」参考: <https://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247483907&idx=1&sn=3d5e1384a36bd59f5fd14135067af1c2&chksm=fb0be897cc7c61815a6a1c3181f3ba3507b199fd7a8c9025e9d8f67b5e9783bc0f0fe1c73903&scene=21#wechat_redirect>

以及[带你用生活大白话理解 NIO](https://mp.weixin.qq.com/s/pSUXoLrwQeaxsI2LI-13EA)

## Netty的重要组件

下面枚举所有的Netty应用程序的基本构建模块，包括客户端和服务端。

#### BOOTSTRAP

Netty 应用程序通过设置bootstrap(引导)类的开始，该类提供了一个用于应用程序网络配置的容器。Netty有两种类型的引导: 客户端(Bootstrap)和服务端(ServerBootstrap)

#### CHANNEL

底层网络传输API必须提供给应用I/O操作的接口，传入(入站)或者传出(出站)数据的载体，如读，写，连接，绑定等等。对于我们来说，这结构几乎总是会成为一个"socket"。

#### CHANNELHANDLER

ChannelHandler 支持很多协议，并且提供用于数据处理的容器。我们已经知道ChannelHandler由特定事件触发。ChannelHandler可专用于几乎所有的动作，包括一个对象转为字节，执行过程中抛出的异常处理。

常用的一个接口是 ChannelInboundHandler，这个类型接收到入站事件(包括接收到的数据)可以处理应用程序逻辑。
当你需要提供相应时，你也可以从ChannelInboundHandler冲刷数据。一句话，业务逻辑经常存活于一个或者多个ChannelInboundHandler。

#### CHANNELPIPELINE

ChannelPipline提供了一个容器给 ChannelHandler链并提供了一个API用于管理沿着链入站和出站事件的流动。每个Channel都有自己的ChannelPipeline，当Channel创建时自动创建的。

#### EVENTLOOP

EventLoop 用于处理 Channel 的 I/O 操作，控制流、多线程和并发。一个单一的 EventLoop通常会处理多个 Channel 事件。一个 EventLoopGroup 可以含有多于一个的 EventLoop 和 提供了一种迭代用于检索清单中的下一个。

#### CHANNELFUTURE

Netty 所有的 I/O 操作都是异步。因为一个操作可能无法立即返回，我们需要有一种方法在以后确定它的结果。
出于这个目的，Netty 提供了接口 ChannelFuture，它的 addListener 方法注册了一个 ChannelFutureListener ，当操作完成时，可以被异步通知（不管成功与否）。

以上组件的关系:

![](https://i.loli.net/2019/07/17/5d2f121ab23d365116.jpg)

几点重要的约定:

* 一个EventLoopGroup包含一个或多个EventLoop
* 一个EventLoop在其生命周期内只能和一个Thread绑定
* EventLoop处理的I/O事件都由它绑定的Thread处理
* 一个Channel在其生命周期内，只能注册于一个EventLoop
* 一个EventLoop可能被分配处理多个Channel。也就是EventLoop与Channel是1:n的关系
* 一个Channel上的所有ChannelHandler的事件由绑定的EventLoop中的I/O线程处理
* 不要阻塞Channel的I/O线程，可能会影响该EventLoop中其他Channel事件处理

## 第一个 Netty 应用: Echo client / server

*本应用的源码请见 [netty仓库](https://github.com/netty/netty)中的example目录。*

接下来，我们来构建一个完整的Netty客户端和服务器，更完整地了解Netty的API是如何实现客户端和服务器的。

先来看看 Netty 应用 - Echo client/server 总览:

![](https://i.loli.net/2019/07/17/5d2f125300c6332893.jpg)

echo应用的客服端和服务器的交互很简单: 客户端启动后，建立一个连接并发送一个或多个消息到服务端，服务端接受到的每个消息再返回给客户端。

#### 服务端代码

* 一个信息处理器(handler): 这个实现是服务端的业务逻辑部分，当连接创建后和接收信息后的处理类。
* 服务器: 主要通过`ServerBootstrap`设置服务器的监听端口等启动部分。

###### EchoServerHandler

通过继承`ChannelInboundHandlerAdapter`，这个类提供了默认的`ChannelInboundHandler`实现，只需覆盖以下的方法:

* channelRead() - 每个消息入站都会调用
* channelReadComplete() - 通知处理器最后的channelRead()是当前批处理中的最后一条消息时调用
* exceptionCaught() - 捕获到异常时调用

```java
@ChannelHandler.Sharable // 标识这类的实例之间可以在 channel 里面共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf in = (ByteBuf) msg;
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));
        ctx.write(in); // 将所接收的消息返回给发送者
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER) // 冲刷所有待审消息到远程节点。关闭通道后，操作完成
            .addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

###### EchoServer

创建`ServerBootstrap`实例来引导服务器，本服务端分配了一个`NioEventLoopGroup`实例来处理事件的处理，如接受新的连接和读/写数据，然后绑定本地端口，分配`EchoServerHandler`实例给Channel，这样服务器初始化完成，可以使用了。

```java
public class EchoServer {

    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public void start() throws Exception {
        NioEventLoopGroup group = new NioEventLoopGroup(); // 创建 EventLoopGroup

        try {
            ServerBootstrap bootstrap = new ServerBootstrap(); // 创建 ServerBootstrap
            bootstrap.group(group)
                    .channel(NioServerSocketChannel.class) // 指定使用 NIO 的传输 Channel
                    .localAddress(new InetSocketAddress(port)) // 设置 socket 地址使用所选的端口
                    .childHandler(new ChannelInitializer<SocketChannel>() { // 添加 EchoServerHandler 到 Channel 的 ChannelPipeline
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new EchoServerHandler());
                        }
                    });

            ChannelFuture future = bootstrap.bind().sync(); // 绑定的服务器;sync 等待服务器关闭
            System.out.println(EchoServer.class.getName() + " started and listen on " + future.channel().localAddress());
            future.channel().closeFuture().sync(); // 关闭 channel 和 块，直到它被关闭
        } finally {
            group.shutdownGracefully().sync(); // 关闭 EventLoopGroup，释放所有资源。
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 4567;
        if (args.length == 1) {
            port = Integer.parseInt(args[0]);
        }
        new EchoServer(port).start(); // 设计端口、启动服务器
    }
}
```

#### 客户端代码

客户端要做的是:

* 连接服务器
* 发送消息
* 等待和接受服务器返回的消息
* 关闭连接

###### EchoClientHandler

继承`SimpleChannelInboundHandler`来处理所有的事情，只需覆盖三个方法:

* channelActive() - 服务器的连接被建立后调用
* channelRead0() - 从服务器端接受到消息调用
* exceptionCaught() - 捕获异常处理调用

```java
@ChannelHandler.Sharable // @Sharable 标记这个类的实例可以在channel里共享
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", CharsetUtil.UTF_8)); // 当被通知该 channel 是活动的时候就发送信息
    }

    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
        System.out.println("Client received: " + byteBuf.toString(CharsetUtil.UTF_8)); // 记录接收到的消息
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // 记录日志错误并关闭 channel
        cause.printStackTrace();
        ctx.close();
    }
}
```

###### EchoClient

通过`Bootstrap`引导创建客户端，另外需要 host 、port 两个参数连接服务器。

```java
public class EchoClient {

    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            Bootstrap bootstrap = new Bootstrap(); // 创建 Bootstrap
            bootstrap.group(group) // 指定EventLoopGroup来处理客户端事件。由于我们使用NIO传输，所以用到了 NioEventLoopGroup 的实现
                    .channel(NioSocketChannel.class) // 使用的channel类型是一个用于NIO传输
                    .remoteAddress(new InetSocketAddress(host, port)) // 设置服务器的InetSocketAddr
                    .handler(new ChannelInitializer<SocketChannel>() { // 当建立一个连接和一个新的通道时。创建添加到EchoClientHandler实例到 channel pipeline
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new EchoClientHandler());
                        }
                    });

            ChannelFuture future = bootstrap.connect().sync(); // 连接到远程；等待连接完成

            future.channel().closeFuture().sync(); // 阻塞到远程; 等待连接完成
        } finally {
            group.shutdownGracefully().sync(); // 关闭线程池和释放所有资源
        }
    }

    public static void main(String[] args) throws Exception {
        final String host = "127.0.0.1";
        final int port = 4567;
        new EchoClient(host, port).start();
    }
}
```

#### 编译和运行 Echo

首先编译、运行服务端，会看到以下log:

```
me.icro.samples.echo.server.EchoServer started and listen on /0:0:0:0:0:0:0:0:4567
```

下一步是编译、运行客服端后，服务端会先接收到信息:

```
Server received: Netty rocks!
```

然后客户端收到反馈:

```
Client received: Netty rocks!
```

## 总结

以上，构建并运行你的第一 个Netty 的客户端和服务器。虽然这是一个简单的应用程序，它可以扩展到几千个并发连接。

我们可以在Netty的Github仓库看到的更多 Netty 如何简化可扩展和多线程的例子。

下一步的深入学习，网上教程很多，大伙可以参考:

* [Netty Github仓库](https://github.com/netty/netty)
* [《Netty 4.x 用户指南》](https://legacy.gitbook.com/book/waylau/netty-4-user-guide/details)
* [Essential Netty in Action 《Netty 实战(精髓)》](https://legacy.gitbook.com/book/waylau/essential-netty-in-action/details)
* [netty-learning-example](https://github.com/sanshengshui/netty-learning-example)

(完)
