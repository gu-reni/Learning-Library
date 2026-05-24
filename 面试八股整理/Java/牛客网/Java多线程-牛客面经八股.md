# Java多线程-牛客面经八股

> 来源：牛客网  |  共 27 题

## 2. 说说线程的创建方式。
### 

### 一句话总结
 1. 继承Thread类并重写run方法。
 2. 实现Runnable接口并实现run方法。
 3. 使用Callable接口配合FutureTask实现带返回值的线程。
 4. 通过线程池（如Executor框架）创建管理线程。
 5. 使用Lambda表达式或匿名内部类简化线程实现。 
 
### 详细解析
 
 在 Java 中，创建线程主要有以下几种方式，每种方式适用于不同的场景： 
 
---
 
#### **1. 继承Thread类**
 
 通过继承Thread类并重写run()方法，直接创建线程对象。
 **特点**： 
 简单易用，但单继承限制灵活性。 每个线程是独立实例，资源消耗较高。 
 **示例**： 
 
```java
class MyThread extends Thread {
 @Override
 public void run() {
 System.out.println("Thread running by extending Thread");
 }
}

// 启动线程
MyThread thread = new MyThread();
thread.start(); // 调用 start() 启动新线程
```
 
---
 
#### **2. 实现Runnable接口**
 
 将任务逻辑封装在Runnable实现类中，通过Thread类包装执行。
 **特点**： 
 避免单继承限制，支持多线程共享同一任务。 推荐基础方式，适合无返回值的场景。 
 **示例**： 
 
```java
class MyRunnable implements Runnable {
 @Override
 public void run() {
 System.out.println("Thread running by implementing Runnable");
 }
}

// 启动线程
Thread thread = new Thread(new MyRunnable());
thread.start();
```
 
 **简化写法（Lambda表达式）**： 
 
```java
new Thread(() -> System.out.println("Lambda Runnable")).start();
```
 
---
 
#### **3. 实现Callable接口 +FutureTask**
 
 通过Callable定义带返回值的任务，结合FutureTask或线程池执行。
 **特点**： 
 支持返回结果和抛出异常。 需配合ExecutorService或FutureTask使用。 
 **示例**： 
 
```java
import java.util.concurrent.*;

class MyCallable implements Callable<String> {
 @Override
 public String call() throws Exception {
 return "Callable result";
 }
}

// 使用 FutureTask 包装 Callable
FutureTask<String> futureTask = new FutureTask<>(new MyCallable());
Thread thread = new Thread(futureTask);
thread.start();
System.out.println(futureTask.get()); // 获取结果

// 或通过线程池提交
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> future = executor.submit(new MyCallable());
System.out.println(future.get());
executor.shutdown();
```
 
---
 
#### **4. 使用线程池（Executor框架）**
 
 通过ExecutorService管理线程池，提交任务执行。
 **特点**： 
 高效管理线程生命周期，避免频繁创建销毁开销。 支持任务队列、线程复用和资源控制。 
 **示例**： 
 
```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// 提交 Runnable 任务
executor.execute(() -> System.out.println("Task in thread pool"));

// 提交 Callable 任务
Future<String> future = executor.submit(() -> "Result from Callable");
System.out.println(future.get());

executor.shutdown(); // 关闭线程池
```
 
---
 
#### **对比与选择建议**
 
| 方式 | 优点 | 缺点 | 适用场景 |
| --- | --- | --- | --- |
| 继承Thread | 简单直接 | 单继承限制，扩展性差 | 简单任务或快速原型开发 |
| 实现Runnable | 灵活，支持共享任务 | 无返回值，无法抛出受检异常 | 大多数多线程场景 |
| 实现Callable | 支持返回值和异常 | 需配合线程池或FutureTask使用 | 需要结果或异常处理的异步任务 |
| 线程池（Executor） | 资源高效，支持复杂任务管理 | 需手动配置参数，避免资源耗尽 | 高并发、长期运行的服务 |
 
---
 
#### **注意事项启动线程必须调用start()**，直接调用run()仅是普通方法调用。 **线程池需显式关闭**：使用完ExecutorService后调用shutdown()或shutdownNow()。 **避免资源竞争**：多线程共享数据时需同步（如synchronized、Lock）。

## 3. 说说线程的生命周期和状态?
#### 

#### 一、线程生命周期的 6 种状态
 
 Java 线程的生命周期由Thread.State枚举定义，包含以下 6 种状态（）： 
 
| 状态名称 | 描述 | 触发条件 |
| --- | --- | --- |
| NEW（新建） | 线程对象被创建但未调用start()方法 | Thread thread = new MyThread(); |
| RUNNABLE（可运行） | 线程已启动，等待 CPU 调度执行run()方法 | 调用start()方法后 |
| BLOCKED（阻塞） | 线程因等待锁资源而暂停执行 | 进入同步代码块/方法时锁被其他线程持有 |
| WAITING（等待） | 线程主动放弃 CPU，进入无限等待状态 | 调用wait()、join()或LockSupport.park() |
| TIMED_WAITING（超时等待） | 线程在指定时间内等待（如睡眠或限时等待） | 调用sleep(long)、wait(long)或join(long) |
| TERMINATED（终止） | 线程执行完毕或异常退出 | run()方法正常结束或抛出未捕获异常 |
 
---
 
#### 二、状态转换图与关键路径
 
 图来自于网上：挑错 |《Java 并发编程的艺术》中关于线程状态的三处错误 
 **三.关键状态转换示例NEW → RUNNABLE** 
 
```java
Thread thread = new Thread(() -> {});
thread.start(); // 状态变为 RUNNABLE
```
 
 **RUNNABLE → BLOCKED** 
 
```java
synchronized (lock) { 
 // 线程A持有锁
}
// 线程B尝试进入同步块时被阻塞（BLOCKED）
```
 
 **RUNNABLE → WAITING** 
 
```java
synchronized (lock) {
 lock.wait(); // 释放锁，进入 WAITING
}
```
 
 **RUNNABLE → TIMED_WAITING** 
 
```java
Thread.sleep(1000); // 进入 TIMED_WAITING
```
 
 **WAITING → RUNNABLE** 
 
```java
synchronized (lock) {
 lock.notify(); // 唤醒等待线程
}
```
 
 **RUNNABLE → TERMINATED** 
 
```java
thread.run() 执行完毕或抛出异常。
```

## 4. 说说wait()和sleep()的区别
### 

### 一句话总结
 1. 所属类不同：wait()来自Object类，sleep()来自Thread类。
 2. 锁释放：wait()会释放对象锁，sleep()保持锁不释放。
 3. 使用条件：wait()必须在同步代码块中调用，sleep()无此限制。
 4. 唤醒机制：wait()需notify()唤醒，sleep()超时自动恢复或中断。
 5. 应用场景：wait()用于线程协作，sleep()用于暂停执行。
 
### 详细解析
 
 在 Java 中，wait()和sleep()都用于线程的暂停，但它们的机制和用途有显著区别。以下是主要区别： 
 
---
 
#### **1. 所属类与锁的释放wait()** 
 是Object类的方法，所有对象均可调用。 **释放对象锁**：调用wait()后，线程会释放当前对象的锁，允许其他线程获取该锁并执行同步代码。 只能在同步代码块（synchronized）中使用，否则抛出IllegalMonitorStateException。 
 **sleep()** 
 是Thread类的静态方法（Thread.sleep()）。 **不释放锁**：线程休眠期间，持有的锁不会被释放，其他线程无法进入同步代码块。 无需在同步代码块中调用。 
---
 
#### **2. 唤醒机制wait()** 
 依赖其他线程调用同一对象的notify()或notifyAll()来唤醒。 可设置超时参数（如wait(1000)），超时后自动唤醒。 
 **sleep()** 
 需等待指定时间结束（如sleep(1000)）或通过interrupt()中断唤醒。 不可被其他线程主动唤醒。 
---
 
#### **3. 使用场景wait()** 
 用于多线程协作（如生产者-消费者模型），当条件不满足时让线程等待，并在条件满足后被唤醒。 
 **sleep()** 
 用于暂停当前线程的执行（如模拟延迟、定时任务），与锁无关。 
---
 
#### **4. 异常处理共同点**：两者均可能抛出InterruptedException（需捕获或声明抛出）。 **区别**：wait()必须在同步上下文中调用，否则抛IllegalMonitorStateException。 
---
 
#### **示例代码**
 
```java
// wait() 示例
synchronized (lock) {
 while (!condition) {
 lock.wait(); // 释放锁，等待唤醒
 }
 // 条件满足后继续执行
}

// sleep() 示例
try {
 Thread.sleep(1000); // 休眠1秒，不释放锁
} catch (InterruptedException e) {
 e.printStackTrace();
}
```
 
---
 
#### **总结**
 
| 特性 | wait() | sleep() |
| --- | --- | --- |
| 所属类 | Object | Thread（静态方法） |
| 锁释放 | 是 | 否 |
| 唤醒方式 | notify()/notifyAll()或超时 | 超时结束或被中断 |
| 使用场景 | 线程间协作 | 单纯暂停执行 |
| 同步要求 | 必须在同步块中 | 无要求 |
| 异常处理 | 必须处理InterruptedException | 同左 |
 
 根据具体需求选择：需要协作释放锁时用wait()，单纯暂停用sleep()。

## 5. 说说你了解的线程同步方式。
### 

### 一句话总结
 互斥锁（Mutex）通过独占访问保证资源安全；信号量（Semaphore）控制线程并发数量；条件变量（Condition Variable）实现线程间状态通知；读写锁（Read-Write Lock）区分读写操作提升效率；原子操作（Atomic）通过硬件指令确保操作的不可分割性。
 
### 详细解析
 
 Java线程同步主要通过以下几种机制实现，确保多线程环境下共享资源的安全访问： 
 
---
 
#### **1.synchronized关键字作用**：提供互斥锁，确保同一时间只有一个线程访问同步代码块或方法。 **使用方式**： **同步实例方法**：锁对象为当前实例。 
```java
public synchronized void method() {
 // 同步代码
}
```
 **同步静态方法**：锁对象为类的Class对象。 
```java
public static synchronized void staticMethod() {
 // 同步代码
}
```
 **同步代码块**：显式指定锁对象（任意对象）。 
```java
public void method() {
 synchronized (lockObject) {
 // 同步代码
 }
}
```
 **特点**： 隐式加锁/解锁，代码简洁。 JDK 1.6 后优化了锁机制（偏向锁、轻量级锁、重量级锁），性能提升显著。 不可中断，无法设置超时。 
---
 
#### **2.Lock接口（如ReentrantLock）作用**：提供更灵活的显式锁操作。 **使用方式**： 
```java
Lock lock = new ReentrantLock();
public void method() {
 lock.lock();
 try {
 // 同步代码
 } finally {
 lock.unlock(); // 确保解锁
 }
}
```
 **特点**： 支持可中断锁（lockInterruptibly()）、超时锁（tryLock(timeout)）、公平锁。 需手动加锁/解锁，避免死锁。 适合高竞争或复杂同步场景。 
---
 
#### **3.volatile关键字作用**：保证变量的可见性，禁止指令重排序。 **使用方式**： 
```java
private volatile boolean flag = false;
```
 **适用场景**： 状态标记（如线程终止标志）。 单例模式的双重检查锁定（DCL）。 **限制**：不保证原子性，无法替代锁。 
---
 
#### **4.wait()与notify()/notifyAll()作用**：实现线程间协调（生产者-消费者模型）。 
 
 **使用方式**（需配合synchronized）： 
 
```java
// 消费者线程
synchronized (lock) {
 while (condition) {
 lock.wait(); // 释放锁并等待
 }
 // 处理逻辑
}

// 生产者线程
synchronized (lock) {
 // 修改条件
 lock.notify(); // 唤醒一个等待线程
 // 或 lock.notifyAll();
}
```
 
 **特点**： 
 wait()释放锁，notify()唤醒等待线程。 需在循环中检查条件（避免虚假唤醒）。 
---
 
#### **5. 原子类（如AtomicInteger）作用**：通过 CAS（Compare and Swap）实现无锁原子操作。 **使用方式**： 
```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // 原子递增
```
 **适用场景**： 计数器、状态标记等简单原子操作。 性能优于锁，但仅支持单一变量。 
---
 
#### **6. 线程安全容器（如ConcurrentHashMap）作用**：内置同步机制，避免手动同步。 **常见实现**： ConcurrentHashMap：分段锁或 CAS 实现高并发访问。 CopyOnWriteArrayList：写时复制，适合读多写少场景。 **特点**：简化代码，直接使用即可保证线程安全。 
---
 
#### **对比与选型建议**
 
| 机制 | 适用场景 | 优点 | 缺点 |
| --- | --- | --- | --- |
| synchronized | 简单同步需求，低竞争场景 | 代码简洁，JVM优化好 | 功能有限，不可中断 |
| Lock | 高竞争场景，需复杂控制（超时、公平） | 灵活，支持多种锁策略 | 需手动管理锁，代码复杂 |
| volatile | 状态标记，可见性需求 | 轻量级，无锁性能高 | 不保证原子性 |
| wait/notify | 线程间协作（生产者-消费者） | 内置线程协调机制 | 需配合锁使用，易出错 |
| 原子类 | 简单原子操作（如计数器） | 无锁高性能 | 仅支持单一变量 |
| 线程安全容器 | 替代手动同步的集合操作 | 简化代码，高效安全 | 功能可能受限 |

## 15. synchronized和Lock有什么区别？
### 

### 一句话总结
 1. 机制：synchronized是Java关键字，基于JVM实现自动锁管理；Lock是接口，需手动加锁解锁。
 2. 灵活性：Lock支持非阻塞尝试锁（tryLock）、可中断锁、超时锁，synchronized无法实现。
 3. 公平性：Lock可设置公平/非公平策略，synchronized仅支持非公平锁。
 4. 条件变量：Lock可通过Condition绑定多个条件，synchronized仅一个等待队列。
 5. 性能：高并发时Lock更高效，synchronized优化后差距缩小，但Lock需显式释放避免死锁。
 
### 详细解析
 
 synchronized和Lock（以ReentrantLock为例）都是 Java 中用于实现线程同步的机制，但它们在设计、功能和使用方式上有显著区别。以下是核心差异的对比： 
 
