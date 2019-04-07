### Concurrent

#### JMM(Java Memory Model)

jmm是一种规范，规范了java虚拟机与计算机内存时如何协同工作的，规定了一个线程何时、如何看到其他线程对共享变量的修改，以及在必须时如何同步的访问共享变量。

##### 有序性-happens-before原则

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
2. 锁定规则：unlock操作先行发生于对同一个锁的lock操作
3. volatile变量规则：volatile些happens-beforevolatile读
4. 传递规则：A happens-before B，B happens-before C，那么A happens-before C
5. 线程启动规则：Thread对象的start()方法happens-before该线程的所有操作
6. 线程中断规则：线程的interrupt()方法的调用happens-before检测到中断事件的发生
7. 线程终结规则：线程中的所有操作happens-before检测到线程终止操作，Thread.join()方法返回代表线程结束，Thread.isAlive()方法可以检测到线程已经终止执行
8. 对象终止规则：一个对象的初始化happens-before对象的finalize()方法

#### 不可变对象

String（final类）

不可变集合将修改集合的方法全部重写，里面直接抛异常

Collections.unmodifiableXXX: Collection、List、Map、Set…….

Guava: ImmutableXXX: Collection、List、Map、Set…….

#### AQS

##### CountDownLatch

并发执行最后合并

##### CyclicBarrier



#### Fork/Join

##### 工作窃取算法

1. 将大任务分割成若干个互不依赖的子任务
2. 为了减少线程之间的竞争，将子任务放到不同的队列里，每个队列对应一个线程来执行该队列里的任务
3. A线程处理完自己队列里的任务后，可以去B线程队列里拿任务，这就是窃取，队列为双端队列，B从B队列头获取任务，A从B队列的队尾获取任务，这样可以减少线程之间的竞争

##### 缺点

1. 双端队列只有一个任务时，还是会存在竞争
2. 消耗更多的资源，多个线程多个队列



#### 线程池

##### ThreadPoolExecutor

corePoolSize：核心线程数

maximumPoolSize：最大线程数

workQueue：阻塞队列，存储等待执行的任务

1. 当前运行的线程少于corePoolSize，直接创建新的线程

2. 当前运行的线程在corePoolSize与maximumPoolSize之间

   1）workQueue没有满：放入工作队列

   2）workQueue已满，创建新的线程，新的线程在keepAliveTime后会关闭

3. 当前运行的线程大于maximumPoolSize

   1）workQueue没有满：放入工作队列

   2）workQueue已满，交给拒绝策略处理