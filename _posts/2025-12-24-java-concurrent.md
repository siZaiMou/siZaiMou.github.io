---
layout: post
title: "Java 多线程：学习记录和补充"
categories: [学习记录]
---

# 并发

# 多线程

## Java线程和操作系统线程

JDK 1.2 之前，Java 线程是基于绿色线程（Green Threads）实现的，这是一种用户级线程（用户线程），也就是说 JVM 自己模拟了多线程的运行，而不依赖于操作系统。

在 JDK 1.2 及以后，Java 线程改为基于原生线程（Native Threads）实现，也就是说 JVM 直接使用操作系统原生的内核级线程（内核线程）来实现 Java 线程，由操作系统内核进行线程的调度和管理。Java 底层会调用 pthread_create 来创建线程，所以本质上 java 程序创建的线程，就是和操作系统线程是一样的，是 1 对 1 的线程模型。

## 线程安全

**竞态（Race Condition，也译为 “竞争条件”）** 指的是**多个线程同时访问共享资源，且至少有一个线程在修改该资源时，最终结果依赖于线程执行的先后顺序或时间调度**，从而导致程序出现不可预期的行为（如数据不一致、逻辑错误等）。

* 原子性：一个操作或多个操作的集合，要么全部执行且执行过程不会被任何其他线程干扰，要么全部不执行。atomic包（这个包提供了一些支持原子操作的类）和synchronized关键字
* 可见性：当一个线程修改了共享变量的值后，其他线程能够 “立即” 看到这个修改后的结果。synchronized和volatile
* 有序性：程序执行的顺序按照代码的 “先后顺序” 执行，即禁止指令重排序对多线程执行结果的干扰。在Java中使用了happens-before原则来确保有序性。
  * 即一个操作的结果需要对另一个操作可见，并且第一个操作在第二个操作之前发生。

## 线程创建方法

1. 继承Thread类

run()方法中定义了线程执行的具体任务。创建该类的实例后，通过调用start()方法启动线程。

优点：

* 实现简单，直接继承即可使用。
* 可直接通过<mark style="background-color: #BBBFC4">`this`</mark>获取当前线程对象。
缺点:

* Java 单继承限制：继承<mark style="background-color: #BBBFC4">`Thread`</mark>后无法再继承其他类，灵活性低。
* 线程任务与线程本身耦合：线程对象与任务逻辑绑定，不利于任务复用。

```Java
public class ExtendsThread extends Thread {
    @Override
    public void run() {
        System.out.println("1......");
    }
    public static void main(String[] args) {
        new ExtendsThread().start();
    }
}
```

1. 实现Runnable接口

实现Runnable接口需要重写run()方法，然后将此Runnable对象作为参数传递给Thread类的构造器，创建Thread对象后调用其start()方法启动线程。

优点：

* 规避单继承限制：可同时继承其他类，灵活性高。
* 任务与线程分离：一个<mark style="background-color: #BBBFC4">`Runnable`</mark>实例可被多个线程共享，适合多线程执行同一任务（如卖票系统）。
缺点：

* 无法直接获取线程执行结果（无返回值）。
* 启动线程需依赖<mark style="background-color: #BBBFC4">`Thread`</mark>类，代码稍繁琐。

```Java
public class ImplementsRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("2......");
    }
    public static void main(String[] args) {
        ImplementsRunnable runnable = new ImplementsRunnable();
        new Thread(runnable).start();
    }
}
```

1. 实现Callable接口和FutureTask

Callable接口类似于Runnable，但Callable的call()方法可以有返回值并且可以抛出异常。要执行Callable任务，需将它包装进一个FutureTask，因为Thread类的构造器只接受Runnable参数，而FutureTask实现了Runnable接口。

**优点**：

* 有返回值：可通过<mark style="background-color: #BBBFC4">`Future`</mark>获取线程执行结果，解决前两种方法无返回值的问题。
* 可抛出异常：<mark style="background-color: #BBBFC4">`call()`</mark>方法声明了异常，便于错误处理。
* 同样规避单继承限制，灵活性高。
**缺点**：

* 代码复杂度高：需结合<mark style="background-color: #BBBFC4">`FutureTask`</mark>和<mark style="background-color: #BBBFC4">`Thread`</mark>使用。
* <mark style="background-color: #BBBFC4">`get()`</mark>方法可能阻塞：若线程未执行完，调用<mark style="background-color: #BBBFC4">`get()`</mark>会阻塞当前线程，可能影响性能。

```Java
public class ImplementsCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("3......");
        return "zhuZi";
    }
    
    public static void main(String[] args) throws Exception {
        ImplementsCallable callable = new ImplementsCallable();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        new Thread(futureTask).start();
        System.out.println(futureTask.get());
    }
}

// 单纯futretask
public class UseFutureTask {
    public static void main(String[] args) {
        FutureTask<String> futureTask = new FutureTask<>(() -> {
            System.out.println("7......");
            return "zhuZi";
        });
        new Thread(futureTask).start();
    }
}
```

1. 线程池


```Java
// Java.util.cocurrent 下

public class UseExecutorService {
    public static void main(String[] args) {
        // Executors类的静态方法
        ExecutorService poolA = Executors.newFixedThreadPool(2);
        poolA.execute(()->{
            System.out.println("4A......");
        });
        poolA.shutdown();
        // 自定义线程池
        ThreadPoolExecutor poolB = new ThreadPoolExecutor(2, 3, 0,
                TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(3),
                Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
        poolB.submit(()->{
            System.out.println("4B......");
        });
        poolB.shutdown();
    }
}
```

## 常用方法

### start和run区别

调用 start()方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。

直接执行 run() 方法，会把 run() 方法当成一个当前线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

### sleep和wait区别

sleep() 方法没有释放锁，而 wait() 方法释放了锁

wait() 通常被用于线程间交互/通信，sleep()通常被用于暂停执行。

wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify()或者 notifyAll() 方法。sleep()方法执行完成后，线程会自动苏醒，或者也可以使用 wait(long timeout) 超时后线程会自动苏醒。

sleep() 是 Thread 类的静态本地方法，wait() 则是 Object 类的本地方法。

### Wait

**wait() 与 notify/notifyAll 方法必须在同步代码块中使用，因此线程在执行它们时，肯定是进入了临界区中的，即该线程肯定是获得了锁的。**

当线程执行wait()时，会把当前的锁释放，然后让出CPU，进入等待状态。

当执行notify/notifyAll方法时，会唤醒一个处于等待该 对象锁 的线程，然后继续往下执行，直到执行完退出对象锁锁住的区域（synchronized修饰的代码块）后再释放锁。

当wait被唤醒或超时，并不是直接进入到运行或者就绪状态，而是先进入到Block状态，抢锁成功后，才能进入到可运行状态

![](/assets/SBTUb1AZwo4JPwxwlurcS7JDnWd.octet-stream)

### Yield

和 sleep 一样都是 Thread 类的方法，都是暂停当前正在执行的线程对象，不会释放资源锁，和 sleep 不同的是 yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。还有一点和 sleep 不同的是 yield 方法只能使同优先级或更高优先级的线程有执行的机会

### Join

不会释放锁

t.join()方法只会使主线程(或者说调用t.join()的线程)进入等待池并等待t线程执行完毕后才会被唤醒。并不影响同一时刻处在运行状态的其他线程。

![](/assets/BsOpbwO2Do0QbWxqyrpchVidnpf.jpeg)

![](/assets/W6svbggGKojrWYxSIYpcVGRonfb.png)

### Notify&NotifyAll

相同点：唤醒等待的线程，同样最多只有一个线程能获得锁，同样不能控制哪个线程获得锁。

* notify：唤醒一个线程，其他线程依然处于wait的等待唤醒状态，如果被唤醒的线程结束时没调用notify，其他线程就永远没人去唤醒，只能等待超时，或者被中断
  * notify在源码的注释中说到notify选择唤醒的线程是任意的，但是依赖于具体实现的jvm。hotspot对notofy()的实现并不是我们以为的随机唤醒，而是“先进先出”的顺序唤醒。
* notifyAll：所有线程退出wait的状态，开始竞争锁，但只有一个线程能抢到，这个线程执行完后，其他线程又会有一个幸运儿脱颖而出得到锁

### Interrupt

每个线程内部都有一个 **boolean 类型的中断标志位**（初始为 <mark style="background-color: #BBBFC4">`false`</mark>），用于标记线程是否 “被请求中断”。中断机制的所有操作都围绕这个标志位展开：

* 当其他线程调用 <mark style="background-color: #BBBFC4">`thread.interrupt()`</mark> 时，会将目标线程的中断标志位设为 <mark style="background-color: #BBBFC4">`true`</mark>；
  中断标志位只是 “信号”，线程如何响应完全由自身逻辑决定：
  * **立即终止**：检测到中断后，释放资源（如关闭文件、释放锁），主动退出 <mark style="background-color: #BBBFC4">`run()`</mark> 方法；
  * **延迟处理**：先完成当前任务，再响应中断；
  * **忽略中断**：继续执行（不推荐，会导致中断机制失效）。