#### **1. 类型与实现层面synchronized**：Java 语言内置的关键字，基于 JVM 实现的 **内置锁（Monitor Lock）**。其底层通过对象的监视器（Monitor）实现，依赖 JVM 的字节码指令（monitorenter和monitorexit）来管理锁的获取和释放。 **Lock**：java.util.concurrent.locks.Lock接口的实现类（如ReentrantLock），属于 **API 层面的显式锁**。通过 Java 代码实现锁逻辑（如使用CAS操作和AQS队列同步器），更灵活但需要手动管理。 
#### **2. 锁的获取与释放方式synchronized**：隐式获取和释放锁。
 进入同步块（或方法）时自动获取锁，退出同步块（正常退出或异常退出）时自动释放锁（由 JVM 保证）。 **Lock**：显式获取和释放锁。
 需手动调用lock()或tryLock()获取锁，并在finally块中调用unlock()释放锁（否则可能导致死锁）。 
#### **3. 锁的特性差异**
 
| 特性 | synchronized | Lock（如 ReentrantLock） |
| --- | --- | --- |
| 可中断性 | 不可中断（除非抛出异常）。 | 支持可中断：通过lockInterruptibly()方法，等待锁时可响应中断。 |
| 超时获取锁 | 不支持（阻塞直到获取锁）。 | 支持超时：通过tryLock(long timeout, TimeUnit unit)，超时未获取则放弃。 |
| 公平性 | 默认非公平锁（线程获取锁的顺序与等待顺序无关）。 | 可通过构造函数ReentrantLock(boolean fair)指定公平锁（按等待队列顺序分配锁）。 |
| 条件变量（Condition） | 仅支持单一等待队列（通过wait()/notify()）。 | 支持多个Condition对象（如newCondition()），实现更细粒度的等待/通知（如生产者-消费者模型中区分“读”和“写”条件）。 |
 
#### **4. 性能与优化synchronized**：早期版本（JDK 1.6 前）性能较差（依赖操作系统互斥量，线程阻塞/唤醒开销大）。
 JDK 1.6 后引入 **锁升级优化**（偏向锁→轻量级锁→重量级锁），减少无竞争或低竞争场景下的性能损耗，与Lock性能接近。 **Lock**：通过CAS（无锁操作）和AQS（队列同步器）实现，在高竞争或复杂场景（如需要可中断、超时）中更灵活，但需手动管理锁。 
#### **5. 使用场景synchronized**：适合简单同步需求（如保护短代码块、避免手动释放锁），代码更简洁。
 例如：保护非静态成员变量的同步方法、短时间的同步代码块。 **Lock**：适合复杂同步需求（如需要可中断、超时、公平锁或多条件变量）。
 例如：需要精确控制锁释放时机、实现生产者-消费者模型中的多条件等待（Condition.await()/signal()）。 
#### **总结**
 
| 维度 | synchronized | Lock（如 ReentrantLock） |
| --- | --- | --- |
| 类型 | JVM 内置关键字 | JUC 包的 API 接口实现 |
| 锁管理 | 隐式（自动获取/释放） | 显式（手动lock()/unlock()） |
| 灵活性 | 低（固定语义） | 高（可中断、超时、公平锁、多条件） |
| 异常安全 | 自动释放（异常时安全） | 需finally块保证释放 |
| 适用场景 | 简单同步、短代码块 | 复杂同步、高竞争、需要细粒度控制 |

## 14. 说说synchronize的用法及原理
### 

### 一句话总结
 synchronized用于控制多线程对共享资源的访问，可修饰方法或代码块，通过指定锁对象实现互斥。其原理基于对象内部的监视器锁（Monitor），线程进入时获取锁，退出时释放锁，底层依赖操作系统的互斥锁及内存屏障机制保证原子性与可见性。JVM会优化锁状态（偏向锁、轻量级锁等）提升性能。
 
### 详细解析
 
 **synchronized** 是 Java 中用于实现线程同步的关键字，通过内置锁（Monitor）机制保证多线程环境下代码块或方法的互斥访问。以下是其用法及底层原理的详细说明： 
 
---
 
#### **一、synchronized 的用法1. 修饰实例方法锁对象**：当前实例对象（this）。 **作用范围**：整个方法体。 **示例**： 
```java
public synchronized void instanceMethod() {
 // 同步代码
}
```
 **2. 修饰静态方法锁对象**：类的Class对象（如MyClass.class）。 **作用范围**：整个静态方法体。 **示例**： 
```java
public static synchronized void staticMethod() {
 // 同步代码
}
```
 **3. 修饰代码块锁对象**：显式指定的任意对象（通常为共享资源）。 
 
 **作用范围**：代码块内部。 
 
 **示例**： 
 
```java
private final Object lock = new Object();

public void blockMethod() {
 synchronized (lock) {
 // 同步代码
 }
}
```
 
---
 
#### **二、synchronized 的原理1. 底层实现字节码指令**：通过monitorenter和monitorexit指令实现锁的获取与释放。 
 
 **对象头结构**：每个 Java 对象在内存中分为三部分： 
 **对象头（Header）**：存储锁状态、GC 分代年龄、哈希码等。 **实例数据（Instance Data）**：对象的字段值。 **对齐填充（Padding）**：补齐内存对齐。 
 **对象头中的锁标记**： 
 **Mark Word**（32/64 位）：存储锁标志位（如无锁、偏向锁、轻量级锁、重量级锁）。 **2. 锁升级过程** 
 JDK 1.6 后，synchronized 引入了 **锁膨胀** 机制，根据竞争情况动态升级锁状态，以平衡性能与开销： 
 
| 锁状态 | 适用场景 | 实现方式 |
| --- | --- | --- |
| 无锁 | 无竞争 | 直接访问资源 |
| 偏向锁 | 单线程重复访问 | 在 Mark Word 中记录线程 ID |
| 轻量级锁 | 多线程交替执行，无实际竞争 | CAS 自旋获取锁 |
| 重量级锁 | 多线程高竞争 | 通过操作系统互斥量（Mutex）实现 |
 
 **锁升级流程**： 
 
```plaintext
无锁 → 偏向锁 → 轻量级锁 → 重量级锁
```
 **3. 锁膨胀机制详解偏向锁**： 
 **目的**：减少无竞争时的锁开销。 **触发条件**：锁对象首次被线程访问。 **实现**：Mark Word 记录线程 ID，后续无需 CAS 操作。 **撤销**：当其他线程尝试获取锁时，偏向锁升级为轻量级锁。 
 **轻量级锁**： 
 **目的**：减少线程阻塞开销，通过自旋（CAS）竞争锁。 **触发条件**：多个线程交替执行，未发生竞争。 **实现**：将对象头替换为指向线程栈中锁记录的指针。 **失败处理**：自旋超过阈值（默认 10 次）后升级为重量级锁。 
 **重量级锁**： 
 **目的**：应对高并发竞争场景。 **实现**：通过操作系统互斥量（Mutex）阻塞线程，进入等待队列。 **开销**：涉及用户态与内核态切换，性能较低。 
---
 
#### **三、synchronized 的特性**
 
| 特性 | 说明 |
| --- | --- |
| 互斥性 | 同一时间仅允许一个线程持有锁。 |
| 可见性 | 锁释放前，变量的修改会刷新到主内存；锁获取时，本地缓存失效。 |
| 可重入性 | 同一线程可重复获取同一把锁（锁计数器递增）。 |
| 非公平锁 | 默认非公平，允许插队竞争，提高吞吐量。 |
 
---
 
#### **四、示例解析1. 单例模式（双重检查锁）** 
```java
public class Singleton {
 private volatile static Singleton instance;

 private Singleton() {}

 public static Singleton getInstance() {
 if (instance == null) {
 synchronized (Singleton.class) {
 if (instance == null) {
 instance = new Singleton();
 }
 }
 }
 return instance;
 }
}
```
 **volatile 作用**：防止指令重排序导致未初始化对象被引用。 **synchronized 作用**：保证单例创建的原子性。

## 25. 说说你对AQS的理解。
### 

### 一句话总结
 AQS（AbstractQueuedSynchronizer）是Java并发包的核心同步框架，通过内置的FIFO队列管理线程竞争，使用volatile变量state表示同步状态。它提供独占和共享两种模式，子类只需重写tryAcquire/tryRelease等模板方法即可实现锁、信号量等同步器（如ReentrantLock/Semaphore），解决了同步控制中线程排队与唤醒的底层细节问题。
 
### 详细解析
 
 **AQS（AbstractQueuedSynchronizer）** 是 Java 并发包（java.util.concurrent.locks）的核心基础框架，几乎所有的同步工具（如ReentrantLock、Semaphore、CountDownLatch）都基于它实现。其核心思想是通过 **共享资源状态管理** 和 **线程等待队列** 实现高效的线程同步机制。以下是其核心要点： 
 
---
 
#### **一、AQS 的核心设计1. 状态管理（State）volatile int state**：
 表示共享资源的状态（如锁的持有次数、信号量许可数）。通过 CAS 操作（compareAndSetState）保证原子性更新。 **状态语义由子类定义**：
 例如： ReentrantLock：state=0表示未锁定，state>0表示锁被持有，且记录重入次数。 Semaphore：state表示剩余可用许可数。 **2. 等待队列（CLH 变种队列）双向链表结构**：
 队列中的每个节点（Node）代表一个等待线程，保存线程引用、等待状态（WAITING、CANCELLED等）及前驱/后继指针。 **自旋 + 阻塞**：
 线程在入队后先自旋尝试获取资源，失败后通过LockSupport.park()挂起，避免无意义的 CPU 占用。 **3. 模板方法模式子类需实现的方法**： tryAcquire(int)：尝试独占式获取资源。 tryRelease(int)：尝试独占式释放资源。 tryAcquireShared(int)：尝试共享式获取资源。 tryReleaseShared(int)：尝试共享式释放资源。 **AQS 提供的核心方法**： acquire(int)：获取资源（可能阻塞）。 release(int)：释放资源并唤醒后续线程。 
---
 
#### **二、AQS 的工作流程（以独占锁为例）1. 线程获取锁（acquire）** 
```plaintext
调用 acquire(arg)
 ↓
调用 tryAcquire(arg)（子类实现）
 ↓
若成功 → 直接执行
若失败 → 创建 Node 并入队
 ↓
循环尝试获取资源（自旋）
 ↓
若仍失败 → 挂起线程（LockSupport.park()）
```
 **2. 线程释放锁（release）** 
```plaintext
调用 release(arg)
 ↓
调用 tryRelease(arg)（子类实现）
 ↓
若释放成功 → 唤醒队列中下一个可用线程（LockSupport.unpark()）
```
 
---
 
#### **三、AQS 的关键优势1. 灵活性与可扩展性** 
 通过继承 AQS 并重写模板方法，可快速实现自定义同步器（如读写锁、信号量）。 
 
 **示例：实现一个不可重入的独占锁**： 
 
```java
class SimpleLock extends AbstractQueuedSynchronizer {
 @Override
 protected boolean tryAcquire(int arg) {
 if (compareAndSetState(0, 1)) {
 setExclusiveOwnerThread(Thread.currentThread());
 return true;
 }
 return false;
 }

 @Override
 protected boolean tryRelease(int arg) {
 if (getState() == 0) throw new IllegalMonitorStateException();
 setExclusiveOwnerThread(null);
 setState(0);
 return true;
 }
}
```
 **2. 高效性能无锁化设计**：通过 CAS 操作减少锁竞争。 **队列唤醒优化**：仅唤醒必要的线程（如ReentrantLock的非公平模式允许插队）。 **3. 支持多种同步模式独占模式**：同一时刻仅一个线程可访问资源（如ReentrantLock）。 **共享模式**：多个线程可同时访问资源（如Semaphore、CountDownLatch）。 
---
 
#### **四、AQS 的典型应用**
 
| 同步工具 | 同步模式 | 状态（state）的语义 |
| --- | --- | --- |
| ReentrantLock | 独占 | 0=未锁定，≥1=锁重入次数 |
| ReentrantReadWriteLock | 独占（写） + 共享（读） | 高16位=读锁计数，低16位=写锁重入次数 |
| Semaphore | 共享 | 剩余可用许可数 |
| CountDownLatch | 共享 | 倒计时计数器（初始值由用户设定） |
| ThreadPoolExecutor.Worker | 独占 | 0=未锁定，1=锁定（表示工作线程是否在执行任务） |

## 21. 如何创建线程池？线程池常见参数有哪些？
### 

### 一句话总结
 创建线程池可通过Java的ThreadPoolExecutor构造函数或Executors工具类实现。核心参数包括：核心线程数（维持常驻线程）、最大线程数（扩容上限）、存活时间（非核心线程空闲时长）、任务队列（存放待执行任务）、拒绝策略（队列满时的处理方式）。
 
### 详细解析
 
#### 一、线程池的创建方式
 
 在 Java 中，线程池的创建主要有两种方式：通过Executors工厂类快速创建和通过ThreadPoolExecutor手动配置。推荐生产环境中使用后者以实现精细化控制。 
 
 1. 通过Executors工厂类创建（快速实现）
 • 固定大小线程池 
 
 适用于负载稳定的场景（如 API 请求处理）： 
 
```java
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5); // 核心线程数=最大线程数=5
```
 
 • 可缓存线程池 
 
 适用于大量短生命周期任务（如批量数据处理）： 
 
```java
 ExecutorService cachedThreadPool = Executors.newCachedThreadPool(); // 线程数动态调整
```
 
 • 单线程线程池 
 
 保证任务顺序执行（如事务处理）： 
 
```java
 ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
```
 
 • 定时/周期性线程池 
 
 用于延迟或周期性任务调度（如心跳检测）： 
 
```java
 ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(3);
 scheduledThreadPool.scheduleAtFixedRate(() -> {
 System.out.println("定时任务执行");
 }, 0, 2, TimeUnit.SECONDS);通过ThreadPoolExecutor手动配置（推荐生产环境）
```
 2. 通过ThreadPoolExecutor手动配置（推荐生产环境）
 直接配置核心参数，灵活控制线程池行为： 
```java
int corePoolSize = 5; // 核心线程数
int maxPoolSize = 10; // 最大线程数
long keepAliveTime = 60L; // 空闲线程存活时间
TimeUnit unit = TimeUnit.SECONDS;
BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(100); // 任务队列
```
 
---
 
#### 二、线程池核心参数详解
 
