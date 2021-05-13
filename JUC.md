## <u>JUC</u>（java util concurrent）

### 一、线程的基本概念

##### 1.程序、进程、线程

- 程序概念：是为完成特定任务、用某种语言编写的一组指令的集合。即指**一段静态的代码**。
- 进程概念：程序的一次执行过程，或是**正在运行的一个程序**。
  说明：**进程作为资源分配的单位**，系统在运行时会为每个进程分配不同的内存区域
- 线程概念：进程可进一步细化为线程，是一个程序内部的一条执行路径。
  说明：**线程作为调度和执行的单位，每个线程拥独立的运行栈和程序计数器(pc)**，线程切换的开销小。
- 补充：进程可以细化为多个线程。
  - 每个线程，拥有自己独立的：虚拟机栈、本地方法栈、程序计数器
  - 多个线程，共享同一个进程中的结构：方法区、堆。

- 举例：一个Java应用程序java.exe，其实至少三个线程：main()主线程，gc()垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。

##### 2.并行与并发

- 并行：多个CPU同时执行多个任务。比如：多个人同时做不同的事。
- 并发：一个CPU(采用时间片)同时执行多个任务。比如：秒杀、多个人做同一件事

### 二、创建多线程的四种方式

##### 1.方式一：继承Thread类的方式

###### 1）步骤：

* a.创建一个继承于Thread类的子类
* b.重写Thread类的run() --> 将此线程执行的操作声明在run()中
* c.创建Thread类的子类的对象
* d.通过此对象调用start()：①启动当前线程 ② 调用当前线程的run()

###### 2）说明两个问题：

- 问题一：我们启动一个线程，必须调用start()，不能调用run()的方式启动线程。
- 问题二：如果再启动一个线程，必须重新创建一个Thread子类的对象，调用此对象的start().

##### 2.方式二：实现Runnable接口的方式

###### 1）步骤：

- 创建一个实现了Runnable接口的类
- 实现类去实现Runnable中的抽象方法：run()
- 创建实现类的对象
- 将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
- 通过Thread类的对象调用start()

###### 2) 前两种方式的对比

* 开发中：优先选择：实现Runnable接口的方式
* 原因：1. 实现的方式没类的单继承性的局限性

     ​             2.实现的方式更适合来处理多个线程共享数据的情况。
     
* 联系：public class Thread implements Runnable
* 相同点：两种方式都需要重写run(),**将线程要执行的逻辑声明在run()中**。
          目前两种方式，要想启动线程，都是调用的Thread类中的start()。

##### 3.方式三：实现Callable接口的方式

###### 1）步骤：

- 创建一个实现Callable的实现类

- 实现call方法，将此**线程需要执行的操作声明在call()中**

- 创建Callable接口实现类的对象

- 将此Callable接口实现类的对象作为参数传递到FutureTask构造器中，创建FutureTask的对象

- 将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()

- 获取Callable中call方法的返回值，get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。




  ```java
  //1.创建一个实现Callable的实现类
  class NumThread implements Callable{
      //2.实现call方法，将此线程需要执行的操作声明在call()中
      @Override
      public Object call() throws Exception {
          int sum = 0;
          for (int i = 1; i <= 100; i++) {
              if(i % 2 == 0){
                  System.out.println(i);
                  sum += i;
              }
          }
          return sum;
      }
  }
  
  
  public class ThreadNew {
      public static void main(String[] args) {
          //3.创建Callable接口实现类的对象
          NumThread numThread = new NumThread();
          //4.将此Callable接口实现类的对象作为传递到FutureTask构造器中，创建FutureTask的对象
          FutureTask futureTask = new FutureTask(numThread);
          //5.将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()
          new Thread(futureTask).start();
      try {
          //6.获取Callable中call方法的返回值
          //get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。
          Object sum = futureTask.get();
          System.out.println("总和为：" + sum);
      } catch (InterruptedException e) {
          e.printStackTrace();
      } catch (ExecutionException e) {
          e.printStackTrace();
      }
  }
  }
  ```

###### 2）面：FutureTask原理

- 在主线程中需要执行比较耗时的操作时，但又不想阻塞主线程时，可以把这些作业交给Future对象在后台完成，
  当主线程将来需要时，就可以通过Future对象获得后台作业的计算结果或者执行状态。
- 一般FutureTask多用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果。
- 仅在计算完成时才能检索结果；如果计算尚未完成，则阻塞 get 方法。一旦计算完成，
  就不能再重新开始或取消计算。get方法而获取结果只有在计算完成时获取，否则会一直阻塞直到任务转入完成状态，然后会返回结果或者抛出异常。 
- 只计算一次，get方法放到最后

###### 3）面：Callable和Runnable接口的对比

- call()可以返回值的。run()方法没有返回值

* call()可以抛出异常，被外面的操作捕获，获取异常的信息。run()不能抛出异常，只能用Try-Catch处理异常
* Callable是支持泛型的，Runnable接口不支持泛型

##### 4.方式四：线程池创建（老年代）

###### 1）面：使用线程池的好处

- ① **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的损耗
- ②**提高响应速度**。当任务到达时，任务可以不需要等到新线程创建就能立即执行
- ③**提高线程的可管理性**。通过参数的设置管理线程

###### 2）面：阻塞队列有哪些选择

- **概念**：阻塞队列支持阻塞插入和移除，当队列满时，阻塞插入元素的线程直到队列不满。当队列为空时，获取元素的线程会被阻塞直到队列非空。阻塞队列常用于生产者和消费者的场景，阻塞队列就是生产者用来存放元素，消费者用来获取元素的容器。

- **好处**：为什么需要BlockingQueue，好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你一手包办了，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤起。

- **实现原理**：使用通知模式实现，生产者往满的队列里添加元素时会阻塞，当消费者消费后，会通知生产者当前队列可用。例如：ArrayBlockingQueue使用Condition来实现