* 线程自身可以通过 <mark style="background-color: #BBBFC4">`Thread.interrupted()`</mark> 或 <mark style="background-color: #BBBFC4">`thread.isInterrupted()`</mark> 检查标志位，判断是否需要中断；
* 线程响应中断后，可自行决定是否停止执行（如退出循环、释放资源后结束）。
  * 若目标线程**正在执行非阻塞代码**（如普通循环）：仅将中断标志位设为 <mark style="background-color: #BBBFC4">`true`</mark>，不影响线程继续执行（需线程自行检查标志位）；
  * 若目标线程**正在执行阻塞方法**（如 <mark style="background-color: #BBBFC4">`Thread.sleep()`</mark>、<mark style="background-color: #BBBFC4">`Object.wait()`</mark>、<mark style="background-color: #BBBFC4">`LockSupport.park()`</mark> 等）：会立即抛出 <mark style="background-color: #BBBFC4">`InterruptedException`</mark>，同时**清除中断标志位**（重置为 <mark style="background-color: #BBBFC4">`false`</mark>），迫使线程从阻塞中唤醒并处理中断。

### 生命周期

![](/assets/IGBqb1WkFoH3lkxnekBcmbEFnAh.png)

### bolcked和waiting区别

* blocked官方定义：线程因等待 Monitor 锁而阻塞的状态。
* 触发条件
  * 线程进入BLOCKED状态通常是因为试图获取一个对象的锁（monitor lock），但该锁已经被另一个线程持有。这通常发生在尝试进入synchronized块或方法时，如果锁已被占用，则线程将被阻塞直到锁可用。在线程的整个生命周期里面，只有 Synchronized 同步锁等待才会存在这个状态。 
  * 线程进入WAITING状态是因为它正在等待另一个线程执行某些操作，例如调用Object.wait()方法、Thread.join()方法或LockSupport.park()方法。在这种状态下，线程将不会消耗CPU资源，并且不会参与锁的竞争。
* 唤醒机制
  * 当一个线程BLOCKED等待锁时，一旦锁被释放，线程将有机会重新尝试获取锁。如果锁此时未被其他线程获取，那么线程可以从BLOCKED状态变为RUNNABLE状态。
  * 线程在WAITING状态中需要被显式唤醒。例如，如果线程调用了Object.wait()，那么它必须等待另一个线程调用同一对象上的Object.notify()或Object.notifyAll()方法才能被唤醒。

# 并发安全

## JUC常用类

### 线程池相关

* ThreadPoolExecutor：最核心的线程池类，用于创建和管理线程池。通过它可以灵活地配置线程池的参数，如核心线程数、最大线程数、任务队列等，以满足不同的并发处理需求。
* Executors：线程池工厂类，提供了一系列静态方法来创建不同类型的线程池：

### 并发集合

* ConcurrentHashMap：线程安全的哈希映射表，用于在多线程环境下高效地存储和访问键值对。它采用了分段锁等技术，允许多个线程同时访问不同的段，提高了并发性能。
* CopyOnWriteArrayList：线程安全的列表，在对列表进行修改操作时，会创建一个新的底层数组，将修改操作应用到新数组上，而读操作仍然可以在旧数组上进行，从而实现了读写分离，提高了并发读的性能，适用于读多写少的场景。

![](/assets/E4yAbQpLQoM4AWxXoGbcG8u7nnb.png)



### 同步工具类

* java.util.concurrent.locks.ReentrantLock、Condition、ReadWriteLock;
* CountDownLatch：允许一个或多个线程等待其他一组线程完成操作后再继续执行。它通过一个计数器来实现，计数器初始化为线程的数量，每个线程完成任务后调用countDown方法将计数器减一，当计数器为零时，等待的线程可以继续执行。常用于多个线程完成各自任务后，再进行汇总或下一步操作的场景。

![](/assets/T83gbyU4vozyz1x3bdFcqVF4noh.png)

* CyclicBarrier：让一组线程互相等待，直到所有线程都到达某个屏障点后，再一起继续执行。CyclicBarrier可以重复使用，当所有线程都通过屏障后，计数器会重置，可以再次用于下一轮的等待。适用于多个线程需要协同工作，在某个阶段完成后再一起进入下一个阶段的场景。

![](/assets/Nzqvbo6TooQMPixDVC3cupm7nVh.png)

* Semaphore：信号量，用于控制同时访问某个资源的线程数量。它维护了一个许可计数器，线程在访问资源前需要获取许可，如果有可用许可，则获取成功并将许可计数器减一，否则线程需要等待，直到有其他线程释放许可。常用于控制对有限资源的访问，如数据库连接池、线程池中的线程数量等。

![](/assets/Mp6YbXGpZoeESSxywaUc8KPtnwd.png)

