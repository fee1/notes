# 多线程(Thread)基础

## 1.并发编程的优缺点

### 优点

- 1.充分利用多核CPU的能力
- 2.方便业务拆分，提升系统的并发能力和性能

### 缺点

- 内存泄漏、上下文切换、线程安全、死锁

	- 内存泄漏（Memory Leak）是指程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。

## 2.并发编程的三要素

### 1.原子性

- 一个或多个操作，要么全部执行成功，要么全部执行失败

	- 线程切换会导致原子性问题

		- JDK Atomic开头的原子类、synchronized、LOCK，可以解决原子性问题

### 2.可见性

- 一个线程对变量的修改，另一个线程能立马看到(synchronized,volatile)

	- 缓存导致可见性问题

		- synchronized、volatile、LOCK，可以解决可见性问题

### 3.有序性

- 程序执行顺序和代码执行的顺序一致(处理器可能会对指令进行重排序)

	- 编译优化带来的有序性问题

		- Happens-Before 规则(保证多线程执行结果不表)可以解决有序性问题

## 3.并发和并行的区别

### 1.并发

- 单位时间内处理多个任务

### 2.并行

- 同一时刻处理多个任务

## 4.什么是上下文切换

### 当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。任务从保存到再加载的过程就是一次上下文切换。

## 5.守护线程和用户线程

### 用户线程：运行在前台，执行具体的任务

### 守护线程：运行在后台，为其他前台线程服务

### 扩展：

- 1.用户线程退出守护线程也会退出，无论是否执行完任务
- 2.setDaemon(true)必须在start()方法前执行，否则会抛出 IllegalThreadStateException 异常
- 3.在守护线程中产生的新线程也是守护线程
- 4.守护 (Daemon) 线程中不能依靠 finally 块的内容来确保执行关闭或清理资源的逻辑。因为我们上面也说过了一旦所有用户线程都结束运行，守护线程会随 JVM 一起结束工作，所以守护 (Daemon) 线程中的 finally 语句块可能无法被执行。

## 6.死锁

### 介绍：两个或两个以上的进程（线程）在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，这些永远在互相等待的进程（线程）称为死锁进程（线程）。

- 

  public class DeadLockDemo {
      private static Object resource1 = new Object();//资源 1
      private static Object resource2 = new Object();//资源 2
  
      public static void main(String[] args) {
          new Thread(() -> {
              synchronized (resource1) {
                  System.out.println(Thread.currentThread() + "get resource1");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread() + "waiting get resource2");
                  synchronized (resource2) {
                      System.out.println(Thread.currentThread() + "get resource2");
                  }
              }
          }, "线程 1").start();
  
          new Thread(() -> {
              synchronized (resource2) {
                  System.out.println(Thread.currentThread() + "get resource2");
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread() + "waiting get resource1");
                  synchronized (resource1) {
                      System.out.println(Thread.currentThread() + "get resource1");
                  }
              }
          }, "线程 2").start();
      }
  }
  
  Thread[线程 1,5,main]get resource1
  Thread[线程 2,5,main]get resource2
  Thread[线程 1,5,main]waiting get resource2
  Thread[线程 2,5,main]waiting get resource1
  
  

### 形成死锁的四个条件

- 1.互斥条件

	- 线程(进程)对于所分配到的资源具有排它性，即一个资源只能被一个线程(进程)占用，直到被该线程(进程)释放

- 2.请求与保持条件

	- 一个线程(进程)因请求被占用资源而发生阻塞时，对已获得的资源保持不放。

- 3.不剥夺条件

	- 线程(进程)已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。

- 4.循环等待条件

	- 当发生死锁时，所等待的线程(进程)必定会形成一个环路（类似于死循环），造成永久阻塞

### 创建线程的四种方式

- 1.继承Thread

  public class MyThread extends Thread {
  
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + " run()方法正在执行...");
      }
  
  }
  
  public class TheadTest {
  
      public static void main(String[] args) {
          MyThread myThread = new MyThread(); 
          myThread.start();
          System.out.println(Thread.currentThread().getName() + " main()方法执行结束");
      }
  
  }
  
  
  

- 2.实现Runnable接口

  public class MyRunnable implements Runnable {
  
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + " run()方法执行中...");
      }
  
  }
  public class RunnableTest {
  
      public static void main(String[] args) {
          MyRunnable myRunnable = new MyRunnable();
          Thread thread = new Thread(myRunnable);
          thread.start();
          System.out.println(Thread.currentThread().getName() + " main()方法执行完成");
      }
  
  }
  
  

