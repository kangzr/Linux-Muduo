#### 由来

##### 为什么需要网络库

连接同一局域网的另外一台服务器，接收数据不完整？

高级语言(Java,  Python等)的Sockets库并没有对Sockets API提供更高层的封装，直接使用容易掉入陷阱。因此需要一个好的网络库来**降低开发难度**，其价值还在于**能方便处理并发连接**。

简化日常的TCP网络编程，让程序员能把精力集中在业务逻辑上，无需天天跟Sockets API较劲。

#### 安装

```shell
# 源码
git clone https://github.com/chenshuo/muduo.git
# 依赖库
sudo apt-get install cmake    # CMake作为buildsysttem
sudo apt-get install libboost-dev libboost-test-dve  # Boost依赖库
sudo apt-get install libcur14-openssl-dev libc-ares-dev
sudo apt-get install protobuf-compiler libprotobuf-dev
# 编译
./build.sh -j2
./build.sh install

```

#### 目录结构

##### 网络核心库

muduo基于Reactor模式，其核心为事件循环EventLoop，用于响应计时器和IO事件。

one loop per thread + thread pool  :  每个IO线程一个事件循环，把IO事件分发到回调函数上。

#### 使用教程

基于事件的非阻塞网络编程是编写并发网络服务程序的主流模式，使用这种模式需要转换思路，由主动接收/发送数据转化为通过回调方式处理。

##### TCP网络编程本质论

TCP网络编程最本质的是处理三个半事件：

- 建立连接，包括服务端接收(accept)新连接、客户端发起(connect)连接。一旦连接建立服务端客户端平等，可各自发送数据
- 断开联系， 包括主动(close shutdown)和被动断开(read(2)返回0)
- 消息到达，文件描述符可读，对它的处理方式决定了网络编程的风格(阻塞还是非阻塞，如何处理分包，应用层的缓冲如何设计等。)
- 消息发送完毕(半个)，指将数据写入操作系统的缓冲区，将由TCP协议栈负责数据的发送和重传，不代表对方收到数据

Q: 为什么要使用应用层发送缓冲区？

A: 假设应用程序需要发送40kB数据，操作系统TCP发送缓冲区只有25KB剩余空间，如果等待缓冲区可用，就会阻塞当前线程，因此网络库把15KB数据缓存起来，放到TCP链接的应用层发送缓冲区中，等socket可写时，立刻发送数据，这样就不会阻塞。如果随后又要发送50kb数据，而此时发送缓冲区尚有未发送数据，需要将其追加到缓冲区。

Q: 为什么要使用应用层接收缓冲区？

A: 假设一次读到数据不够一个完整数据包，需将数据暂存，等剩余数据收到后一并处理。

Q: 如何设计并缓冲区？一方面希望**减少系统调用**，尽可能一次读取更多的数据，则需要一个大的缓冲区；另一方面，希望**减少内存占用**，如果10000个并发连接，每个分配50kb，将占用1GB内存，且大多数时候这些缓冲区使用率很低。

A:  muduo使用readv(2)结合栈上空间巧妙地解决了这个问题。

Q: 如果使用发送缓冲区，万一接收方处理缓慢，数据会不会一直堆积在发送方，造成内存暴涨？如何做应用层的流量控制？如何设计并实现定时器？使之与网络IO共用一个线程，以避免思索。 (muduo代码中)













































