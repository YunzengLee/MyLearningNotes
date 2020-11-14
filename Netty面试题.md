### 同步与异步 

[参考文章](www.zhihu.com/question/19732473)

同步异步关注的是消息通信机制。同步，就是在发出一个调用时，没有得到结果之前，该调用就不返回。就是由调用者主动等待这个调用的结果。

异步相反，调用在发出后就返回了，没有返回结果。也就是说，一个异步过程调用后，调用者不会立刻得到结果。而是在调用后，被调用者通过状态、通知来通知调用者，或通过回调函数处理这个调用。

### 阻塞与非阻塞

关注的是程序在等待调用结果（消息，返回值）时的状态

阻塞调用是指调用结果返回之前，当前线程挂起。调用线程只有在结果返回之后才会返回。

非阻塞调用指在不能立刻得到结果时，线程继续处理其他任务，该调用不会阻塞当前线程。

## BIO NIO IO多路复用 AIO的区别

[这篇文章](https://www.cnblogs.com/zingp/p/6863170.html)讲得很好。

对于一次IO访问，数据先拷贝到操作系统内核的缓冲区中，然后才会从操作系统的内核缓冲区拷贝到应用程序的缓冲区，最后交给进程。因此包含两个阶段：1. 等待数据准备；2. 将数据从内核拷贝到进程中。

### BIO

1. 用户进程向操作系统内核发起IO请求
2. 内核准备数据并拷贝到用户空间，此时进程阻塞。
3. 进程解除阻塞。

特点是用户进程在内核准备数据和数据从内核拷贝到进程内存中这两个过程中处于阻塞等待状态。

### NIO

1. 用户进程向操作系统内核发起IO请求，如果数据没有准备好，会返回一个error
2. 用户进程得到返回结果，说明数据没有准备好，可以处理其他任务，一段时间后再次请求，直到数据返回。

特点是用户进程在内核准备数据时需要不断主动询问数据准备好没有。

### IO多路复用

实际上是用select poll epoll监听多个io对象，有数据时就通知用户进程。好处是单个进程可以处理多个连接，而不是单个连接处理的更快。

1. 用户进程调用select，内核监听所有select负责的socket；
2. 当任何一个socket的数据准备好了，select就返回
3. 此时用户进程调用read操作，将数据从内核拷贝到用户进程。

### AIO

1. 用户进程发起IO请求后立刻去做其他事情
2. 内核等待数据准备完成将数据拷贝到用户内存，然后向用户进程发送一个signal，告诉他read操作完成了。



### BIO BlockingIO

同步阻塞IO，数据的读取写入必须阻塞在一个线程内等待其完成。

活动链接数不高的情况下，可以让每个连接专注自己的IO且编程模型简单。当面对大量IO连接时，就无能为力。

并发量高，来一个客户端连接就开启一个线程；同时阻塞方法多，很多线程都在等待，CPU在线程之间反复切换，因此效率低。

```java
package com.cloud.bio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocket ss = new ServerSocket();
        ss.bind(new InetSocketAddress("127.0.0.1",8888));//开启一个插座，指定端口和地址（里面有很多插口供客户端连接）
        while(true){
            Socket s = ss.accept(); //阻塞方法，等在此处等待有客户端接入ss，如果没有线程就进入wait，直到客户端接入才被唤醒
            new Thread(()->{  //一旦有客户端接入，开启一个新线程处理，主线程继续等待
                handle(s);
            }).start();
        }
    }

    public static void handle(Socket s){
        try{
            byte[] bytes = new byte[1024];
            int len = s.getInputStream().read(bytes);// 此处的read方法也是阻塞方法，如果客户端没发来消息，就会一直wait
            System.out.println(new String (bytes,0,len));
            s.getOutputStream().write(bytes,0,len); //write也是阻塞方法，如果客户端不接收，就阻塞在此处
            s.getOutputStream().flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

```java
package com.cloud.bio;

import java.io.IOException;
import java.net.Socket;

public class Client {
    public static void main(String[] args) throws IOException {
        Socket s = new Socket("127.0.0.1",8888); //连接服务器
        s.getOutputStream().write("helloServer".getBytes()); //s是客户端这边看到的channel
        s.getOutputStream().flush();

        System.out.println("write over, waiting for msg back..");
        byte[] bytes = new byte[1024];
        int len = s.getInputStream().read(bytes);
        System.out.println(new String(bytes,0,len));
        s.close();
    }
}

```



### NIO New/Non-Blocking

单线程：Server 端有一个Selector选择器只专注客户端的连接，读取，和写入，（会盯在Server端的插座旁边，如果有客户连接，就接收进来建立连接，如果有客户写入，就读取进来。即发现一件事就处理一件事）

```java
package com.cloud.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocketChannel ssc = ServerSocketChannel.open();
        //ServerSocketChannel是NIO对ServerSocket进行的封装，这个是双向的，可以同时读，同时写
        //而BIO需要拿到InputStream，和OutputStream才能读和写。
        ssc.socket().bind(new InetSocketAddress("127.0.0.1",8888));
        ssc.configureBlocking(false);//设定为非阻塞
        System.out.println("server started listening on " +ssc.getLocalAddress());
        Selector selector = new Selector.open();//打开一个Selector
        ssc.register(selector, SelectionKey.OP_ACCEPT);//这句话意思是注册感兴趣的事，OP_ACCEPT就是对客户端发起连接这件事
        while (true){ //轮询
            selector.select();//
            Set<SelectionKey> keys = selector.selectedKeys();//key就是服务器插座上的每一个插头（相当于channel），如果有客户端建立连接，该插头就会被选出来，进入selector里去。
            Iterator<SelectionKey> it =keys.iterator();
            while (it.hasNext()){
                SelectionKey key = it.next();
                it.remove();//取出来处理之后就把key从it里remove掉，避免反复处理
                handle(key);
            }
        }
    }
    public static void handle(SelectionKey key){
       if(key.isAcceptable()){//说明这个key是有客户端要连上的
           try{
               ServerSocketChannel ssc = (ServerSocketChannel) key.channel(); //获取channel
               SocketChannel sc = ssc.accept(); //接收连接，获得连接的通道  与BIO处代码类似
               sc.configureBlocking(false);

               sc.register(key.selector(),SelectionKey.OP_READ);//给这个channel注册一个用来监听READ事件
           } catch (IOException e) {
               e.printStackTrace();
           }
       }else if(key.isReadable()){
           SocketChannel sc = null;
           try{
               sc = (SocketChannel) key.channel();
               ByteBuffer buffer = ByteBuffer.allocate(512); //BIO读数据是一个字节一个字节的，效率低。
               // 而NIO每一个channel都和一个Buffer（内存中的字节数组）放在一起，从字节数组中读和写，效率高
                                                            //
               buffer.clear();
               int len =sc.read(buffer);
               if(len!=-1){
                   System.out.println(new String(buffer.array(),0,len));

               }
               ByteBuffer bufferToWrite = ByteBuffer.wrap("HelloClient".getBytes());
               sc.write(bufferToWrite);
           } catch (IOException e) {
               e.printStackTrace();
           } finally {
               if(sc!=null){
                   try{
                       sc.close();
                   } catch (IOException e) {
                       e.printStackTrace();
                   }
               }
           }
       }
    }
}
/**
 * ServerSocket建立一个插座，Selector对每一个插口放一个key，这个key记录着对哪件事感兴趣，第一个感兴趣的事件是ACCEPT，就是客户端尝试往这个插口上连接的事件。
 * Selector进行轮询，哪个key有事件发生就把key取出来，挨个进行进行处理
 * 如果处理时，发现是有客户端要建立连接，就给他注册一个channel，并再注册一个感兴趣事件READ。
 */