- 3.实现Callable接口

  public class MyCallable implements Callable<Integer> {
  
      @Override
      public Integer call() {
          System.out.println(Thread.currentThread().getName() + " call()方法执行中...");
          return 1;
      }
  
  }
  
  public class CallableTest {
  
      public static void main(String[] args) {
          FutureTask<Integer> futureTask = new FutureTask<Integer>(new MyCallable());
          Thread thread = new Thread(futureTask);
          thread.start();
  
          try {
              Thread.sleep(1000);
              System.out.println("返回结果 " + futureTask.get());
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (ExecutionException e) {
              e.printStackTrace();
          }
          System.out.println(Thread.currentThread().getName() + " main()方法执行完成");
      }
  
  }
  
  

- 4.使用 Executors 工具类创建线程池

## 刁专问题

### 1.为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？

- 从JVM角度出发，直接调用run()方法是在把run方法加入当前调用线程的方法栈。start()方法重新开启一个新线程，JVM分配独立的栈空间。
- 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行。

### 2.什么是 Callable 和 Future?

- Future 接口表示异步任务，是一个可能还没有完成的异步任务的结果。所以说 Callable用于产生结果，Future 用于获取结果。

## 7.线程的生命周期及五种基本状态

### 图解

### 1.新建(new)

- 新创建一个线程对象

### 2.可运行(runable)

- 调用start方法以后就是进入可运行就绪状态，等待获取cpu使用权

### 3.运行(running)

- 获得了cpu使用权的线程

### 4.阻塞(block)

- 暂时放弃cpu使用权，进入阻塞状态

	- 1.等待阻塞：线程执行 wait()方法，JVM会把该线程放入等待队列(waitting queue)中，使本线程进入到等待阻塞状态；
	- 2.同步阻塞：线程在获取 synchronized 同步锁失败(因为锁被其它线程所占用)，，则JVM会把该线程放入锁池(lock pool)中，线程会进入同步阻塞状态；
	- 3.其他阻塞: 通过调用线程的 sleep()或 join()或发出了 I/O 请求时，线程会进入到阻塞状态。当 sleep()状态超时、join()等待线程终止或者超时、或者 I/O 处理完毕时，线程重新转入就绪状态。

### 5.死亡(dead)

- 线程run()、main()方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。

## 8.JAVA中的线程调度

### Java虚拟机采用抢占式调度模型，优先让运行池中优先级高的线程占用CPU

- wait()

	- 使线程阻塞，并且释放锁

- sleep()

	- 是线程睡眠，静态方法，强行唤醒线程会出现InterruptedException异常

- notify()

	- 唤醒一个处于等待的线程，随机性的，有JVM确定唤醒那个线程，与优先级无关

- notityAll()

	- 唤醒所有处于等待状态的线程

## 问题

### 1.wait()方法和sleep()方法有什么区别？

- 类不同：sleep是Thread线程类的方法，wait是object方法
- 是否释放锁：sleep不释放锁，wait释放锁
- 用途不同：wait方法通常被用于线程间交互和通信，sleep通常被用于暂停执行

### 2.为什么wait()，notify()，notifyAll被定义在Object中？

- 任何对象都可以作为锁，为了方便区分，把方法定义在对象上，该对象通知使用到该对象线程的唤醒和等待。（说白了就是为了方便，定义在线程中会比较复杂，一个线程持有的锁太多了就放弃锁的时候就很麻烦）

	- wait/notify方法也是依赖于monitor对象，不在同步块或者同步方法中使用，就会抛出java.lang.illegalMonitorStateException的异常。synchronized关键字底层也是依赖于monitor完成。

### 3.为什么wait()，notify(),notifyAll()必须在同步方法或者同步代码块中才能被调用？(重要)

- 一个线程需要调用wait方法的时候必须持有锁，然后释放锁并且进入等待阻塞状态，notify方法也是一样的，它会释放锁，以便其他线程获得锁。这些方法都需要线程持有锁，所以只能在同步方法或者同步代码块中调用

### 4.Thread类中的yield方法有什么作用？

- 让出cpu的使用权，使线程从运行状态变成可运行状态，当然下一次运行的线程也可能还是它，看系统如何分配。

### 5.sleep()方法和yield()方法有什么区别？

- 1.sleep方法给其他线程运行机会不会考虑线程的优先级，yield方法只会考虑优先级更高的线程
- 2.sleep方法会让线程转入阻塞状态，yield方法会让线程转入就绪状态
- 3.sleep方法声明会抛出InterruptedException，yield不会有任何异常

### 6.如何停止线程？

- 1.使用Interrupt停止线程，是线程正常退出通知线程应该停止了
- 2.stop方法停止容易出现资源未释放的情况，suspend容易出现死锁

### 7.interrupted和isInterrupted方法的区别？

- 1.Interrupted查看当前的中断信号，并清楚中断信号(把信号变成false)
- 2.isInterrupted查看当前中断信号是true还是false

### 8.什么是阻塞式方法？

- 程序一直等待该方法完成期间不会做任何事情，ServerSocket的accept方法就是一直等待客户端链接。

### 9.怎么唤醒一个线程？

- notify和notifyAll方法都是唤醒线程的方法(wait方法只是释放锁，不会让阻塞的线程唤醒)

### 10.notify和notifyAll有什么区别？

- notify只会随机唤醒一个线程，notifyAll会唤醒所有的线程

	- notify可能会出现信号丢失的情况

- notifyAll调用后会将所有线程由等待池移到锁池，参与锁的竞争，竞争失败继续留在锁池竞争

### 11.线程间如何共享数据？

- 1.公共的数据
- 2.共同拥有的数据，比如同一个runnable实现类

### 12.Java如何实现线程间的通讯和协作？

- 1.syncrhoized加锁的线程使用wait、notify、notifyAll方法
- 2.ReentrantLock类加锁的线程的Condition类的await()、signal()、signalAll()线程间的直接数据交换。
- 3.通过管道进行线程通信：字节流和字符流

### 13.什么是线程互斥和同步？

- 互斥：线程使用到某个资源时别的线程就要等待
- 同步：某一代码块有其他线程在执行时，其他线程就要等待
- 总结：互斥就是一种特殊的同步

### 14.servlet是线程安全的吗？

- Servlet不是线程安全的，它是单实例的，当多个线程同时访问同一个方法，是不能保证共享变量的线程安全的。

	- 扩展

		- 1.struts2的action是多实例多线程的，是线程安全的，每次请求都会new一个新的action分配给请求
		- 2.SpringMVC的Controller不是线程安全的，底层使用dispaterServlet继承的HttpServlet，采用的也是单实例

### 15.保证线程安全的三种方式

- 1.使用安全类。比如java.util.concurrent下的类AtomicInteger
- 2.使用自动锁synchronized
- 3.使用手动锁Lock

  Lock lock = new ReentrantLock();
  lock. lock();
  try {
      System. out. println("获得锁");
  } catch (Exception e) {
      // TODO: handle exception
  } finally {
      System. out. println("释放锁");
      lock. unlock();
  }
  
  

### 16.线程的构造方法，run方法是那个线程调用的？

- 那个线程下new的线程，那个线程调用的构造方法，run方法都是new出来的线程调用的

	- 例子：如果说上面的说法让你感到困惑，那么我举个例子，假设 Thread2 中 new 了Thread1，main 函数中 new 了 Thread2，那么：

（1）Thread2 的构造方法、静态块是 main 线程调用的，Thread2 的 run()方法是Thread2 自己调用的

（2）Thread1 的构造方法、静态块是 Thread2 调用的，Thread1 的 run()方法是Thread1 自己调用的

### 17.如何获取线程的dump文件？

- linux：kill -3 PID
- win: Ctrl+break

### 18.线程出现异常会怎么样？

- 1.处理了异常的情况：线程正常执行
- 2.没有处理的情况：有一个Thread.UncaughtExceptionHander类用于处理为捕获的异常造成线程中断的内嵌接口

	- 1.synchronized: 有锁的情况下会释放锁
	- 2.lock: 没有处理异常主动释放锁，可能会造成死锁或者资源未释放

### 19.java线程过多会怎么样？

- 1.消耗过多CPU
- 2.太多无用线程竞争锁
- 3.减低JVM稳定性(可能是开辟过多的线程栈)

### 20.为什么代码会重排序？

- 为了提供更好的性能。必须满足的条件

	- 1.单线程环境不能改变程序运行的结果
	- 2.存在数据依赖关系，不能重排序

### 21.两种线程规则

- 1.as-if-serial规则

	- as-if-serial语义保证单线程内程序的执行结果不被改变

- 2.happens-before规则

	- happens-before关系保证正确同步的多线程程序的执行结果不被改变

## 9.synchronized锁关键字

### 1.修饰方法和代码块，保证线程安全

### 2.单例模式情况是怎么样的？

public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}