- **常用阻塞队列**：
  
  - **ArrayBlockingQueue**，由数组结构组成的有界阻塞队列，默认情况下不保证线程公平，有可能先阻塞的线程最后才访问队列。
  - **LinkedBlockingQueue**，由链表结构组成的有界阻塞队列，队列的默认和最大长度为 Integer 最大值。
  - **SynchronousQueue**，不存储元素的阻塞队列，每一个 put 必须等待一个 take。默认使用非公平策略，也支持公平策略，适用于传递性场景，吞吐量高。
  - PriorityBlockingQueue，支持优先级的无界阻塞队列，默认情况下元素按照升序排序。可自定义 `compareTo` 方法指定排序规则，或者初始化时指定 Comparator 排序，不能保证同优先级元素的顺序。
- DelayQueue，支持延时获取元素的无界阻塞队列，使用优先级队列实现。创建元素时可以指定多久才能从队列中获取当前元素，只有延迟期满时才能从队列中获取元素，适用于缓存和定时调度。
  
- BlockingQueue的核心方法：

  ![image-20210310155656028](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210310155656028.png)

   抛出异常	       当阻塞队列满时，再往队列里add插入元素会抛IllegalStateException:Queue full
                 		  	当阻塞队列空时，再往队列里remove移除元素会抛NoSuchElementException
                 
  
   特殊值	            插入方法，成功ture失败false
                               移除方法，成功返回出队列的元素，队列里没有就返回null
                 
  
   一直阻塞	       当阻塞队列满时，生产者线程继续往队列里put元素，队列会一直阻塞生产者线程直到put数据or
  
  ​                              响应中断退出；
  
  ​                             当阻塞队列空时，消费者线程试图从队列里take元素，队列会一直阻塞消费者线程直到队列可用         
  
   超时退出	       当阻塞队列满时，队列会阻塞生产者线程一定时间，超过限时后生产者线程会退出
###### 3）面：创建线程池有哪些参数

- ① corePoolSize：常驻核心线程数，如果为 0，当执行完任务没有任何请求时会消耗线程池；如果大于 0，即使本地任务执行完，核心线程也不会被销毁。该值设置过大会浪费资源，过小会导致线程的频繁创建与销毁。

- ② maximumPoolSize：线程池能够容纳同时执行的线程最大数，必须大于等于 1，如果与核心线程数设置相同代表固定大小线程池。

- ③ keepAliveTime：线程空闲时间，线程空闲时间达到该值后会被销毁，直到只剩下 corePoolSize 个线程为止，避免浪费内存资源。

- ④ unit：keepAliveTime 的时间单位。

- ⑤ workQueue：工作队列，当线程请求数大于等于 corePoolSize 时线程会进入阻塞队列。

- ⑥ threadFactory：线程工厂，用来生产一组相同任务的线程。可以给线程命名，有利于分析错误。

- ⑦ handler：拒绝策略，等待队列已经排满了，再也塞不下新任务了。同时，线程池中的max线程数也达到了，无法继续为新任务服务，需要拒绝策略机制合理的处理这个问题。4种拒绝策略。

  - 默认使用 AbortPolicy 丢弃任务并抛出异常，

  - CallerRunsPolicy 表示用调用者线程执行，
  - DiscardOldestPolicy 表示抛弃队列里等待最久的任务并把当前任务加入队列，执行当前任务，
  - DiscardPolicy 表示直接抛弃当前任务但不抛出异常。

###### 4）面：创建线程池的方法

- 使用Executors静态工厂方法创建线程池：

  - ① `newFixedThreadPool`，固定大小的线程池，该线程池使用的**工作队列是无界阻塞队列** LinkedBlockingQueue，队列的默认和最大长度为 Integer 最大值。适用于负载较重的服务器。

  - ② `newSingleThreadExecutor`，一池一线程，使用单线程，相当于单线程串行执行所有任务，适用于需要保证顺序执行任务的场景。该线程池使用的**工作队列是无界阻塞队列** LinkedBlockingQueue，队列的默认和最大长度为 Integer 最大值。

  - ③ `newCachedThreadPool`，maximumPoolSize 设置为 Integer 最大值，是高度可伸缩的线程池。该线程池使用的***工作队列是没有容量的 SynchronousQueue**，如果主线程提交任务的速度高于线程处理的速度，线程池会不断创建新线程，极端情况下会创建过多线程而耗尽CPU 和内存资源。适用于执行很多短期异步任务的小程序或负载较轻的服务器。

  - ④ `newScheduledThreadPool`：线程数最大为 Integer 最大值，存在 OOM 风险。支持定期及周期性任务执行，适用需要多个后台线程执行周期任务，同时需要限制线程数量的场景。相比 Timer 更安全，功能更强，与 `newCachedThreadPool` 的区别是不回收工作线程。

    ![image-20210310160858269](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210310160858269.png)



- 实际开发中不使用Executors工厂创建线程池，而是手动创建

  ```java
  ExecutorService threadPool = new ThreadPoolExecutor(2,
                  5,
                  2L,
                  TimeUnit.SECONDS,
                  new ArrayBlockingQueue<Runnable>(3),
                  Executors.defaultThreadFactory(),
                  //new ThreadPoolExecutor.AbortPolicy()
                  //new ThreadPoolExecutor.CallerRunsPolicy()
                  //new ThreadPoolExecutor.DiscardOldestPolicy()
                  new ThreadPoolExecutor.DiscardOldestPolicy()
  
  );
  ```

- **手动创建线程池原因**：

  ![image-20210310164407529](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210310164407529.png)

###### 5）面：线程池的拒绝策略有哪些

- 默认使用 AbortPolicy 丢弃任务并抛出异常，

- CallerRunsPolicy 表示用调用者线程执行，
- DiscardOldestPolicy 表示抛弃队列里等待最久的任务并把当前任务加入队列，执行当前任务，
- DiscardPolicy 表示直接抛弃当前任务但不抛出异常。

###### 6）面：线程池处理任务的流程（工作原理）

![image-20210310165010631](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210310165010631.png)

