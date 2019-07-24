## 锁

### `synchronized`关键字有什么了解？
`synchronized`关键字是Java解决多个线程并发访问临界资源问题的一种解决方案，它保证了在同一时刻只能有一个线程访问临界资源。

在JDK1.6之前，`synchronized`关键字属于重量级锁，线程是映射到操作系统的原生线程之上的，如果要挂起或者唤醒一个线程，都需要操作系统把线程从用户态转换到内核态，这个操作时很耗时的，所以之前的`synchronized`关键字效率很低。

在JDK1.6之后，Java官方从JVM层面对synchronized进行了较大优化，引入了偏向锁、轻量级锁，最后才升级到重量级锁，在很多场景下其实偏向锁、轻量级锁都可以解决问题，所以效率倍增！

### `synchronized`关键字有哪些用法？在实际项目中你是怎么用的呢？
1. **修饰成员方法**
加在成员方法上，作用的是当前对象实例，调用实例方法时需要先获取实例的锁。

2. **修饰静态方法**


3. **修饰代码块**