- 1.为uniqueInstance提供内存空间
- 2.初始化 uniqueInstance
- 3.将 uniqueInstance 指向分配的内存地址
- 注意：多线程情况下，经过系统的指令重排序，执行顺序很有可能变成1-3-2，第一个线程还未初始化完毕，就已经提前将对象返回就有可能使用报错。

### 3.底层实现原理

- 1.底层通过monitor指令来实现
- 2.在锁代码块前添加指令monitorenter
- 3.在锁代码块后后添加monitorexit
- 4.之后又再次monitorexit，防止出现异常没有释放锁的情况
- 案例：

  public class SynchronizedDemo {
      public void method() {
          synchronized (this) {
              System.out.println("synchronized 代码块");
          }
      }
  }
  
  

	- 

### 4.可重入原理

- 重入锁原理意思是一个线程获取锁以后重新进入这个锁代码块，线程可以继续获取到锁
- 每次获取锁是计数器加一，释放锁时计数器减一，直到锁的计数器为0时才算是完全释放(可能出现在递归调用的同步代码中有吧)

### 5.自旋锁原理

- 很多锁住的部分都是一些很简单的代码，很快就运行完毕，让线程进入阻塞等待不太合适。为了解决线程从挂起阻塞到可运行的过程(用户态到内核态切换的问题)，在synchronized边界做忙循环，多次循环还没有拿到锁再做阻塞

