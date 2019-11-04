#### 只使用非递归的mutex

**互斥器(mutex)是加锁原语，用来排他性地访问共享数据，不是等待原语，使用mutex时，期望加锁不要阻塞，立刻拿到锁，然后尽快访问数据，尽快解锁，这样才能不影响并发性和性能**

线程间(inter-thread)同步工具： mutex分为递归(recursive)/可重入(reentrant)和非递归(non-recursive)/非可重入，唯一的区别：同一个线程可以重复对recursive mutex加锁，不可重复对non-recursive mutex加锁

#### 条件变量(condition variable)

**当需要等待某个条件成立，可使用条件变量(condition variable)**

正确使用方式：

- 必须与mutex一起使用，该布尔表达式的读写需受此mutex保护
- 在mutex已上锁的才能调用wait()
- 把判断条件和wait()放到while循环中

#### 不要用读写锁和信号量

#### 封装MutexLock MutexLockGuard Condition

这几个class需要深刻理解并默写

#### 线程安全的Singleton实现

#### sleep(3)不是同步原语

#### 归纳总结

- 线程同步的四项原则， 尽量使用高层同步设施(线程池、队列、倒计时)
- 使用普通互斥器和条件变量完成剩余的同步任务，采用RAII惯用手法(idiom)和Scoped Locking