| 参数 | 作用 | 配置建议 | 典型场景 |
| --- | --- | --- | --- |
| corePoolSize | 核心线程数，线程池长期保持的最小线程数 | CPU 密集型任务设为 CPU 核心数；I/O 密集型任务设为2 * CPU 核心数 | 高并发 I/O 任务（如 Web 服务器） |
| maximumPoolSize | 最大线程数，线程池允许的最大并发线程数 | 根据系统资源（内存、CPU）和任务类型调整，避免资源耗尽 | 突发流量场景（如秒杀） |
| keepAliveTime | 非核心线程空闲存活时间 | 结合任务间隔设置（如 60 秒），及时回收空闲线程 | 低频任务（如定时任务） |
| unit | keepAliveTime的时间单位 | 常用TimeUnit.SECONDS或TimeUnit.MILLISECONDS | 与业务需求匹配 |
| workQueue | 任务队列，存放待执行的任务 | 优先使用有界队列（如LinkedBlockingQueue），避免内存溢出 | 任务量波动较大的场景 |
| threadFactory | 线程工厂，用于创建线程 | 自定义线程名称、优先级等，便于监控和调试 | 需追踪线程来源的生产环境 |
| handler | 拒绝策略，当队列满且线程数达上限时的处理方式 | 根据业务需求选择策略（如丢弃、抛出异常、由调用线程执行） | 高负载下的容错处理 |
 
---
 
#### 三、线程池参数配置最佳实践
 
 核心线程数（corePoolSize）
 • CPU 密集型任务：设置为CPU 核心数（如Runtime.getRuntime().availableProcessors()）。 
 
 • I/O 密集型任务：设置为2 * CPU 核心数（因线程等待 I/O 时释放 CPU）。 
 
 任务队列（workQueue）
 • 有界队列（如LinkedBlockingQueue）：防止内存溢出，需设置合理容量。 
 
 • 无界队列（如SynchronousQueue）：适用于任务直接交接的场景，但可能引发 OOM。 
 
 拒绝策略（handler） 
 
| 策略 | 行为 | 适用场景 |
| --- | --- | --- |
| AbortPolicy（默认） | 抛出RejectedExecutionException异常 | 需快速失败并记录日志的场景 |
| CallerRunsPolicy | 由提交任务的线程直接执行任务 | 任务可降级处理的场景 |
| DiscardPolicy | 直接丢弃任务，不抛出异常 | 可容忍任务丢失的场景 |
| DiscardOldestPolicy | 丢弃队列头部任务，尝试重新提交当前任务 | 需优先处理新任务的场景 |
 
 线程工厂（threadFactory）
 • 自定义线程名称，便于排查问题： 
 
```java
 ThreadFactory threadFactory = r -> {
 Thread thread = new Thread(r);
 thread.setName("custom-thread-" + UUID.randomUUID());
 return thread;
 };
```
 
---
 
#### 四、线程池使用示例
 
```java
public class ThreadPoolDemo {
 public static void main(String[] args) {
 // 自定义线程池参数
 int corePoolSize = 5;
 int maxPoolSize = 10;
 long keepAliveTime = 30L;
 TimeUnit unit = TimeUnit.SECONDS;
 BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(100);
 ThreadFactory threadFactory = Executors.defaultThreadFactory();
 RejectedExecutionHandler handler = new ThreadPoolExecutor.CallerRunsPolicy();

 ThreadPoolExecutor executor = new ThreadPoolExecutor(
 corePoolSize, maxPoolSize, keepAliveTime, unit, workQueue, 
 threadFactory, handler
 );

 // 提交任务
 for (int i = 0; i < 20; i++) {
 executor.execute(() -> {
 System.out.println(Thread.currentThread().getName() + " 执行任务");
 try {
 Thread.sleep(1000);
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 });
 }

 // 关闭线程池
 executor.shutdown();
 }
}
```
 
---
 
#### 五、线程池的关闭与监控
 
 优雅关闭
 •shutdown()：停止接收新任务，等待已提交任务执行完毕。 
 
 •shutdownNow()：立即终止所有任务，返回未执行的任务列表。 
 
 监控线程池状态
 • 通过ThreadPoolExecutor的方法获取实时状态： 
 
```java
 int activeCount = executor.getActiveCount(); // 活跃线程数
 long completedTaskCount = executor.getCompletedTaskCount(); // 已完成任务数
 int poolSize = executor.getPoolSize(); // 当前线程数
```

## 8. 说说volatile的用法及原理。
**一、 volatile 的主要作用（用法）** 
 volatile是一个轻量级的同步机制，它主要解决了两种问题：**可见性** 和**有序性**。但它**不能保证原子性**。 
 
 **保证可见性问题**：在多线程环境下，每个线程为了执行效率，可能会将主内存中的变量拷贝到自己的本地内存（如CPU缓存）中进行操作。如果一个线程修改了变量的值，但未能及时写回主内存，其他线程就看不到这个最新的值，从而导致数据不一致。 **volatile的作用**：当一个线程修改了一个volatile变量，这个新值会**立即被强制刷新到主内存**。同时，当其他线程读取这个volatile变量时，它会**强制使本地缓存失效，直接从主内存中重新读取**。 **简单比喻**：就像公告板，一个人（线程A）把通知写在公告板（主内存）上，其他人（线程B、C）都会来看这个公告板，而不是看自己手里可能过时的小抄。 
 **禁止指令重排序问题**：为了优化性能，编译器和处理器可能会对指令的执行顺序进行重排序。在单线程下，这不会影响最终结果，但在多线程下，可能会导致意想不到的错误。 
 
 **volatile的作用**：通过插入**内存屏障**，禁止编译器和对volatile变量读写操作前后的指令进行重排序，从而保证程序执行的有序性。 
 
 **经典场景**：**双重检查锁定单例模式**。 
 
```java
public class Singleton {
 private static volatile Singleton instance; // 使用volatile

 public static Singleton getInstance() {
 if (instance == null) { // 第一次检查
 synchronized (Singleton.class) {
 if (instance == null) { // 第二次检查
 instance = new Singleton(); // ！！！没有volatile这里可能会出问题
 }
 }
 }
 return instance;
 }
}
```
 instance = new Singleton();这行代码在JVM中分为三步： 分配内存空间 初始化对象 将instance引用指向这块内存 如果没有volatile，步骤2和步骤3**可能被重排序**。如果线程A执行完1和3后（此时instance不为null，但对象还未初始化），线程B进入第一次检查，发现instance不为null，就会直接返回一个**未初始化完成**的对象，导致程序出错。 volatile通过禁止重排序，确保了对象的初始化一定在引用赋值之前完成。 **二、 volatile 的实现原理** 
 volatile的底层实现是通过**内存屏障** 来实现的。 
 
 内存屏障（Memory Barrier）是一组CPU指令，用于控制特定条件下的重排序和内存可见性问题。 
 
 在JVM层面，对于volatile变量的访问，会插入以下屏障： 
 **写操作时**： 在每个volatile写操作**之前**插入一个StoreStore屏障，确保前面的所有普通写操作已经对其它处理器可见。 在每个volatile写操作**之后**插入一个StoreLoad屏障，强制将写缓冲区的数据刷新到主内存，并让其它处理器的缓存失效。 **读操作时**： 在每个volatile读操作**之后**插入一个LoadLoad屏障和一个LoadStore屏障，确保后面的所有读/写操作都必须等待本次volatile读从主内存加载完成。 
 这些屏障共同作用，保证了volatile写操作对所有线程的**可见性**，并防止了指令的**重排序**。 
 **三、 重要注意事项：volatile 不保证原子性** 
 这是一个非常关键的考点。volatile**不能保证复合操作的原子性**。 
 **例子**：count++这个操作，看似一步，实际上分为三步： 读取count的值 将值加1 写回新的值 **问题**：如果线程A在执行完第1步后，CPU时间片被线程B抢走，线程B顺利地将count加1并写回。随后线程A恢复执行，它仍然基于自己读取的旧值进行加1并写回，这就会覆盖线程B的操作，导致最终结果小于预期。 
 **解决方案**： 
 对于需要保证原子性的操作（如i++），需要使用synchronized关键字或java.util.concurrent.atomic包下的原子类（如AtomicInteger）。 **总结** 
| 特性 | volatile | synchronized |
| --- | --- | --- |
| 原子性 | ❌ 不保证 | ✅ 保证 |
| 可见性 | ✅ 保证 | ✅ 保证 |
| 有序性 | ✅ 保证（有限） | ✅ 保证（完全） |

## 19. ThreadLocal 的用法和实现原理。
### 

### 一句话总结
 ThreadLocal用于存储线程私有数据，通过set()/get()在当前线程内共享变量。每个Thread内部维护ThreadLocalMap，以ThreadLocal实例为键存储独立副本。数据隔离通过线程专属的Map实现，避免多线程竞争。需注意使用后及时remove()防止内存泄漏，弱引用键设计可辅助垃圾回收。
 
### 详细解析
 
#### 一、核心概念
 
 ThreadLocal是 Java 提供的线程本地存储工具，为每个线程维护独立的变量副本，实现线程间的数据隔离。其核心价值在于 避免共享数据竞争，同时 简化跨方法参数传递。 
 
#### **二、核心用法基本操作** 
 
```java
public class ThreadLocalDemo {
 // 创建 ThreadLocal 实例（静态变量避免重复创建）
 private static final ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

 public static void main(String[] args) {
 // 设置值
 threadLocal.set(100);
 // 获取值
 System.out.println(threadLocal.get()); // 输出 100
 // 移除值（防止内存泄漏）
 threadLocal.remove();
 }
}
```
 
 **典型场景**
 • 数据库连接管理：每个线程独立持有连接，避免多线程竞争。 
 
 • 用户会话信息：存储当前线程的登录用户 ID 或请求 ID。 
 
 • 线程上下文传递：在多层方法调用中传递参数（如日志追踪的 TraceID）。 
 
 • 线程安全对象池：如SimpleDateFormat的线程隔离实例。 
 
---
 
#### **三、实现原理内部数据结构**
 • ThreadLocalMap：每个线程（Thread对象）内部维护的哈希表，键为ThreadLocal实例（弱引用），值为线程本地变量副本。 
 
```java
// Thread 类中的成员变量
transient ThreadLocal.ThreadLocalMap threadLocals = null;
```
 
 • Entry 结构：WeakReference<ThreadLocal<?>>作为键，避免ThreadLocal对象被回收后内存泄漏。 
 
 **核心方法流程** 
 2.1 set(T value) 获取当前线程的ThreadLocalMap。 若不存在则创建。 以当前ThreadLocal实例为键，存储值。
 2.2 get()：
 从当前线程的ThreadLocalMap中获取值。
 若不存在，调用initialValue()初始化并存储。
 2.3 remove()： 清除当前线程的ThreadLocalMap中对应键值对。 
 四、与同步机制的对比 
| 维度 | ThreadLocal | synchronized/Lock |
| --- | --- | --- |
| 数据隔离 | 每个线程独立副本 | 共享变量，需加锁保护 |
| 适用场景 | 线程独立数据（如连接、用户上下文） | 共享资源并发访问 |
| 性能 | 无锁，高性能 | 锁竞争导致性能下降 |
| 内存泄漏风险 | 存在（需手动清理） | 无 |
 
 五、实际案例 
 
 **数据库连接隔离** 
 
```java
public class DBConnectionHolder {
 private static final ThreadLocal<Connection> connectionHolder = new ThreadLocal<>();

 public static Connection getConnection() throws SQLException {
 Connection conn = connectionHolder.get();
 if (conn == null) {
 conn = DriverManager.getConnection(url, user, password);
 connectionHolder.set(conn);
 }
 return conn;
 }

 public static void close() throws SQLException {
 Connection conn = connectionHolder.get();
 if (conn != null) {
 conn.close();
 connectionHolder.remove(); // 防止内存泄漏
 }
 }
}
```
 
 **Web 请求上下文传递** 
 
```java
public class RequestContext {
 private static final ThreadLocal<String> traceId = new ThreadLocal<>();

 public static void setTraceId(String id) {
 traceId.set(id);
 }

 public static String getTraceId() {
 return traceId.get();
 }

 public static void clear() {
 traceId.remove();
 }
}
```

## 12. 讲一下Java里的CAS。
### 

### 一句话总结
 CAS（Compare-And-Swap）是Java中基于硬件的无锁并发机制，通过比较内存值与预期值进行原子更新。核心方法包含内存地址、预期值和新值，若匹配则更新，否则重试或放弃。常用于AtomicInteger等原子类实现线程安全操作，避免加锁开销。存在ABA问题（可通过版本号标记解决），适用于低竞争场景。
 
### 详细解析
 
 在 Java 中，**CAS（Compare-And-Swap，比较并交换）** 是一种无锁的原子操作技术，用于在多线程环境下实现变量的线程安全修改。它通过 CPU 底层指令保证操作的原子性，避免了传统锁（如synchronized）带来的线程阻塞和上下文切换开销，是 Java 并发编程（JUC 包）的核心技术之一。 
 
#### **一、CAS 的核心思想**
 
 CAS 的操作逻辑可以概括为三个步骤：
 **比较（Compare）** → **交换（Swap）**
 具体来说，CAS 包含三个关键参数： 
 **V（Value）**：变量在内存中的实际值（目标内存地址的值）。 **A（Expected）**：线程认为变量当前应有的“预期值”。 **B（NewValue）**：线程希望将变量更新为的“新值”。 
 **操作逻辑**：
 当且仅当内存中的实际值 V **等于** 预期值 A 时，才将 V 更新为 B；否则不执行任何操作（保持 V 不变）。整个过程由 CPU 指令保证原子性，不可被中断。 
 
#### **二、Java 中的 CAS 实现**
 
 Java 中 CAS 的底层实现依赖于sun.misc.Unsafe类（JDK 内部使用，不建议直接调用）。Unsafe提供了一系列compareAndSwapXxx方法（如compareAndSwapInt、compareAndSwapObject），这些方法直接调用 CPU 的原子指令（如 x86 的cmpxchg指令），确保操作的原子性。 
 示例：AtomicInteger的原子递增 
 Java 的原子类（如AtomicInteger）是 CAS 的典型应用。以incrementAndGet()方法为例，其内部通过循环 CAS 实现原子递增： 
 
```java
public class AtomicInteger extends Number implements java.io.Serializable {
 private static final Unsafe unsafe = Unsafe.getUnsafe();
 private static final long valueOffset; // value 字段的内存偏移量

 static {
 try {
 valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
 } catch (Exception ex) { throw new Error(ex); }
 }

 private volatile int value; // 保证可见性

 public final int incrementAndGet() {
 return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
 }
}
```
 
 unsafe.getAndAddInt方法的底层逻辑是一个 **自旋循环**： 
 