### 6.synchronized的锁升级是什么原理？

- 1.锁对象头里有一个threadid字段，第一次访问的时候threadid为空，JVM让其持有偏向锁，并且将threadid置为当前线程id
- 2.第二次访问的时候先判断线程id是否一致，如果一致直接可以使用此锁对象，不一致升级为轻量级锁(答错过一次)

	- 扩展

		- 偏向锁

			- 简介

				-  一个线程反复的去获取/释放一个锁，如果这个锁是轻量级锁或者重量级锁，不断的加解锁显然是没有必要的，造成了资源的浪费。于是引入了偏向锁，偏向锁在获取资源的时候会在资源对象上记录该对象是偏向该线程的，偏向锁并不会主动释放，这样每次偏向锁进入的时候都会判断改资源是否是偏向自己的，如果是偏向自己的则不需要进行额外的操作，直接可以进入同步操作

			- 偏向锁获取过程

				- （1）访问Mark Word中偏向锁标志位是否设置成1，锁标志位是否为01——确认为可偏向状态
				- （2）如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤（5），否则进入步骤（3）
				- （3）如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行（5）；如果竞争失败，执行（4）
				- （4）如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。
				- （5）执行同步代码

		- 轻量级锁

			- 轻量级锁是偏向锁升级来的，当第二个线程加入锁竞争时，偏向锁就会升级为轻量级锁

		- 重量级锁

			- synchronized就是重量级锁

- 3.通过一定次数的循环还没有拿到锁对象，就会把锁升级为重量级锁synchronized(答错过一次)
- 锁的升级的目的：锁升级是为了减低了锁带来的性能消耗。

## 10.线程之间怎么知道变量被修改了？

### 1.volatile修饰变量，线程都能知道最新的修改

### 2.synchronized加锁

### 3.wait/notify线程之间相互等待通知

## 11.volatile关键字

### 作用

- 保证可见性和禁止指令重排序

	- 扩展

		- 提供的happens-before来保证一个线程的修改了另一个线程可见，被volatile修改的值会立即被更新到主存中去

- 和CAS结合，保证原子性，比如AtomicInteger类

### 问题

- 能用来修饰数组吗？

	- 可以，但是只是能保证数组的引用内立即可见，而不是整个数组都受到保护

- volatile变量和atomic变量有什么不同？

	- 1.volatile可以保证先行关系，无法保证原子性，例如count++就不是原子性的
	- 2.AtomicInteger的getAndIncrement就可以保证原子性

## 12.Lock锁

### 1.优势

- 1.可以使锁更公平
- 2.可以使线程在等待锁的时候响应中断
- 3.可以让线程尝试获取锁，并且无法获取锁的时候立即返回或者等待一段时间
- 4.可以在不同的范围内，以不同的顺序获取和释放锁
- 5.支持公平锁也支持非公平锁

	- 扩展：synchronized是非公平锁

### 2.什么是乐观锁和悲观锁

- 悲观锁

	- 总是设想最坏的情况，别人拿到数据的时候都会做修改，所以每次拿数据的时候都会加锁，想拿这个数据的线程都要阻塞

- 乐观锁

	- 设想每次拿数据的时候别人都不去修改，就没必要上锁，可以用个版本号判断别人有没有修改数据

### 3.什么是CAS