* 条件变量：条件变量（Condition）是与锁（`Lock`）结合使用的一种线程同步机制，用于实现更复杂的线程间协作。条件变量允许线程在特定条件下等待，并在条件满足时被唤醒。它是 `Object` 的 `wait()` 和 `notify()` 机制的增强版，提供了更灵活和可扩展的功能。
  * 为什么互斥锁需要条件变量：以一个<span style="color: #C7D5F6">[生产者消费者](https://zhida.zhihu.com/search?content_id=189206560&content_type=Article&match_order=1&q=%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85&zhida_source=entity)</span>的例子来看，生产者和消费者通过一个队列连接，因为队列属于共享变量，所以在访问队列时需要加锁。生产者向队列中放入消息的时间是不一定的，因为消费者不知道队列什么时候有消息，所以只能不停循环判断或者sleep一段时间，不停循环会浪费cpu资源，如果sleep那么要sleep多久，sleep太短又会浪费资源，sleep太长又会导致消息消费不及时。
  * 为什么条件变量需要互斥锁：消费者的逻辑 可以简单分为两步：
    1. 消费消息直至消费完；
    1. 执行cond.wait(lock)开始等待下一次通知。
    1. 如果有互斥锁的情况下，这两步是原子的，就是在这个过程中是不会有新的消息添加到队列中的。那如果没有互斥锁保护，那么这两步就不是原子的了，比如刚执行完步骤1，生产者在队列里添加了一个消息，生产者添加消息并发送通知之后消费者才开始执行步骤2，这个时候就会导致这个新添加的消息无法及时被消费者消费到。

![](/assets/NvCqbfswXoWKiwxcY6ncSoQHnQg.png)

![](/assets/Uz1lbJYcaoAb04x8a9ycJniUnHh.png)

### 原子类

* AtomicInteger：原子整数类，提供了对整数类型的原子操作，如自增、自减、比较并交换等。通过硬件级别的原子指令来保证操作的原子性和线程安全性，避免了使用锁带来的性能开销，在多线程环境下对整数进行计数、状态标记等操作非常方便。
  * 内部通过 <mark style="background-color: #BBBFC4">`volatile int value`</mark> 存储整数
  * 依赖 <mark style="background-color: #BBBFC4">`Unsafe`</mark> 类（Java 底层的 unsafe 工具类，提供直接操作内存的方法）的 <mark style="background-color: #BBBFC4">`compareAndSwapInt`</mark> 实现原子更新
  
```Java
// AutomicInteger、AutomicLong、AutomicBoolean
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue, lazySet 提供了一种比 set 方法更弱的语义，可能导致其他线程在之后的一小段时间内还是可以读到旧的值，但可能更高效。
```

* AtomicIntegerArray：原子数组类
  * 内部通过 <mark style="background-color: #BBBFC4">`private final int[] array`</mark> 存储数组，数组本身不具备 <mark style="background-color: #BBBFC4">`volatile`</mark> 语义，但通过 CAS 直接操作数组元素的内存地址保证原子性。
  * 数组元素的内存地址 = 数组对象的基地址 + 元素索引 × 元素大小（<mark style="background-color: #BBBFC4">`int`</mark> 占 4 字节）。<mark style="background-color: #BBBFC4">`Unsafe`</mark> 提供 <mark style="background-color: #BBBFC4">`arrayBaseOffset`</mark> 和 <mark style="background-color: #BBBFC4">`arrayIndexScale`</mark> 方法获取基地址和元素大小
  * 对数组元素的原子操作（如 <mark style="background-color: #BBBFC4">`compareAndSet`</mark>、<mark style="background-color: #BBBFC4">`getAndIncrement`</mark>）通过 <mark style="background-color: #BBBFC4">`Unsafe`</mark> 的 <mark style="background-color: #BBBFC4">`compareAndSwapInt`</mark> 直接操作内存地址实现
  
```Java
public final int get(int i) //获取 index=i 位置元素的值
public final int getAndSet(int i, int newValue)//返回 index=i 位置的当前的值，并将其设置为新值：newValue
public final int getAndIncrement(int i)//获取 index=i 位置元素的值，并让该位置的元素自增
public final int getAndDecrement(int i) //获取 index=i 位置元素的值，并让该位置的元素自减
public final int getAndAdd(int i, int delta) //获取 index=i 位置元素的值，并加上预期的值
boolean compareAndSet(int i, int expect, int update) //如果输入的数值等于预期值，则以原子方式将 index=i 位置的元素值设置为输入值（update）
public final void lazySet(int i, int newValue)//最终 将index=i 位置的元素设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

* AtomicReference：原子引用类，用于对对象引用进行原子操作。可以保证在多线程环境下，对对象的更新操作是原子性的，即要么全部成功，要么全部失败，不会出现数据不一致的情况。常用于实现无锁数据结构或需要对对象进行原子更新的场景。
  * 内部通过 <mark style="background-color: #BBBFC4">`volatile V value`</mark> 存储对象引用（<mark style="background-color: #BBBFC4">`volatile`</mark> 保证引用的可见性，即多线程能看到最新的对象引用）。
  * 依赖 <mark style="background-color: #BBBFC4">`Unsafe`</mark> 的 <mark style="background-color: #BBBFC4">`compareAndSwapObject`</mark> 方法，直接操作对象引用的内存地址


```Java
@Data
public class BankCard {

    private final String accountName;
    private final int money;

    // 构造函数初始化 accountName 和 money
    public BankCard(String accountName, int money) {
        this.accountName = accountName;
        this.money = money;
    }
}

public class BankCardARTest {
    private static AtomicReference<BankCard> *bankCardRef*= new AtomicReference<>(new BankCard("cxuan", 100));

    public static void main(String[] args) {

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                while (true) {
                    // 使用 AtomicReference.get 获取
                    final BankCard card = *bankCardRef*.get();
                    BankCard newCard = new BankCard(card.getAccountName(), card.getMoney() + 100);
                    // 使用 CAS 乐观锁进行非阻塞更新
                    if (*bankCardRef*.compareAndSet(card, newCard)) {
                        System.*out*.println(newCard);
                    }
                    try {
                        TimeUnit.*SECONDS*.sleep(1);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}
```

## 保证线程安全的关键字或类

### synchronized关键字

同步代码块或方法，确保同一时刻只有一个线程可以访问这些代码。对象锁是通过关键字锁定对象的监视器（monitor）来实现的。可以修饰实例方法、静态方法(和synchronized(this)一样，都是给类加锁)、代码块。

#### 原理

在.class字节码文件中，synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。


```PlainText
monitorenter //会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。
...
monitorexit //对象锁的的拥有者线程才可以执行 monitorexit 指令来释放锁。在执行 monitorexit 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁。
```

Java会为每一个对象和对象的Class对象分配一个ObjectMonitor对象，他是一个C++结构体，ObjectMonitor用来维护当前持有锁的线程，阻塞等待锁释放的线程链表，调用了wait阻塞等待notify的线程链表。

每个对象中都内置了一个 ObjectMonitor对象。执行monitorenter指令时，线程试图获取锁也就是获取对象监视器monitor的持有权。monitor由c++实现。

### volatile关键字

https://blog.csdn.net/bfj11/article/details/123949405?sharetype=blog&shareId=123949405&sharerefer=APP&sharesource=mzs0mzs&sharefrom=link

1. 保证（**共享变量/成员变量**）变量的可见性，每次使用它都需要到主存中进行读取(线程共享)。普通的变量则会被线程放在本地内存里。
    * 普通变量在多线程环境下可能会被线程缓存，导致一个线程的修改对其他线程不可见。
      * 现代 CPU 为提高效率，会将共享内存中的数据缓存到线程对应的**CPU 缓存。**
    * 当一个线程修改了 <mark style="background-color: #BBBFC4">`volatile`</mark> 变量的值，其他线程可以立即看到这个修改，而不会使用本地缓存中的旧值。
2. <mark style="background-color: #BBBFC4">`volatile`</mark> 变量会禁止 JVM 和 CPU 对其进行指令重排序优化，确保代码的执行顺序与编写顺序一致。
    * 大多数现代<span style="color: #C7D5F6">[微处理器](https://zhida.zhihu.com/search?content_id=179422856&content_type=Article&match_order=1&q=%E5%BE%AE%E5%A4%84%E7%90%86%E5%99%A8&zhida_source=entity)</span>都会采用将指令乱序执行（out-of-order execution，简称OoOE或OOE）的方法，在条件允许的情况下，直接运行当前有能力立即执行的后续指令，避开获取下一条指令所需数据时造成的等待3。通过乱序执行的技术，处理器可以大大提高执行效率。

### Lock接口和ReentrantLock类

Lock接口和ReentrantLock类java.util.concurrent.locks.Lock接口提供了比synchronized更强大的锁定机制，ReentrantLock是一个实现该接口的例子，提供了更灵活的锁管理和更高的性能。

### 原子类

AtomicInteger等。

### 线程局部变量

java.lang.ThreadLocal类可以为每个线程提供独立的变量副本，这样每个线程都拥有自己的变量，消除了竞争条件。让每个线程绑定自己的值，可以将ThreadLocal类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。

### JUC集合&工具类

java.util.concurrent

## 常用的锁-在什么场景下使用

### 常见锁

1. 内置锁（synchronized）：
    1. 是内置锁机制的基础，可以用于方法或代码块。当一个线程进入代码块或方法时，它会获取关联对象的锁；当线程离开该代码块或方法时，锁会被释放。如果其他线程尝试获取同一个对象的锁，它们将被阻塞，直到锁被释放。
    2. 其中，syncronized加锁时有无锁、偏向锁、轻量级锁和重量级锁几个级别。偏向锁用于当一个线程进入同步块时，如果没有任何其他线程竞争，就会使用偏向锁，以减少锁的开销。轻量级锁使用线程栈上的数据结构，避免了操作系统级别的锁。重量级锁则涉及操作系统级的互斥锁。
2. ReentrantLock
    是一个显式的锁类，提供了比synchronized更高级的功能，如可中断的锁等待、定时锁等待、公平锁选项等。ReentrantLock使用lock()和unlock()方法来获取和释放锁。其中，公平锁按照线程请求锁的顺序来分配锁，保证了锁分配的公平性，但可能增加锁的等待时间。非公平锁不保证锁分配的顺序，可以减少锁的竞争，提高性能，但可能造成某些线程的饥饿。
3. 读写锁（ReadWriteLock）：
    允许多个读取者同时访问共享资源，但只允许一个写入者。读写锁通常用于读取远多于写入的情况，以提高并发性。
    典型问题：**写锁饥饿-**大量读操作持续持有读锁，导致写锁长期无法获取（读锁可重入且共享，写锁需等待所有读锁释放）。创建读写锁时指定 <mark style="background-color: #BBBFC4">`fair = true`</mark>，确保线程按请求顺序获取锁（写锁不会被读锁无限插队）。
4. 自旋锁
    自旋锁是一种锁机制，线程在等待锁时会持续循环检查锁是否可用，而不是放弃CPU并阻塞。通常可以使用CAS来实现。这在锁等待时间很短的情况下可以提高性能，但过度自旋会浪费CPU资源。

### 分类

![](/assets/VkvXbOs3AoTQUMx08wxcvWjLnOh.png)

## synchronized和reentrantlock及其应用场景

### synchronized工作原理

使用synchronized之后，会在编译之后在同步的代码块前后加上monitorenter和monitorexit字节码指令，他依赖操作系统底层互斥锁实现。他的作用主要就是实现原子性操作和解决共享变量的内存可见性问题。

执行monitorenter指令时会尝试获取对象锁，如果对象没有被锁定或者已经获得了锁，锁的计数器+1。此时其他竞争锁的线程则会进入等待队列中。执行monitorexit指令时则会把计数器-1，当计数器值为0时，则锁释放，处于等待队列中的线程再继续竞争锁。

Java会为每一个对象和对象的Class对象分配一个ObjectMonitor对象，他是一个C++结构体，ObjectMonitor用来维护当前**持有锁的线程**，**阻塞**等待锁释放的**线程链表**，调用了**wait**等待notify的**线程链表**。

每个对象中都内置了一个 ObjectMonitor对象。执行monitorenter指令时，线程试图获取锁也就是获取对象监视器monitor的持有权。monitor由c++实现。

![](/assets/Zo2cbUq9boSkFIxJbvtclphcnLf.png)

![](/assets/VyICbPtgboFD96xmYpwc1VEonLh.png)

synchronized是排它锁，当一个线程获得锁之后，其他线程必须等待该线程释放锁后才能获得锁，而且由于Java中的线程和操作系统原生线程是一一对应的，线程被阻塞或者唤醒时时会从用户态切换到内核态，这种转换非常消耗性能。

适用于简单同步需求、对代码块而非整个方法同步、内置锁的使用(尤其是在对象状态与锁保护的代码紧密相关时)

如果再深入到源码来说，synchronized实际上有两个队列waitSet(存等待被唤醒的线程，被唤醒后会进入entryList)和entryList(存竞争锁的线程)。

1. 当多个线程进入同步代码块时，首先进入entryList
2. 有一个线程获取到monitor锁后，就赋值给当前线程，并且计数器+1
3. 如果线程调用wait方法，将释放锁，当前线程置为null，计数器-1，同时进入waitSet等待被唤醒，调用notify或者notifyAll之后又会进入entryList竞争锁
4. 如果线程执行完毕，同样释放锁，计数器-1，当前线程置为null

![](/assets/TninbnUFgog861xFD8oc3w4fn0g.png)

### reentrantlock工作原理

ReentrantLock 的底层实现主要依赖于 AbstractQueuedSynchronizer（AQS）这个抽象类。AQS 是一个提供了基本同步机制的框架，其中包括了队列、状态值等。

reentrantlock实现了 Lock 接口，是一个可重入且独占式的锁，有轮询、超时、中断、多个条件变量、公平锁和非公平锁等高级功能。

* 公平锁：线程获取锁的顺序与请求顺序一致，AQS 同步队列的 天然FIFO 特性，公平锁在尝试获取锁（`tryAcquire` 方法）时，会额外判断 “当前线程是否是队列的头节点”，只有头节点线程才能获取锁，避免新线程插队。
* 中断：`ReentrantLock.lockInterruptibly()` 方法，其底层调用 AQS 的 `acquireInterruptibly`
  线程在同步队列中等待时，若被中断，AQS 会触发以下流程：
  * 检查线程中断标志（`Thread.interrupted()`）；
  * 若已中断，抛出 `InterruptedException` 并退出等待；
  * 未中断则继续阻塞，直到被唤醒或获取锁
* 定时：线程竞争锁失败后，进入同步队列，通过 `LockSupport.parkNanos(this, nanosTimeout)` 进行**限时阻塞**（而非无限期 `park()`）。若阻塞时间超过剩余超时时间，线程会自动唤醒并检查是否获取到锁。
默认使用非公平锁(锁被释放之后，后申请的线程可能会先获取到锁，是随机或者按照其他优先级排序的。性能更好，但可能会导致某些线程永远无法获取到锁)。

synchronized在jvm层面实现，reentrantlock在jdk层面实现。

Reentrantlock是可中断锁，获取锁的过程中可以被中断，不需要一直等到获取锁之后 才能进行其他逻辑处理。synchronized是不可中断锁。

适用于高级锁功能需求、性能优化、复杂同步结构

`ReentrantLock` 本身不直接提供 `wait()` 和 `notify()` 方法，而是通过 `Condition` 接口 实现类似功能，且比 `Object` 的 `wait/notify` 更灵活（可创建多个条件对象，实现更精细的通知控制）。

### synchronized和reentrantlock区别

* 用法不同：synchronized 可用来修饰普通方法、静态方法和代码块，而 ReentrantLock 只能用在代码块上。
* 获取锁和释放锁方式不同：synchronized 会自动加锁和释放锁，当进入 synchronized 修饰的代码块之后会自动加锁，当离开 synchronized 的代码段之后会自动释放锁。而 ReentrantLock 需要手动加锁和释放锁
* 锁类型不同：synchronized 属于非公平锁，而 ReentrantLock 既可以是公平锁也可以是非公平锁。
  * <span style="color: #E0E1E4">synchronized 属于非公平锁体现在</span><span style="color: #E0E1E4">**JVM**</span><span style="color: #E0E1E4"> 的内部实现上。当多个线程竞争锁时，</span><span style="color: #FBBFBC">`synchronized`</span><span style="color: #E0E1E4"> 并不会按照线程请求锁的顺序来安排锁的获取，而是让竞争最激烈的线程尽快获取锁-</span><span style="color: #E0E1E4">**当前处于 “活跃状态”（正在运行或刚被调度）的线程更有可能优先获取锁**</span>
* 底层实现不同：synchronized 是 JVM 层面通过监视器实现的，而 ReentrantLock 是基于 AQS 实现的。

## 可重入锁及原理

可重入锁是指同一个线程在获取了锁之后，可以再次重复获取该锁而不会造成死锁或其他问题。当一个线程持有锁时，如果再次尝试获取该锁，就会成功获取而不会被阻塞。

### Synchronized的可重入机制

synchronized底层是利用计算机系统mutex Lock实现的。每一个可重入锁都会关联一个线程ID和一个锁状态status。当一个线程请求方法时，会去检查锁状态。

1. 如果锁状态是0，代表该锁没有被占用，使用CAS操作获取锁，将线程ID替换成自己的线程ID。
2. 如果锁状态不是0，代表有线程在访问该方法。此时，如果线程ID是自己的线程ID，如果是可重入锁，会将status自增1，然后获取到该锁，进而执行相应的方法；如果是非重入锁，就会进入阻塞队列等待。
3. 释放锁时，如果是可重入锁的，每一次退出方法，就会将status减1，直至status的值为0，最后释放该锁。
    1. 如果非可重入锁的，线程退出方法，直接就会释放该锁。

### Reentrantlock

ReentrantLock实现可重入锁的机制是基于线程持有锁的计数器。

* 当一个线程第一次获取锁时，计数器会加1，表示该线程持有了锁。在此之后，如果同一个线程再次获取锁，计数器会再次加1。每次线程成功获取锁时，都会将计数器加1。
* 当线程释放锁时，计数器会相应地减1。只有当计数器减到0时，锁才会完全释放，其他线程才有机会获取锁。

## Synchronized锁升级

### 无锁->偏向锁->轻量级锁->重量级锁

https://blog.csdn.net/wangshuai6707/article/details/133306554?sharetype=blog&shareId=133306554&sharerefer=APP&sharesource=mzs0mzs&sharefrom=link

1. 无锁：对于共享资源，不涉及多线程的竞争访问。这是没有开启偏向锁的时候的状态，在JDK1.6之后偏向锁的默认开启的，可以通过JVM参数进行设置偏向延迟(需要在JVM启动之后的多少秒之后才能开启)，同时是否开启偏向锁也可以通过JVM参数设置。
2. 偏向锁：偏向锁的核心思想是，在无竞争的情况下，把整个同步消除掉。也就是说，如果一个锁只被一个线程锁定，而没有其他线程来竞争这个锁，那么这个锁就会偏向于这个线程，从而消除这个锁的同步操作。获得过锁的线程更容易再获得锁。共享资源首次被访问时，JVM会对该共享资源对象做一些设置，比如将对象头中是否偏向锁标志位置为1，对象头中的线程ID设置为当前线程ID（注意：这里是操作系统的线程ID），后续当前线程再次访问这个共享资源时，会根据偏向锁标识跟线程ID进行比对是否相同，比对成功则直接获取到锁，进入临界区域（就是被锁保护，线程间只能串行访问的代码），这也是synchronized锁的可重入功能。jdk15版本后默认关闭了偏向锁。**时机：升级时机：首次被线程获取且无竞争**
3. 轻量级锁：多个线程竞争时，JVM会先尝试使用轻量级锁，以CAS方式来获取锁（一般就是自旋加锁，不阻塞线程采用循环等待的方式），成功则获取到锁，状态为轻量级锁，失败（达到一定的自旋次数还未成功）则锁升级到重量级锁。**时机：当其他线程尝试获取该锁时，偏向锁无法继续维持，必须撤销并升级。**
4. 重量级锁 ：锁已经被某个线程持有，此时是偏向锁状态，未释放锁前，再有其他线程来竞争时，则会升级到重量级锁，另外轻量级锁状态多线程竞争锁时，也会升级到重量级锁，重量级锁由操作系统来实现，所以性能消耗相对较高。之前介绍的synchronized原理均为重量级锁。**时机：轻量级锁竞争时，未获取到锁的线程会先自旋（默认最多 10 次，可通过**<mark style="background-color: #BBBFC4">`**-XX:PreBlockSpin**`</mark>**调整）。、当第三个线程（或更多）尝试获取已处于轻量级锁状态的对象时，JVM 会直接判定 “竞争激烈”，无需等待自旋结束，立即触发升级。**

### 具体过程：

线程A进入 synchronized 开始抢锁，JVM 会判断当前是否是偏向锁的状态，如果是就会根据 Mark Word 中存储的线程 ID 来判断，当前线程A是否就是持有偏向锁的线程。如果是，则忽略 check，线程A直接执行临界区内的代码。

但如果 Mark Word 里的线程不是线程 A，就会通过自旋尝试获取锁，如果获取到了，就将 Mark Word 中的线程 ID 改为自己的;如果竞争失败，就会立马撤销偏向锁，膨胀为轻量级锁。

后续的竞争线程都会通过自旋来尝试获取锁，如果自旋成功那么锁的状态仍然是轻量级锁。然而如果竞争失败，锁会膨胀为重量级锁，后续等待的竞争的线程都会被阻塞。

## Synchronized锁的优化

* 锁膨胀：synchronized 从无锁升级到偏向锁，再到轻量级锁，最后到重量级锁的过程，它叫做锁膨胀也叫做锁升级。JDK 1.6 之前，synchronized 是重量级锁，也就是说 synchronized 在释放和获取锁时都会从用户态转换成内核态，而转换的效率是比较低的。但有了锁膨胀机制之后，synchronized 的状态就多了无锁、偏向锁以及轻量级锁了，这时候在进行并发操作时，大部分的场景都不需要用户态到内核态的转换了，这样就大幅的提升了 synchronized 的性能。
* 锁消除：指的是在某些情况下，JVM 虚拟机如果检测不到某段代码被共享和竞争的可能性，就会将这段代码所属的同步锁消除掉，从而到底提高程序性能的目的。
* 锁粗化：将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。
* 自适应自旋锁：指通过自身循环，尝试获取锁的一种方式，优点在于它避免一些线程的挂起和恢复操作，因为挂起线程和恢复线程都需要从用户态转入内核态，这个过程是比较慢的，所以通过自旋的方式可以一定程度上避免线程挂起和恢复所造成的性能开销。

## AQS

AQS全称为AbstractQueuedSynchronizer，是Java中的一个抽象类。 AQS是一个用于构建锁、同步器、协作工具类的工具类（框架）。Java中的大部分同步类（Lock、Semaphore、ReentrantLock、CountDownLatch等）都是基于AbstractQueuedSynchronizer（简称为AQS）实现的。

AQS核心思想：

* 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
* 如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁(下图) 实现的，即将暂时获取不到锁的线程加入到队列中。

CLH：Craig、Landin and Hagersten队列，是单向链表，AQS中的队列是CLH变体的虚拟双向队列（FIFO），AQS是通过将每条请求共享资源的线程封装成一个节点来实现锁的分配。在 CLH 队列锁中，一个节点表示一个线程，它保存着线程的引用（thread）、 当前节点在队列中的状态（waitStatus）、前驱节点（prev）、后继节点（next）。

![](/assets/XbTNbqlVsoLypixb4T1c20Vmnng.png)

AQS使用一个Volatile的int类型的成员变量State来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作，通过CAS完成对State值的修改。阻塞和唤醒是操作系统级别的。

![](/assets/NDsXbLtjPoteufxShP5cccStnVc.png)

### 具体原理

与具体的实现类有关

#### 状态state

* state的具体含义，会根据具体实现类的不同而不同：比如在Semapore里，他表示剩余许可证的数量；在CountDownLatch里，它表示还需要倒数的数量；在ReentrantLock中，state用来表示“锁”的占有情况，包括可重入计数，当state的值为0的时候，标识该Lock不被任何线程所占有。
* state是volatile修饰的，并被并发修改，所以修改state的方法都需要保证线程安全。

#### FIFO队列

* 这个队列用来存放“等待的线程，AQS就是“排队管理器”，当多个线程争用同一把锁时，必须有排队机制将那些没能拿到锁的线程串在一起。当锁释放时，锁管理器就会挑选一个合适的线程来占有这个刚刚释放的锁。
* AQS会维护一个等待的线程队列，把线程都放到这个队列里，这个队列是双向链表形式。

#### 实现获取/释放等方法

* 这里的获取和释放方法，是利用AQS的协作工具类里最重要的方法，是由协作类自己去实现的，并且含义各不相同；
* 获取方法：获取操作会以来state变量，经常会阻塞（比如获取不到锁的时候）。在Semaphore中，获取就是acquire方法，作用是获取一个许可证； 而在CountDownLatch里面，获取就是await方法，作用是等待，直到倒数结束；
* 释放方法：在Semaphore中，释放就是release方法，作用是释放一个许可证； 在CountDownLatch里面，获取就是countDown方法，作用是将倒数的数减一；
* 需要每个实现类重写tryAcquire和tryRelease等方法。

#### 使用


```Java
public class LeeLock  {
    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire (int arg) {
            return compareAndSetState(0, 1);
        }
        @Override
        protected boolean tryRelease (int arg) {
            setState(0);
            return true;
        }
        @Override
        protected boolean isHeldExclusively () {
            return getState() == 1;
        }
    }

    private Sync sync = new Sync();

    public void lock () {
        sync.acquire(1);
    }

    public void unlock () {
        sync.release(1);
    }
}
public class LeeMain {
    static int count = 0;
    static LeeLock leeLock = new LeeLock();
    public static void main (String[] args) throws InterruptedException {
        Runnable runnable = new Runnable() {
            @Override
            public void run () {
                try {
                    leeLock.lock();
                    for (int i = 0; i < 10000; i++) {
                        count++;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    leeLock.unlock();
                }
            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(count);
    }
}
```

## ThreadLocal

是Java中用于解决线程安全问题的一种机制，它允许创建线程局部变量，即每个线程都有自己独立的变量副本，从而避免了线程间的资源共享和同步问题。

线程内的全局变量/线程内部跨方法传递参数，而不用修改方法签名

#### 使用


```Java
import java.text.SimpleDateFormat;
import java.util.Random;
public class ThreadLocalExample implements Runnable{
     // SimpleDateFormat 不是线程安全的，所以每个线程都要有自己独立的副本
    private static final ThreadLocal<SimpleDateFormat> formatter = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyyMMdd HHmm"));
    public static void main(String[] args) throws InterruptedException {
        ThreadLocalExample obj = new ThreadLocalExample();
        for(int i=0 ; i<10; i++){
            Thread t = new Thread(obj, ""+i);
            Thread.sleep(new Random().nextInt(1000));
            t.start();
        }
    }
    @Override
    public void run() {
        System.out.println("Thread Name= "+Thread.currentThread().getName()+" default Formatter = "+formatter.get().toPattern());
        try {
            Thread.sleep(new Random().nextInt(1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //formatter pattern is changed here by thread, but it won't reflect to other threads
        formatter.set(new SimpleDateFormat());
        System.out.println("Thread Name= "+Thread.currentThread().getName()+" formatter = "+formatter.get().toPattern());
    }
}
```

#### 原理

每个线程内部有ThreadLocalMap类型的对象，类似于定制化的hashmap。默认情况下对象为null。

作用：一个线程内的全局变量

线程栈中可能有对thread对象的引用，thread对象又引用threadlocalMap对象，对象都在堆里。

ThreadLocal变量是放在了当前线程的 ThreadLocalMap 中，每个Entry代表一个完整的对象，key是ThreadLocal本身，value是ThreadLocal的泛型对象值。

`位置：ThreadLocal` 的数据存储在 当前线程（`Thread` 对象）的内部成员变量 中

![](/assets/MS77bHvd5o9vXexTzylcWOSWn3b.png)

ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，而 value 是强引用。所以，如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。可能会出现内存泄漏。实际应用中需要在使用完`ThreadLocal`变量后调用`remove()`方法释放资源。

线程池会复用线程对象，与线程对象绑定的类的静态属性 ThreadLocal 变量也会被重用，这就导致一个线程可能获取到其他线程的ThreadLocal 值。

![](/assets/B5llbEypcoXiebx8mnncSEtenuh.png)

![](/assets/WyzWbYXTFomYDVxXhF1cc4P5nnf.png)

![](/assets/TpTIbVSjComhUdxdxn4cAerpntb.png)

#### 线程已经有虚拟机方法栈了 为什么要再设置threadLocal

线程的虚拟机栈中确实存储局部变量（方法内定义的变量），且局部变量天然线程私有（其他线程无法访问），但局部变量的作用域仅限于当前方法（或方法调用链），无法满足 **“跨方法、跨类共享线程私有数据”** 的需求。

![](/assets/AmkpbgwagoJeJYx6ym5cPfhsnVH.png)

## 乐观锁和悲观锁

1. 悲观锁


```Java
public void performSynchronisedTask() {
    synchronized (this) {
        // 需要同步的操作
    }
}
private Lock lock = new ReentrantLock();
lock.lock();
try {
   // 需要同步的操作
} finally {
    lock.unlock();
}
```

1. 乐观锁

* 版本号：一般是在数据表中加上一个数据版本号 version 字段，表示数据被修改的次数。当数据被修改时，version 值会加1。当线程 A 要更新数据值时，在读取数据的同时也会读取 version 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的 version 值相等时才更新，否则重试更新操作，直到更新成功。
* 时间戳：使用时间戳记录数据的更新时间，在更新数据时，在比较时间戳。如果当前时间戳大于数据的时间戳，则说明数据已经被其他线程更新，更新失败。
* CAS操作：Compare And Swap（比较与交换） ，用于实现乐观锁，被广泛应用于各大框架中。CAS 的思想很简单，就是用一个预期值和要更新的变量值进行比较，两值相等才会进行更新。如Atomic 原子类使用CAS机制。问题：
  ![](/assets/AVpibv4ClokpwtxAfg5cUvymnSg.png)

  * 自旋CAS的方式如果长时间不成功，会给CPU带来很大的开销。
    * 解决：限制自旋次数、使用指数退避策略增加重试间隔，或在竞争非常激烈时考虑使用传统的锁机制
  * ABA问题：在CAS更新的过程中，当读取到的值是A，然后准备赋值的时候仍然是A，但是实际上有可能A的值被改成了B，然后又被改回了A，这个CAS更新的漏洞就叫做ABA。只是ABA的问题大部分场景下都不影响并发的最终效果。
    * 解决：引入一个版本号（stamp） 或布尔标记（mark）来记录变量的修改历史。每次更新时不仅比较值，还比较版本号，只有两者都符合预期时才进行更新
  * 只能保证一个共享变量的原子操作：只对一个共享变量操作可以保证原子性，但是多个则不行，多个可以通过AtomicReference来处理或者使用锁synchronized实现。
    * 解决：将多个变量封装到一个不可变对象中，然后使用 `AtomicReference` 来对这个对象引用进行 CAS 操作

## voliatle关键字与指令重排序

### voliatle作用

1. 保证变量对所有线程的可见性。当一个变量被声明为volatile时，它会保证对这个变量的写操作会立即刷新到主存中，而对这个变量的读操作会直接从主存中读取，从而确保了多线程环境下对该变量访问的可见性。这意味着一个线程修改了volatile变量的值，其他线程能够立刻看到这个修改，不会受到各自线程工作内存的影响。
2. 禁止指令重排序优化。volatile关键字在Java中主要通过内存屏障来禁止特定类型的指令重排序。
    1. 写-写（Write-Write）屏障：在对volatile变量执行写操作之前，会插入一个写屏障。这确保了在该变量写操作之前的所有普通写操作都已完成，防止了这些写操作被移到volatile写操作之后。
    2. 读-写（Read-Write）屏障：在对volatile变量执行读操作之后，会插入一个读屏障。它确保了对volatile变量的读操作之后的所有普通读操作都不会被提前到volatile读之前执行，保证了读取到的数据是最新的。
    3. 写-读（Write-Read）屏障：这是最重要的一个屏障，它发生在volatile写之后和volatile读之前。这个屏障确保了volatile写操作之前的所有内存操作（包括写操作）都不会被重排序到volatile读之后，同时也确保了volatile读操作之后的所有内存操作（包括读操作）都不会被重排序到volatile写之前。
    4. **普通读、写操作**通常指的是对**非volatile变量**的读写操作。
        例如，有如下代码逻辑：
        int a = 1; // 普通写操作
        volatile int b = 2; // volatile写操作
        如果没有volatile，b=2可能被重排序到a=1之前，如果其它线程先依赖b，再依赖a，且b被写入内存，a没有被写入，就会出现不一致。
        正常情况下应该先给<mark style="background-color: #BBBFC4">`a`</mark>赋值为 1，再给<mark style="background-color: #BBBFC4">`b`</mark>赋值为 2。但如果发生了普通写操作被移到 volatile 写操作之后，就可能先执行对<mark style="background-color: #BBBFC4">`b`</mark>的赋值，再执行对<mark style="background-color: #BBBFC4">`a`</mark>的赋值。
        如果在其他线程中依赖于普通变量先被写入后的结果来进行后续操作，而普通写操作被移到了 volatile 写操作之后，那么其他线程可能会先看到 volatile 变量的更新，而此时普通变量还没有被正确写入，就会导致数据不一致。

* volatile关键字可以保证可见性，但不能保证原子性，因此不能完全保证线程安全。volatile关键字用于修饰变量，当一个线程修改了volatile修饰的变量的值，其他线程能够立即看到最新的值，从而避免了线程之间的数据不一致。
* 但是，volatile并不能解决多线程并发下的复合操作问题，比如i++这种操作不是原子操作，如果多个线程同时对i进行自增操作，volatile不能保证线程安全。对于复合操作，需要使用synchronized关键字或者Lock来保证原子性和线程安全。

### 指令重排序作用

在执行程序时，为了提高性能，处理器和编译器常常会对指令进行重排序，但是重排序要满足下面 2 个条件才能进行：

1. 在单线程环境下不能改变程序运行的结果
2. 存在数据依赖关系的不允许重排序。

## 死锁

### 死锁条件

1. 互斥条件：互斥条件是指多个线程不能同时使用同一个资源
2. 请求并保持条件：请求并保持条件是指，当线程 A 已经持有了资源 1，又想申请资源 2，而资源 2 已经被线程 C 持有了，所以线程 A 就会处于等待状态，但是线程 A 在等待资源 2 的同时并不会释放自己已经持有的资源 1。
3. 不可剥夺条件：不可剥夺条件是指，当线程已经持有了资源 ，在自己使用完之前不能被其他线程获取，线程 B 如果也想使用此资源，则只能在线程 A 使用完并释放后才能获取。
4. 环路等待条件：环路等待条件指的是，在死锁发生的时候，两个线程获取资源的顺序构成了环形链。

### 死锁避免

避免死锁问题就只需要破环其中一个条件就可以，最常见的并且可行的就是使用资源有序分配法，来破环环路等待条件。

那什么是资源有序分配法呢？线程 A 和 线程 B 获取资源的顺序要一样，当线程 A 是先尝试获取资源 A，然后尝试获取资源 B 的时候，线程 B 同样也是先尝试获取资源 A，然后尝试获取资源 B。也就是说，线程 A 和 线程 B 总是以相同的顺序申请自己想要的资源。

# 线程池

## 工作原理

线程池是为了减少频繁的创建线程和销毁线程带来的性能损耗：

![](/assets/Z4bCbx1GSoMENPxO9Z5cKjbbnkF.png)

线程池分为核心线程池，线程池的最大容量，还有等待任务的队列，提交一个任务，如果核心线程没有满，就创建一个线程，如果满了，就是会加入等待队列，如果等待队列满了，就会增加线程，如果达到最大线程数量，如果都达到最大线程数量，就会按照一些丢弃的策略进行处理。<span style="color: #FBBFBC">**当工作线程数量超过核心线程数且没有新任务时，线程池中的非核心线程会被回收，线程数量会减少。**</span>这是线程池管理资源、避免闲置消耗的关键机制。如果你通过调用 `allowCoreThreadTimeOut(true)`方法设置了允许核心线程超时，那么核心线程在空闲时间超过`keepAliveTime`后也会被回收​，直到线程池中的线程数为0。

## 默认线程池种类

* 通过 Executor 框架的工具类 Executors 来创建：
  * FixedThreadPool：固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
  * SingleThreadExecutor： 只有一个线程的线程池。若多于一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
  * CachedThreadPool： 可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。
  * ScheduledThreadPool：给定的延迟后运行任务或者定期执行任务的线程池。
* 不推荐用此方法：
  * FixedThreadPool 和 SingleThreadExecutor:使用的是有界阻塞队列是 LinkedBlockingQueue ，其任务队列的最大长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致 OOM。
  * CachedThreadPool:使用的是同步队列 SynchronousQueue, 允许创建的线程数量为 Integer.MAX_VALUE ，如果任务数量过多且执行速度较慢，可能会创建大量的线程，从而导致 OOM。
  * ScheduledThreadPool 和 SingleThreadScheduledExecutor :使用的无界的延迟阻塞队列 DelayedWorkQueue ，任务队列最大长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致 OOM。

## 线程池参数

1. corePoolSize：线程池核心线程大小。当核心线程数为0的时候，会创建一个非核心线程进行执行。
2. maximumPoolSize：线程池最大线程数量。
3. keepAliveTime空闲线程存活时间：一个线程如果处于空闲状态，并且当前的线程数量大于corePoolSize则销毁。
4. unit 空闲线程存活时间单位。
5. workQueue 工作队列：ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue。
    * ArrayBlockingQueue：基于数组实现的有界阻塞队列，一旦创建容量固定不变。遵循 FIFO（先进先出）原则，支持公平和非公平两种访问策略（默认非公平）。
      * **选择：**
        * 基于数组的有界队列，容量固定（初始化时指定大小）。
        * **特点：结构简单、效率高，支持公平 / 非公平锁（默认非公平），适合任务量可预测的场景、高并发情况下，需要严格限制任务数量以避免资源耗尽，例如流量控制场景。**
        * 任务处理速度与生产速度接近，适合轻量级任务处理系统。
        * 缺点：容量固定，满了之后会触发拒绝策略（或阻塞提交线程，若使用<mark style="background-color: #BBBFC4">`put()`</mark>方法）。
      * **实现**：
        * 内部通过一个定长数组存储元素，配合两个索引（takeIndex 和 putIndex）分别标记出队和入队位置
        * 使用**单把重入锁**（ReentrantLock）控制并发，锁的条件变量（notEmpty 和 notFull）分别处理非空和非满的等待 / 唤醒逻辑
        * 公平性通过锁的公平性实现，公平模式下线程按阻塞顺序获取访问权，非公平模式可能存在插队现象
        * 元素入队和出队操作均需获取锁，当队列满时 put 操作阻塞，队列空时 take 操作阻塞
    * LinkedBlockingQueue：基于单向链表实现的可选有界阻塞队列，默认容量为 Integer.MAX_VALUE（可视为无界）。同样遵循 FIFO 原则。
      * 选择：
        * 基于链表的可选有界队列（默认容量为<mark style="background-color: #BBBFC4">`Integer.MAX_VALUE`</mark>，几乎可视为无界）。
        * 特点：插入 / 删除效率高，适合任务量波动较大的场景；若指定容量则可避免内存溢出。
        * 缺点：默认无界可能导致任务堆积过多，耗尽内存（OOM）。
        * **适用：任务处理速度较快，任务积压风险低。系统资源充足，对响应时间要求不高，例如日志收集服务。**
      * **实现**：
        * 内部由节点链表构成，每个节点包含元素和下一个节点的引用
        * 使用**两把分离的重入锁**（takeLock 和 putLock）分别控制出队和入队操作，提高并发性能
        * 各自的锁拥有独立的条件变量（notEmpty 和 notFull），避免了 ArrayBlockingQueue 中单一锁导致的操作互斥
        * 维护一个原子性的计数器（count）记录元素数量，用于判断队列空 / 满状态
        * 链表结构使其在插入和删除元素时具有更好的性能（相对数组结构），但需要额外的节点对象创建开销
    * SynchronousQueue：同步队列不存储任务，任务必须直接交给线程处理。如果没有空闲线程，任务提交会被阻塞。
      * 选择
        * **希望每个任务立即被执行，不希望排队、需要线程池灵活扩展到最大线程数，例如短时间高负载任务、无缓冲的同步队列，不存储任务，提交的任务必须立即被线程执行（若没有空闲线程则创建新线程，直到达到最大线程数）。**
        * 特点：零内存占用，适合任务执行速度快、任务数量少的场景（如 RPC 通信）。
        * 配合<mark style="background-color: #BBBFC4">`CachedThreadPool`</mark>（核心线程 0，最大线程数无限）使用时，可快速响应短期任务。
        1. 队列长度始终为 0，任务直接交由线程处理。
        1. 线程池必须动态创建新线程来处理任务，否则提交线程会等待。
        1. 适合高强度并发任务，要求线程池有足够的资源。
      * 实现：
        * 没有实际的元素存储结构，仅维护等待的生产者和消费者线程队列
        * 支持公平和非公平模式，公平模式使用链表维护等待线程，非公平模式使用栈结构
        * 内部通过 Transferer 接口实现核心交换逻辑，根据模式不同有 QueueTransferer（公平）和 StackTransferer（非公平）两种实现
        * 适用于线程间直接传递数据的场景，如 ExecutorService 中的 CachedThreadPool 就使用了该队列
    * PriorityBlockingQueue：基于优先级堆实现的无界阻塞队列，元素按照自然顺序或自定义比较器排序，不遵循 FIFO 原则。
      * 选择
        * 带优先级的无界阻塞队列，任务按优先级排序（需实现<mark style="background-color: #BBBFC4">`Comparable`</mark>接口）。
        * 特点：保证高优先级任务先执行，适合任务有轻重缓急的场景（如实时任务优先于普通任务）。
        * 缺点：无界可能导致 OOM，且排序会带来额外性能开销。
      * 内部使用数组实现的二叉堆结构存储元素，默认是小顶堆（可通过比较器改变排序方式）
      * 使用**单把重入锁**控制并发，配合 notEmpty 条件变量处理出队等待
      * 由于是无界队列，put 操作不会阻塞（除非内存不足），take 操作在队列为空时阻塞
      * 当元素数量超过当前容量时，会自动扩容（类似于 ArrayList 的扩容机制）
      * 不保证相同优先级元素的顺序，如需保证可使用 PriorityBlockingQueue 并让元素实现 Comparable 接口时考虑加入时间戳等辅助排序字段
6. threadFactory 线程工厂：可以用来设定线程名、是否为daemon线程等等。
    1. 线程池中的所有工作线程都会通过该工厂创建，通过实现 <mark style="background-color: #BBBFC4">`ThreadFactory`</mark> 接口，可以在创建线程时统一设置以下属性：
        1. **线程名称**：按业务场景命名（如 <mark style="background-color: #BBBFC4">`order-process-1`</mark>、<mark style="background-color: #BBBFC4">`payment-handler-2`</mark>），方便日志追踪和问题排查。
        2. **守护线程（Daemon）**：设置线程为守护线程（<mark style="background-color: #BBBFC4">`setDaemon(true)`</mark>），使其在主线程退出时自动销毁（适用于后台辅助线程）。
        3. **线程优先级**：通过 <mark style="background-color: #BBBFC4">`setPriority(int)`</mark> 调整线程优先级（1-10，默认 5），确保关键任务的线程优先执行。
        4. **未捕获异常处理器**：通过 <mark style="background-color: #BBBFC4">`setUncaughtExceptionHandler()`</mark> 统一处理线程中未捕获的异常（避免异常导致线程静默退出）。
7. handler 拒绝策略：
    1. CallerRunsPolicy，使用线程池的调用者所在的线程去执行被拒绝的任务，除非线程池被停止或者线程池的任务队列已有空缺。 核心业务：保证任务不丢失（推荐<mark style="background-color: #BBBFC4">`CallerRunsPolicy`</mark>或自定义策略）
    2. AbortPolicy，直接丢弃任务并抛出RejectedExecutionException异常。核心业务中未处理该异常可能导致流程中断，非核心业务中会丢失任务且无感知。
    3. DiscardPolicy，不做任何处理，静默拒绝提交的任务。非核心业务：允许任务丢失（可选<mark style="background-color: #BBBFC4">`DiscardPolicy`</mark>或<mark style="background-color: #BBBFC4">`DiscardOldestPolicy`</mark>）
    4. DiscardOldestPolicy，抛弃进入队列最早的那个任务，然后尝试把这次拒绝的任务放入队列

## 参数设置经验 

Bytedance：最佳的线程池核数计算公式=cpu核数 * (IO耗时 + cpu耗时)/cpu耗时

CPU密集型：corePoolSize = CPU核数 + 1（避免过多线程竞争CPU）

* 任务的主要操作是**计算、逻辑处理**，几乎不涉及 IO 操作（如数学运算、数据排序、复杂算法），执行过程中 CPU 一直处于忙碌状态（利用率接近 100%）。
* 若线程数**等于 CPU 核数**：理论上 CPU 可满负荷运行，但实际中可能因线程偶尔的阻塞（如缓存失效、分支预测失败）导致 CPU 短暂空闲。
* 增加**1 个额外线程**：当某个线程因短暂阻塞（非 IO 阻塞，如等待 CPU 缓存）时，额外的线程可利用 CPU 的空闲时间，避免资源浪费，提高 CPU 利用率。
* 为何不设置更多？若线程数远大于 CPU 核数，会导致**频繁的上下文切换**（CPU 在多个线程间切换执行），切换成本（保存 / 恢复线程状态）会抵消多线程的优势，反而降低效率。
IO密集型：corePoolSize = CPU核数 x 2（或更高，具体看IO等待时间）

* 任务的主要操作是**等待 IO 响应**（如网络请求、文件读写、数据库操作），执行过程中 CPU 大部分时间处于空闲状态（等待 IO 完成）。
* 例如：假设 CPU 核数为 4，1 个线程执行 IO 任务时，90% 的时间在等待 IO（CPU 空闲），仅 10% 的时间使用 CPU。此时若有 8 个线程（4×2），CPU 可在 8 个线程的 IO 等待间隙交替处理它们的计算需求，充分利用空闲的 CPU 资源。
* 为何是 “×2”？这是经验值，实际可根据 IO 等待时间调整：
  * IO 等待时间越长（如远程网络请求），可设置更多线程（如 <mark style="background-color: #BBBFC4">`CPU核数 × 5`</mark>），因为 CPU 空闲时间更久。
  * IO 等待时间较短（如本地文件读写），可适当减少（如 <mark style="background-color: #BBBFC4">`CPU核数 × 1.5`</mark>），避免线程过多导致切换开销。
### 场景一：电商场景，特点瞬时高并发、任务处理时间短，线程池的配置可设置如下：

![](/assets/OvFIbmSuLoZ2qrx52bBcGGZknAf.png)

* SynchronousQueue确保任务直达线程，避免队列延迟。
* 拒绝策略快速失败，前端返回“活动火爆”提示，结合降级策略（如缓存预热）。

### 场景二：后台数据处理服务，特点稳定流量、任务处理时间长（秒级）、允许一定延迟，线程池的配置可设置如下：

![](/assets/Ouwmbx4t7o6a3QxIllRcYFXRnne.png)

* 固定线程数避免资源波动，队列缓冲任务，拒绝策略兜底。
* 配合监控告警（如队列使用率>80%触发扩容）。

### 场景三：微服务HTTP请求处理，特点IO密集型、依赖下游服务响应时间，线程池的配置可设置如下：

![](/assets/RlkxbKU1woEz0txK5NbcFAvqnJc.png)

* 根据下游RT（响应时间）调整线程数，队列防止瞬时峰值。
* 自定义拒绝策略将任务暂存Redis，异步重试。

### 动态设置线程池参数

![](/assets/K2nWbbYkMomTcUx9bvWckduJnGf.png)

## shutdown ()和shutdownNow()

* shutdown使用了以后会置状态为SHUTDOWN，正在执行的任务会继续执行下去，没有被执行的则中断。此时，则不能再往线程池中添加任何任务，否则将会抛出 RejectedExecutionException 异常
* 而 shutdownNow 为STOP，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务，当然，它会返回那些未执行的任务。 它试图终止线程的方法是通过调用 Thread.interrupt() 方法来实现的，但是这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。

## 任务如何提交

队列中的任务实现runnable或callable，线程池新建线程直接task.start即可。

线程start方法中 while不断循环，从队列中拿一runnable或callable直接run即可。


```Java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class SimpleThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final List<Worker> workers;
    private volatile boolean isShutdown;

    public SimpleThreadPool(int coreSize, int queueCapacity) {
        taskQueue = new ArrayBlockingQueue<>(queueCapacity);
        workers = new ArrayList<>(coreSize);
        
        // 初始化工作线程
        for (int i = 0; i < coreSize; i++) {
            Worker worker = new Worker();
            workers.add(worker);
            worker.start();
        }
    }

    public void submit(Runnable task) throws InterruptedException {
        if (!isShutdown) taskQueue.put(task);
    }

    public void shutdown() {
        isShutdown = true;
        workers.forEach(Thread::interrupt);
    }

    // 工作线程
    private class Worker extends Thread {
        @Override
        public void run() {
            while (!isShutdown && !isInterrupted()) {
                try {
                    taskQueue.take().run(); // 取任务并执行
                } catch (InterruptedException e) {
                    if (isShutdown) break;
                }
            }
        }
    }

    // 使用示例
    public static void main(String[] args) throws InterruptedException {
        SimpleThreadPool pool = new SimpleThreadPool(3, 5);
        
        // 提交任务
        for (int i = 0; i < 10; i++) {
            final int id = i;
            pool.submit(() -> {
                try { Thread.sleep(1000); } 
                catch (InterruptedException e) { return; }
                System.out.println("任务" + id + "完成");
            });
        }
        
        Thread.sleep(12000);
        pool.shutdown();
    }
}

```

## 取消提交的任务

当向线程池提交任务时，会得到一个Future对象。这个对象提供了几种方法来管理任务的执行，包括取消任务。主要是cancel(boolean mayInterruptIfRunning)方法方法。这个方法尝试取消执行的任务。

![](/assets/JLznbCxauoxjz2x8K1Vc2BUhnGh.png)

Future执行某一耗时的任务时，可以将这个耗时任务交给一个子线程去异步执行，同时执行其它任务，不用等待耗时任务执行完成。再通过 Future 类获取到耗时任务的执行结果。

参数mayInterruptIfRunning指示是否允许中断正在执行的任务。如果设置为true，则表示如果任务已经开始执行，那么允许中断任务；如果设置为false，任务已经开始执行则不会被中断。

## 线程异常

* 线程异常，异常会被当前线程的 catch 块捕获并处理，不会影响到其他线程的执行。如果没有通过 try-catch 块来捕获异常，异常会传播到线程的 run() 方法外部。此时，线程会因为未被捕获的异常而异常终止，后续的代码不会再执行。
* 线程池中的线程异常，如果是execute方法(Runnable),可以看异常输出在控制台，而submit(Callable)在控制台没有直接输出，必须调用Future.get()方法时，可以捕获到异常。一个线程出现异常不会影响线程池里面其他线程的正常执行。
* 若线程在执行过程中抛出**未被捕获的 checked 异常**（编译时不允许，需显式处理）或**未被捕获的 unchecked 异常**（如 <mark style="background-color: #BBBFC4">`NullPointerException`</mark>、<mark style="background-color: #BBBFC4">`OutOfMemoryError`</mark>），线程会立即终止。可通过 <mark style="background-color: #BBBFC4">`Thread.setUncaughtExceptionHandler()`</mark> 捕获未处理异常，做日志记录等操作，但无法阻止线程终止。
* 线程不是被回收而是线程池把这个线程移除掉，同时创建一个新的线程放到线程池中。即当线程异常之后，按照正常情况来说线程就直接消失了，但是通过`processWorkerExit`方法的补救，增加了一个新的线程，保证线程池的运行。
  * 回收是标记线程为可终止状态，移除是实际删除线程对象。

## 线程池的任务顺序问题

线程池（如 `ThreadPoolExecutor`）的核心工作流程如下：

1. 当核心线程都在忙碌时，新来的任务会被放入一个阻塞队列（BlockingQueue）​ 中等待。
2. 当队列也满了时，线程池才会创建新的线程（直到达到最大线程数）来执行任务。

场景 “旧任务在队列里，新任务被新线程执行” 正是发生在第2步。一个刚被提交的任务B，因为队列已满，由一个新创建的非核心线程直接执行了。而之前提交的任务A，还在队列里苦苦等待核心线程空闲下来。

这样，后提交的任务B就先于先提交的任务A执行了，导致了执行顺序的错乱。

![](/assets/KqwZblmQpo4Xl7xT9MuczBAsn7e.png)

## 阿里线程池规范

![](/assets/VqQfbr9FEoZ9dMx6temcYrnNngg.png)

# 虚拟线程

虚拟线程（Virtual Thread）是 JDK 而不是 OS 实现的轻量级线程(Lightweight Process，LWP），由 JVM 调度。许多虚拟线程共享同一个操作系统线程，虚拟线程的数量可以远大于操作系统线程的数量。

虚拟线程是`Java19`提出来的一个概念，`Java19`提供特性预览，开放实装是`Java21`（2023年9月）

![](/assets/RxJIbwbCromnMKx9F9ncXrDdn9f.png)

虚拟线程（Virtual Threads）是 Java 21 正式引入的轻量级线程，属于用户态线程（User-Level Thread），由 JVM 管理而非直接映射到操作系统内核线程。它的设计目标是**大幅提升高并发场景下的吞吐量**，尤其适合 I/O 密集型任务（如 Web 服务、微服务、消息处理等）。

## 核心特点

1. **极致轻量**
    * 虚拟线程的创建成本极低：初始栈空间仅几十 KB（传统平台线程约 1MB），支持单 JVM 中创建数百万甚至上千万个线程，突破了传统线程的资源限制。
    * 生命周期管理（创建 / 销毁）效率是平台线程的千倍以上，几乎消除了线程本身的性能开销。
2. **M:N 调度模型**虚拟线程通过 “载体线程（Carrier Thread）” 与操作系统内核线程关联，采用 **M:N 映射**（多虚拟线程映射到少数字内核线程）：
    * 当虚拟线程执行阻塞操作（如网络 I/O、锁等待、<mark style="background-color: #BBBFC4">`Thread.sleep()`</mark>）时，JVM 会自动将其从载体线程上 “卸载”，释放载体线程去执行其他虚拟线程。
    * 阻塞操作完成后，虚拟线程再被 “挂载” 到某个载体线程上继续执行。这种 “非阻塞式调度” 避免了传统线程阻塞时的资源浪费，大幅提升了线程利用率。
3. **完全兼容现有 API**虚拟线程是 <mark style="background-color: #BBBFC4">`java.lang.Thread`</mark> 的子类，完全兼容现有的线程 API（<mark style="background-color: #BBBFC4">`Runnable`</mark>、<mark style="background-color: #BBBFC4">`Callable`</mark>、<mark style="background-color: #BBBFC4">`ExecutorService`</mark> 等）。现有代码几乎无需修改，只需通过特定方式创建虚拟线程即可迁移

## 创建

* 使用 Thread.startVirtualThread() 创建


```Java
public class VirtualThreadTest {
  public static void main(String[] args) {
    CustomThread customThread = new CustomThread();
    Thread.startVirtualThread(customThread);
  }
}
static class CustomThread implements Runnable {
  @Override
  public void run() {
    System.out.println("CustomThread run");
  }
}
```

* 使用Thread.ofVirtual()创建


```Java
public class VirtualThreadTest {
  public static void main(String[] args) {
    CustomThread customThread = new CustomThread();
    // 创建不启动
    Thread unStarted = Thread.ofVirtual().unstarted(customThread);
    unStarted.start();
    // 创建直接启动
    Thread.ofVirtual().start(customThread);
  }
}
static class CustomThread implements Runnable {
  @Override
  public void run() {
    System.out.println("CustomThread run");
  }
}
```

* 使用 ThreadFactory() 创建


```Java
public class VirtualThreadTest {
  public static void main(String[] args) {
    CustomThread customThread = new CustomThread();
    ThreadFactory factory = Thread.ofVirtual().factory();
    Thread thread = factory.newThread(customThread);
    thread.start();
  }
}
static class CustomThread implements Runnable {
  @Override
  public void run() {
    System.out.println("CustomThread run");
  }
}
```

* 使用 Thread.startVirtualThread() 创建


```Java
public class VirtualThreadTest {
  public static void main(String[] args) {
    CustomThread customThread = new CustomThread();
    Thread.startVirtualThread(customThread);
  }
}
static class CustomThread implements Runnable {
  @Override
  public void run() {
    System.out.println("CustomThread run");
  }
}
```

## 基本原理

virtual thread = continuation + scheduler

虚拟线程会把任务（一般是java.lang.Runnable）包装到一个Continuation实例中：

* 当任务需要阻塞挂起的时候，会调用Continuation的yield操作进行阻塞
* 当任务需要解除阻塞继续执行的时候，Continuation会被继续执行

Scheduler也就是执行器，会把任务提交到一个载体线程池中执行：

* 执行器是java.util.concurrent.Executor的子类
* 虚拟线程框架提供了一个默认的ForkJoinPool用于执行虚拟线程任务

平台线程即常见线程

载体线程 Java平台线程与系统线程一一映射，所以平台线程被操作系统调度，但是虚拟线程是由JVM调度。

* mount操作：虚拟线程挂载到平台线程，虚拟线程中包装的Continuation栈数据帧或者引用栈数据会被拷贝到平台线程的线程栈，这是一个从堆复制到栈的过程
* unmount操作：虚拟线程从平台线程卸载，大多数虚拟线程中包装的Continuation栈数据帧会留在堆内存中

# 经典场景

## 多线程打印奇偶数-怎么控制打印的顺序

![](/assets/VRi9bmSVnoCIudx502zc4Ruinqh.png)

## 双重校验锁实现对象单例


```Java
public class Singleton {
    private volatile static Singleton uniqueInstance;
    private Singleton() {
    }
    public  static Singleton getUniqueInstance() {
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
```

* 第一次校验，也就是第一个判断if(singleton == null)，意义是由于单例模式只需创建一个实例，所以当第一次创建实例成功之后，再次调用Singleton.getInstance()就没有必要进入同步锁代码块，直接返回之前创建的实列即可。
* 第二次校验，也就是第二次判断if(singleton == null),是为了防止二次创建实列，我们假设一种状况，当singleton还未被创建的时候，线程r1 调用了getInstance 方法，由于此时的singleton 为空，则可以进入第一层判断，线程r1正准备继续执行，此时，线程r2抢占cpu资源，此时r2也调用了getInstance 方法，同理线程r1并没有实例化singleton，线程r2也可以进去判断，然后继续往下执行，进入到同步代码块，进入第二层判断，完成了singleton 的创建，并分配空间，r2线程运行周期结束。执行任务又回到了r1,如果没有第二层判断，线程r1 也会创建一个实列(r2线程已经创建一个实列，第二层判断为false)，这样就完全避免掉多线程环境下会创建多个实列的的问题。
* 为什么要volatile：由于 Java 内存模型允许编译器和处理器对指令进行重排序，在没有volatile的情况下，可能会出现重排序，例如先将对象引用赋值给instance，但对象的实例化操作尚未完成。这样，其他线程在检查 `instance == null` 时，会认为单例已经创建，从而得到一个未完全初始化的对象，导致错误。

## 如何在多个子线程中捕获异常并引发主线程异常？

* 用try-catch块捕获异常。在多线程中，每个子线程都可以使用try-catch块来捕获异常并进行处理。这样可以确保每个子线程的异常不会传播到其他线程，但需要注意的是，这并不会将异常传播到主线程。因此主线程无法感知到子线程的异常情况。
* 使用Thread.UncaughtExceptionHandler接口来处理未捕获的异常。这种方式可以让我们在主线程中捕获子线程的异常，并做出相应的处理。

![](/assets/ZtldbMzRcogwlox8g9JcpHV0nFh.png)

* 基于 Future + Callable（适用于需要子线程返回结果的场景）.
  `Callable` 允许子线程返回结果，主线程通过 `Future` 的 `get()` 方法获取结果时，若子线程抛出异常，`get()` 会将异常包装为 `ExecutionException` 抛出，主线程捕获该异常即可。
  1. 子线程实现 `Callable` 接口（重写 `call()` 方法，可抛异常）；
  1. 通过 `ExecutorService` 提交 `Callable`，获取 `Future` 对象；
  1. 主线程调用 `future.get()`，捕获 `ExecutionException`，通过 `getCause()` 拿到子线程原始异常。
