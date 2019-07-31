## 锁

### `synchronized`关键字？
`synchronized`关键字是Java解决多个线程并发访问临界资源问题的一种解决方案，它保证了在同一时刻只能有一个线程访问临界资源。

在JDK1.6之前，`synchronized`关键字属于重量级锁，线程是映射到操作系统的原生线程之上的，如果要挂起或者唤醒一个线程，都需要操作系统把线程从用户态转换到内核态，这个操作时很耗时的，所以之前的`synchronized`关键字效率很低。

在JDK1.6之后，Java官方从JVM层面对synchronized进行了较大优化，引入了偏向锁、轻量级锁，最后才升级到重量级锁，在很多场景下其实偏向锁、轻量级锁都可以解决问题，所以效率倍增！

### `synchronized`关键字有哪些用法？在实际项目中是怎么用的呢？
1. **修饰成员方法**：加在成员方法上，作用的是当前**对象**，调用实例方法时需要先获取对象的锁。

2. **修饰静态方法**：修饰静态方法时，作用的是整个**类的所有对象**，也就是说调用这个类对象的任意方法时，都会先获取类的锁。

3. **修饰代码块**：修饰代码块时，锁的可以是**任意对象或类**，进入同步代码块前会获取指定对象的锁。

在目前分布式泛滥的背景下，大部分的场景用的都是分布式锁，我感觉单机锁其实用处不大，但是在懒汉式的单例模式中，就采用了**双重校验锁**的机制：
```java
public class Singleton {

	// 使用volatile修饰
	private volatile static Singleton singleton;

	// 构造函数私有化，保证外部无法通过构造函数实例化
	private Singleton() {}
	
	public static Singleton getInstance() {
		if (singleton == null) {
			// 先加锁
			synchronized (Singleton.class) {
				// 进来需要再次校验，防止重复创建实例
				if (singleton == null) {
					singleton = new Singleton();
				}
			}
		}
		return singleton;
	}
}
```
### volatile关键字用处？
#### 可见性问题
在JDK1.2之前，Java中变量是直接从主存（也就是JVM的那块内存）中读取的，这是没问题的，但是效率不够快，因此当前的Java内存模型中，变量是可以在[CPU缓存](https://zh.wikipedia.org/wiki/CPU%E7%BC%93%E5%AD%98)中有一份拷贝的，如图：

![](./images/volatile1.png)

很明显，当一个线程把数据写入工作内存但还没有写入主存，另外一个线程读取该数据时就会出现**数据不一致**的问题，因此有了`volatile`关键字，使用`volatile`修饰的变量将直接读取主存，如图：

![](./images/volatile2.png)

#### 指令重排序问题
拿懒汉式的单例模式中`singleton = new Singleton();`这句话来看，这句话其实大致分为三步进行：
1. 给singleton分配一块内存空间
2. 初始化singleton
3. 将singleton指向分配的内存空间

由于JVM具有指令重排的特性，原本1-2-3的顺序可能变成了1-3-2，这样在单线程下是没问题的，可以获得一样的效果，但是在多线程下，可能会让另一个线程拿到一个还没初始化的实例！这时如果使用`volatile`修饰变量，JVM将不会对此变量进行指令重排序。

**总的来说，`volatile`关键字有以下用处：**
- 保证变量的可见性
- 防止指令重排序

### `synchronized`的底层原理
`synchronized`是Java中的一个关键字，因此它的运作是由JVM来控制的，我们可以使用`javap`命令来查看字节码信息，具体分情况来看：

1. `synchronized`同步代码块时，Java文件如下：
```java
public class Demo {
	public void method() {
		synchronized(this) {
			System.out.println("do something");
		}
	}
}
```
找到class文件，执行`javap -c -s -v -l .\Demo.class`，可以看到字节码文件，其中关键部分如下：

![](./images/sync-monitor.png)
> 完整字节码请看[这里](./source/Demo.class)

可以看到，标号为4、7、9的内容是我们`synchronized`代码块中的内容，在标号为3、13的地方分别有`monitorenter`和`monitorexit`，因此可以看出：

**synchronized 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。** 当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 `monitor`(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 `monitorexit` 指令后，将锁计数器设为0，表明锁被释放。**如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。**

2. `synchronized`修饰成员方法时，Java文件如下：
```java
public class Demo2 {
	public synchronized void method() {
		System.out.println("do something");
	}
}
```
同上查看字节码文件，关键如下：
![](./images/sync-method.png)
> 完整字节码请看[这里](./source/Demo2.class)

通过对比上个字节码文件，可以看到这时没有了`monitor`的指令，取而代之的是flags的内容多了个`ACC_SYNCHRONIZED`，也就是说，**`ACC_SYNCHRONIZED`标记标识了这个方法是一个同步方法，JVM统统此标记来判断是否要进行同步方法调用。**

3. `synchronized`修饰静态方法时，，Java文件如下：
```java
public class Demo3 {
	public synchronized static void method() {
		System.out.println("do something");
	}
}
```
查看字节码文件，关键如下：
![](./images/sync-static-method.png)
> 完整字节码请看[这里](./source/Demo3.class)

我们发现，相比同步成员方法时，这里的flags只是多了一个`ACC_STATIC`标识而已（*PS.由此也可以看出方法的修饰符都是在flags里面声明的*），因此`synchronized`在修饰静态方法和成员方法时其实原理是一样的，因此**不管是类直接调用静态方法还是通过实例调用静态方法，JVM都会读取到`ACC_SYNCHRONIZED`标记，从而进行同步调用！**

### 在JDK1.6以后，`synchronized`做了哪些优化？

### `synchronized`和`ReentrantLock`的区别


### `synchronized`和`volatile`的区别

### 乐观锁与悲观锁

### 公平锁和非公平锁
### 可重入锁和不可重入锁