- compare and swap，比较交换，基于乐观锁的操作。

	- 例子：CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存地址里面的值和 A 的值是一样的，那么就将内存里面的值更新成 B。CAS是通过无限循环来获取数据的，若果在第一轮循环中，a 线程获取地址里面的值被b 线程修改了，那么 a 线程需要自旋，到下次循环才有可能机会执行。
	- 使用 CAS 操作来实现的(AtomicInteger,AtomicBoolean,AtomicLong)

		- 自己使用预期值的时候，会出现ABA问题

- 会产生什么问题？

	- 1.ABA问题

		- 线程A和B都要修改Map中的对象，A和B判断为没有线程修改数据，A阻塞了B继续，B把对象删除了，继续把判断标志位变为可以执行，A此时去执行就会出现找不到或者数据被修改的问题

			- 从 Java1.5 开始 JDK 的 atomic包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。

	- 2.循环时间长开销大

		- 资源竞争严重的情况，CAS自旋概率大，浪费CPU资源，效率比重量级锁低

	- 3.只能保证一个共享变量的原子操作

### 4.死锁

- 1.什么是死锁？

	- 互相持有对方需要的锁，发生阻塞现象，成为死锁

- 2.产生死锁的条件是什么？

	- 1.互斥条件

		- 互斥就是线程在某一时间内独占资源

	- 2.请求与保持条件

		- 一个进程因请求资源阻塞时，对已获得资源保持不放

	- 3.不剥夺条件

		- 进程已获得资源，在未使用完之前，不能强行剥夺

	- 4.循环等待条件

		- 若干线程之间形成一种头尾相接的循环等待条件资源的条件

- 3.怎么防止死锁？

	- 1.尽量使用tryLock(long timeout, TimeUtil unit)方法，设置超时时间，超时可以退出防止死锁
	- 2.尽量使用java的并发类代替手写锁
	- 3.尽量降低锁的使用粒度，尽量不要几个功能共用一把锁
	- 4.尽量减少同步的代码块

### 5.活锁

- 任务或者被执行者没有被阻塞，由于没有满足条件，导致一致重复尝试，失败，尝试，失败

	- 活锁不停地在改变状态，死锁变现为等待

### 6.饥饿

- 一个或者多个线程因为某种原因无法获得所需要的资源，导致一直无法执行的状态

	- 导致饥饿的原因？

		- 1.高优先级线程吞噬所有的低优先级线程的时间
		- 2.线程永久阻塞在一个等待进入同步块的状态，因为其他线程总能在它之前访问

### 7.多线程锁线程升级

- 无状态锁，偏向锁，轻量级锁，重量级锁，锁只能锁着竞争情况升级，不能降级

### 8.AQS(AbstractQueuedSynchronizer)锁

- 1.AQS原理概念

	- 如果被请求的共享资源空闲，当前请求资源的线程设置为有限工作线程，并且将共享资源设置为锁定状态，如果被请求的资源被占用，那么就需要一套线程阻塞等待机制以及唤醒时锁分配机制，这个机制AQS使用CLH队列实现的，暂时获取不到锁的线程加入到队列中。

		- 

- 2.底层代码

	- 使用volatile修饰保证线程可见性，状态信息通过protected类型的getState、setState、compareAndSetState进行操作

	  private volatile int state;//共享变量，使用volatile修饰保证线程可见性
	  //返回同步状态的当前值
	  protected final int getState() {  
	          return state;
	  }
	   // 设置同步状态的值
	  protected final void setState(int newState) { 
	          state = newState;
	  }
	  //原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
	  protected final boolean compareAndSetState(int expect, int update) {
	          return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
	  }
	  
	  

		- LockSupport源码(底层使用Unsafe类)

- 3.AQS对资源的共享方式

	- 1.独占(Exclusive)

		- 只有一个线程能执行，如ReentrantLock

			- 公平锁

				- 按照线程在队列中的顺序，先到者先拿到锁

			- 非公平锁

				- 当线程要获得锁时，无视队列顺序，谁抢到就是谁的

					- 通常非公平锁效率更高，A获取锁，B就要等待，此时C过来，如果是公平锁，那么C就要去B后面排队等待，不公平的话C直接执行了，这就是额外的开销。

	- 2.共享(share)

		- 多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock

- 4.如何使用

	- AQS的设计是基于模板方法模式
	- 自定义同步器只需要实现对共享资源state的获取和释放方式即可，线程等待队列的维护AQS已经在顶层实现好了
	- 需要实现的方法

	  isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
	  tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
	  tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
	  tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
	  tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
	  
	  
	  