![image-20210310165019969](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210310165019969.png)

- 工作原理
  - 1、在创建了线程池后，线程池中的线程数为零。
  - 2、当调用execute()方法添加一个请求任务时，线程池会做出如下判断：
    - 2.1如果正在运行的线程数量小于corePoolSize，那么马上创建线程运行这个任务；
    -  2.2如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务**放入队列**；
    -  2.3如果这个时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；**注意：此时执行的是当前任务**
    -  2.4如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，那么线程池会**启动饱和拒绝策略来执行**。
  - 3、当一个线程完成任务时，它会从队列中取下一个任务来执行。
  - 4、当一个线程无事可做超过一定的时间（keepAliveTime）时，线程会判断：如果当前运行的线程数大于corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，**它最终会收缩到corePoolSize的大小**。

###### 7）面：关闭线程池的方法

- 可以调用 `shutdown` 或 `shutdownNow` 方法关闭线程池，原理是遍历线程池中的工作线程，然后逐个调用线程的 `interrupt` 方法中断线程，无法响应中断的任务可能永远无法终止。

### 三、Thread类中的常用方法

- start():启动当前线程；调用当前线程的run()
2. run(): 通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中
3. currentThread():静态方法，返回执行当前代码的线程
4. getName():获取当前线程的名字
5. setName():设置当前线程的名字
6. yield():释放当前cpu的执行权，但下一刻CPU可能还把资源分配到当前线程。 
7. join():在线程a中调用线程b的join(),此时线程a就进入阻塞状态，直到线程b完全执行完以后，线程a才结束阻塞状态。
8. stop():已过时。当执行此方法时，强制结束当前线程。
9. sleep(long millitime):让当前线程“睡眠”指定的millitime毫秒。在指定的millitime毫秒时间内，当前线程是阻塞状态。
10. isAlive():判断当前线程是否存活

- 线程的优先级：
  - 1.
    MAX_PRIORITY：10
    MIN _PRIORITY：1
    NORM_PRIORITY：5  -->默认优先级
  - 2.如何获取和设置当前线程的优先级：
      getPriority():获取线程的优先级
      setPriority(int p):设置线程的优先级
  - 说明：高优先级的线程要抢占低优先级线程cpu的执行权。但是只是从概率上讲，高优先级的线程高概率的情况下被执行。并不意味着只当高优先级的线程执行完以后，低优先级的线程才执行。

### 四、Thread的生命周期

![image-20210311093017065](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210311093017065.png)

- ###### 线程的生命周期状态：

  - NEW：新建状态，线程被创建且未启动，此时还未调用 `start`  方法。
  - RUNNABLE：Java 将操作系统中的就绪和运行两种状态统称为 RUNNABLE，此时线程有可能在等待时间片，也有可能在执行。
  - BLOCKED：阻塞状态，可能由于锁被其他线程占用、调用了 `sleep` 或 `join` 方法、执行了 `wait`方法等。
  - WAITING：等待状态，该状态线程不会被分配 CPU 时间片，需要其他线程通知或中断。可能由于调用了无参的 `wait` 和 `join` 方法。
  - TIME_WAITING：限期等待状态，可以在指定时间内自行返回。导可能由于调用了带参的 `wait` 和 `join` 方法。
  - TERMINATED：终止状态，表示当前线程已执行完毕或异常退出。

### 五、线程的同步机制

##### 1. 线程的安全问题举例

例子：创建个窗口卖票，总票数为100张.使用实现Runnable接口的方式
* 1.问题：卖票过程中，出现了重票、错票 -->出现了线程的安全问题
* 2.问题出现的原因：当某个线程操作车票的过程中，尚未操作完成时，其他线程参与进来，也操作车票。
* 3.如何解决：当一个线程a在操作ticket的时候，其他线程不能参与进来。直到线程a操作完ticket时，其他线程才可以开始操作ticket（共享数据）。这种情况即使线程a出现了阻塞，也不能被改变。

##### 2. Java解决方案：同步机制

- 在Java中，我们通过同步机制，来解决线程的安全问题。

- 同步的方式，解决了线程的安全问题。  ----优点
- 操作同步代码时，只能有一个线程参与，其他线程等待，相当于单线程的过程，效率低。----- 缺点

##### 3. 面：Java解决线程安全的三种方式及代码实现

- 方式一：**同步代码块**

  ```java
  synchronized(同步监视器){
       //需要被同步的代码
  
   }
  ```

  - 1.操作共享数据的代码，即为需要被同步的代码。  -->不能包含代码多了，也不能包含代码少了。
  -  2.**共享数据**：多个线程共同操作的变量。比如：ticket就是共享数据。  
  -  3.**同步监视器**，俗称：锁。任何一个类的对象，都可以充当锁。 要求：多个线程必须要共用同一把锁。
  - 4.在实现Runnable接口创建多线程的方式中，我们可以考虑**使用this充当同步监视器**。
  -  5.在继承Thread类创建多线程的方式中，慎用this充当同步监视器，考虑**使用当前类充当同步监视器**。

- 方式二：**同步方法**

     ​     如果操作共享数据的代码完整的声明在一个方法中，我们不妨将此方法声明同步的。总结：
     
     - 1.同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
     - 2.**非静态的同步方法**，同步监视器是：this
     - 3.**静态的同步方法**，同步监视器是：当前类本身

-  方式三：Lock锁  --- JDK5.0新增

  ​     手动的启动同步（lock()，同时结束同步也需要手动的实现（unlock()）
  