```java
public final int getAndAddInt(Object o, long offset, int delta) {
 int v;
 do {
 v = getIntVolatile(o, offset); // 读取当前值（volatile 保证可见性）
 // 循环 CAS：如果当前值等于预期值 v，则更新为 v + delta
 } while (!compareAndSwapInt(o, offset, v, v + delta)); 
 return v;
}
```
 **自旋**：如果 CAS 失败（说明其他线程修改了值），则重新读取当前值并再次尝试 CAS，直到成功。 **volatile 变量**：value被声明为volatile，保证多线程间的可见性（任何线程对value的修改会立即被其他线程感知）。 
#### **三、CAS 的应用场景**
 
 CAS 在 Java 并发编程中被广泛使用，典型场景包括： 
 **原子类（AtomicXXX）**：如AtomicInteger、AtomicBoolean，用于实现无锁的计数器、状态标记等。 **并发容器（JUC 包）**：如ConcurrentHashMap（JDK 8+）的节点更新、ConcurrentLinkedQueue的链表操作，通过 CAS 替代锁提升性能。 **锁的底层实现**：如ReentrantLock的tryLock()方法通过 CAS 尝试获取锁，减少线程阻塞概率。 
#### **四、CAS 与锁的对比**
 
| 特性 | CAS（无锁） | 锁（如 synchronized） |
| --- | --- | --- |
| 线程状态 | 自旋（不阻塞） | 阻塞（需唤醒） |
| 适用场景 | 低竞争、短时间操作 | 高竞争、长时间操作 |
| 上下文切换 | 无（避免线程阻塞） | 可能频繁（阻塞/唤醒开销大） |
| 复杂度 | 需处理 ABA、自旋开销等问题 | 简单（JVM 优化后性能提升） |

## 13. CAS会出现什么问题？ABA如何解决？
### 

### 一句话总结

 CAS操作可能遇到ABA问题：某变量被其他线程从A改为B又改回A，CAS误判未修改。

 ABA问题的解决方案包括：

 1. 使用版本号（如AtomicStampedReference），每次修改递增版本号，CAS同时检查值和版本号；

 2. 通过标记位或时间戳机制确保状态唯一性，避免值被重复使用。

### 详细解析

 在 Java 并发编程中，**CAS（Compare and Swap）** 是一种无锁的原子操作，用于实现线程安全。尽管它高效，但在某些场景下会引发问题，其中最典型的是 **ABA 问题**。以下是详细分析及解决方案：

---

#### **1. CAS 的潜在问题(1) ABA 问题** 

 - **现象**： 线程 1 读取变量值为A，随后被挂起； 线程 2 将值修改为B，后又改回A； 线程 1 恢复执行，发现值仍为A，认为未被修改，CAS 操作成功。 **本质问题**：值虽然相同，但中间发生过变化，可能导致逻辑错误（如链表节点被重复修改）。

 **(2) 自旋开销** 

 - **问题**：CAS 失败后会不断重试（自旋），若长时间竞争激烈，会消耗 CPU 资源。

 **(3) 仅支持单一变量** 

 - **限制**：CAS 只能保证单个变量的原子性，无法直接作用于多个变量或复杂对象。

---

#### **2. ABA 问题的解决方案(1) 版本号（Stamp）机制** 

 通过维护一个 **版本号（或时间戳）**，每次修改值的同时递增版本号。CAS 操作需同时校验 **值** 和 **版本号**，确保值未被其他线程修改过。

 **(2) Java 实现：AtomicStampedReference** 

 - **原理**： 将值与一个int类型的版本号绑定，每次更新时版本号递增。 示例代码： ```java AtomicStampedReference<Integer> atomicRef = new AtomicStampedReference<>(100, 0); // 初始版本号 int oldStamp = atomicRef.getStamp(); int oldValue = atomicRef.getReference(); // 更新值并校验版本号 boolean success = atomicRef.compareAndSet( oldValue, 200, oldStamp, oldStamp + 1 ); ```

 **(3) 其他方案：AtomicMarkableReference** 

 - **原理**： 使用一个boolean标记代替版本号，标记值是否被修改过。适用于只需简单标记的场景。

---

#### **3. 其他问题的应对策略(1) 自旋开销优化** 

 - **退避策略**：失败后延迟重试（如指数退避），减少竞争。

 - **锁升级**：当自旋次数超过阈值，转为悲观锁（如synchronized）。

 **(2) 多变量原子性** 

 - 使用AtomicReference封装对象，通过对象的不可变性保证多状态一致性。

---

#### **4. ABA 问题示例场景**

 假设一个无锁栈结构，栈顶节点为A，线程 1 准备将栈顶替换为B：

 - 线程 1 读取栈顶为A；

 - 线程 2 弹出A，压入C，再弹出C，压入A；

 - 线程 1 执行 CAS，发现栈顶仍为A，操作成功，但此时栈的实际状态可能已不一致（如A的next指针被修改过）。

 **使用AtomicStampedReference后**：即使栈顶值仍为A，版本号不同会导致 CAS 失败，强制线程 1 重新检查状态。

---

#### **总结**

| 问题 | 解决方案 | 适用场景 |
| --- | --- | --- |
| ABA 问题 | 版本号（AtomicStampedReference） | 需严格校验值未被中途修改的场景 |
| 自旋开销 | 退避策略或锁升级 | 高并发竞争场景 |
| 多变量原子性 | 封装为不可变对象 | 需要维护多个关联变量的场景 |

 **核心原则**：

 - 如果值的中间变化不影响业务逻辑，ABA 问题可忽略；

 - 若中间变化可能导致错误（如状态机、链表等），必须使用版本号机制。

## 1. Java 线程和操作系统的线程有什么区别？
### 

### 一句话总结

 Java线程是JVM层面的抽象，早期采用用户态线程（绿色线程），现代主流实现（如HotSpot）采用1:1模型直接映射操作系统内核线程。区别在于：1.抽象层次不同（JVM管理状态/优先级，操作系统实际调度）；2.部分特性不完全对应（如中断机制）；3.JVM可能优化线程创建/销毁过程。最终执行仍依赖操作系统线程，但提供跨平台一致性。

### 详细解析

#### 一、本质关系

 - 依赖关系 Java 线程是操作系统线程的封装，其底层实现依赖于宿主系统的线程库（如 Windows 的 Win32 线程、Linux 的 Pthread）。JVM 通过调用系统 API 创建和管理操作系统线程，Java 程序中的Thread类本质上是操作系统的线程代理。

 - 映射模型 • 早期 JDK（1.2 之前）：使用纯用户态的“绿色线程”（User-Level Threads），由 JVM 自行调度，无需内核支持，但无法利用多核 CPU。 • 现代 JDK（1.2+）：采用“原生线程模型”（1:1 映射），每个 Java 线程对应一个操作系统线程，由内核调度。

---

 **二、核心区别** 

| 维度 | Java 线程 | 操作系统线程 |
| --- | --- | --- |
| 调度主体 | JVM 负责线程状态管理（如start()/stop()） | 操作系统内核直接调度（通过时间片轮转、优先级等） |
| 资源开销 | 创建/销毁成本低（JVM 抽象封装） | 创建/销毁成本高（需内核分配 TCB、栈空间等） |
| 同步机制 | 提供synchronized、Lock等高级接口 | 依赖系统调用（如信号量、互斥锁） |
| 跨平台性 | 线程行为与操作系统无关（JVM 屏蔽差异） | 行为依赖具体内核实现（如 Linux 的调度策略） |
| 生命周期 | 由 JVM 管理（如 GC 回收线程栈） | 由内核管理（如线程阻塞、终止） |

---

 **三、状态映射 

 Java 线程的状态（定义在Thread.State枚举）是对操作系统线程状态的抽象：

| Java 线程状态 | 对应操作系统线程状态 | 说明 |
| --- | --- | --- |
| NEW | 未创建 | Java 线程对象已实例化，但未调用start()，底层内核线程未创建。 |
| RUNNABLE | 就绪（Ready）或运行（Running） | 表示 Java 线程已启动，可能正在 CPU 上执行（运行）或等待调度（就绪）。 |
| BLOCKED | 阻塞（Blocked） | 因竞争锁（如synchronized）被挂起，等待锁释放。 |
| WAITING/TIMED_WAITING | 阻塞（Blocked） | 因wait()、join()、sleep()等方法主动挂起，等待特定条件（如超时、通知）。 |
| TERMINATED | 终止（Terminated） | 线程执行结束，底层内核线程已销毁。 |

四、实现细节对比** 

 - 线程创建 • Java 线程：调用new Thread()时，JVM 通过 JNI 调用系统 API（如pthread_create）创建操作系统线程。 • 操作系统线程：直接通过系统调用（如 Linux 的clone()）创建，需显式处理线程上下文和资源分配。

 - 上下文切换 • Java 线程：JVM 在用户态管理线程状态（如栈帧、程序计数器），切换时仅需保存/恢复 JVM 线程上下文，无需进入内核态。 • 操作系统线程：切换需内核介入，保存/恢复寄存器、内存映射等，开销较大。

 - 优先级控制 • Java 线程：通过setPriority()设置优先级，但最终由操作系统映射实现（不同平台优先级策略不同）。 • 操作系统线程：直接使用内核定义的优先级（如 Linux 的SCHED_FIFO）。

## 7. Java中如何检测死锁？如何预防和避免线程死锁?
### 

### 一句话总结

 检测死锁：使用jstack分析线程转储或通过ThreadMXBean的findDeadlockedThreads方法。

预防死锁：1.按固定顺序获取锁，破坏循环等待；2.避免嵌套锁；3.使用tryLock设置超时机制；4.减少同步代码块范围。

### 详细解析

#### 一、死锁的检测方法

 - **线程转储分析** 1.1 工具：jstack、VisualVM、JConsole。

 1.2 步骤：

 • 通过jstack <pid>生成线程转储文件。

 • 搜索关键词deadlock，自动高亮死锁线程及锁依赖关系。

 示例输出片段：

```text
 Found one Java-level deadlock:
 "Thread-1": waiting to lock monitor 0x00007f8b1c004280 (object 0x000000076b39e4e8, a java.lang.Object),
 which is held by "Thread-2"
 "Thread-2": waiting to lock monitor 0x00007f8b1c003ae0 (object 0x000000076b39e4f0, a java.lang.Object),
 which is held by "Thread-1"
```

 ** 2. 编程检测（ThreadMXBean）** 

 • 代码示例：

```java
import java.lang.management.ManagementFactory;
import java.lang.management.ThreadMXBean;

public class DeadLockDetector {
 public static void main(String[] args) {
 ThreadMXBean detector = ManagementFactory.getThreadMXBean();
 long[] deadlockedThreads = detector.findDeadlockedThreads();
 if (deadlockedThreads != null) {
 System.out.println("检测到死锁线程：" + deadlockedThreads.length);
 }
 }
}
```

 ** 3.监控工具**

 • VisualVM：实时监控线程状态，自动标记死锁线程。

 • APM工具：如 SkyWalking、Pinpoint，集成死锁检测告警功能。

---

 **三、死锁的预防与避免策略 1. 破坏互斥条件（不可行）** 

 原因：锁的互斥性是线程安全的基础，无法破坏。

 2. **破坏请求与保持条件** 

 方案：线程一次性申请所有所需资源，未获取成功则释放已占资源

 代码示例：

```java
synchronized (lock1) {
 if (tryLock(lock2)) { // 尝试获取lock2
 try {
 // 执行任务
 } finally {
 unlock(lock2);
 }
 }
}
```

 ** 3. 破坏不剥夺条件**

 方案：允许线程主动释放已持有的锁（如ReentrantLock.unlock()）。

 代码示例：

```java
Lock lock = new ReentrantLock();
if (lock.tryLock()) {
 try {
 // 临界区
 } finally {
 lock.unlock(); // 主动释放
 }
}
```

 ** 4.破坏循环等待条件**

 方案：按固定顺序获取锁（全局锁排序）。

 代码示例：

```java
// 定义锁的顺序
private static final Object[] locks = {lockA, lockB, lockC};

public void doWork() {
 Arrays.sort(locks); // 按对象哈希值排序
 synchronized (locks[0]) {
 synchronized (locks[1]) {
 synchronized (locks[2]) {
 // 临界区
 }
 }
 }
}
```

 ** 5.超时机制**

 方案：使用Lock.tryLock(timeout, unit)设置超时时间。

 代码示例：

```java
if (lock1.tryLock(1, TimeUnit.SECONDS)) {
 try {
 if (lock2.tryLock(1, TimeUnit.SECONDS)) {
 try {
 // 执行任务
 } finally {
 lock2.unlock();
 }
 }
 } finally {
 lock1.unlock();
 }
}
```

 ** 6.资源分配策略**

 • 银行家算法：动态检测资源分配安全性，避免系统进入不安全状态（适用于实时系统）。

 • 锁分级：将锁按优先级分级，高优先级锁优先分配。

** 7. 并发工具替代锁**

 • 无锁数据结构：如ConcurrentHashMap、AtomicInteger。

 • 信号量（Semaphore）：控制并发访问数量。

 • CountDownLatch/CyclicBarrier：协调线程执行顺序，避免嵌套锁。

## 6. 写一个Java死锁的例子
### 

### 一句话总结

 创建两个线程，每个线程先锁定一个对象，再尝试获取对方持有的对象。当线程1锁定A后请求B，线程2锁定B后请求A时，形成循环等待导致死锁。核心代码使用synchronized嵌套锁，通过Thread.sleep(100)确保锁竞争顺序，运行后线程会相互阻塞无法继续执行。

### 详细解析

 以下是一个简单的 Java 死锁代码示例，演示两个线程互相等待对方释放锁的场景：

```java
public class DeadlockExample {
 private static final Object lock1 = new Object();
 private static final Object lock2 = new Object();

 public static void main(String[] args) {
 Thread threadA = new Thread(() -> {
 synchronized (lock1) {
 System.out.println("Thread-A 持有 lock1");
 try {
 Thread.sleep(100); // 模拟操作耗时，增加死锁概率
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 System.out.println("Thread-A 等待获取 lock2...");
 synchronized (lock2) {
 System.out.println("Thread-A 成功获取 lock2");
 }
 }
 });

 Thread threadB = new Thread(() -> {
 synchronized (lock2) {
 System.out.println("Thread-B 持有 lock2");
 try {
 Thread.sleep(100); // 模拟操作耗时，增加死锁概率
 } catch (InterruptedException e) {
 e.printStackTrace();
 }
 System.out.println("Thread-B 等待获取 lock1...");
 synchronized (lock1) {
 System.out.println("Thread-B 成功获取 lock1");
 }
 }
 });

 threadA.start();
 threadB.start();
 }
}
```

