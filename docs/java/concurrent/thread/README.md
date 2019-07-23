## Java线程深入

### Java中如何创建线程有几种方式？
#### 创建无需返回值的线程
1. 直接继承Thread类，重写`run()`方法：
```java
public class AThread extends Thread {
	@Override
	public void run() {
		// TODO something
		System.out.println("直接继承Thread实现线程");
	}
}
```
2. 因为Java只允许单继承，所以也可以实现`Runnable`接口实现`run()`方法，然后调用`Thread`类的`Thread(Runnable target)`方法去创建一个线程：
```java
public class BThread implements Runnable {
	@Override
	public void run() {
		// TODO something
		System.out.println("实现Runnable接口的线程");
	}
}
```
这两种方式创建线程的启动如下：
```java
public static void main(String[] args) {
    AThread a = new AThread();
    a.start();// 直接继承Thread实现线程
    Thread b = new Thread(new BThread());
    b.start();// 实现Runnable接口的线程
}
```
#### 创建需要返回值的线程
与实现`Runnable`接口的`run()`方法类比，带返回值的线程需要实现`Callable`接口的`call()`方法，通过一个泛型来指定返回的类型：
```java
public class CThread implements Callable<String> {
	@Override
	public String call() throws Exception {
		// TODO something
		return "这是带返回值的线程";
	}
}
```
启动线程和获取返回值需要借助`FutureTask`工具类，线程启动后可以通过`isDone()`方法获取线程状态，方式如下：
```java
public static void main(String[] args) {
    FutureTask<String> futureTask = new FutureTask<>(new CThread());
    Thread c = new Thread(futureTask);
    c.start();
    while (true) {
        if (futureTask.isDone()) {// 线程执行完毕
            try {
                System.out.println(futureTask.get());// 这是带返回值的线程
                break;
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
}
```
### 有了线程，为什么要使用线程池？
> 使用线程池的好处是减少在创建和销毁线程上所消耗的时间及系统资源，解决资源不足的问题。  
> 如果不使用线程池，有可能系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。  
> ——引用自《阿里巴巴Java开发手册》P28

通俗一点，就是线程每次创建是必然要消耗系统资源的，但是线程池不会每次都创建新线程，所以需要使用线程时，尽量采用线程池吧~
### 如何创建线程池？
#### 使用`Executors`工具类创建（不推荐）
- 不设置上限的两个：
  - `Executors.newCachedThreadPool()`：创建核心线程为0，最大线程数为int型最大值数量，超时时间为60s，阻塞队列为`SynchronousQueue`的线程池
  - `Executors.newScheduledThreadPool(int corePoolSize)`：创建核心线程为corePoolSize，最大线程数为int型最大值的线程池，超时时间为0，阻塞队列为`DelayedWorkQueue`
- 设置上限的两个：
  - `Executors.newFixedThreadPool(int nThreads)`：核心线程和最大线程都为nThreads，超时时间为0，阻塞队列为`LinkedBlockingQueue`
  - `Executors.newSingleThreadExecutor()`：核心线程和最大线程都为1，超时时间为0，阻塞队列为`LinkedBlockingQueue`

> 为什么不推荐？  
> Executors 返回线程池对象的弊端如下：  
> - `FixedThreadPool`和`SingleThreadExecutor`：允许请求的队列长度为`Integer.MAX_VALUE`，可能堆积大量的请求，从而导致OOM。
> - `CachedThreadPool`和`ScheduledThreadPool`：允许创建的线程数量为`Integer.MAX_VALUE`，可能会创建大量线程，从而导致OOM。  
> ——引用自《阿里巴巴Java开发手册》P29

#### 直接使用`ThreadPoolExecutor`构造方法创建（推荐）
四个构造方法：

![](./images/thread-pool-executor.png)

### ThreadPoolExecutor类构造方法参数有哪些？
`ThreadPoolExecutor`共有四个构造方法，最终都调入了一个构造方法：
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```
解释：
- **corePoolSize**：核心线程数。线程池会一直维护已创建的线程的最大数量。
- **maximumPoolSize**：最大线程数。用与当前线程达到`corePoolSize`并且阻塞队列也满了以后，会创建临时线程执行任务，直到`maximumPoolSize`大小。
- **keepAliveTime**：存活时间。用于控制线程的闲置时间，空闲超时后线程会退出，直到达到`corePoolSize`大小。
- **unit**：`keepAliveTime`的单位。从纳秒到天共7种。
- **workQueue**：阻塞队列。用于当前任务数超过`corePoolSize`时，新来的任务会进入`workQueue`排队，等待核心线程空闲后执行。
- **threadFactory**：线程工厂。可以自行重写方法，以实现创建线程属性的个性化，一般来说默认实现就够用了。
- **handler**：拒绝策略。用于当前线程达到`maximumPoolSize`时，线程池该怎么处理新来的任务，默认是抛出异常。

### 阻塞队列有哪些？
- **LinkedBlockingQueue**：
- **SynchronousQueue**：
- **ArrayBlockingQueue**：
- **PriorityBlockingQueue**：

### 拒绝机制有哪些？
- ThreadPoolExecutor.**AbortPolicy**：默认值，丢弃任务并抛出`RejectedExecutionException`异常
- ThreadPoolExecutor.**DiscardPolicy**：也是丢弃任务，但是不抛出异常
- ThreadPoolExecutor.**DiscardOldestPolicy**：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
- ThreadPoolExecutor.**CallerRunsPolicy**：由调用线程处理该任务 

### `execute()`方法和`submit()`方法的区别是什么呢？

### 线程池的执行原理是什么？

### 线程池的核心线程数量如何设置？