-  代码实现：

  ```java
  package com.atguigu.thread;
   
  import java.util.concurrent.locks.Lock;
  import java.util.concurrent.locks.ReentrantLock;
   
  class Ticket //实例例eld +method
  {
   private int number=30;
  /* //1同步 public synchronized void sale() 
   {//2同步  synchronized(this) {}
    if(number > 0) {
      System.out.println(Thread.currentThread().getName()+"卖出"+(number--)+"\t 还剩number);
     }
   }*/
  
   private Lock lock = new ReentrantLock();//List list = new ArrayList()
   
   public void sale() 
   {
     lock.lock();
     
     try {
      if(number > 0) {
       System.out.println(Thread.currentThread().getName()+"卖出"+(number--)+"\t 还剩number);
      }
     } catch (Exception e) {
      e.printStackTrace();
     } finally {
      lock.unlock();
     }
     
   }
   
  }
   
  /**
   * 
   * @Description:卖票程序个售票出  0张票
   @author xiale
   * 笔记：J里面如何 1 多线程编-上
   *   1.1 线程  (资里源类 *   1.2 高内聚 
   */
  public class SaleTicket 
  {
   public static void main(String[] args)//main所有程序
     Ticket ticket = new Ticket();
    
   new Thread(() -> {for (int i = 1; i < 40; i++)ticket.sale();}, "AA").start();
   new Thread(() -> {for (int i = 1; i < 40; i++)ticket.sale();}, "BB").start();
   new Thread(() -> {for (int i = 1; i < 40; i++)ticket.sale();}, "CC").start();
   
  /*  new Thread(new Runnable() {
      @Override
      public void run() 
      {
       for (int i = 1; i <=40; i++) 
       {
         ticket.sale();
       }
      }
     }, "AA").start();
     
     new Thread(new Runnable() {
      @Override
      public void run() 
      {
       for (int i = 1; i <=40; i++) 
       {
         ticket.sale();
       }
      }
     }, "BB").start();
     new Thread(new Runnable() {
      @Override
      public void run() 
      {
       for (int i = 1; i <=40; i++) 
       {
         ticket.sale();
       }
      }
     }, "CC").start();
     */
   }
  }
   
  ```

##### 4.面：synchronized 与 Lock的异同

- 相同：二者都可以解决线程安全问题

- 不同：
  - synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器
    Lock需要手动的启动同步（lock()，同时结束同步也需要手动的实现（unlock()）
  - 在Lock的实现类重入锁ReentrantLock中，增加了高级功能：
    - **公平锁：** 公平锁指多个线程在等待同一个锁时，必须按照申请锁的顺序来依次获得锁，而非公平锁不保证这一点，在锁被释放时，任何线程都有机会获得锁。synchronized 是非公平的，ReentrantLock 在默认情况下是非公平的，可以通过构造方法指定公平锁。一旦使用了公平锁，性能会急剧下降，影响吞吐量。 
    - **锁绑定多个条件：** 一个 ReentrantLock 可以同时绑定多个 Condition。synchronized 中锁对象的 `wait` 跟 `notify` 可以实现一个隐含条件，如果要和多个条件关联就不得不额外添加锁，而 ReentrantLock 可以多次调用 `newCondition` 创建多个条件。 

##### 5. 面：synchronized的实现原理理解

- synchronized实现同步的基础：Java中的每一个对象都可以作为锁、Java的每一个对象都有一个monitor与之对应
  - Java中的每一个对象都可以作为锁。具体表现为以下3种形式。对于普通同步方法，锁是当前实例对象；对于静态同步方法，锁是当前类的Class对象；对于同步方法块，锁是 Synchonized括号里配置的对象。synchronized用的锁是存在Java对象头中，Java对象头里的Mark Word里存储了对象的hashCode、分代年龄和锁标记位，1 bit的是否是偏向锁偏向锁标记位以及2 bit的锁标记位，Mark Word 里存储的数据会随着锁标记位的变化而变化。
  - Java的每一个对象都有一个monitor与之对应，JVM基于进入和退出Monitor对象来实现同步方法和同步代码块，使用 synchronized 时 JVM 会 找到对象的 monitor，根据 monitor 的状态进行加解锁的判断。如果成功加锁就成为该 monitor 的唯一持有者，monitor 在被释放前不能再被其他线程获取。此时Java对象头的锁标记位也会相应改变。

##### 6.面：锁的优化策略：

- JDK 6 对 synchronized 做了很多优化，引入了自适应自旋、锁消除、锁粗化、偏向锁和轻量级锁等提高锁的效率，锁一共有 4 个状态，级别从低到高依次是：无锁、偏向锁、轻量级锁和重量级锁，状态会随竞争情况升级。锁可以升级但不能降级，这种只能升级不能降级的锁策略是为了提高锁获得和释放锁的效率。

##### 7.面：自旋锁是什么

- 背景

  ​        线程的阻塞和唤醒需要CPU从用户态转为核心态，频繁的阻塞和唤醒对CPU来说是一件负担很重的工作，势必会给系统的并发性能带来很大的压力。在许多应用上面，对象锁的锁状态只会持续很短一段时间，为了这一段很短的时间频繁地阻塞和唤醒线程是非常不值得的。所以引入自旋锁。

- 何谓自旋锁？
              所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。而是执行一段无意义的循环（自旋）。自旋等待不能替代阻塞，虽然它可以避免线程切换带来的开销，但是它占用了处理器的时间。如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源，它不会做任何有意义的工作，这样反而会带来性能上的浪费。所以说，自旋等待的时间（自旋的次数）必须要有一个限度，如果自旋超过了定义的时间仍然没有获取到锁，则应该被挂起。
  自旋锁在JDK 1.4.2中引入，默认关闭，但是可以使用-XX:+UseSpinning开开启，在JDK1.6中默认开启。同时自旋的默认次数为10次，可以通过参数-XX:PreBlockSpin来调整；