---

#### **代码解析**

 - **定义两个锁对象**： ```java private static final Object lock1 = new Object(); private static final Object lock2 = new Object(); ```

 - **线程A的执行逻辑**： 先获取lock1，然后尝试获取lock2。

 - 中间通过Thread.sleep(100)模拟耗时操作，让线程B有机会拿到lock2。

 - **线程B的执行逻辑**： 先获取lock2，然后尝试获取lock1。

 - 同样通过Thread.sleep(100)增加死锁概率。

 - **死锁产生条件**： **互斥**：synchronized保证锁的互斥性。

 - **持有并等待**：线程A持有lock1并等待lock2，线程B持有lock2并等待lock1。

 - **不可抢占**：锁只能由持有线程释放。

 - **循环等待**：线程A等待线程B，线程B等待线程A。

---

#### **运行结果**

 程序会输出以下内容后卡住（死锁）：

```cpp
Thread-A 持有 lock1
Thread-B 持有 lock2
Thread-A 等待获取 lock2...
Thread-B 等待获取 lock1...
```

---

#### **如何验证死锁？**

 - **使用jstack工具**： 运行程序后，通过jps查找 Java 进程 ID。

 - 执行jstack <pid>，输出中会显示检测到的死锁： ```plaintext Found one Java-level deadlock: ============================= "Thread-B": waiting to lock monitor 0x00007f... (object 0x...), which is held by "Thread-A" "Thread-A": waiting to lock monitor 0x00007f... (object 0x...), which is held by "Thread-B" ```

 - **使用 VisualVM 或 JConsole**： 图形化工具会直接标注出死锁线程。

---

#### **如何解决这个死锁？**

 - **统一锁的获取顺序**： 强制两个线程按相同顺序获取锁（例如都先获取lock1，再获取lock2）。

 - **使用超时锁**： 将synchronized替换为ReentrantLock.tryLock(timeout)，超时失败时释放已有锁。

 - **减少锁的嵌套**： 尽量避免在持有锁时请求其他锁。

## 9. volatile 可以保证原子性么？
### 

### 一句话总结

 volatile关键字不能保证原子性，只能保证可见性和有序性。对于多线程环境下的复合操作（如i++），即使变量被声明为volatile，仍需通过加锁或使用原子类（如AtomicInteger）来确保原子性操作。

### 详细解析

 一、原子性的定义与要求 

 原子性指一个操作要么完全执行，要么完全不执行，不会被其他线程中断。例如：

• 原子操作：i = 1（单次赋值）、flag = true（状态标记）。

 • 非原子操作：i++（需读取、计算、写入三步）、count = count + 1（复合运算）。

---

 **二、volatile 的局限性：无法保证复合操作原子性** 

 - **底层机制分析** • 读/写操作的原子性： volatile 对单个变量的 读 和 写 操作是原子的（如int a = volatileVar或volatileVar = 10），因为 JVM 会通过内存屏障确保直接操作主内存。 • 复合操作的原子性： 若操作涉及 多步骤（如自增i++），即使变量是 volatile，仍可能被其他线程打断。例如： ```java // 线程A和B同时执行以下操作 volatile int counter = 0; counter++; // 实际分解为：读counter → 加1 → 写回counter ``` 可能出现 竞态条件：两个线程均读取counter=0，各自加1后写回，最终结果为1而非预期的2。

 - **字节码验证** 通过反编译volatile int counter++的字节码，可见其包含 4 条指令： ```bytecode getstatic // 读取counter值 iconst_1 // 加载常量1 iadd // 执行加法 putstatic // 写回结果 ``` 这四步操作在多线程环境下可能被中断，导致原子性失效。

---

 **三、适用场景与替代方案** 

 - **volatile 的适用场景** • 单次读/写操作：如状态标志（volatile boolean isShutdown）。

 • DCL 单例模式：通过 volatile 禁止指令重排序，确保对象初始化完成后再被其他线程访问。

 • 安全发布对象：确保对象引用对其他线程可见时已完成初始化。

 ** 2.需要原子性的场景解决方案** 

| 场景 | 问题 | 解决方案 |
| --- | --- | --- |
| 计数器自增 | 多线程并发导致结果错误 | AtomicInteger（CAS 实现） |
| 复合条件判断 | 读取-修改-写入非原子 | synchronized或ReentrantLock |
| 单例双重检查锁定 | 指令重排序导致半初始化对象泄露 | volatile+ 双重检查锁定（DCL） |

 代码示例（AtomicInteger 替代 volatile）：

```java
public class AtomicCounter {
 private AtomicInteger counter = new AtomicInteger(0);

 public void increment() {
 counter.incrementAndGet(); // 原子性自增
 }

 public int getCount() {
 return counter.get();
 }
}
```

## 10. 公平锁与非公平锁有什么区别？
### 

### 一句话总结

 公平锁按请求顺序分配锁，保证线程先到先得；非公平锁允许插队，可能出现线程饥饿但吞吐量更高。公平锁通过队列严格维护顺序，获取时检查是否有等待队列；非公平锁直接尝试抢占锁，失败后才入队。非公平锁减少线程切换开销，性能更好但公平性差。

### 详细解析

#### 一、核心定义与实现原理对比

| 维度 | 公平锁 | 非公平锁 |
| --- | --- | --- |
| 线程获取顺序 | 按请求锁的顺序分配（FIFO），确保先到先得 | 允许插队，新线程可直接竞争锁，可能抢占已等待线程的资源 |
| 底层实现 | 通过AQS的等待队列维护顺序，获取锁时检查队列是否有前驱节点（hasQueuedPredecessors()） | 直接尝试CAS获取锁，失败后再进入队列等待 |
| Java 实现类 | ReentrantLock（通过构造函数fair=true指定） | ReentrantLock（默认非公平）、synchronized（仅非公平） |
| 锁释放逻辑 | 释放锁时唤醒队列头部的线程 | 释放锁时唤醒队列中的线程（可能随机或按优先级） |

---

 **二、性能与资源消耗差异** 

 - **吞吐量** • 非公平锁： 允许线程直接竞争锁，减少上下文切换和队列维护开销，吞吐量更高（通常比公平锁高 3-5 倍）。 示例：在高并发场景下，非公平锁的吞吐量可达 10,000 ops/ms，而公平锁可能仅 2,000 ops/ms。

 • 公平锁：

 需维护严格的等待队列，频繁的队列操作（如入队、出队）和上下文切换导致性能下降。

 ** 2. 资源竞争** 

 • 非公平锁：

 新线程可能“插队”获取锁，导致等待队列中的线程长时间饥饿（Starvation）。

 • 公平锁：

 严格按顺序分配锁，避免饥饿问题，但需牺牲性能。

---

 **三、代码实现与锁竞争流程** 

 - **非公平锁（ReentrantLock 默认）** ```java // NonfairSync 的 lock 方法 final void lock() { if (compareAndSetState(0, 1)) // 直接尝试 CAS 获取锁 setExclusiveOwnerThread(Thread.currentThread()); else acquire(1); // 失败则进入 AQS 队列 } ``` • 特点： 线程优先尝试直接获取锁，失败后再排队，减少等待时间。

 - **公平锁（ReentrantLock 构造时指定 fair=true）** ```java // FairSync 的 lock 方法 protected final boolean tryAcquire(int acquires) { final Thread current = Thread.currentThread(); int c = getState(); if (c == 0) { if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) // 检查队列是否有前驱节点 setExclusiveOwnerThread(current); return true; } // ... 重入逻辑 } ``` • 特点： 获取锁前检查队列，确保无等待线程时才允许获取，保证公平性。

---

 **四、适用场景与选型建议** 

| 场景需求 | 推荐锁类型 | 原因 |
| --- | --- | --- |
| 高并发、吞吐量优先 | 非公平锁 | 减少锁竞争开销，提升系统吞吐量 |
| 严格顺序处理（如事务顺序） | 公平锁 | 避免线程饥饿，确保请求按到达顺序处理 |
| 短任务、低延迟 | 非公平锁 | 插队机制可快速响应高优先级任务 |
| 长任务、资源密集型 | 公平锁 | 避免长任务独占锁导致后续任务长时间等待 |

 典型应用示例：

• 非公平锁：网络请求处理、缓存更新、日志记录。

 • 公平锁：数据库连接池、订单处理队列、实时交易系统。

---

 **五、性能测试对比（示例）** 

```java
public class LockPerformanceTest {
 private static final int THREAD_COUNT = 10;
 private static final int ITERATIONS = 1_000_000;

 public static void main(String[] args) {
 testLockPerformance(new ReentrantLock(true), "Fair Lock");
 testLockPerformance(new ReentrantLock(false), "Non-Fair Lock");
 }

 private static void testLockPerformance(Lock lock, String lockType) {
 long startTime = System.currentTimeMillis();
 ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
 for (int i = 0; i < THREAD_COUNT; i++) {
 executor.submit(() -> {
 for (int j = 0; j < ITERATIONS; j++) {
 lock.lock();
 try { /* 模拟临界区操作 */ } finally { lock.unlock(); }
 }
 });
 }
 executor.shutdown();
 while (!executor.isTerminated()) {}
 long endTime = System.currentTimeMillis();
 System.out.println(lockType + "吞吐量: " + (THREAD_COUNT * ITERATIONS / (endTime - startTime)) + " ops/ms");
 }
}
```

 输出：

```cpp
Fair Lock吞吐量: 1,850 ops/ms
Non-Fair Lock吞吐量: 9,200 ops/ms
```

## 11. 讲一下乐观锁和悲观锁。
### 

### 一句话总结

 乐观锁假设操作冲突少，只在提交时检查数据是否被修改（如版本号机制），适用于读多写少场景。悲观锁预先加锁（如行锁、表锁），确保独占操作，适合写多场景，但影响并发性能。前者减少锁竞争，后者保证强一致性。

### 详细解析

#### 一、核心思想对比

| 维度 | 悲观锁（Pessimistic Locking） | 乐观锁（Optimistic Locking） |
| --- | --- | --- |
| 核心假设 | 假设并发冲突必然发生，需通过加锁避免数据竞争 | 假设并发冲突极少发生，仅在提交时验证数据是否被修改 |
| 加锁时机 | 读取数据时立即加锁，阻塞其他线程访问 | 读取数据时不加锁，更新时通过版本号或CAS机制验证 |
| 锁粒度 | 行级锁、表级锁（如数据库的SELECT ... FOR UPDATE） | 无物理锁，通过版本号或哈希值实现逻辑控制 |
| 典型实现 | 数据库行锁、Java的synchronized、ReentrantLock | 数据库版本号、Java的Atomic类（CAS）、时间戳机制 |

---

 **二、实现原理与技术细节** 

 - **悲观锁的实现** • 数据库层面： 通过SELECT ... FOR UPDATE对查询结果加排他锁，其他事务需等待锁释放才能修改数据。 ```sql BEGIN; SELECT * FROM orders WHERE id = 1 FOR UPDATE; -- 加锁 UPDATE orders SET status = 'paid' WHERE id = 1; COMMIT; -- 释放锁 ``` • Java层面： 使用synchronized关键字或ReentrantLock实现线程级互斥。

 - **乐观锁的实现** • 版本号机制： 数据库表中增加version字段，更新时校验版本一致性。 ```sql -- 读取数据时获取版本号 SELECT stock, version FROM products WHERE id = 1; -- 更新时校验版本 UPDATE products SET stock = stock - 1, version = version + 1 WHERE id = 1 AND version = 2; -- 若版本不匹配则更新失败 ``` • CAS（Compare and Swap）： 通过原子操作比较预期值与实际值，若一致则更新（如AtomicInteger.incrementAndGet()）。

---

 **三、性能与适用场景对比** 

| 指标 | 悲观锁 | 乐观锁 |
| --- | --- | --- |
| 吞吐量 | 低（锁竞争导致线程阻塞和上下文切换） | 高（无锁竞争，适合高并发读场景） |
| 冲突处理 | 直接阻塞，避免冲突 | 冲突时回滚或重试（如CAS循环） |
| 适用场景 | 写操作频繁、冲突概率高（如银行转账、库存扣减） | 读操作为主、冲突概率低（如商品浏览、配置读取） |
| 死锁风险 | 高（需严格按顺序释放锁） | 无 |
| 实现复杂度 | 低（依赖数据库或语言原生锁） | 中（需设计版本号或CAS逻辑） |

---

 **四、典型应用场景** 

 - **悲观锁适用场景** • 金融交易：确保转账操作的原子性，避免超卖或重复扣款。

 • 库存管理：在高并发下单场景中，防止超卖（如秒杀系统）。

 • 强一致性需求：如数据库事务的隔离级别为SERIALIZABLE时。

 ** 2.乐观锁适用场景** 

 • 电商库存：读多写少场景，通过版本号控制库存更新。

 • 配置系统：频繁读取配置，偶尔更新时避免锁竞争。

 • 分布式系统：结合Redis的WATCH命令实现乐观锁

## 16. synchronized和volatile有什么区别？
### 

### 一句话总结

 1. 作用范围：synchronized用于方法或代码块，保证原子性和可见性；volatile仅修饰变量，保证可见性和禁止指令重排序。
2. 原子性：synchronized支持多操作原子性（如i++）；volatile不保证复合操作的原子性。
3. 性能：volatile无锁机制，开销更小；synchronized涉及锁竞争和上下文切换，开销较大。
4. 适用场景：synchronized解决多线程竞争问题；volatile适用于单写多读的变量同步。

### 详细解析

 synchronized和volatile是 Java 中用于解决多线程并发问题的两个关键工具，但它们的设计目标、实现机制和适用场景有本质区别。以下是两者的核心差异：

#### **1. 核心作用不同**

 - **synchronized**： 是**锁机制**，通过互斥（同一时间仅允许一个线程执行同步代码）保证**原子性**，并通过内存屏障保证**可见性**。它用于保护共享资源的复合操作（如多步读写、计算），确保操作的完整性和线程安全。

 - **volatile**： 是**变量修饰符**，通过强制变量直接读写主内存（而非线程本地缓存）保证**可见性**，并通过禁止特定指令重排（如禁止写后读重排）避免多线程下的逻辑混乱。它不保证原子性（除非是单个变量的读写操作）。