```

单线程的缺陷：如果在处理某一个客户端的读写请求时阻塞，那么就无法为其他客户端提供服务。

多线程：

reactor模式，响应式编程

Selector只负责客户端的连接，客户端的读和写交给线程池处理。（这就是netty的做法，只是netty做了很好的封装）

```java
package com.cloud.nio;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class PoolServer {
    ExecutorService pool = Executors.newFixedThreadPool(50);
    private Selector selector;

    public static void main(String[] args) throws IOException {
        PoolServer server = new PoolServer();
        server.initServer(8000);
        server.listen();
    }

    public void initServer(int port) throws IOException {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);
        serverSocketChannel.socket().bind(new InetSocketAddress(port));
        this.selector = Selector.open();
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("服务器启动成功");

    }
    public void listen() throws IOException {
        while(true){
            selector.select();
            Iterator ite = this.selector.selectedKeys().iterator();
            while(ite.hasNext()){
                SelectionKey key = (SelectionKey) ite.next();
                ite.remove();
                if(key.isAcceptable()){
                    ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                    SocketChannel channel = serverSocketChannel.accept();
                    channel.configureBlocking(false);
                    channel.register(this.selector,SelectionKey.OP_READ);
                }else if(key.isReadable()){
                    key.interestOps(key.interestOps()&(-SelectionKey.OP_READ));
                    pool.execute(new ThreadHandlerChannel(key));
                }
            }
        }
    }
}