- 代码演示

  ```java
  package com.xzp.thread;
  
  import java.util.concurrent.TimeUnit;
  import java.util.concurrent.atomic.AtomicReference;
  
  /**
   * 自旋锁的实现原理:
   * 尝试获取锁的线程不会立即阻塞，而是采用循环的方式去获取锁，这样做的好处是减少线程上下文切换带来的消耗，缺点是循环会消耗CPU资源
   * @author xzp
   * @create 2021-03-12 15:46
   */
  public class SpinLockTest {
  
      // 原子引用线程，初始化为null
      AtomicReference<Thread> atomicReference = new AtomicReference<>();
  
      // 获得锁
      public void myLock() {
          Thread thread = Thread.currentThread();
          System.out.println(Thread.currentThread().getName() + "\t come in!!!");
          // 自旋（while循环 + CAS）
          while (!atomicReference.compareAndSet(null,thread)) {
              //System.out.println("lmy");
          }
      }
  
      // 销毁锁
      public void myUnLock() {
          Thread thread = Thread.currentThread();
          atomicReference.compareAndSet(thread,null);
          System.out.println(Thread.currentThread().getName() + "\t out!!!");
      }
  
      public static void main(String[] args) throws InterruptedException {
  
          SpinLockTest spinLockTest = new SpinLockTest();
          new Thread(()->{
              spinLockTest.myLock();
  
              try {
                  TimeUnit.SECONDS.sleep(5);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
  
              spinLockTest.myUnLock();
          },"AA").start();
  
  
          //Thread.sleep(1000);
  
          try {
              TimeUnit.SECONDS.sleep(1);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  
  
          new Thread(()->{
              spinLockTest.myLock();
  
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
  
              spinLockTest.myUnLock();
          },"BB").start();
      }
  }
  ```

  

##### 8.面：自适应自旋锁是什么

- JDK6 对自旋锁进行了优化，自旋时间不再固定，而是由前一次的自旋时间及锁拥有者的状态决定。

  如果在同一个锁上，自旋刚刚成功获得过锁且持有锁的线程正在运行，虚拟机会认为这次自旋也很可能成功，进而允许自旋持续更久。如果自旋很少成功，以后获取锁时将可能直接省略掉自旋，避免浪费处理器资源。

  有了自适应自旋，随着程序运行时间的增长，虚拟机对程序锁的状况预测就会越来越精准。


##### 9.面：锁消除是什么

- 锁消除指即时编译器对检测到不可能存在共享数据竞争的锁进行消除。主要判定依据来源于逃逸分析，如果判断一段代码中堆上的所有数据都只被一个线程访问，就可以当作栈上的数据对待，认为它们是线程私有的而无须同步。


##### 10.面：锁粗化是什么？

- 原则需要将同步块的作用范围限制得尽量小，只在共享数据的实际作用域中进行同步，这是为了使等待锁的线程尽快拿到锁。
- 但如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体之内的，即使没有线程竞争也会导致不必要的性能消耗。因此如果虚拟机探测到有一串零碎的操作都对同一个对象加锁，将会把同步的范围扩展到整个操作序列的外部。比如循环体内的vector.add()操作

##### 11.面：偏向锁是什么？

- 偏向锁是为了在没有竞争的情况下减少锁开销，锁会偏向于第一个获得它的线程，如果在执行过程中锁一直没有被其他线程获取，则持有偏向锁的线程将不需要进行同步。
- 当锁对象第一次被线程获取时，虚拟机会将对象头中的偏向模式设为 1，同时使用 CAS 把获取到锁的线程 ID 记录在对象的 Mark Word 中。如果 CAS 成功，持有偏向锁的线程以后每次进入锁相关的同步块都不再进行任何同步操作。一旦有其他线程尝试获取锁，偏向模式立即结束，根据锁对象是否处于锁定状态决定是否撤销偏向，后续同步按照轻量级锁那样执行。
- 偏向锁的释放采用了一种只有竞争才会释放锁的机制，线程是不会主动去释放偏向锁，需要等待其他线程来竞争。

##### 12.面：轻量级锁是什么？

- 目的：轻量级锁是为了在没有竞争的前提下减少重量级锁使用操作系统互斥量产生的性能消耗。其性能提升的依据是“对于绝大部分的锁，在整个生命周期内都是不会存在竞争的”

- 在代码即将进入同步块时，如果同步对象没有被锁定，虚拟机将在当前线程的栈帧中建立一个锁记录空间，存储锁对象目前 Mark Word 的拷贝。然后虚拟机使用 CAS 尝试把对象的 Mark Word 更新为指向锁记录的指针，如果更新成功即代表该线程拥有了锁，锁标志位将转变为 00，表示处于轻量级锁定状态。

  如果更新失败就意味着至少存在一条线程与当前线程竞争。虚拟机检查对象的 Mark Word 是否指向当前线程的栈帧，如果是则说明当前线程已经拥有了锁，直接进入同步块继续执行，否则说明锁对象已经被其他线程抢占。如果出现两条以上线程争用同一个锁，轻量级锁就不再有效，将膨胀为重量级锁，锁标志状态变为 10，此时Mark Word 存储的就是指向重量级锁的指针，后面等待锁的线程也必须阻塞。

  解锁同样通过 CAS 进行，如果对象 Mark Word 仍然指向线程的锁记录，就用 CAS 把对象当前的 Mark Word 和线程复制的 Mark Word 替换回来。假如替换成功同步过程就顺利完成了，如果失败则说明有其他线程尝试过获取该锁，就要在释放锁的同时唤醒被挂起的线程。

##### 13.面：偏向锁、轻量级锁和重量级锁的区别？

- 偏向锁的优点是加解锁不需要额外消耗，和执行非同步方法比仅存在纳秒级差距，缺点是如果存在锁竞争会带来额外锁撤销的消耗，适用只有一个线程访问同步代码块的场景。
- 轻量级锁的优点是竞争线程不阻塞，程序响应速度快，缺点是如果线程始终得不到锁会自旋消耗 CPU，适用追求响应时间、同步代码块执行快的场景。
- 重量级锁的优点是线程竞争不使用自旋不消耗CPU，缺点是线程会阻塞，响应时间慢，适应追求吞吐量、同步代码块执行慢的场景。

##### 14.面：读写锁是什么