#### **2. 原子性 vs 可见性**

 - **原子性**： synchronized能保证原子性。由于它通过锁的互斥性，同一时间只有一个线程执行同步代码块，因此复合操作（如count++）不会被其他线程打断。 volatile**不保证原子性**。即使变量被volatile修饰，像count++（包含“读取-修改-写入”三步）这样的复合操作仍可能因多线程交错执行导致数据错误。

 - **可见性**： 两者都能保证可见性，但实现方式不同： synchronized：通过“锁的释放-获取”机制（JVM 插入内存屏障），确保锁释放前所有变量修改对其他线程可见（即其他线程获取锁时会强制从主内存读取最新值）。

 - volatile：通过内存屏障（如StoreLoad屏障），强制变量修改后立即写入主内存，且其他线程读取时直接从主内存获取最新值，避免使用本地缓存的旧值。

#### **3. 指令重排的控制**

 - volatile可以**禁止特定指令重排**。例如，在单例模式的双重检查锁定（DCL）中，volatile能防止“对象初始化”和“赋值给引用”的操作顺序被重排（避免其他线程看到未完全初始化的对象）。

 - synchronized不直接禁止指令重排。同步代码块内的操作可能被编译器或 CPU 重排，只要重排后的结果与顺序执行一致（即 **as-if-serial 语义**）。但由于锁的互斥性，其他线程无法看到未完成的中间状态，因此间接保证了逻辑顺序。

#### **4. 使用范围**

 - synchronized可以修饰： 实例方法（锁为当前对象this）；

 - 静态方法（锁为类的Class对象）；

 - 代码块（显式指定锁对象）。

 - volatile只能修饰**变量**（成员变量或静态变量），不能修饰方法或代码块。

#### **5. 性能差异**

 - volatile是轻量级的，没有锁竞争的开销（无需上下文切换、线程阻塞）。

 - synchronized在低竞争场景下（JDK 1.6 后通过偏向锁、轻量级锁优化）性能接近volatile，但高竞争场景下可能升级为重量级锁（涉及操作系统内核态切换），开销较大。

#### **6. 典型应用场景**

 - **volatile**： 状态标志（如boolean isRunning，控制线程启停）；

 - 单例模式的双重检查锁定（防止指令重排导致的空指针）；

 - 独立读写的变量（如volatile int version，仅需保证每次读取最新值，无需复合操作）。

 - **synchronized**： 保护复合操作（如i++、账户转账的金额修改）；

 - 需要保证原子性和可见性的共享资源操作（如计数器、缓存更新）。

#### **总结**

| 特性 | synchronized | volatile |
| --- | --- | --- |
| 核心作用 | 锁机制，保证原子性和可见性 | 变量可见性，禁止指令重排 |
| 原子性 | 保证（互斥访问） | 不保证（仅单个读写是原子的） |
| 可见性 | 保证（通过锁的内存屏障） | 保证（强制主内存读写） |
| 指令重排 | 不直接禁止（但互斥保证逻辑顺序） | 禁止特定重排（如写后读重排） |
| 使用范围 | 方法、代码块 | 变量 |
| 性能 | 高竞争场景开销大（但优化后改善） | 轻量级，无锁竞争开销 |
| 典型场景 | 复合操作、共享资源的完整保护 | 状态标志、单例 DCL、独立变量读写 |

 **一句话总结**：synchronized是“重量级”的锁，全面保证原子性和可见性；volatile是“轻量级”的变量修饰符，仅保证可见性和禁止重排，适合简单场景。

## 17. 介绍下ReentrantLock的定义和特性。
### 

### 一句话总结

 ReentrantLock是Java中的可重入互斥锁，提供线程同步功能。支持公平/非公平模式，允许线程重复获取锁且不会死锁。特性包括可中断锁获取、超时尝试锁及Condition条件变量。需手动加锁解锁，相比synchronized更灵活控制并发，适用于复杂多线程场景。

### 详细解析

 ReentrantLock是 Java 并发包（java.util.concurrent.locks）中提供的一种**显式可重入互斥锁**，用于在多线程环境中控制对共享资源的访问。它通过手动获取和释放锁的方式，提供了比synchronized关键字更灵活、更细粒度的锁控制能力。

#### **一、核心特性**

 1. **可重入性（Reentrant）** 

 与synchronized类似，ReentrantLock支持**可重入性**：同一个线程可以多次获取同一把锁而不会被阻塞（通过记录锁的“持有次数”实现）。例如，线程 A 获取锁后，在未释放的情况下可以再次获取该锁，锁的计数器会递增，直到所有获取操作都被对应释放（计数器归零），其他线程才能获取该锁。

 2. **显式锁管理** 

 与synchronized隐式（自动获取/释放锁）不同，ReentrantLock是**显式锁**：

 - 通过lock()方法主动获取锁；

 - 通过unlock()方法主动释放锁（必须在finally块中调用，避免因异常导致锁未释放而死锁）。

 示例：

```java
ReentrantLock lock = new ReentrantLock();
try {
 lock.lock(); // 显式获取锁
 // 临界区代码（操作共享资源）
} finally {
 lock.unlock(); // 显式释放锁（必须在finally中）
}
```

 3. **公平锁与非公平锁（Fair vs Non-fair）** 

 ReentrantLock支持两种锁策略，通过构造函数参数控制：

 - **非公平锁（默认）**：线程获取锁的顺序与等待队列无关，新线程可能“插队”直接获取锁（提高吞吐量，但可能导致部分线程长时间等待）。

 - **公平锁**：严格按照等待队列的顺序分配锁（先等待的线程先获取锁，避免“线程饥饿”，但性能略低）。

 构造方式：

```java
ReentrantLock fairLock = new ReentrantLock(true); // 公平锁
ReentrantLock nonFairLock = new ReentrantLock(); // 非公平锁（默认）
```

 4. **可中断的锁获取** 

 ReentrantLock提供lockInterruptibly()方法，允许在等待锁的过程中响应中断（通过Thread.interrupt()）。这避免了synchronized中线程一旦进入阻塞状态就无法被中断的问题。

 示例：

```java
try {
 lock.lockInterruptibly(); // 可中断的锁获取
 // 临界区代码
} catch (InterruptedException e) {
 // 线程被中断时的处理逻辑
 Thread.currentThread().interrupt(); // 恢复中断状态
} finally {
 if (lock.isHeldByCurrentThread()) { // 检查当前线程是否持有锁（避免异常释放）
 lock.unlock();
 }
}
```

 5. **超时获取锁** 

 通过tryLock(long timeout, TimeUnit unit)方法，线程可以尝试在指定时间内获取锁。若超时未获取到锁，则返回false，避免无限等待。

 示例：

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) { // 尝试1秒内获取锁
 try {
 // 临界区代码
 } finally {
 lock.unlock();
 }
} else {
 // 超时处理逻辑（如放弃操作）
}
```

 6. **条件变量（Condition）支持** 

 ReentrantLock可以通过newCondition()方法创建多个Condition对象（条件变量），实现更细粒度的线程等待/通知机制。相比synchronized仅能通过wait()/notify()共享一个等待队列，Condition支持**多个独立的等待队列**，适合复杂的线程协调场景（如生产者-消费者模型中的不同条件）。

 示例（生产者-消费者）：

```java
ReentrantLock lock = new ReentrantLock();
Condition notFull = lock.newCondition(); // 队列未满条件
Condition notEmpty = lock.newCondition(); // 队列未空条件

// 生产者放入元素
lock.lock();
try {
 while (queue.size() == MAX_SIZE) {
 notFull.await(); // 等待队列非满
 }
 queue.add(element);
 notEmpty.signal(); // 通知消费者队列非空
} finally {
 lock.unlock();
}