class ThreadHandlerChannel extends Thread{
    private SelectionKey key;
    ThreadHandlerChannel(SelectionKey key){
        this.key = key;
    }
    @Override
    public void run() {
        SocketChannel channel = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try{
            int size = 0;
            while ((size=channel.read(buffer))>0){
                buffer.flip();
                baos.write(buffer.array(),0,size);
                buffer.clear();
            }
            baos.close();
            byte[] content = baos.toByteArray();
            ByteBuffer writeBuf = ByteBuffer.allocate(content.length);
            writeBuf.put(content);
            writeBuf.flip();
            channel.write(writeBuf);
            if(size == -1){
                channel.close();
                
            } else{
              key.interestOps(key.interestOps()|SelectionKey.OP_READ);  
              key.selector().wakeup();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```






### netty是什么

- 基于NIO的客户端服务器（client-server）框架，可以快速简单的开发网络应用程序。
- 简化了TCP、UDP套接字服务器等网络编程，提高了性能和安全性
- 支持多种协议：FTP SMTP HTTP

### 为什么使用netty（优点）

- 统一的API，支持多种传输模型，包括阻塞和非阻塞
- 简单强大的线程模型
- 自带编解码器解决TCP粘包拆包问题
- 自带各种协议栈
- 真正的无连接数据包套接字支持。

应用场景：

- 实现HTTP服务器（一般使用Tomcat），处理常见的POST GET请求
- 作为RPC框架的网络通信工具
- 实现即时通讯系统
- 实现消息推送系统

### 核心组件及作用

#### 1 Channel

该接口是netty对网络操作的抽象类，它包括基本的IO操作，如bind(),connect(), read(), write() 等

常见的Channel接口实现类是NioServerSocketChannel（服务器）和NioSocketChannel(客户端),这两个Channel可以和BIO编程模型中的ServerSocket和Socket两个概念对应上。netty中Channel提供的API降低了直接使用Socket的复杂性。

#### 2 EventLoop

EventLoop（事件循环）接口是netty最核心的概念，定义了netty的核心抽象，用于处理连接的生命周期中所发生的事件。

主要作用就是：负责监听网络事件并调用事件处理器进行相关的IO操作。

Channel与EventLoop的联系：

Channel为netty网络操作的抽象类，EventLoop负责处理注册到其上的Channel处理IO操作。

#### 3 ChannelFuture

netty中所有IO操作都为异步，无法立刻得到操作是否成功，可以通过ChannelFuture接口的addListener()方法注册一个ChannelFutureListener，当操作执行成功或失败时，监听就会自动触发返回结果。

另外，该接口的channel（）方法可以获取相关联的Channel。

另外，该接口的sync（）方法可以使异步的操作变成同步的。

```java
public interface ChannelFuture extends Future<Void>{
    Channel channel();
    ChannelFuture addListener(GenericFutureListener<? extends Future<? super Void>> var1);
    ChannelFuture sync() throws InterruptedException;
}
```



#### 4 ChannelHandler 和ChannelPipline

前者是消息的具体处理器，负责处理读写操作，客户端连接等。

后者为前者的链，Channel创建时自动分配一个ChannelPipline，ChannelPIPline可以通过addLast（）方法添加多个ChannelHandler,因为一个数据或事件可以被多个Handler处理，一个处理完就交给下一个Handler。

#### 5 EventLoopGroup

![image-20200616181408833](C:\Users\Wensheng\AppData\Roaming\Typora\typora-user-images\image-20200616181408833.png)

EventLoopGroup包含多个EventLoop（每一个EventLoop通常内部包含一个线程，它的任务是监听网络事件并调用事件处理器来进行IO操作）

上图是一个服务器端对EventLoopGroup使用的大致模块图，bossEventLoopGroup用于客户端连接请求处理，而workerEventLoopGroup用于具体处理器IO相关操作。

#### Bootstrap和ServerBootstrap

前者是客户端的启动引导类，使用如下

```java
EventLoopGroup group = new NioEventLoopGroup();
try{
    Bootstrap b = new Bootstrap();
    b.group(group).
        ......
    //建立连接
    ChannelFuture f = b.connect(host,port).sync();
    f.channel().closeFuture().sync();
}finally{
    group.shutdownGracefully();
}

```

后者是服务器端的启动类，使用如下：

```java
EventLoopGroup boss  = new NioEventLoopGroup(1);
EventLoopGroup worker  = new NioEventLoopGroup();
try{
    ServerBootstrap b = new ServerBootstrap();
    b.group(boss,worker).
        .....
    ChannelFuture f = b.bind(port).sync();
    f.channel().closeFuture().sync();
    
}finally{
    boss.shutdownGracefully();
    worker.shutdownGracefully();
}
```

可以看出：

1. Bootstrap通常用connect（）方法连接到远程的主机和端口，作为TCP协议通信中的客户端。也可以通过bind方法绑定端口，作为UDP协议通信中的一端。
2. ServerBootstrap通常使用bind方法绑定本地端口，等待客户端连接
3. Bootstrap需要一个线程组，ServerBootstrap需要配置两个线程组，一个用于接收连接，一个用于具体的处理。

### netty线程模型

大部分网络框架都是基于Reactor模式设计开发的。

Reactor模式基于事件驱动，采用多路复用将事件分发给相应的Handler处理。

在netty主要靠NIOEventLoopGroup线程池实现具体的线程模型。

1. 单线程模型：

   一个线程处理所有的accept、read、decode、process、encode、send事件。不适用高负载高并发的场景。

   ```java
   EventLoopGroup worker  = new NioEventLoopGroup();
   
   ServerBootstrap b = new ServerBootstrap();
   b.group(worker,worker).
   ```

2. 多线程模型

   一个Acceptor线程只负责监听客户端的连接，一个NIO线程池负责具体处理：accept、read、decode、Process、encode、send事件。适合并发连接量不大的场景。

   ```java
   EventLoopGroup boss  = new NioEventLoopGroup(1);
   EventLoopGroup worker  = new NioEventLoopGroup();
   try{
       ServerBootstrap b = new ServerBootstrap();
       b.group(boss,worker).
   ```

3. 主从多线程模型

   从主NIO线程池中选一个线程作为Acceptor线程，绑定监听端口，接收客户端连接，其他线程负责后续接入认证工作。从NIO线程池负责具体处理IO读写。

   ```java
   EventLoopGroup boss  = new NioEventLoopGroup();
   EventLoopGroup worker  = new NioEventLoopGroup();
   try{
       ServerBootstrap b = new ServerBootstrap();
       b.group(boss,worker).
   ```

### netty客户端和服务器端的启动过程

服务器端

```java
 // 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //2.创建服务端启动引导/辅助类：ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            //3.给引导类配置两大线程组,确定了线程模型
            b.group(bossGroup, workerGroup)
                    // (非必备)打印日志
                    .handler(new LoggingHandler(LogLevel.INFO))
                    // 4.指定 IO 模型
                .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) {
                            ChannelPipeline p = ch.pipeline();
                            //5.可以自定义客户端消息的业务处理逻辑
                            p.addLast(new HelloServerHandler());
                        }
                    });
            // 6.绑定端口,调用 sync 方法阻塞知道绑定完成
            ChannelFuture f = b.bind(port).sync();
            // 7.阻塞等待直到服务器Channel关闭(closeFuture()方法获取Channel 的CloseFuture对象,然后调用sync()方法)
            f.channel().closeFuture().sync();
        } finally {
            //8.优雅关闭相关线程组资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
```

1.  创建两个NioEventLoopGroup实例，一个处理客户端的TCP连接请求，一个负责每一条连接的具体读写数据的处理逻辑，真正负责IO读写操作，交由对应的Handler处理。
2. 创建一个服务器启动类ServerBootstrap
3. 通过.group()方法给启动类配置线程组，确定线程模型
4. 通过channel（）方法给启动类ServerBootstrap指定了IO模型为NIO
5. 通过.childHandler()给启动类创建一个ChannelInitializer，指定了业务处理逻辑HelloServerHandler对象
6. 调用bind方法绑定端口

### [TCP粘包/拆包及解决](https://my.oschina.net/u/3837147/blog/3023833)

基于TCP发送数据时，出现多个字符串粘在一起或者一个字符串被拆开的情况。这是因为TCP有个缓存区，如果一个数据包发送的数据量太小没有达到缓存区大小，TCP会将多个数据包合并为一个发送，如果一个数据包太大，超过了缓冲区的大小，TCP就会拆分成多个发送。

解决方案：

1. 发送数据包的时候每个包都使用固定长度，不足就使用空格补全。
2. 发送端在每个数据包的末尾加固定的分隔符，接收端通过识别分隔符来恢复完整的数据包。
3. 将消息分为消息头和消息体两部分，消息头保存消息长度，根据长度信息读取完整的消息内容。
4. 通过自定义协议进行粘包拆包处理。



#### netty提供的解决方案：

1. 使用netty自带的解码器（netty只提供了解码器，编码器需要自己实现）

    对于通过分隔符进行粘包和拆包问题的处理，Netty提供了两个编解码的类，`LineBasedFrameDecoder`和`DelimiterBasedFrameDecoder`。

   - **`LineBasedFrameDecoder`** : 发送端发送数据包的时候，每个数据包之间以换行符作为分隔，`LineBasedFrameDecoder` 的工作原理是它依次遍历 `ByteBuf` 中的可读字节，判断是否有换行符，然后进行相应的截取。
   - **`DelimiterBasedFrameDecoder`** : 可以自定义分隔符的解码器，**`LineBasedFrameDecoder`** 实际上是一种特殊的 `DelimiterBasedFrameDecoder` 解码器。
   - **`FixedLengthFrameDecoder`**: 固定长度解码器，它会一次读取指定长度的消息，能够按照指定的长度对消息进行相应的拆包。
   - **`LengthFieldBasedFrameDecoder`**： 这里`LengthFieldBasedFrameDecoder`与`LengthFieldPrepender`需要配合起来使用，其实本质上来讲，这两者一个是解码，一个是编码的关系。它们处理粘拆包的主要思想是在生成的数据包中添加一个长度字段，用于记录当前数据包的长度。

2. 自定义序列化编解码器

   在 Java 中自带的有实现 `Serializable` 接口来实现序列化，但由于它性能、安全性等原因一般情况下是不会被使用到的。

   通常情况下，我们使用 Protostuff、Hessian2、json 序列方式比较多，另外还有一些序列化性能非常好的序列化方式也是很好的选择：

   - 专门针对 Java 语言的：Kryo，FST 等等
   - 跨语言的：Protostuff（基于 protobuf 发展而来），ProtoBuf，Thrift，Avro，MsgPack 等等

### netty长连接、心跳机制

#### TCP长连接和短连接

短连接： server 端 与 client 端建立连接之后，读写完成之后就关闭掉连接，如果下一次再要互相发送消息，就要重新连接。管理和实现都比较简单，但每一次的读写都要建立连接必然会带来大量网络资源的消耗，并且连接的建立也需要耗费时间。

长连接：client 向 server 双方建立连接之后，即使 client 与 server 完成一次读写，它们之间的连接并不会主动关闭，后续的读写操作会继续使用这个连接。长连接的可以省去较多的 TCP 建立和关闭的操作，降低对网络资源的依赖，节约时间。对于频繁请求资源的客户来说，非常适用长连接。

#### 为什么需要心跳机制？netty的心跳机制

##### 心跳

是什么：

在 TCP 保持长连接的过程中，可能会出现断网等网络异常出现，异常发生的时候， client 与 server 之间如果没有交互的话，它们是无法发现对方已经掉线的。为了解决这个问题, 我们就需要引入 **心跳机制** 。

心跳机制的工作原理是: 在 client 与 server 之间在一定时间内没有数据交互时， 客户端或服务器就会发送一个特殊的数据包给对方, 当接收方收到这个数据报文后, 也立即发送一个特殊的数据报文, 回应发送方, 此即一个 PING-PONG 交互。所以, 当某一端收到心跳消息后, 就知道了对方仍然在线, 这就确保 TCP 连接的有效性.

##### netty的心跳机制：

TCP 实际上自带的就有长连接选项，本身是也有心跳包机制，也就是 TCP 的选项：`SO_KEEPALIVE`。但是，TCP 协议层面的长连接灵活性不够。所以，一般情况下我们都是在应用层协议上实现自定义心跳机制的，也就是在 Netty 层面通过编码实现。通过 Netty 实现心跳机制的话，核心类是 `IdleStateHandler` 。

### 零拷贝

> 零复制（英语：Zero-copy；也译零拷贝）技术是指计算机执行操作时，CPU 不需要先将数据从某处内存复制到另一个特定区域。这种技术通常用于通过网络传输文件时节省 CPU 周期和内存带宽。

在 OS 层面上的 `Zero-copy` 通常指避免在 `用户态(User-space)` 与 `内核态(Kernel-space)` 之间来回拷贝数据。而在 Netty 层面 ，零拷贝主要体现在对于数据操作的优化。

Netty 中的零拷贝体现在以下几个方面：

1. 使用 Netty 提供的 `CompositeByteBuf` 类, 可以将多个`ByteBuf` 合并为一个逻辑上的 `ByteBuf`, 避免了各个 `ByteBuf` 之间的拷贝。
2. `ByteBuf` 支持 slice 操作, 因此可以将 ByteBuf 分解为多个共享同一个存储区域的 `ByteBuf`, 避免了内存的拷贝。
3. 通过 `FileRegion` 包装的`FileChannel.tranferTo` 实现文件传输, 可以直接将文件缓冲区的数据发送到目标 `Channel`, 避免了传统通过循环 write 方式导致的内存拷贝问题.