- ReentrantLock 是排他锁，同一时刻只允许一个线程访问，读写锁ReentrantReadWriteLock在同一时刻允许多个读线程访问，在写线程访问时，所有的读写线程均阻塞。读写锁维护了一个读锁和一个写锁，通过分离读写锁使并发性相比排他锁有了很大提升。

  ```java
  
  import java.util.HashMap;
  import java.util.Map;
  import java.util.concurrent.TimeUnit;
  import java.util.concurrent.locks.ReadWriteLock;
  import java.util.concurrent.locks.ReentrantReadWriteLock;
  
  class MyCache {
      private volatile Map<String, Object> map = new HashMap<>();
      private ReadWriteLock rwLock = new ReentrantReadWriteLock();
  
      public void put(String key, Object value) {
          rwLock.writeLock().lock();
          try {
              System.out.println(Thread.currentThread().getName() + "\t 正在写" + key);
              //暂停一会儿线程
              try {
                  TimeUnit.MILLISECONDS.sleep(300);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              map.put(key, value);
              System.out.println(Thread.currentThread().getName() + "\t 写完了" + key);
              System.out.println();
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              rwLock.writeLock().unlock();
          }
      }
  
      public Object get(String key) {
          rwLock.readLock().lock();
          Object result = null;
          try {
              System.out.println(Thread.currentThread().getName() + "\t 正在读" + key);
              try {
                  TimeUnit.MILLISECONDS.sleep(300);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              result = map.get(key);
              System.out.println(Thread.currentThread().getName() + "\t 读完了" + result);
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              rwLock.readLock().unlock();
          }
          return result;
      }
  }
  
  public class ReadWriteLockDemo {
  
      public static void main(String[] args) {
          MyCache myCache = new MyCache();
  
          for (int i = 1; i <= 5; i++) {
              final int num = i;
              new Thread(() -> {
                  myCache.put(num + "", num + "");
              }, String.valueOf(i)).start();
          }
          for (int i = 1; i <= 5; i++) {
              final int num = i;
              new Thread(() -> {
                  myCache.get(num + "");
              }, String.valueOf(i)).start();
          }
      }
  }
  ```

##### 15.面：AQS 是什么

- AbstractQueuedSynchronizer 的简写AQS，队列同步器
- 同步队列器是实现锁的关键，同步器依赖内部的同步队列（一个FIFO双向队列）来完成同步状态的管理，当前线程获取同步状态失败时，同步器会将当前线程以及等待状态信息构造成一个节点（Node）将其加入同步队列，同时阻塞当前线程，当同步状态释放时，会唤醒首节点中线程，使其再次尝试获取同步状态。
- 同步队列中的节点（Node）用来保存获取同步状态失败的线程引用、等待状态、前驱，后继节点信息等。

##### 16.面： AQS 有哪两种模式

- **独占模式**表示锁只会被一个线程占用，其他线程必须等到持有锁的线程释放锁后才能获取锁，同一时间只能有一个线程获取到锁。
- **共享模式**表示多个线程获取同一个锁有可能成功，ReadLock 就采用共享模式。
- 独占模式通过 acquire 和 release 方法获取和释放锁，共享模式通过 acquireShared 和 releaseShared 方法获取和释放锁。

##### 17.面：AQS 独占式获取/释放锁的原理

- 获取同步状态时，调用 `acquire` 方法，维护一个同步队列，使用 `tryAcquire` 方法安全地获取线程同步状态，获取失败的线程会被构造同步节点并通过 `addWaiter` 方法加入到同步队列的尾部，在队列中自旋。之后调用 `acquireQueued` 方法使得该节点以死循环的方式获取同步状态，如果获取不到则阻塞，被阻塞线程的唤醒主要依靠前驱节点的出队或被中断实现，移出队列或停止自旋的条件是前驱节点是头结点且成功获取了同步状态。
- 释放同步状态时，同步器调用 `tryRelease` 方法释放同步状态，然后调用 `unparkSuccessor` 方法唤醒头节点的后继节点，使后继节点重新尝试获取同步状态。

##### 18.面：AQS 共享式式获取/释放锁的原理

- 获取同步状态时，调用 `acquireShared` 方法，该方法调用 `tryAcquireShared` 方法尝试获取同步状态，返回值为 int 类型，返回值不小于 0 表示能获取同步状态。因此在共享式获取锁的自旋过程中，成功获取同步状态并退出自旋的条件就是该方法的返回值不小于0。

- 释放同步状态时，调用 `releaseShared` 方法，释放后会唤醒后续处于等待状态的节点。它和独占式的区别在于 `tryReleaseShared` 方法必须确保同步状态安全释放，通过循环 CAS 保证，因为释放同步状态的操作会同时来自多个线程。

### 六、线程间的通信

##### 1.线程通信涉及到的三个方法：

- wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。
- notify():一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。
- notifyAll():一旦执行此方法，就会唤醒所有被wait的线程。

##### 2.三个方法的注意点

- wait()，notify()，notifyAll()三个方法必须使用在**同步代码块或同步方法中**。
- wait()，notify()，notifyAll()三个方法的调用者必须是同步代码块或同步方法中的同步监视器。否则，会出现IllegalMonitorStateException异常
- wait()，notify()，notifyAll()三个方法是定义在java.lang.Object类中。

##### 3.面：sleep() 和 wait()的异同？

- **相同点**：一旦执行方法，都可以使得当前的线程进入阻塞状态。
- **不同点**：
  - 1）两个方法声明的位置不同：Thread类中声明sleep() , Object类中声明wait() 
  -  2）调用的要求不同：sleep()可以在任何需要的场景下调用。 wait()必须使用在同步代码块或同步方法中      
  - 3）关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，wait()会释放锁。	 
  -  4）wait()需要被唤醒，否则一直被阻塞。sleep()过了睡眠时间自然唤醒。

##### 4.小结释放锁的操作

![image-20210311134404251](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210311134404251.png)

##### 5.小结不会释放锁的操作

![image-20210312091434009](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210312091434009.png)

##### 6. 示例代码一