// 消费者取出元素
lock.lock();
try {
 while (queue.isEmpty()) {
 notEmpty.await(); // 等待队列非空
 }
 queue.poll();
 notFull.signal(); // 通知生产者队列非满
} finally {
 lock.unlock();
}
```

#### **二、与synchronized的对比**

| 特性 | synchronized | ReentrantLock |
| --- | --- | --- |
| 锁类型 | 隐式锁（自动获取/释放） | 显式锁（手动获取/释放） |
| 可重入性 | 支持 | 支持 |
| 公平性 | 非公平（默认） | 支持公平/非公平（可配置） |
| 可中断性 | 不支持（阻塞后无法中断） | 支持（lockInterruptibly()） |
| 超时获取锁 | 不支持 | 支持（tryLock(timeout)） |
| 条件变量 | 仅1个（wait()/notify()） | 支持多个独立Condition对象 |
| 锁状态查询 | 不支持（无法感知锁是否被持有） | 支持（如isLocked()、getQueueLength()） |
| 代码复杂度 | 简单（语法糖） | 较高（需手动管理锁） |

#### **三、适用场景**

 ReentrantLock适合需要**更灵活锁控制**的场景，例如：

 - 需要公平锁策略（避免线程饥饿）；

 - 需要可中断或超时获取锁（防止死锁）；

 - 需要多个条件变量（如复杂的生产者-消费者模型）；

 - 需要监控锁的状态（如调试或性能优化）。

#### **四、注意事项**

 - **必须手动释放锁**：unlock()需在finally块中调用，确保锁必然释放，避免死锁。

 - **公平锁的性能**：公平锁虽然更“公平”，但需维护严格的等待队列，通常性能低于非公平锁（默认策略）。

 - **条件变量的正确使用**：Condition.await()必须在锁的保护下调用（即先获取锁），否则会抛出IllegalMonitorStateException。

#### **五、总结**

 ReentrantLock是synchronized的增强版，通过显式锁管理提供了更灵活的并发控制能力（如可中断、超时、公平锁、多条件变量）。在需要高级锁特性时（如复杂线程协调），ReentrantLock是更优选择；而简单同步场景（如基础互斥）则推荐使用synchronized（代码更简洁）。

## 18. ReentrantReadWriteLock的使用场景是？
### 

### 一句话总结

 ReentrantReadWriteLock适用于读多写少的高并发场景，如缓存系统。读操作共享资源时可允许多线程并行访问，提升性能；写操作独占资源保证数据一致性。通过分离读写锁降低线程竞争，适合需要频繁读取但较少修改的数据同步需求。

### 详细解析

 ReentrantReadWriteLock（可重入读写锁）是 Java 并发包中用于解决“读多写少”场景下并发问题的工具，其核心设计是**分离读锁和写锁**：读锁（ReadLock）支持多线程共享，写锁（WriteLock）独占。这种机制能显著提升“读多写少”场景下的并发效率。以下是其典型适用场景及原理分析：

#### **一、核心适用场景：读多写少**

 ReentrantReadWriteLock具有以下特性。

 - 可重入性：同一线程可多次获取读锁或写锁（如递归调用），避免死锁。

 - 锁降级（Write Lock → Read Lock）：持有写锁时可获取读锁，再释放写锁（降级为读锁），保证“写后读”的一致性（如更新缓存后立即验证最新值）。

 - 公平/非公平策略：默认非公平模式（提升吞吐量），公平模式（减少线程饥饿，适用于对公平性要求高的场景）。

 ReentrantReadWriteLock最适合的场景是 **读操作远多于写操作**，且需要保证数据一致性的场景。此时，读锁的共享特性可以大幅减少线程间的竞争，而写锁的独占性则确保写操作的原子性和可见性。

#### **二、具体典型场景**

 1. **缓存系统（高频读、低频写）** 

 缓存的核心特点是“读多写少”：大量线程需要读取缓存中的数据（如热点数据、配置信息），但缓存的更新（如过期后重新加载、手动刷新）频率较低。

 - **普通互斥锁的问题**：若用synchronized或ReentrantLock，每次读操作都会互斥，即使数据未修改，读线程也会互相阻塞，导致性能瓶颈。

 - **读写锁的优势**：读操作使用读锁（可共享），允许多线程同时读取缓存；写操作使用写锁（独占），保证更新时无其他线程干扰。例如： ```java class Cache { private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock(); private final Lock readLock = rwLock.readLock(); private final Lock writeLock = rwLock.writeLock(); private Map<String, Object> data = new HashMap<>(); // 读缓存（多线程共享） public Object get(String key) { readLock.lock(); try { return data.get(key); } finally { readLock.unlock(); } } // 写缓存（独占） public void put(String key, Object value) { writeLock.lock(); try { data.put(key, value); } finally { writeLock.unlock(); } } } ```

 2. **配置/状态管理（高频读、低频修改）** 

 系统中的全局配置（如开关、阈值）或状态（如服务是否可用）通常被频繁读取，但修改操作（如运维调整配置）很少。此时，读写锁能高效支持多线程并发读取，同时保证修改时的原子性。

 - **示例**：一个线程安全的配置管理器，允许多个线程同时读取配置，但修改时必须独占锁。

 3. **统计/报表数据（批量读、偶尔更新）** 

 统计类数据（如在线用户数、订单总量）常被多个线程读取分析，但更新（如定时任务统计）频率较低。读写锁允许统计时多线程并发读取，更新时独占锁保证数据完整性。

 - **优势**：避免了普通锁下“统计时所有读操作阻塞”的问题，提升系统吞吐量。

 4. **高并发数据结构（读多写少的自定义结构）** 

 对于需要自定义的高并发数据结构（如线程安全的字典、列表），若读操作远多于修改（如get远多于put），用读写锁替代synchronized能减少读操作的竞争。

 - **对比**：ConcurrentHashMap内部也使用了类似读写分离的机制（分段锁/CAS），但ReentrantReadWriteLock更灵活，可自定义锁的范围和逻辑。

#### **三、不适用场景**

 若**写操作频繁**（如读写比例接近 1:1 或写更多），ReentrantReadWriteLock的优势会被削弱，甚至可能因锁的维护开销（如读锁的共享状态管理）导致性能下降。此时，普通互斥锁（如ReentrantLock）或无锁数据结构（如原子类）可能更高效。

#### ** 

 四、与其他锁的对比

| 场景 | ReentrantReadWriteLock | ReentrantLock | synchronized |
| --- | --- | --- | --- |
| 读多写少 | ✅ 高吞吐 | ❌ 低效 | ❌ 低效 |
| 写多读少 | ❌ 性能下降 | ✅ 高效 | ✅ 简单 |
| 锁降级 | ✅ 支持 | ✅ 支持 | ❌ 不支持 |
| 公平性控制 | ✅ 支持 | ✅ 支持 | ❌ 仅非公平 |

**

 五、总结 

 ReentrantReadWriteLock适合 **读操作远多于写操作，且需要保证写操作独占性、读操作并发性** 的场景（如缓存、配置管理、统计数据等）。其核心价值是通过“读写分离”平衡了并发效率与数据一致性，在“读多写少”时显著优于普通互斥锁。

## 20. ThreadLocal 内存泄露问题是怎么导致的？
### 

### 一句话总结

 ThreadLocal内存泄露是由于Entry的key使用弱引用，当ThreadLocal对象被回收后key变为null，但value仍被强引用导致无法回收。若未及时调用remove()清理，线程长期存活（如线程池复用）会导致value累积占用内存，从而引发内存泄露。

### 详细解析

 ThreadLocal的内存泄漏问题，本质是由 **线程生命周期与ThreadLocal对象的引用关系**、**ThreadLocalMap的设计缺陷** 共同导致的。以下是具体原因的分层解析：

#### 一、基础背景：ThreadLocal 的工作机制

 ThreadLocal的核心是为 **每个线程独立存储数据**，避免多线程竞争。其实现依赖于每个线程内部的ThreadLocalMap（一个类似HashMap的容器）：

 - 每个线程（Thread对象）持有一个ThreadLocalMap成员变量；

 - ThreadLocalMap的存储单元是Entry对象，其中： **键（key）**：是ThreadLocal对象的 **弱引用**（WeakReference<ThreadLocal<?>>）；

 - **值（value）**：是用户通过ThreadLocal.set()存入的实际对象（强引用）。

#### 二、内存泄漏的直接原因：Entry 的弱引用键与强引用值的矛盾

 ThreadLocalMap.Entry的设计中，键是弱引用，值是强引用，这是内存泄漏的关键矛盾点：

 - **弱引用键的特性**：当外部代码不再持有ThreadLocal对象的强引用（即ThreadLocal变量被回收或作用域结束），Entry中的弱引用键会被垃圾回收器（GC）自动回收（键变为null）。

 - **强引用值的残留**：即使键被回收（变为null），Entry中的值（value）仍然被Entry以强引用形式持有。此时，这个值对象已经 **无法通过任何ThreadLocal键访问到**（因为键是null），但由于被Entry强引用，GC 无法回收它。

#### 三、内存泄漏的触发条件：线程生命周期过长

 上述矛盾本身不会直接导致内存泄漏，只有当 **线程的生命周期长于ThreadLocal的使用周期** 时，泄漏才会发生：

 - **普通线程**：如果线程执行结束并被销毁，线程的ThreadLocalMap也会被回收，其中的所有Entry（包括残留的值）会被一并清理，不会泄漏。

 - **线程池中的线程**：线程池的核心线程是 **复用的、不会被销毁** 的（例如FixedThreadPool）。此时，线程的ThreadLocalMap会长期存在，残留的Entry值无法被回收，随着时间积累，内存泄漏会越来越严重。

#### 四、为什么使用弱引用键？这是设计缺陷吗？

 ThreadLocalMap选择弱引用作为键，**并非缺陷，而是一种权衡**：

 - 如果键是强引用：当外部ThreadLocal变量被回收后，ThreadLocalMap中的键（强引用）仍然指向ThreadLocal对象，导致ThreadLocal无法被回收（即使已不再使用），反而会造成更严重的ThreadLocal对象本身的泄漏。

 - 弱引用键的优势：确保ThreadLocal对象本身可以被及时回收（键被 GC 后变为null），避免了ThreadLocal自身的泄漏，但代价是可能残留值对象。

#### 五、总结：内存泄漏的本质

 ThreadLocal内存泄漏的根本原因是：

 **当ThreadLocal对象的外部强引用被回收后，ThreadLocalMap.Entry的弱引用键会被 GC 回收（变为null），但值对象（value）仍被Entry强引用。若线程长期存活（如线程池中的线程），这些无法被访问的 value 对象会一直残留在线程的ThreadLocalMap中，无法被 GC 回收，导致内存泄漏。** 

#### 如何避免？

 虽然ThreadLocal设计上存在潜在泄漏风险，但可以通过以下方式规避：

 - **主动清理**：使用完ThreadLocal后，调用ThreadLocal.remove()方法，显式删除Entry（包括键和值）。

 - **限制作用域**：尽量将ThreadLocal定义为static final（单例），避免频繁创建/销毁ThreadLocal对象，减少弱引用键被回收的概率。

 - **线程池合理配置**：控制线程池的生命周期，避免核心线程无限制存活（如使用ScheduledThreadPool时注意线程复用策略）。

## 22. 线程池常用的阻塞队列有哪些？
### 

### 一句话总结

 线程池常用的阻塞队列包括：

1. ArrayBlockingQueue（基于数组的有界队列）；

2. LinkedBlockingQueue（基于链表的可选有界/无界队列）；

3. SynchronousQueue（不存储元素，直接传递任务）；

4. PriorityBlockingQueue（支持优先级排序的无界队列）。

此外，ScheduledThreadPoolExecutor使用延迟队列DelayedWorkQueue处理定时任务。

### 详细解析

#### 一、核心阻塞队列类型及特性

 在 Java 线程池中，阻塞队列（BlockingQueue） 是任务存储和调度的核心组件。以下是常用的 5 种阻塞队列及其特性：

| 队列类型 | 底层结构 | 容量特性 | 核心机制 | 适用场景 |
| --- | --- | --- | --- | --- |
| ArrayBlockingQueue | 数组 | 有界（需指定） | FIFO 顺序，支持公平/非公平锁 | 需控制任务量，避免内存溢出 |
| LinkedBlockingQueue | 链表 | 可选有界/无界 | FIFO 顺序，两把锁（生产/消费分离） | 高吞吐量，任务量波动大 |
| SynchronousQueue | 无存储结构 | 零容量 | 直接传递任务，无缓冲 | 线程数可动态扩展（如短时任务） |
| PriorityBlockingQueue | 堆结构 | 无界 | 按优先级排序（自然序或自定义 Comparator） | 需优先处理高优先级任务 |
| DelayQueue | 优先级堆 | 无界 | 延迟到期后才能被消费 | 定时任务、缓存过期处理 |

---

### **二、各队列的详细解析**

#### 1. ArrayBlockingQueue（数组有界阻塞队列）

 - 底层结构：基于固定大小的数组实现，初始化时需显式指定容量（capacity）。

 - 特性： 有界性：队列容量固定，无法动态扩展。当队列满时，新任务会触发线程池的拒绝策略（如RejectedExecutionHandler）。

 - 顺序性：遵循 FIFO（先进先出） 原则，任务按提交顺序被处理。

 - 线程安全：通过ReentrantLock保证并发安全，支持公平锁（按等待顺序访问）或非公平锁（默认，吞吐量更高）。

 - 适用场景： 适合需要严格控制队列大小、避免内存溢出的场景（如有界的任务缓冲）。例如，对资源使用敏感的系统中，防止任务堆积导致OOM。

#### 2. LinkedBlockingQueue（链表阻塞队列）

 - 底层结构：基于链表实现，支持有界（需显式指定容量）或无界（默认容量为Integer.MAX_VALUE，近似无界）。

 - 特性： 有界 vs 无界： 无界模式（默认）：队列理论上无上限，任务会无限堆积，可能导致内存溢出（OOM）。

 - 有界模式：需显式指定容量（如new LinkedBlockingQueue<>(100)），队列满时触发拒绝策略。

 - 顺序性：遵循 FIFO 原则。

 - 线程安全：通过分离的读锁和写锁（putLock和takeLock）实现更高的并发效率。

 - 适用场景： 无界模式：适用于任务处理速度较快、任务量可控的场景（但需警惕OOM风险，如Executors.newFixedThreadPool()默认使用此队列）。

 - 有界模式：适合需要限制任务缓冲大小，同时希望比ArrayBlockingQueue有更高并发性能的场景。

#### 3. SynchronousQueue（同步队列）

 - 底层结构：不存储任务的特殊队列（容量为0）。每个插入操作（put）必须等待另一个线程的取出操作（take），反之亦然。

 - 特性： 无存储能力：任务无法在队列中停留，必须立即被线程处理。

 - 直接移交（Direct Handoff）：任务提交后，若没有空闲线程，会直接触发线程池创建新线程（直到达到maxPoolSize）；若maxPoolSize也被耗尽，则触发拒绝策略。

 - 适用场景： 适合处理短平快的任务（任务执行时间短），且需要快速响应的场景。例如，Executors.newCachedThreadPool()默认使用此队列，通过动态扩缩线程来应对突发任务（maxPoolSize为Integer.MAX_VALUE）。

#### 4. PriorityBlockingQueue（优先阻塞队列）

 - 底层结构：基于堆（Heap）实现的无界阻塞队列，任务按优先级排序。

 - 特性： 无界性：队列容量无上限（仅受内存限制），任务会无限堆积（需警惕OOM）。

 - 优先级排序：任务需实现Comparable接口或通过Comparator指定排序规则，优先级高的任务先被处理。

 - 线程安全：通过ReentrantLock保证并发安全。

 - 适用场景： 适合需要按优先级处理任务的场景（如任务有轻重缓急之分）。例如，支付系统中优先处理高金额订单，或实时系统中优先处理低延迟任务。

#### 5. DelayQueue（延迟阻塞队列）

 - 底层结构：基于堆（Heap）实现的无界阻塞队列，任务需实现Delayed接口（定义延迟时间）。

 - 特性： 延迟触发：任务只有在延迟时间到期后才能被取出（take()操作会阻塞直到任务延迟到期）。

 - 无界性：队列容量无上限（受内存限制）。

 - 优先级：堆结构保证延迟时间短的任务先被处理（即使后提交）。

 - 适用场景： 适合需要延迟执行或定时执行任务的场景。例如，订单超时自动取消（30分钟未支付）、缓存条目定时失效等。

---

### 三、线程池与阻塞队列的匹配关系

不同线程池通过选择不同的阻塞队列实现其设计目标：

| 线程池类型 | 使用的阻塞队列 | 设计目标 |
| --- | --- | --- |
| FixedThreadPool | LinkedBlockingQueue | 固定线程数，任务队列无界，避免线程频繁创建 |
| CachedThreadPool | SynchronousQueue | 线程数动态扩展，任务直接传递，适合短时任务 |
| ScheduledThreadPool | DelayedWorkQueue | 延迟或周期性任务调度 |
| SingleThreadExecutor | LinkedBlockingQueue | 单线程顺序执行任务 |

---

### 四、阻塞队列的选型建议

 - 任务特性优先： • 高并发短任务：SynchronousQueue（减少队列存储开销）。 • 需延迟处理：DelayQueue。 • 优先级敏感：PriorityBlockingQueue。

 - 资源控制优先： • 内存敏感：ArrayBlockingQueue（有界队列，避免 OOM）。 • 吞吐量优先：LinkedBlockingQueue（无界队列，高并发性能好）。

 - 特殊场景： • 任务直接移交：SynchronousQueue（如CachedThreadPool）。 • 定时任务：DelayQueue（如ScheduledThreadPool）。

## 23. 设计一个线程池，可以根据任务的优先级来运行。
### 

### 一句话总结

 设计线程池时使用优先队列（如PriorityBlockingQueue）存储任务，通过实现Comparator接口或任务对象的Comparable接口来定义优先级规则。工作线程从队列中按优先级顺序取出任务执行，核心线程数、最大线程数等参数通过ThreadPoolExecutor自定义配置。需确保任务添加时的线程安全，并处理可能存在的线程饥饿问题。

### 详细解析

 基于 Java 的 ThreadPoolExecutor，通过 优先级队列 和 任务排序机制 实现任务优先级调度。

 1. 定义优先级任务 

 任务类需包含优先级字段，并实现Comparable接口或提供Comparator，以便优先级队列排序。

```java
public class PriorityTask implements Runnable, Comparable<PriorityTask> {
 private final Runnable task; // 实际任务逻辑
 private final int priority; // 优先级（数值越小优先级越高）

 public PriorityTask(Runnable task, int priority) {
 this.task = task;
 this.priority = priority;
 }

 @Override
 public void run() {
 task.run();
 }

 @Override
 public int compareTo(PriorityTask other) {
 return Integer.compare(this.priority, other.priority);
 }
}
```

---

#### **2. 创建优先级阻塞队列**

 使用PriorityBlockingQueue作为线程池的任务队列，确保任务按优先级排序。

```java
BlockingQueue<Runnable> priorityQueue = new PriorityBlockingQueue<>(11, Comparator.comparingInt(task -> {
 if (task instanceof PriorityTask) {
 return ((PriorityTask) task).getPriority();
 }
 return 0; // 默认优先级
}));
```

---

#### **3. 自定义线程池**

 通过ThreadPoolExecutor构建线程池，指定优先级队列，并配置核心参数（核心线程数、最大线程数等）。

```java
int corePoolSize = 4;
int maxPoolSize = 8;
long keepAliveTime = 60L;
TimeUnit unit = TimeUnit.SECONDS;

ThreadPoolExecutor executor = new ThreadPoolExecutor(
 corePoolSize,
 maxPoolSize,
 keepAliveTime,
 unit,
 priorityQueue
);
```

---

#### **4. 提交任务时指定优先级**

 向线程池提交任务时，封装为PriorityTask并指定优先级值。

```java
executor.execute(new PriorityTask(() -> {
 System.out.println("高优先级任务执行");
}, 1)); // 优先级 1（最高）

executor.execute(new PriorityTask(() -> {
 System.out.println("低优先级任务执行");
}, 3)); // 优先级 3
```

---

#### **5. 处理拒绝策略**

 自定义拒绝策略，处理队列满时的任务提交（如记录日志或动态扩容）。

```java
executor.setRejectedExecutionHandler((task, executor) -> {
 System.err.println("任务被拒绝: " + task);
 // 可选：降级处理或重试逻辑
});
```

---

#### **6. 验证执行顺序**

 测试线程池，确保高优先级任务先执行：

```java
public static void main(String[] args) {
 // 创建线程池（代码同上）

 // 提交不同优先级任务
 executor.execute(new PriorityTask(() -> System.out.println("Task 3 (优先级 3)"), 3));
 executor.execute(new PriorityTask(() -> System.out.println("Task 1 (优先级 1)"), 1));
 executor.execute(new PriorityTask(() -> System.out.println("Task 2 (优先级 2)"), 2));

 // 关闭线程池
 executor.shutdown();
}
```

 **输出结果**：

```cpp
Task 1 (优先级 1)
Task 2 (优先级 2)
Task 3 (优先级 3)
```

---

#### **7. 高级优化(1) 动态调整优先级** 

 若需运行时修改任务优先级，需自定义队列并重写offer()方法，触发重新排序。

```java
public class ResizablePriorityBlockingQueue<E> extends PriorityBlockingQueue<E> {
 @Override
 public boolean offer(E e) {
 boolean added = super.offer(e);
 if (added) {
 // 触发堆重建（实际需通过锁和条件变量实现）
 PriorityQueue<E> pq = new PriorityQueue<>(this);
 this.clear();
 this.addAll(pq);
 }
 return added;
 }
}
```

 **(2) 混合优先级与公平性** 

 若需兼顾优先级和提交顺序，可使用多层队列（如高优先级队列 + 普通队列）。

---

#### **总结**

| 组件 | 作用 |
| --- | --- |
| PriorityTask | 封装任务和优先级，实现排序逻辑。 |
| PriorityBlockingQueue | 按优先级存储任务，确保高优任务先出队。 |
| ThreadPoolExecutor | 管理线程生命周期和任务调度。 |
| 拒绝策略 | 处理队列满时的任务降级或重试。 |

 通过以上设计，线程池将优先执行高优先级任务，适用于订单处理、实时计算等需要任务分级的场景。

## 24. 说一下：Callable 和 Future。
### 

### 一句话总结

 Callable是Java中可返回结果和抛出异常的任务接口，常用于多线程异步执行。

 Future代表异步计算结果，提供检查任务状态、获取结果或取消任务的方法。

 两者配合使用时，Callable定义具体任务逻辑，Future用于监控和获取任务执行结果。通过线程池提交Callable任务会返回Future对象，实现异步处理机制。主要解决传统Runnable接口无法获取返回值的局限性。

### 详细解析

#### 一、核心关系：生产者-消费者模型

 Callable 是任务的生产者，负责定义需要执行的任务并生成结果；Future 是任务的消费者，用于获取任务执行的结果或状态。两者通过 异步执行 和 结果传递 实现协作。

---

 **二、功能对比与协作流程** 

| 角色 | Callable | Future |
| --- | --- | --- |
| 定义目的 | 定义可返回结果的任务（实现call()） | 获取异步任务的结果或状态 |
| 核心方法 | V call() throws Exception | V get(),cancel(),isDone()等 |
| 协作流程 | 1. 线程池提交 Callable 任务
2. 生成 Future 对象
3. 通过 Future 管理任务 | 1. 通过Future.get()阻塞等待结果
2. 检查任务状态（完成/取消） |

---

 **三、具体协作步骤** 

 - 任务提交 通过线程池的submit(Callable<T>)方法提交任务，返回一个Future<T>对象： ```java ExecutorService executor = Executors.newSingleThreadExecutor(); Future<Integer> future = executor.submit(() -> { Thread.sleep(1000); return 42; }); ```

 - 结果获取 调用Future.get()阻塞等待结果（可设置超时）： ```java try { Integer result = future.get(2, TimeUnit.SECONDS); // 超时 2 秒 } catch (TimeoutException e) { future.cancel(true); // 超时后取消任务 } ```

 - 状态管理 •isDone()：检查任务是否完成（无论成功或失败） •isCancelled()：检查任务是否被取消 •cancel(boolean mayInterrupt)：尝试取消任务（true表示中断执行中的线程）

---

 **四、异常处理机制**

• 任务执行异常：若Callable.call()抛出异常，Future.get()会抛出ExecutionException，需通过getCause()获取原始异常。

 • 取消任务：调用cancel()后，若任务未开始则直接取消；若已开始且mayInterrupt为true，则中断线程。

---

 **五、Future 的局限性及解决方案** 

| 局限性 | 解决方案 |
| --- | --- |
| get()阻塞调用线程 | 使用get(timeout, unit)设置超时，或结合回调机制（如CompletableFuture） |
| 无法主动推送任务进度 | 通过自定义状态监听器或轮询isDone()实现 |
| 单任务结果管理 | 使用CompletableFuture实现多任务依赖和结果聚合 |

---

 **六、FutureTask：Callable 与 Future 的结合体**

FutureTask是Future的实现类，同时实现了Runnable接口，可直接作为线程任务提交：

```java
FutureTask<Integer> futureTask = new FutureTask<>(() -> 42);
new Thread(futureTask).start(); // 直接启动线程执行
Integer result = futureTask.get(); // 获取结果
```

 优势：

• 兼容ExecutorService.submit()和Thread两种提交方式

 • 内部自动管理Callable与Future的状态同步

---

 **七、实际应用场景** 

 - 异步计算 如耗时数据处理（图片压缩、文件解析），主线程可继续处理其他逻辑。

 - 任务超时控制 通过get(timeout, unit)避免无限等待。

 - 批量任务管理 提交多个Callable任务，通过Future数组批量获取结果： ```java List<Future<String>> futures = new ArrayList<>(); for (int i = 0; i < 10; i++) { futures.add(executor.submit(() -> "Result")); } futures.forEach(f -> { try { System.out.println(f.get()); } catch (Exception e) { e.printStackTrace(); } }); ```

## 26. Semaphore 有什么用？
### 

### 一句话总结

 Semaphore 是用于控制多线程/进程对共享资源访问的同步机制。通过维护计数器实现资源管理，提供 wait() 和 signal() 操作协调线程执行顺序。主要解决并发场景下的互斥访问（二进制信号量）和资源数量限制（计数信号量）问题，防止数据竞争和系统资源耗尽。

### 详细解析

#### 一、核心功能：资源并发控制

 Semaphore（信号量） 是 Java 并发包中用于 控制同时访问共享资源的线程数量 的同步工具。其核心机制是通过 许可证（Permits） 的获取与释放，实现线程的并发调度。

核心价值：

• 限制资源过载（如数据库连接池、线程池）

 • 实现流量控制（如 API 限流）

 • 协调多线程协作（如生产者-消费者模型）

---

 **二、核心应用场景** 

 - **限流与资源保护** • 场景：限制同时访问某个服务的线程数（如 API 接口限流）。

 • 实现：初始化 Semaphore 的许可数为最大并发量，线程执行前需获取许可。

```java
 Semaphore semaphore = new Semaphore(100); // 允许 100 个并发请求
 public void handleRequest() {
 semaphore.acquire();
 try {
 // 处理请求
 } finally {
 semaphore.release();
 }
 }
```

 优势：避免系统因突发流量崩溃。

 ** 2.资源池管理** 

 • 场景：控制数据库连接、线程池等有限资源的并发使用。

 • 实现：许可数等于资源池容量。

```java
 // 数据库连接池示例
 Semaphore dbConnectionPool = new Semaphore(10); // 最大 10 个连接
 Connection getConnection() throws InterruptedException {
 dbConnectionPool.acquire();
 return dataSource.getConnection();
 }
 void releaseConnection(Connection conn) {
 conn.close();
 dbConnectionPool.release();
 }
```

 优势：防止资源耗尽。

 ** 3.多线程限量执行** 

 • 场景：控制同时执行的任务数量（如批量处理）。

 • 实现：通过acquire()和release()动态调整并发度。

```java
 // 限制同时处理的任务数为 5
 Semaphore taskSemaphore = new Semaphore(5);
 for (int i = 0; i < 100; i++) {
 new Thread(() -> {
 taskSemaphore.acquire();
 try {
 processTask();
 } finally {
 taskSemaphore.release();
 }
 }).start();
 }
```

 优势：平衡资源占用与处理效率。

 ** 4.生产者-消费者模型** 

 • 场景：协调生产者和消费者的执行节奏。

 •实现：通过信号量控制缓冲区容量。

```java
 Semaphore emptySlots = new Semaphore(10); // 缓冲区容量 10
 Semaphore filledSlots = new Semaphore(0);

 // 生产者
 void produce() throws InterruptedException {
 emptySlots.acquire();
 addToBuffer();
 filledSlots.release();
 }

 // 消费者
 void consume() throws InterruptedException {
 filledSlots.acquire();
 removeFromBuffer();
 emptySlots.release();
 }
```

 优势：避免生产者过快或消费者过慢导致的阻塞。

 ** 5.实现互斥锁** 

 • 场景：将 Semaphore 退化为互斥锁（独占模式）。

 • 实现：初始化许可数为 1。

```java
 Semaphore mutex = new Semaphore(1);
 public void criticalSection() throws InterruptedException {
 mutex.acquire();
 try {
 // 临界区代码
 } finally {
 mutex.release();
 }
 }
```

 优势：相比synchronized，支持公平性配置。

---

 **三、与锁的对比** 

| 特性 | Semaphore | synchronized/ReentrantLock |
| --- | --- | --- |
| 并发控制 | 允许多个线程同时访问 | 仅允许一个线程访问 |
| 资源管理 | 适用于有限资源池 | 适用于临界区保护 |
| 灵活性 | 支持动态调整许可数 | 固定锁状态 |
| 公平性 | 可配置公平模式 | 非公平锁（默认） |

---

 **四、底层实现原理** 

 - AQS 框架：Semaphore 基于AbstractQueuedSynchronizer实现，通过state变量记录可用许可数。

 - 队列管理：使用 CLH 队列管理等待线程，非公平模式下新线程可能抢占许可。

 - CAS 操作：通过compareAndSetState()保证许可数修改的原子性。

---

 **五、实际代码示例（带超时与异常处理）** 

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class SemaphoreDemo {
 private static final Semaphore semaphore = new Semaphore(3, true); // 公平模式，3 个许可

 public static void main(String[] args) {
 for (int i = 0; i < 10; i++) {
 new Thread(new Worker(i)).start();
 }
 }

 static class Worker implements Runnable {
 private final int id;

 Worker(int id) {
 this.id = id;
 }

 @Override
 public void run() {
 try {
 System.out.println("Worker-" + id + " 正在申请资源...");
 if (semaphore.tryAcquire(2, 1, TimeUnit.SECONDS)) { // 尝试获取 2 个许可，超时 1 秒
 try {
 System.out.println("Worker-" + id + " 获取资源，开始工作...");
 Thread.sleep(2000);
 } finally {
 semaphore.release(2);
 System.out.println("Worker-" + id + " 释放资源");
 }
 } else {
 System.out.println("Worker-" + id + " 获取资源超时");
 }
 } catch (InterruptedException e) {
 Thread.currentThread().interrupt();
 }
 }
 }
}
```

## 27. CountDownLatch 有什么用？
### 

### 一句话总结

 CountDownLatch 用于同步线程，允许一个或多个线程等待其他线程完成操作。初始化时设定计数器，线程完成任务后调用 countDown() 减少计数，当计数器归零时，等待的线程被唤醒。常用于主线程等待多个子线程初始化完成，或控制多个线程同时开始执行任务。

### 详细解析

 CountDownLatch 是 Java 并发包（java.util.concurrent）中的同步工具类，其核心作用是 协调多个线程的执行顺序，允许一个或多个线程等待其他线程完成操作后再继续执行。以下是其核心用途、工作原理及典型场景的详细解析：

---

 **一、核心功能与作用** 

 - 等待多个线程完成 主线程（或某个线程）需要等待多个子线程执行完毕后再继续处理后续逻辑。 示例：主线程等待所有服务初始化完成后再启动业务逻辑。

 - 控制并行任务的起始点 多个线程在某一时刻同时开始执行任务（如赛跑发令枪）。 示例：同时启动多个线程进行数据处理，确保所有线程同时进入执行阶段。

 - 实现最大并行性 通过等待所有线程就绪后同时释放，最大化资源利用率。

---

 **二、工作原理** 

 - 计数器机制 • 初始化时设定一个计数值count，表示需要等待的任务数量。 • 每个任务完成后调用countDown()方法将计数器减 1。 • 当计数器减至 0 时，所有调用await()的线程被唤醒。

 - 底层依赖 AQS 基于AbstractQueuedSynchronizer（AQS）实现，通过同步状态（state）管理计数器，线程阻塞和唤醒由 AQS 的队列控制。

---

 **三、典型应用场景** 

 - **主线程等待子线程完成** 场景：主线程需要汇总多个子线程的计算结果。 代码示例： ```java CountDownLatch latch = new CountDownLatch(3); for (int i = 0; i < 3; i++) { new Thread(() -> { // 子线程执行任务 latch.countDown(); }).start(); } latch.await(); // 主线程等待所有子线程完成 System.out.println("所有子线程已完成，主线程继续执行"); ``` 输出： ```cpp 子线程1完成 子线程2完成 子线程3完成 所有子线程已完成，主线程继续执行 ```

 - **多服务依赖启动** 场景：主服务需等待数据库、缓存等依赖服务启动完成后才能运行。 代码示例： ```java CountDownLatch serviceLatch = new CountDownLatch(3); // 启动数据库服务 new Thread(() -> { startDatabase(); serviceLatch.countDown(); }).start(); // 启动缓存服务 new Thread(() -> { startCache(); serviceLatch.countDown(); }).start(); serviceLatch.await(); // 等待所有服务启动 startMainService(); ```

 - **模拟并发测试** 场景：模拟多个用户同时发起请求，测试系统性能。 代码示例： ```java CountDownLatch startLatch = new CountDownLatch(1); // 发令枪 CountDownLatch endLatch = new CountDownLatch(100); // 等待100个线程完成 for (int i = 0; i < 100; i++) { new Thread(() -> { try { startLatch.await(); // 等待发令枪响 sendRequest(); // 模拟请求 } catch (InterruptedException e) { Thread.currentThread().interrupt(); } finally { endLatch.countDown(); } }).start(); } startLatch.countDown(); // 开始测试 endLatch.await(); // 等待所有请求完成 ```

 - **任务分阶段执行** 场景：将任务拆分为多个阶段，每个阶段需等待前一阶段完成。 示例： ```java CountDownLatch phase1Latch = new CountDownLatch(5); CountDownLatch phase2Latch = new CountDownLatch(5); // 阶段1：数据预处理 for (int i = 0; i < 5; i++) { new Thread(() -> { preprocessData(); phase1Latch.countDown(); }).start(); } phase1Latch.await(); // 等待阶段1完成 // 阶段2：数据分析 for (int i = 0; i < 5; i++) { new Thread(() -> { analyzeData(); phase2Latch.countDown(); }).start(); } phase2Latch.await(); // 等待阶段2完成 ```

---

 **四、与 CyclicBarrier 的区别** 

| 特性 | CountDownLatch | CyclicBarrier |
| --- | --- | --- |
| 计数器重置 | 一次性使用，计数器归零后不可重置 | 可重复使用，计数器归零后自动重置 |
| 等待方向 | 一个或多个线程等待其他线程完成 | 所有线程相互等待，直到全部到达屏障点 |
| 典型用途 | 任务汇总、服务依赖启动 | 分阶段任务协调（如多阶段计算） |