- 5.ReentrantLock(可重入锁)

	- 1.概念

		- 支持重入性，表示能够对共享资源能够重复加锁，即当前线程获取该锁再次获取不会被阻塞

			- 扩展

				- synchronized是隐式支持重入性，通过自增自减的方式实现重入

- 6.ReentrantLockReadWriteLock(可重入读写锁)

	- 1.概念

		- ReentrantLockReadWriteLock是ReadWriteLock接口的一个具体实现，实现了读写分离，写锁独占，读锁共享，不会互斥，读和写，写和写才会互相排斥，提升了读写性能

	- 2.特性

		- 1.公平选择性

			- 支持非公共和公平的锁获取方式，吞吐量还是非公平优于公平

		- 2.重入性

			- 读和写都支持线程重入

		- 3.锁降级

			- 遵循获取写锁、获取读锁再释放的次序，写锁能够降级为读锁

- 7.Condition源码分析与等待通知机制

### 9.ConcurrentHashMap锁实现

- JDK1.6

	- 2.Segment维护了哈希散列的若干个桶，每个桶由HashEntry构成的链表
	- 1.Segment继承了ReentrantLock充当锁的角色，为每一个segment提供线程保障

- JDK1.8

	- 抛弃原有的Segment分段锁，使用CAS+synchronized保证线程安全

- 并发度都是默认16

### 10.并发容器之CopyOnWriteArrayList

- 1.优点

	- 读写分离
	- 最终一致性
	- 开辟另一个空间的思路解决并发冲突

- 2.缺点

	- 写操作的时候需要拷贝数组，消耗内存，可能会导致full gc
	- 没法满足实时性读取的要求

### 11.问题

- 1.Lock锁可以中断吗？Synchronized可以中断吗？

	- Lock可以中断，Synchronized不能中断

## 13.ThreadLocal

### 概念

- 是一个本地线程副本变量工具类，在每一个线程中都创建一个ThreadLocalMap对象，简单来说ThreadLocal就是一种以空间换取时间的做法，每个线程都可以访问自己内部的ThreadLocalMap对象的value，通过这种方式避免资源在多线程之间共享

	- 扩展：任何线程局部变量在工作完成后没有释放，java应用就存在内存泄露的风险

### 经典使用场景

- 为每一个线程分配一个JDBC连接的Connection，这样可以保证线程都在各自的Connection上进行数据库操作，不会出现线程A关了，线程B还在使用的情况；还有Session管理问题

### 问题

- 1.什么是线程局部变量？

	- 线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程之间共享。java提供ThreadLocal类来支持线程局部变量，是线程安全的方式，但是存在内存泄漏的风险，比如在线程执行完毕，没有关闭资源

- 2.ThreadLocal造成内存泄漏的原因？

	- ThreadLocalMap的key是弱引用，而value是强引用，ThreadLocal没有被外部强引用的情况下在垃圾回收的时候，key会被清理掉，而value不会被清理。这个时候可能就会出现key为null的情况而value没有被清理，value无法被GC回收，这时候可能会出现内存泄漏

		- WeakReference类做的弱引用，下次GC就会被回收

- 3.ThreadLocal解决内存泄露的方案？

	- 1.每次使用完ThreadLocal，调用它的remove方法，清理数据

## 14.并发容器BlockingQueue

### 介绍

- BlockingQueue 接口是 Queue 的子接口，它的主要用途并不是作为容器，而是作为线程同步的的工具，因此他具有一个很明显的特性，当生产者线程试图向 BlockingQueue 放入元素时，如果队列已满，则线程被阻塞，当消费者线程试图从中取出一个元素时，如果队列为空，则该线程会被阻塞，正是因为它所具有这个特性，所以在程序中多个线程交替向 BlockingQueue 中放入元素，取出元素，它可以很好的控制线程之间的通信

	- 阻塞队列使用最经典的场景就是 socket 客户端数据的读取和解析，读取数据的线程不断将数据放入队列，然后解析线程不断从队列取数据解析。

### JDK7的七个阻塞队列

- 1.ArrayBlockingQueue

	- 一个由数组结构组成的有界阻塞队列

- 2.LinkedBlockingQueue

	- 一个由链表结构组成的有界阻塞队列

		- 默认容量为Integer.MAX_VALUE，也就是差不多21亿，使用前要注意

- 3.PriorityBlockingQueue

	- 一个支持优先级排序的无界阻塞队列

- 4.DelayQueue

	- 一个使用优先级队列实现的无界阻塞队列

- 5.SynchronousQueue

	- 一个不存储元素的阻塞队列