```java
package com.atguigu.thread;
 
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
 
import org.omg.IOP.Codec;
/**
 * 
 * @Description:
 *现在两个线程，
 * 可以操作初始值为零的一个变量，
 * 实现一个线程对该变量加1，一个线程对该变量减1，
 * 交替，来10轮。 

 *  * 笔记：Java里面如何进行工程级别的多线程编写
 * 1 多线程变成模板
 *     1.1  线程    操作    资源类  
 *     1.2  高内聚  低耦合
 * 2 多线程变成模板
 *     2.1  判断
 *     2.2  逻辑操作
 *     2.3  通知
 * 3 防止虚假唤醒用while
 */ 
 
class ShareDataOne//资源类
{
  private int number = 0;//初始值为零的一个变量
 
  public synchronized void increment() throws InterruptedException 
  {
     //1判断
     while (number !=0 ) {
       this.wait();
     }
     //2逻辑操作
     ++number;
     System.out.println(Thread.currentThread().getName()+"\t"+number);
     //3通知
     this.notifyAll();
  }
  
  public synchronized void decrement() throws InterruptedException 
  {
     // 1判断
     while (number == 0) {
       this.wait();
     }
     // 2逻辑操纵
     --number;
     System.out.println(Thread.currentThread().getName() + "\t" + number);
     // 3通知
     this.notifyAll();
  }
}
 

public class NotifyWaitDemoOne
{
  public static void main(String[] args)
  {
     ShareDataOne sd = new ShareDataOne();
     new Thread(() -> {
       for (int i = 1; i < 10; i++) {
          try {
            sd.increment();
          } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          }
       }
     }, "A").start();
     new Thread(() -> {
       for (int i = 1; i < 10; i++) {
          try {
            sd.decrement();
          } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
          }
       }
     }, "B").start();
  }
}

```



##### 7. 示例代码二

```java
package com.atguigu.thread;
 
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 
 * @Description: 
 * 多线程之间按顺序调用，实现A->B->C
 * 三个线程启动，要求如下：
 * 
 * AA打印5次，BB打印10次，CC打印15次
 * 接着
 * AA打印5次，BB打印10次，CC打印15次
 * ......来10轮  
 */
 
class ShareResource 
{
  private int number = 1;//1:A 2:B 3:C 
  private Lock lock = new ReentrantLock();
  private Condition c1 = lock.newCondition();
  private Condition c2 = lock.newCondition();
  private Condition c3 = lock.newCondition();
 
  public void print5(int totalLoopNumber)
  {
     lock.lock();
     try 
     {
       //1 判断
       while(number != 1)
       {
          //A线程阻塞
          c1.await();
       }
       //2 逻辑操作
       for (int i = 1; i <=5; i++) 
       {
          System.out.println(Thread.currentThread().getName()+"\t"+i+"\t totalLoopNumber: "+totalLoopNumber);
       }
       //3 通知
       number = 2;
       c2.signal();
     } catch (Exception e) {
       e.printStackTrace();
     } finally {
       lock.unlock();
     }
  }
  public void print10(int totalLoopNumber)
  {
     lock.lock();
     try 
     {
       //1 判断
       while(number != 2)
       {
          //B线程阻塞
          c2.await();
       }
       //2 逻辑操作
       for (int i = 1; i <=10; i++) 
       {
          System.out.println(Thread.currentThread().getName()+"\t"+i+"\t totalLoopNumber: "+totalLoopNumber);
       }
       //3 通知
       number = 3;
       c3.signal();
     } catch (Exception e) {
       e.printStackTrace();
     } finally {
       lock.unlock();
     }
  }  
  
  public void print15(int totalLoopNumber)
  {
     lock.lock();
     try 
     {
       //1 判断
       while(number != 3)
       {
          //C线程阻塞
          c3.await();
       }
       //2 逻辑操作
       for (int i = 1; i <=15; i++) 
       {
          System.out.println(Thread.currentThread().getName()+"\t"+i+"\t totalLoopNumber: "+totalLoopNumber);
       }
       //3 通知
       number = 1;
       c1.signal();
     } catch (Exception e) {
       e.printStackTrace();
     } finally {
       lock.unlock();
     }
  }  
}
 
public class ThreadOrderAccess
{
  public static void main(String[] args)
  {
     ShareResource sr = new ShareResource();
     
     new Thread(() -> {
       for (int i = 1; i <=10; i++) 
       {
          sr.print5(i);
       }
     }, "AA").start();
     new Thread(() -> {
       for (int i = 1; i <=10; i++) 
       {
          sr.print10(i);
       }
     }, "BB").start();
     new Thread(() -> {
       for (int i = 1; i <=10; i++) 
       {
          sr.print15(i);
       }
     }, "CC").start();   
  
  }
}
 
```



### 七. java中的并发工具类

##### 1. 等待多线程完成的CountdownLatch

功能：让一些线程阻塞直到另一些线程完成一系列操作后才被唤醒。

```java
import java.util.concurrent.CountDownLatch;

/**
 * 功能：让一些线程阻塞直到另一些线程完成一系列操作后才被唤醒。
 * CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，这些线程会阻塞。
 * 其它线程调用countDown方法会将计数器减1(调用countDown方法的线程不会阻塞)，
 * 当计数器的值变为0时，因await方法阻塞的线程会被唤醒，继续执行。
 * 
 * 解释：6个同学陆续离开教室后值班同学才可以关门。
 * main主线程必须要等前面6个线程完成全部工作后，自己才能开干 

 * @author xzp
   */
   public class CountDownLatchDemo
   {
   public static void main(String[] args) throws InterruptedException
   {
         CountDownLatch countDownLatch = new CountDownLatch(6);
       

       for (int i = 1; i <=6; i++) //6个上自习的同学，各自离开教室的时间不一致
       {
          new Thread(() -> {
              System.out.println(Thread.currentThread().getName()+"\t 号同学离开教室");
              countDownLatch.countDown();
          }, String.valueOf(i)).start();
       }
       countDownLatch.await();
       System.out.println(Thread.currentThread().getName()+"\t****** 班长关门走人，main线程是班长");

   }

}
```

