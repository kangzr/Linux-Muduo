#### 单线程服务器的常用编程模型

- lighttpd, 单线程服务器.
- libevent, libev
- ACE. Poco C++ libraries
- Java NIO
- POE (perl)
- Twisted (Python)

#### 多线程服务器的常用编程模型

- one loop per thread
- 线程池
- 推荐模式

#### 进程间通信只用TCP

#### 多线程服务器的使用场合

1. 必须单线程场合
   - fork
   - 限制程序的CPU使用率
2. 