- 6.LinkedTransferQueue

	- 一个由链表结构组成的无界阻塞队列

- 7.LinkedBlockingDeque

	- 一个由链表结构组成的双向阻塞队列

### 并发容器之ConcurrentLinkedQueue详解与源码分析

### 并发容器之ArrayBlockingQueue与LinkedBlockingQueue详解

## 15.线程池

### 在面向对象编程中，创建和销毁对象是很费时间的，因为创建一个对象要获取内存资源或者其它更多资源。在 Java 中更是如此，虚拟机将试图跟踪每一个对象，以便能够在对象销毁后进行垃圾回收。所以提高服务程序效率的一个手段就是尽可能减少创建和销毁对象的次数，特别是一些很耗资源的对象创建和销毁，这就是”池化资源”技术产生的原因

- 和springbean容器一样的意思原理

### 工具类 Executors创建线程池

- newSingleThreadExecutor

	- 创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

		- 底层使用LinkedBlockingQueue默认队列大小是Integer.MAX_VALUE，很可能造成OOM

- newFixedThreadPool

	- 创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

		- 底层使用LinkedBlockingQueue默认队列大小是Integer.MAX_VALUE，很可能造成OOM

- newCachedThreadPool

	- 创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60 秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说 JVM）能够创建的最大线程大小。

		- 底层使用SynchronousQueue，不存储元素的阻塞队列，最大线程是Integer.MAX_VALUE也会照成OOM

- newScheduledThreadPool

	- 创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

		- 源码设置最大线程数为Integer.MAX_VALUE，容易OOM

### 优点

- 1.降低资源消耗：重用存在的线程，减少对象创建销毁的开销。
- 2.提高响应速度：有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。当任务达到时，任务可以不需要等到线程创建就能立即执行。
- 3.提高性能的可管理性：线程是稀缺资源，无限创建，不仅消耗系统资源，还会降低系统稳定新，使用线程池可以进行统一的分配、调优和监控

### 线程池状态

- 图：
- running

	- 正常状态，接收新的任务，处理等待队列中的任务

- shutdown

	- 不接受新的任务提交，但提交会继续处理等待队列中的任务

- stop

	- 不接受新的任务提交，不在处理等待队列中的任务，中断正在执行任务的线程

- tidying

	- 所有的任务都销毁了，workConut为0，线程池的状态在转换为tidying状态时，会执行钩子方法terminated()

- terminated

	- terminated()方法结束后，线程池的状态就会变成这个

### ThreadPoolExecutor创建线程池的区别

- 阿里巴巴开发规范不允许使用Executors去创建，原因

	- 1.newFixedThreadPool和newSingleThreadExecutor，堆积请求处理队列可能会耗费非常大的内存，甚至OOM
	- 2.newCachedThreadPool和newScheduledThreadPool，线程的最大数量是Integer.MAX_VALUE，可能会创建非常多的线程甚至OOM

- 创建线程池

	- 构造函数创建，参数

		- corePoolSize

			- 核心线程数，线程定义了最小可以同时运行的线程数量

		- maxinumPoolSize

			- 线程池中允许存在的工作线程的最大数量

		- workQueue

			- 当新的任务来的时候会先判断当前运行的线程数量是否达到核心线程数量，如果达到的话，任务就会被存放在队列中

		- keepAliveTime

			- 线程池中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，知道等待大的时间超过keepAliveTime还会被回收

		- unit

			- keepAliveTime参数的时间单位

		- threadFactory

			- 为线程池提供创建新线程的工厂

		- handler

			- 线程池任务队列超过maxinumPoolSize之后的拒绝策略

	- 案例

		- 任务

		  import java.util.Date;
		  
		  /**
		   * 这是一个简单的Runnable类，需要大约5秒钟来执行其任务。
		   */
		  public class MyRunnable implements Runnable {
		  
		      private String command;
		  
		      public MyRunnable(String s) {
		          this.command = s;
		      }
		  
		      @Override
		      public void run() {
		          System.out.println(Thread.currentThread().getName() + " Start. Time = " + new Date());
		          processCommand();
		          System.out.println(Thread.currentThread().getName() + " End. Time = " + new Date());
		      }
		  
		      private void processCommand() {
		          try {
		              Thread.sleep(5000);
		          } catch (InterruptedException e) {
		              e.printStackTrace();
		          }
		      }
		  
		      @Override
		      public String toString() {
		          return this.command;
		      }
		  }
		  
		  

		- 线程池

		  import java.util.concurrent.ArrayBlockingQueue;
		  import java.util.concurrent.ThreadPoolExecutor;
		  import java.util.concurrent.TimeUnit;
		  
		  public class ThreadPoolExecutorDemo {
		  
		      private static final int CORE_POOL_SIZE = 5;
		      private static final int MAX_POOL_SIZE = 10;
		      private static final int QUEUE_CAPACITY = 100;
		      private static final Long KEEP_ALIVE_TIME = 1L;
		      public static void main(String[] args) {
		  
		          //使用阿里巴巴推荐的创建线程池的方式
		          //通过ThreadPoolExecutor构造函数自定义参数创建
		          ThreadPoolExecutor executor = new ThreadPoolExecutor(
		                  CORE_POOL_SIZE,
		                  MAX_POOL_SIZE,
		                  KEEP_ALIVE_TIME,
		                  TimeUnit.SECONDS,
		                  new ArrayBlockingQueue<>(QUEUE_CAPACITY),
		                  new ThreadPoolExecutor.CallerRunsPolicy());
		  
		          for (int i = 0; i < 10; i++) {
		              //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
		              Runnable worker = new MyRunnable("" + i);
		              //执行Runnable
		              executor.execute(worker);
		          }
		          //终止线程池
		          executor.shutdown();
		          while (!executor.isTerminated()) {
		          }
		          System.out.println("Finished all threads");
		      }
		  }
		  
		  