##### 2. 同步屏障 CyclicBarrier

   功能：让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。
```java
package com.atguigu.thread;
 
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
 
/**
 * @Description: TODO(这里用一句话描述这个类的作用)  
 * @author xzp
 *
 * CyclicBarrier
 * 的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，
 * 让一组线程到达一个屏障（也可以叫同步点）时被阻塞，
 * 直到最后一个线程到达屏障时，屏障才会开门，所有
 * 被屏障拦截的线程才会继续干活。
 * 线程进入屏障通过CyclicBarrier的await()方法。
 * 
 * 集齐7颗龙珠就可以召唤神龙
 */
public class CyclicBarrierDemo
{
  private static final int NUMBER = 7;
  
  public static void main(String[] args)
  {
     //CyclicBarrier(int parties, Runnable barrierAction) 
     
     CyclicBarrier cyclicBarrier = new CyclicBarrier(NUMBER, ()->{System.out.println("*****集齐7颗龙珠就可以召唤神龙");}) ;
     
     for (int i = 1; i <= 7; i++) {
       new Thread(() -> {
          try {
            System.out.println(Thread.currentThread().getName()+"\t 星龙珠被收集 ");
            cyclicBarrier.await();
          } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
          }
       }, String.valueOf(i)).start();
     }
  }
}
```

##### 3. 控制并发线程数的Semaphore

功能：一个是用于共享资源有限的互斥使用，另一个用于并发线程数的控制

```java
package com.atguigu.thread;
 
import java.util.Random;
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;
 
/**
 * 
 * @Description: TODO(这里用一句话描述这个类的作用)  
 * @author xzp
 * 
 * 在信号量上我们定义两种操作：
 * acquire（获取） 当一个线程调用acquire操作时，它要么通过成功获取信号量（信号量减1），
 *               要么一直等下去，直到有线程释放信号量，或超时。
 * release（释放）实际上会将信号量的值加1，然后唤醒等待的线程。
 * 
 * 信号量主要用于两个目的，一个是用于共享资源有限的互斥使用，另一个用于并发线程数的控制。
 */
public class SemaphoreDemo
{
  public static void main(String[] args)
  {
     Semaphore semaphore = new Semaphore(3);//模拟3个停车位
     
     for (int i = 1; i <=6; i++) //模拟6部汽车
     {
       new Thread(() -> {
          try 
          {
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName()+"\t 抢到了车位");
            TimeUnit.SECONDS.sleep(new Random().nextInt(5));
            System.out.println(Thread.currentThread().getName()+"\t------- 离开");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }finally {
            semaphore.release();
          }
       }, String.valueOf(i)).start();
     }   
  }
}

```



### N.面试汇总

###### 1. 经典面试题：i++在两个线程分别执行100次，最大值和最小值分别多少

i++只需要执行一条指令，并不能保证多个线程i++，操作同一个i，可以得到正确的结果。因为还有寄存器的因素，多个cpu对应多个寄存器。每次要先把i从内存复制到寄存器，然后++，然后再把i复制到内存中，这需要至少3步。从这个意义上讲，说i++是原子的并不对。

如此，假设两个线程的执行步骤如下：

1、 线程A执行第一次i++，取出内存中的i，值为0，存放到寄存器后执行加1，此时CPU1的寄存器中值为1，内存中为0；

```
注：对于多线程，线程共用一个内存，如果线程A在寄存器执行操作后而没有写入内存，则会切换到另一个线程。
```

2、线程B执行第一次i++，取出内存中的i，值为0，存放到寄存器后执行加1，此时CPU2的寄存器中值为1，内存中为0；

3、线程A继续执行完成第99次i++，并把值放回内存，此时CPU1中寄存器的值为99，内存中为99；

```
注：线程A在此的每一步执行，都写回内存，直到完成第99次1
```

4、线程B继续执行第一次i++，将其值放回内存，此时CPU1中的寄存器值为1，内存中为1；

```
注：由于线程B之前未写回内存，因此停留在第一次，而当其写回时，覆盖了原来的99，内存中的值变为1
```

5、线程A执行第100次i++，将内存中的值（现在是1）取回CPU1的寄存器！！！，并执行加1，此时CPU1的寄存器中的值为2，内存中为1；

6、线程B执行完所有操作，并将其放回内存，此时CPU2的寄存器值为100，内存中为100；

```
注：B执行完100次i++，则CPU2寄存器值为100，写入内存，100覆盖了原来的1
```

7、线程A执行100次操作的最后一部分，将CPU1中的寄存器值放回内存（值为2，见第5步），内存中值为2；

8、结束！

```
注：最大值200的情况：线程A和B每次执行完i++均写入内存，交替进行1
```

所以该题目便可以得出最终结果，最小值为2，最大值为200。

相似的题目：

j=100，两个线程j–，均执行50次，可能值是多少？

思路：过程和上题类似。第一次j–：两个线程从内存取到100值，执行j–,此时寄存器值都为99，内存值100。后边线程A和B交替执行j–，不写入内存，直到执行到第50次j–时，线程A先将内存中的值（100）取到CPU1的寄存器，再执行减1，变为99，写入内存；线程B将内存中的值（99）取到CPU2的寄存器，再执行减1，变为98，写入内存，结束。此时内存值最大为98！！！ 
因此，该题的范围是0到98之间！！！！！！！！

###### 2. 面：CAS的实现原理

博客链接：https://blog.csdn.net/qq_32998153/article/details/79529704?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=1328626.10641.16153610625442837&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control

###### 3.面：写时复制技术：

博客链接：https://blog.csdn.net/m0_37941483/article/details/103142000

- CopyOnWriteArrayList:既保证并发的读请求，又保证独占的写请求
- CopyOnWriteArraySet
- ConcurrentHashMap 与 synchronizedHashMap

###### 4.集合的线程安全问题