- 饱和策略

	- ThreadPoolExecutor.AbortPolicy

		- 抛出RejectedExecutionException拒绝新任务的处理

	- ThreadPoolExecutor.CallerRunsPolicy

		- 调用自己的线程执行任务。这种策略会降低对与新任务提交的速度，影响程序的性能

	- ThreadPoolExecutor.DiscardPolicy

		- 不处理新的任务，直接丢弃

	- ThreadPoolExecutor.DiscardOldestPolicy

		- 此策略将丢弃最早的未处理的任务请求

- 线程池实现原理

	- 

### ScheduledThreadPoolExecutor详解

## 16.Atomic类

### 1.原理

- 类的基本特性就是在多线程的环境下，当有多个线程同时对单个(包括基本类型和引用类型)变量进行操作时，具有排他性，即当多个线程同时对该变量的值进行更新的时候，仅有一个线程能成功，而未成功的线程可以像自旋锁一样继续尝试，一直等到执行成功
- 利用CAS+volatile和native方法来保证原子操作，从而避免synchronized的高开销，执行效率大为提升

	- CAS原理就是拿期望值和原本的一个值做比较，如果值相同则更新成新的值。UnSafe类的objectFieldOffset()方法是一个本地方法，这个方法是用来拿到"原来的值"的内存地址，返回值是valueOffset，另外value是一个volatile变量，在内存中可见，保证JVM可以在任何时刻任何线程拿到该变量的最新值

		- 代码

		  // setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
		  private static final Unsafe unsafe = Unsafe.getUnsafe();
		  private static final long valueOffset;
		  
		  static {
		   try {
		   valueOffset = unsafe.objectFieldOffset
		   (AtomicInteger.class.getDeclaredField("value"));
		   } catch (Exception ex) { throw new Error(ex); }
		  }
		  
		  private volatile int value;
		  ————————————————
		  版权声明：本文为CSDN博主「ThinkWon」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
		  原文链接：https://blog.csdn.net/ThinkWon/article/details/104863992
		  

## 17.并发工具

### CountDownLatch(常用)

- 一般用于某个线程A等待若干个线程执行完任务之后，它才执行

	- 强调一个线程等待多个线程完成某件事情

		- 调用countDown方法后，当前线程不会阻塞，会继续往下执行

			- 不能复用

### CyclicBarrier(常用)

- cyclicBarrier一般用于一组线程互相等待至某个状态，然后这组线程在执行

	- 多个线程等待某一件事完成，在一起完成某件事

		- 会阻塞当前线程，直到CyclicBarrier指定的线程达到指定点的时候，才会往下执行

			- 可以复用

### Semaphore(常用)

- 信号量

	- 作用是限制某段代码的并发数，Semaphore的构造函数可以传入一个int型整数n，表示某段代码最多只有n个线程可以访问，如果超出了n，那么请等待，等到某个线程执行完毕这块代码，下个线程在进入，如果设置为1，相当于synchronized

		- 相比较synchionized和ReentrantLock都是一次只允许一个线程访问资源，Semaphore(信号量)可以允许多个线程访问某个资源

### Exchanger

- 线程协作工具类，用于两个线程之间的数据交换，两个线程都执行到exchange方法，那么这两个线程就可以交换数据

## 

