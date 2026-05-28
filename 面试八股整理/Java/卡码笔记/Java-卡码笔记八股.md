# Java面试题合集

> 来源：卡码笔记（notes.kamacoder.com）

## SpringMVC执行流程｜大厂面试题｜Java高频面试题｜DispatcherServlet、HandlerMapping、HandlerAdapter

#  SpringMVC执行流程

##  简要回答

- SpringMVC的核心是**前端控制器与组件协作**，全程**由前端控制器DispatcherServlet统一调度，解耦组件**。执行流程可以概括为：
  1. 用户发送请求至前端控制器（DispatcherServlet）。
  2. 前端控制器调用HandlerMapping，获取对应处理器和拦截器并返回执行链。
  3. HandlerAdapter进行适配调用具体的处理器Handler（页面控制器Controller）。
  4. 处理器Handler执行完成后返回ModelAndView。
  5. 视图解析器解析视图名后返回具体的View实例。
  6. View实例结合模型数据渲染并相应给用户。
##  详细回答

1. **请求接收**：客户端发送HTTP请求，经过Tomcat等Web服务器接收后转发到SpringMVC的**前端控制器(DispatcherServlet)** 。
2. **查找处理器映射器**：前端控制器收到请求后，根据请求信息（请求URL、请求方法、请求参数）调用 **处理器映射器HandlerMapping**。
3. **返回处理器执行链**：处理器映射器根据xml配置、注解找到具体的处理器，生成**处理器执行链**(HandlerExecutionChain)，包含处理器Handler以及处理器拦截器Interceptor，返回给DispatcherServlet。
4. **适配处理器**：DispatcherServlet调用**处理器适配器(HandlerAdapter)** ，使用supports()方法找到适配当前处理器的适配器。
5. **拦截器前置执行、处理器执行**：处理器适配器(HandlerAdapter)先调用处理器执行链中的拦截器preHandle()方法，再调用具体的**处理器(Handler)** 的业务方法。
6. **处理器返回**：Handler执行完成返回ModelAndView对象，若为REST接口则直接返回JSON等数据。
7. **拦截器后置执行**：HandlerAdapter调用拦截器的postHandle()方法，然后将Handler的执行结果ModelAndView返回给DispatcherServlet。
8. **视图解析**：DispatcherServlet将ModelAndView传给**视图解析器(ViewResolver)** ，视图解析器(ViewResolver)根据ModelAndView的视图名解析为真实视图对象，返回具体的View实例。
9. **视图渲染**：DispatcherServlet将模型数据传递给**View实例** ，View根据自身的视图模版与模型数据生成最终的响应内容，返回给DispatcherServlet。
10. **拦截器完成执行、响应客户端**：DispatcherServlet调用拦截器的afterCompletion()方法后将渲染后的视图返回给**客户端**。
##  知识图解

1. SpringMVC的执行流程  知识扩展

  1. 扩展
    2. **SpringMVC**
      3. 是Spring中一个重要模块，能够让Spring快速构建MVC架构的Web程序。MVC就是模型Model，视图View和控制器Controller，核心思想是通过将业务逻辑、数据和显示分离，主要关注处理Web请求、管理用户会话。控制应用程序流程。
      4. **模型Model**：分为数据模型和业务模型，可以作为实体类承载用户数据，也可以作为Service或DAO对象处理用户提交的请求，负责业务逻辑的处理。
      5. **视图View**：负责将模型数据展示给用户，直接与用户进行交互。如果使用@RestController注解代替传统的@Controller注解，方法会返回JSON格式数据，不会解析视图。
      6. **控制器Controller**：接收用户请求，将用户请求转发给响应的Model处理，根据Model的处理结果给用户提供响应，实现模型与视图的分离，不处理具体业务逻辑。
      7. 用户通过View界面向服务端提交请求，Controller接收到请求后对请求进行解析，找到相应的Model后，对用户请求进行处理后返回给Controller，对响应页面渲染后返回到客户端。
    8. **前端控制器**(DispatcherServlet)
      9. DispatcherServlet是SpringMVC的核心控制器，它负责接收请求，调用相应的处理器，并返回相应的结果。核心作用是协调HandlerMapping、HandlerAdapter等组件，解耦各模块。
    10. **处理器适配器**(HandlerAdapter)
      11. SpringMVC有多种处理器(Controller)，注解式、接口式、Servlet式，不同处理器的调用方式不同，HandlerAdapter可以将不同处理器的调用接口统一，让DispatcherServlet无需关心处理器类型，调用适配器的handle()方法符合开闭原则。
  12. 面试官可能追问
  2. Q1：DispatcherServlet是怎么初始化的？
    3. 服务器启动时，DispatcherServlet会作为Servlet被初始化，执行init()方法，加载SpringMVC的配置文件并初始化核心组件如HandlerMapping、HandlerAdapter和ViewResolver并缓存，初始化完成后等待接收请求。
  4. Q2：处理器返回null时，SpringMVC怎么处理？
    5. 如果方法标注@ResponseBody，会通过MessageConverter将返回值转换成空响应体(JSON)并返回给客户端。如果不是REST接口，DispatcherServlet会跳过视图解析和渲染步骤，返回空响应。
  6. Q3：@RestController和@Controller有什么区别？如果@Controller想返回JSON可以怎么做？
    7. RestController注解是Controller注解和ResponseBody注解的结合，类中的所有方法默认返回JSON；所以在有Controller注解的方法上添加@ResponseBody注解可以返回JSON。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## synchronized关键字｜高频面试题｜Java高频面试题｜线程同步、互斥锁、锁升级

#  synchronized关键字

##  简要回答


synchronized是Java中最基础的**线程同步关键字**，通过**互斥锁机制**保证多线程对共享资源的安全访问，可以**修饰方法或代码块**。

##  详细回答

- synchronized是Java提供的**内置锁（监视器锁）**，其实现依赖软件层面上的JVM，其性能会随着Java版本的升级而提高。
- 当方法或代码快被synchronized修饰时，它将成为**临界区**，同一时刻只能由一个线程访问。
- 当线程通过synchronized等待锁的时候**不能被Thread.interrupt()中断**，需要保证程序合理防止造成死锁。
- 使用synchronized后，会在编译之后在代码前后加上**monitorenter**和**monitorexit**字节码指令。
  - 执行**monitorenter**指令时尝试获取对象锁，如果对象没有被锁或者已经获得了锁，锁的**计数器加一**。此时其他竞争锁的线程会进入阻塞队列等待。
  - 执行**monitorexit**指令时会把计数器减一，当**计数器值为0时，锁释放**，处于等待队列中的线程会竞争锁。
- **加锁**的过程中会清除工作内存中的共享变量，再从主内存读取，**释放锁**的过程则是将工作内存中的共享变量写回主内存。

```
`public synchronized void synchronizedMethod(){
  //这个方法内部的代码被锁定，同一时间只有一个线程可以执行
}

public void someMethod(){
  synchronized(this){
    //这个代码块被锁定
  }
}
`
```
 1
2
3
4
5
6
7
8
9

- Java对象在内存中包含“对象头”，其中有一个**Mark Word字段会记录锁的状态**（无锁/偏向锁/轻量级锁/重量级锁）。
- synchronized版本更新：
  1. JDK6之前，synchronized是重量级锁，线程阻塞/唤醒时需要进行操作系统内核态切换，造成很大开销。
  2. JDK6及以后引入了**锁升级机制**，即偏向锁-轻量级锁-重量级锁的升级规则，大幅减小开销。
##  知识图解

1. synchronized底层示意图  知识扩展

  1. 扩展：
  2. Monitor结构（适用于**重量级锁**）
    3. Monitor是synchronized实现线程同步的核心底层结构，通过**EntryList**和**WaitSet**管理线程的竞争和等待。
    4. Monitor核心结构包含：
      1. **Owner**（指向当前持有锁的线程）
      2. **EntryList**（未获取到锁的线程）
      3. **WaitSet**（调用wait()方法的线程，如果被唤醒线程进入EntrySet）
      4. **recursions**（当前线程持有锁的重入次数）
  1. 面试官可能追问：
  5.

Q1：详细介绍一下锁的升级机制？

    1. 无锁：没有开启偏向锁时为无锁，**JDK6之后默认开启偏向锁**。
    2. 偏向锁：没有线程拿到锁，称为匿名偏向。有线程拿到偏向锁会**记录线程ID**，如果还有相同ID的线程竞争锁则直接获取。**JDK8后默认为轻量级锁**，可以设置附加偏向锁。
    3. 轻量级锁：**新线程竞争偏向锁时升级**为轻量级锁。每个线程通过CAS操作尝试将锁对象头的MarkWord设置为指向自身，如果成功设置则已获得锁。
    4. 重量级锁：**自旋次数/自旋线程数超过阈值升级**为重量级锁。线程被挂起节约CPU资源，但是内核态和用户态切换消耗大。
  6.

Q2：synchronized锁静态方法和普通方法有什么区别？
 锁静态方法 锁普通方法 锁当前类对象 锁当前对象实例 对所有实例调用都互斥 仅对同一对象互斥 同一时间只有一个线程能执行 多线程可同时访问不同对象  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 什么是死锁？如何避免和排查死锁？

#  什么是死锁？如何避免和排查死锁？

##  简要回答

- 死锁是指**多个线程在竞争多个资源时，互相持有对方需要的资源**，导致所有线程都无法继续执行的永久阻塞状态。
- 发生死锁通常需要同时满足四个条件：**互斥条件、持有并等待条件、不可剥夺条件、环路等待条件**。
- 避免死锁的核心思路是**破坏这四个条件中的任意一个**，工程上最常见的方法是：**统一加锁顺序、减少嵌套锁、使用tryLock超时机制、不要在锁内执行耗时操作**。
- 排查死锁时可以通过**jps + jstack**、**jcmd Thread.print -l**、**jconsole/arthas**查看线程栈，重点关注`BLOCKED`线程、锁持有关系以及输出中的`Found one Java-level deadlock`。
##  详细回答

1. 什么是死锁
  2. 死锁本质上是**循环等待**。线程A持有资源1等待资源2，线程B持有资源2等待资源1，双方都不释放自己已经拿到的资源，最终谁也执行不下去。
  3. 在Java里，死锁不仅可能发生在**synchronized**之间，也可能发生在**ReentrantLock**、数据库锁、分布式锁以及多种锁组合使用的场景中。
4. 死锁产生的四个必要条件
  1. **互斥条件**：同一时刻一个资源只能被一个线程占有。
  2. **持有并等待条件**：线程已经持有一个资源，同时继续申请其他资源。
  3. **不可剥夺条件**：线程已经拿到的资源，在自己释放前不能被强行抢走。
  4. **环路等待条件**：多个线程之间形成首尾相接的资源等待环。
  5. 只有这四个条件同时成立时，才会形成死锁，所以**只要破坏其中任意一个条件，就能避免死锁**。
6. Java中常见的死锁场景
  7. **加锁顺序不一致**：线程A先锁A再锁B，线程B先锁B再锁A，是最常见的死锁原因。
  8. **多资源混合加锁**：本地锁、数据库行锁、分布式锁混用，如果获取顺序不统一，容易出现循环等待。
  9. **锁内执行耗时操作**：在线程持锁期间执行RPC、数据库查询、文件IO，会放大锁竞争时间，增加死锁概率。
  10. **线程池任务互相等待**：固定大小线程池中的任务互相`Future.get()`等待，也可能形成“线程池死锁”或饥饿型死锁。
11. 如何避免死锁
  1. **统一资源申请顺序**：这是最常用的方法。比如所有线程都先锁A再锁B，就不会形成环路等待。
  2. **一次性申请所有资源**：如果某个资源拿不到，就释放已经获取的资源后重试，避免“持有并等待”。
  3. **使用可超时锁**：`ReentrantLock.tryLock(timeout)`可以让线程在等待一段时间后放弃，避免永久阻塞。
  4. **使用可中断锁**：`lockInterruptibly()`允许线程在等待锁时响应中断，适合复杂协作场景。
  5. **缩小锁粒度，减少嵌套锁**：只锁真正需要同步的代码，尽量不要在一个锁里再套另一个锁。
  6. **不要在锁内做耗时操作**：例如网络调用、长事务、远程接口、复杂计算等，都会显著增加死锁风险。
  7. **优先使用并发工具类**：能用`ConcurrentHashMap`、原子类、阻塞队列解决的问题，就尽量不要手写多把锁配合。
##  代码示例


```
`public class DeadLockDemo {
    private static final Object LOCK_A = new Object();
    private static final Object LOCK_B = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (LOCK_A) {
                sleep(100);
                synchronized (LOCK_B) {
                    System.out.println("t1 done");
                }
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            synchronized (LOCK_B) {
                sleep(100);
                synchronized (LOCK_A) {
                    System.out.println("t2 done");
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }

    private static void sleep(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35

- 上面代码中，`t1`先拿`LOCK_A`再等`LOCK_B`，`t2`先拿`LOCK_B`再等`LOCK_A`，满足死锁四个条件后就会永久卡住。
- 如果想避免这类问题，可以让所有线程都**按照固定顺序获取锁**，例如统一先拿`LOCK_A`再拿`LOCK_B`。
##  排查步骤

1. **先看现象**
  2. 接口长时间无响应、线程一直不结束、CPU不一定很高，但业务明显“卡住”。
  3. 线程状态中经常能看到多个线程长期处于`BLOCKED`或等待锁的状态。
4. **定位Java进程**
  5. 使用`jps -l`查看Java进程ID。
6. **导出线程栈**
  7. 使用`jstack <pid>`导出线程栈。
  8. 或者使用`jcmd <pid> Thread.print -l`查看更详细的线程和锁信息。
9. **重点看线程栈中的锁关系**
  10. 如果输出中直接出现`Found one Java-level deadlock`，说明JVM已经帮你识别出Java层面的死锁。
  11. 继续看死锁线程名称、线程栈、等待的锁对象以及锁的持有者是谁。
  12. 如果没有直接打印死锁，也要关注多个线程是否互相`waiting to lock`对方持有的对象。
13. **结合代码和日志回溯**
  14. 根据线程名、类名、方法栈定位到具体业务代码。
  15. 检查不同线程的加锁顺序是否一致，是否存在锁内调用数据库、RPC、MQ等耗时逻辑。
  16. 如果涉及数据库事务，还要结合慢SQL日志、InnoDB死锁日志一起分析。
17. **线上临时处理**
  18. 如果已经确认实例死锁且无法自行恢复，通常需要先**重启实例或摘流处理**，避免请求持续堆积。
  19. 后续再通过线程栈、日志和代码复盘根因，不能只靠重启掩盖问题。
##  知识图解

1. 死锁的出现  知识扩展

  1. 扩展：
  2. `jstack`典型会打印出线程的等待关系，例如某个线程“waiting to lock”某个对象，而这个对象又被另一个线程持有。
  3. 如果输出中出现`Found one Java-level deadlock`，说明JVM已经检测到Java层面的循环等待，这也是排查死锁时最直接的证据。
  4. 在业务代码中还可以使用`ThreadMXBean#findDeadlockedThreads()`做定时探测，适合接入监控系统做告警。
  1. 面试官可能追问：
  5. Q1：`synchronized`和`ReentrantLock`都可能发生死锁吗？
    6. **都会**。只要线程之间形成循环等待，就可能死锁。区别在于`ReentrantLock`支持`tryLock()`、超时和可中断等待，因此**更容易做死锁预防**。
  7. Q2：死锁、活锁、饥饿有什么区别？
    8. **死锁**：线程彼此等待，完全不再向前推进。
    9. **活锁**：线程没有阻塞，一直在重试或让步，但是业务仍然没有进展。
    10. **饥饿**：某个线程长期抢不到资源，其他线程还能继续执行。
  11. Q3：线程池也会发生死锁吗？
    12. **会**。例如固定大小线程池中，任务A等待任务B结果，任务B又等待任务A或者等待线程池中尚未执行的任务，就可能出现线程池内部互相等待的问题。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## SpringBoot启动流程｜大厂面试题｜Java高频面试题｜SpringApplication.run()、自动装配、内嵌Tomcat

#  SpringBoot启动流程

##  简要回答

- SpringBoot主要通过SpringApplication.run()方法启动，可以分成创建SpringApplication实例和调用run()方法两个阶段。**初始化SpringApplication实例**需要启动SpringBoot需要先创建一个SpringApplication构造器，在这个阶段主要做的是获取应用类型，加载所有初始化器与监听器，并通过调用栈寻找执行main方法的主类。**执行run()方法**需要执行run()方法会启动计时器并设置系统属性，初始化监听器并启动监听器，设置命令行参数，准备环境对象和应用上下文，刷新应用上下文并发布启动事件结束计时器，完成SpringBoot的启动。
##  详细回答

- SpringBoot启动流程：
1. **初始化SpringApplication实例**：启动SpringBoot需要先创建一个SpringApplication构造器，在这个阶段获取应用类型，加载所有初始化器与监听器，并通过调用栈寻找执行main方法的主类。
  2. **获取应用类型**：通过deduceFromClasspath()方法，当返回值为NONE是应用为非web应用，不会启动服务器；如果返回SERVLET则是基于Servlet的web应用，会启动Tomcat服务器；如果返回REACTIVE则当前应用是响应式应用。
  3. **加载初始化器**：使用getSpringFactoriesInstances()方法可以获取所有初始化器的名称集合，然后根据名称实例化这些初始化器，进行排序后返回设置到initializers属性中。
  4. **加载监听器**：与初始化器类似，使用getSpringFactoriesInstances()读取spring.factories中的ApplicationListener，实例化并启动事件监听器。
5. **执行run()方法**：执行run()方法会启动计时器并设置系统属性，初始化监听器并启动监听器，设置命令行参数，准备环境对象和应用上下文，刷新应用上下文并发布启动事件结束计时器，完成SpringBoot的启动。
  6. **启动计时器**：调用run方法时，首先会创建并启动计时，同时配置java.awt.headless属性为true，保证无GUI界面。
  7. **初始化监听器**：通过getRunListeners()方法加载全周期事件监听器(SpringApplicationRunListener)，并发布启动准备的监听事件。
  8. **设置命令行参数**：将启动时的args参数设置到Environment对象中，并设置到SpringApplication对象中。
  9. **准备环境对象**：创建Environment对象，然后加载系统属性、环境变量和配置文件，然后把配置环境(Environment)加入监听对象。
  10. **打印banner图**：根据应用类型和配置打印banner图，默认是ASCII字符画。
  11. **准备异常报告器**：初始化启动失败时的异常分析器。
  12. **创建应用上下文**：根据应用类型创建对应的应用上下文实例，如果是Web应用，创建AnnotationConfigServletWebServerApplicationContext。
  13. **准备上下文环境**：需要给上下文绑定环境对象，执行ApplicationContextInitializer的初始化操作，然后加载配置类、扫描组件等资源。
  14. **刷新上下文**：是SpringBoot启动的关键步骤，调用refreshContext(context)方法，Spring在此完成自动装配，包括扫描主类及其子包的注解，对Bean进行注册和初始化，筛选生效的配置，加载自动配置类，如果是Web应用还会创建并启动Tomcat服务器。
  15. **finishRefresh**：上下文刷新完成后，用户可以在afterRefresh()方法中进行后置处理，自定义扩展功能。
  16. **结束阶段**：计时器结束，打印出启动耗时，触发上下文启动完成事件并通知监听器，同时调用用户自定义的启动后逻辑，即ApplicationRunner/CommandLineRunner，随后发布就绪事件，SpringBoot应用启动完成，内嵌容器开始监听端口，对外服务。
##  知识图解

1. SpringBoot启动流程  知识扩展

  1. 扩展
    2. @SpringBootApplication
      3. 是SpringBoot启动的关键注解，这个组合注解包含了@SpringBootConfiguration、@EnableAutoConfiguration和@ComponentScan三个注解。 **@SpringBootConfiguration** 注解表示这是一个配置类， **@ EnableAutoConfiguration** 注解开启自动配置，能够通过内部方法扫描classpath的META-INF/spring.factories配置文件，并且将其中EnableAutoConfiguration对应的配置项实例化并注册到spring容器中； **@ ComponentScan** 注解扫描主类所在包及其子包下的组件。
  4. 面试官可能追问
  2. Q1：SpringBoot启动时**Bean的加载顺序**是什么？
    3. SpringBoot 启动时先通过自动配置类注册Bean定义，再加载用户自定义配置类的 Bean 定义；实例化阶段优先处理无依赖的 Bean，再处理有依赖的（按依赖关系）；初始化时按构造方法→依赖注入→@PostConstruct→InitializingBean→@Bean 指定的 initMethod 顺序执行；如果要控制顺序，建议用 @DependsOn 显式声明，避免依赖扫描或方法顺序，懒加载 Bean 则在首次使用时才初始化。
  4. Q2：SpringBoot是怎么**实现内嵌Tomcat**的？
    5. SpringBoot会自动配置TomcatServletWebServerFactory并创建Tomcat实例，然后配置Tomcat连接器(Connector)、引擎(Engine)和主机(Host)，将SpringMVC的DispatcherServlet注册为Tomcat的Servlet，随后调用Tomcat.start()启动容器，绑定端口监听请求。
  6. Q3：SpringBoot启动时，为什么自定义的Bean会被自动配置的Bean覆盖？
    7. 自定义Bean被自动配置的Bean覆盖可能是因为自动配置类使用了@Primary注解导致用户Bean被覆盖，或者用户Bean所在的包被扫描，可以使用@Primary注解标注用户Bean或排除对应的自动配置类解决。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## volatile关键字｜高频面试题｜Java高频面试题｜内存可见性、指令重排序、单例模式

#  volatile关键字

##  简要回答

- volatile是Java并发编程中常用的关键字，作为变量修饰符，无法修饰方法以及代码块，被修饰的共享变量保证了不同线程对该变量操作的内存可见性且禁止指令重排序。
##  详细回答

1. 保证可见性：一个线程修改了变量值，其他线程能够立即看到修改的值。
  2. 对非volatile变量进行读写时，每个线程先从主内存拷贝变量到CPU缓存中，如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，所以每个线程可以拷贝到不同CPU cache中。
  3. volatile修饰的变量不会被缓存在寄存器或者对其他处理器不可见的地方，保证每次读写变量都从主内存中读，跳过CPU cache。所以当一个线程修改了这个变量的值，新值对于其他线程是立即得知的。
  4. 避免了多线程环境下因缓存不一致导致的读取脏数据问题。
5. 禁止指令重排：对一个volatile变量的写操作，执行在任意后续对这个volatile变量的读操作之前。
  6. 在读写操作指令前后插入内存屏障，指令重排序时不能把后面的指令重排序到内存屏障。
  7. 指令重排是JVM为了优化指令，提高程序运行效率，在不影响单线程程序执行结果的前提下，尽可能提高并行度。
  8. 指令重排序包括编译器重排序和运行时重排序。
9. 不能保证原子性：volatile不能保证复合操作的原子性,需要结合synchronized或原子类来保证原子性。
##  使用场景

- volatile适用于读写操作不依赖其当前值（如状态标志、单例模式中的延迟初始化）的场景，比synchronized更轻量级，不会引起线程上下文切换和调度。
##  代码示例

1. 状态标记变量：用volatile修饰线程的中断/停止标记，确保一个线程能及时感知到其他线程的状态变化。
  2. isRunning不加volatile修饰可能导致一直读取本地缓存的true,无法停止。
  3. isRunning加volatile修饰，修改会立即被感知，可以保证线程安全。

```
`public class VolatileDemo {
    // 用volatile修饰状态标记
    private static volatile boolean isRunning = true;

    public static void main(String[] args) throws InterruptedException {
        // 启动线程
        Thread worker = new Thread(() -> {
            while (isRunning) { // 读取标记
                // 执行任务...
            }
            System.out.println("线程已停止");
        });
        worker.start();

        // 主线程休眠后修改标记
        Thread.sleep(1000);
        isRunning = false; // 修改标记，会立即同步到主内存
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19

1. 单例模式的双重检查锁定：volatile用于禁止指令重排序，避免因初始化对象时的指令重排序导致的线程安全问题。
  2. 禁止instance = new Singleton()指令重排序，确保instance在初始化完成后才会被其他线程可见。

```
`public class Singleton {
    // 用volatile修饰单例对象
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) { // 第一次检查（避免不必要的同步）
            synchronized (Singleton.class) {
                if (instance == null) { // 第二次检查（确保线程安全）
                    // 若不加volatile，可能发生指令重排序：
                    // 1. 分配内存 -> 2. 初始化对象 -> 3. 赋值给instance
                    // 重排后可能变成 1->3->2，导致其他线程获取到未初始化的对象
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20

1. 独立观察变量：多个线程共享同一个变量，变量的更新不依赖当前值时，volatile保证读取及时性。
  2. 记录程序运行状态（加载中、运行中、已停止）

```
`public class StatusTracker {
    private volatile String status = "INIT"; // 初始状态

    // 线程A更新状态
    public void updateStatus(String newStatus) {
        status = newStatus; // 简单赋值，无依赖
    }

    // 线程B观察状态
    public String getStatus() {
        return status; // 保证读取到最新值
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13

##  知识图解

1. volatile关键字的作用示意图

 知识扩展

1. 扩展：
- JVM的内存屏障：
  1. LoadLoad屏障:在第二段数据指令被访问前保证第一大段读数据指令执行完毕。
  2. StoreStore屏障:在第二段数据指令被访问前保证第一段写数据指令执行完毕。
  3. LoadStore屏障:在第二段数据指令被访问前保证第一段读数据指令执行完毕。
  4. StoreLoad屏障:在第二段数据指令被访问前保证第一段写数据指令执行完毕。
1. 面试官可能追问:
- Q1：如果一个volatile变量是引用类型（比如对象），它能保证对象内部字段的可见性吗？
  1. 不能。当volatile修饰引用类型变量时，它只能保证的是引用本身的可见性（即引用指向的对象地址的变化能被其他线程及时感知），但无法保证该对象内部字段的可见性。
- Q2：volatile和原子类（如AtomicInteger）的适用场景有何不同？原子类能替代volatile吗？
  1. 不能简单替代，volatile保证变量的可见性和禁止指令重排序，不支持原子性；原子类则可以通过CAS保证原子性，不能保证可见性。
  2. volatile适合做状态标记和线程间通信。
  3. 原子类适合做计数器/累加器，需要原子更新的变量（如i++）。
- Q3：针对volatile变量JVM采用了什么内存屏障？
  1. 对于写操作，在写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障。
  2. 对于读操作，在读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## SpringBoot概述｜大厂面试题｜Java高频面试题｜自动配置、起步依赖、内置服务器

#  SpringBoot概述

##  简要回答

- SpringBoot是Spring框架的简化版，通过**自动配置**减少XML配置内容，**起步依赖**整合常用的技术栈如SpringMVC、MyBatis等，**内置Tomcat**实现jar包的独立运行，解决Spring配置繁琐，依赖管理复杂的问题，使开发者专注业务而非框架配置。
##  详细回答

-

SpringBoot是基于Spring的**快速开发脚手架**。它提供一种快速启动的方式，自动配置和约定优于配置的原则极大地简化了Spring应用的搭建、开发和部署过程。

-

SpringBoot使用**内嵌服务器**的方式，将Tomcat、Jetty等服务器嵌入到应用中，可以将应用程序打包成一个可执行的JAR文件，无需部署到外部容器，简化项目的部署和运行。

-

SpringBoot采用**自动配置**的机制，根据应用程序中引入的依赖和配置，SpringBoot自动配置整个应用程序的环境。

  - SpringBoot的自动配置是**基于条件的按需配置**，本质是通过注解驱动+SPI机制，根据项目依赖、环境配置、自定义规则，自动向IoC容器注入对应Bean，替代传统Spring的XML手动配置。
  - **@EnableAutoConfiguration**是实现自动装配的核心注解，该注解中 **@AutoConfigurationPackage** 注解会将主应用程序类所在包及其子包下的所有类注册到IoC容器中，@Import注解会导入**AutoConfigurationImportSelector类** ，该类实现了ImportSelector接口，可以动态选择需要导入的自动配置类：在应用程序启动时，AutoConfigurationImportSelector类会**扫描类路径**，加载META-INF/spring.factories中的自动配置类，然后对每一个发现的自动配置类使用**条件判断**，通过条件注解（如@ConditionalOnClass）筛选出符合当前环境的配置类，如果满足导入条件，则将自动配置类注册到IoC容器中。遵循 “自定义优先” 原则，开发者可通过手动配置 Bean 或禁用自动配置类，覆盖默认行为，最终实现 “按需配置、简化开发” 的目标。
-

SpringBoot提供了**快速的项目启动器**，不同的Starter将常用的技术栈的依赖整合，比如spring-boot-starter-web包含了SpringMVC、Jackson等Web开发常用的依赖，开发者只需引入一个依赖，无需手动管理版本，避免冲突。

-

SpringBoot遵循**约定优于配置**的原则，预设默认的配置和约定，开发者按照这些约定进行开发，可以大大减少配置文件的编写。

  - SpringBoot提供**特定的项目结构**，将主应用程序类置于根包，将控制器类、服务类、数据访问类等分别放在相应子包中，使开发者更易理解项目结构与组织，新成员加入项目也能快速定位各功能代码的位置，提升协作效率。
  - SpringBoot提供了大量**默认配置**，比如连接数据库、设置Web服务器、处理日志等，开发者无需配置日志级别、输出格式与位置。
  - SpringBoot的**自动化配置**也是约定大于配置的体现，通过分析项目的依赖和环境，自动配置应用程序的行为。
##  知识图解

1. Spring框架与Spring Boot的关系架构图  知识扩展

  1. 扩展
    2. SpringBoot的**生态支持**：提供spring-boot-actuator实现应用监控，进行健康检查与指标统计等；spring-boot-devtools实现热部署，开发时无需重启应用；spring-boot-test简化单元测试与集成测试，覆盖开发全生命周期。
  3. 面试官可能追问
  2. Q1：Spring和微服务的关系是什么？
    3. SpringBoot是微服务架构的基础，单个微服务节点通常是一个SpringBoot应用；微服务治理框架(Spring Cloud)是基于SpringBoot实现的，Spring Cloud可以管理多个SpringBoot应用，通过Starter整合Eureka、Nacos等组件实现服务注册与发现、配置中心、熔断降级等能力。
  4. Q2：SpringBoot的自动配置可以禁用吗？
    5. 可以禁用。在@SpringBootApplication中通过exclude属性来禁用，如@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})。还可以在配置文件中设置spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration。
  6. Q3：为什么SpringBoot的Starter能解决版本冲突？
    7. SpringBoot维护了spring-boot-dependencies版本管理器，定义了所有Starter依赖的兼容版本；Starter引入时无需引入指定版本，由SpringBoot统一管控可以避免手动引入不同版本依赖的冲突。
  8. Q4：自动配置的条件注解优先级怎么排？
    9. 条件注解按照从类到方法的层级进行执行，同一层级内的条件注解按照代码顺序执行。如果某个类/方法上有条件注解失败，则该类/方法会失效。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Error与Exception｜高频面试题｜Java高频面试题｜异常体系、Throwable、可恢复性

#  Error与Exception

##  简要回答

###  Exception 和 Error 的概念

1. **Exception (异常)** ：表示程序在运行过程中**可能遇到的、可以被捕获和处理**的异常情况。这些异常通常是由于**外部因素或程序逻辑错误**导致的，是可恢复的。
2. **Error (错误)** ：表示 **JVM 内部**或**系统级别**的严重问题，通常是致命的，应用程序无法预料和恢复。
###  Exception 和 Error 的区别

-

如下表所示：
 **维度** **Exception** **Error** **继承体系** 继承自 `java.lang.Throwable`，是 `Throwable` 的一个主要子类。 继承自 `java.lang.Throwable`，是 `Throwable` 的另一个主要子类。 **发生时机** 通常在程序运行时发生，但有些在编译时强制检查。 通常在程序运行时发生，表示JVM或系统层面的问题。 **严重性** 相对不严重，通常是可预见和可恢复的。 严重，通常是致命的，表示系统资源或JVM的故障。 **可恢复性** 大多数情况下是可恢复的，可以通过 `try-catch` 捕获处理，使程序继续运行。 通常不可恢复，即使捕获也难以有效处理，程序往往会终止。 **处理方式** 编译器强制或建议处理（`try-catch` 或 `throws`）。 通常不建议捕获和处理，应由JVM或系统层面解决。 **产生原因** 外部环境问题（如I/O错误）、程序逻辑错误、非法参数等。 系统资源耗尽、JVM内部错误、硬件故障等。

---

##  详细回答

###  Exception 和 Error 的概念

1. **Exception (异常)** ： `Exception` 类及其子类表示程序在运行过程中可能遇到的、**可以被捕获和处理**的异常情况。这些异常通常是由于**外部因素**（如文件不存在、网络中断）或**程序逻辑错误**导致的。
2. **Error (错误)** ： `Error` 类及其子类表示了在 **程序运行期间发生的**、**JVM** 或 **系统级别的** 严重问题，这些问题通常是应用程序**无法预料和恢复**的。例如，JVM 内存耗尽、JVM 错误、线程死锁等。
###  Exception 和 Error 的区别

1. **继承体系** ：
  2. **Exception**：`Exception` 类继承自 `java.lang.Throwable`，是 `Throwable` 的一个主要子类。它下面又派生出众多子类，形成一个庞大的异常体系，包括编译期异常（如 `IOException等`）和运行期异常（如 `RuntimeException及其子类`）。
  3. **Error**：`Error` 类也继承自 `java.lang.Throwable`，是 `Throwable` 的另一个主要子类。它与 `Exception` 在 `Throwable` 层面是并列关系，表示不同性质的问题。
4. **发生时机**：
  5. **Exception**：`Exception` 通常在程序运行时发生，但 **编译期异常** 在编译阶段就会被编译器强制检查。`Exception`是**程序逻辑或外部环境**在运行时可能出现的**可预见问题**。
  6. **Error**：`Error` 通常在程序运行时发生，但它们表示的是**JVM或系统层面**的问题，而非应用程序本身的逻辑问题。这些问题往往是**突发的、无法预料的**。
7. **严重性**：
  8. **Exception**：相对不严重。`Exception` 表示的是**程序可以尝试处理和恢复**的异常情况，例如文件未找到、网络连接中断等，这些问题通常不会导致整个应用程序崩溃。
  9. **Error**：严重。`Error` 表示的是**非常严重的、通常是致命的**问题，例如内存溢出（`OutOfMemoryError`）、栈溢出（`StackOverflowError`）。这些问题往往意味着JVM或底层系统已经处于崩溃边缘，程序无法继续正常运行。
10. **可恢复性**：
  11. **Exception**：大多数情况下是**可恢复**的。我们可以通过 `try-catch` 块捕获 `Exception`，并执行相应的异常处理逻辑（如重试、给出提示、记录日志等），从而使程序能够从异常中恢复并继续执行。
  12. **Error**：通常是**不可恢复**的。`Error` 表示的问题超出了应用程序的控制范围，即使捕获了 `Error`，也往往无法采取有效的措施来恢复程序的正常运行。所以，通常遇到 `Error` 意味着程序需要终止并进行系统级别的检查或重启。
13. **处理方式**：
  14. **Exception**：对于**编译期异常**，Java编译器会强制要求开发者进行处理，例如使用 `try-catch` 块进行捕获处理，或者 使用 `throws` 关键字声明抛出。对于**运行期异常**，编译器不强制处理，但开发者可以选择性地捕获和处理，以增强程序的健壮性。
  15. **Error**：通常不建议在应用程序层面捕获和处理 `Error`。因为它们代表的是系统级别的故障，捕获它们并试图恢复往往是徒劳的，甚至可能掩盖更深层次的问题。当 `Error` 发生时，通常最好的做法是**让程序终止**，并由运维人员或系统管理员介入解决。
16. **产生原因**：
  17. **Exception**：主要是**外部环境问题** （如文件不存在、网络连接超时、数据库连接失败等），也有可能是程序逻辑错误（如空指针引用、数组越界、类型转换失败、非法参数传递等）。
  18. **Error**：产生原因通常与**JVM或系统资源**有关，例如内存不足（`OutOfMemoryError`）、栈空间耗尽（`StackOverflowError`）等，也可能是JVM自身的bug或者出现了损坏，甚至还有可能是硬件的故障间接导致JVM无法正常运行。

---

##  知识拓展

1.

**Exception 和 Error的对比 示意图如下**：
   →

### 评论
 

验证登录状态...

## 线程start与run｜高频面试题｜Java高频面试题｜多线程、并发、线程状态

#  线程start与run

##  简要回答

- run() 方法不会创建一个新的线程，只会在当前线程中执行run方法的代码。
- start() 方法可以启动一个新线程，能够实现多线程执行。
##  详细回答


在Java多线程中：

1. run方法
- 是**线程的执行体**，包含线程要执行的**代码**。
- 当**直接调用**run方法时，代码直接在当前线程中作为**普通方法**执行，不会创建新的线程。
- run方法也**可以被子类重写**，实现特定任务。
1. start方法
- 会**启动一个新的线程**，会在新线程中执行run方法代码。
- start方法会为线程分配系统资源，并将线程置于**就绪状态**，当**调度器选择该线程**时，会**执行run方法**中的代码。
- start方法**只能调用一次**，如果再次调用，将抛出IllegalThreadStateExeception异常。 方法 创建线程 执行run 多线程 run() 不创建线程 立即执行 没有多线程 start() 创建新的线程 创建线程后 多线程，允许并行
##  使用场景

- start()：适用于**任务需要与主线程或其他线程并行执行**时，例如后台下载、异步处理数据。**利用多线程来提高效率**，实现并行处理。
- run()：适用于需要重写来**实现线程任务**，或者需要**在当前线程同步执行**的场景。
- 使用时注意，**直接调用run()方法无法实现多线程**，**一个线程对象只能调用一次start()** 。
##  知识图解

1. **run()与start()区别示意**  知识扩展

  1. 扩展：
  2. 线程的状态：
    3. 线程的状态能够反映线程在生命周期的不同阶段，由JVM和操作系统共同管理，状态转换一般就是“就绪-运行-阻塞/等待-就绪”的循环，直到线程终止。
    1. NEW：尚未启动线程状态。线程创建，**还没有调用start方法**，未分配系统资源。
    2. RUNNABLE： **就绪状态** 已经获取除了CPU外全部资源，**等待操作系统调度**分配CPU时间片。 **正在运行状态** 线程获得时间片，**执行run()** 方法。两个状态合并称为Runnable。
    3. BLOCKED：阻塞状态。竞争同步锁失败，**等待锁释放**。
    4. WAITING：等待状态的线程正在等待另一线程执行特定的操作。
    5. TIMED_WAITING：线程在**指定时间内等待**，超时后**自动唤醒**。
    6. TERMINATED：线程完成执行，终止状态。
  1. 面试官可能追问:
  4. Q1：sleep和wait的区别是什么？
    1. sleep是Thread类的静态方法，可以在任何地方**通过Thread.sleep()调用**，无需依赖对象实例。wait是Object类的实例方法，必须**通过对象实例调用**。
    2. sleep()在调用时，线程会暂停执行指定的时间，**不会释放持有的对象锁**。wait()在调用时**会释放持有的对象锁**，进入**等待状态**。
    3. sleep可以在**任意位置调用**，无需事先获取锁。wait必须在同步块或同步方法内调用，线程**需持有该对象的锁**，否则会抛出IlleaglMonitorStateException。
    4. sleep**休眠时间结束后，线程自动恢复**到就绪状态。等待CPU调度。wait需要其他线程**调用相同对象的notify()或notifyAll()** 方法才能被唤醒。
  5. Q2：为什么不能调用多次start()方法？
    6. 线程对象的生命周期只允许调用一次start()
    7. 首次调用会将线程状态**从New转为Runnable**，同时触发操作系统创建新线程，若再次调用，**线程状态已经不是New，JVM会抛出IllegalThreadStateException**。
  8. Q3：多个线程调用start()，执行顺序由什么决定？
    9. 由操作系统的**线程调度算法**决定，与start()调用顺序无关。
  10. Q4：线程从New到Runnable的过程中，JVM会做什么？
    11. 调用start()后，JVM会**向操作系统申请**创建线程，分配栈空间、程序计数器等。随后操作系统将线程**加入调度队列**，此时状态变为Runnable。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 策略模式｜高频面试题｜设计模式高频面试题｜行为型设计模式、算法封装、if

#  策略模式

##  简要回答

- 策略模式(Strategy Pattern)是行为型设计模式，它定义了一系列算法，将每个算法都封装起来，使它们可以互相替换。策略模式让算法的变化独立于使用算法的客户，也就是说这些算法完成的功能类型和对外接口是一样的，只是不同的策略引起环境角色表现出不同的行为。
- 优点：使用策略模式可以替代使用if...else或者switch...case语句，降低代码编写的复杂度，提高代码的可维护性。
- 缺点：需要定义大量的策略类，且每个策略类都要提供给客户端，只适用于客户端知道所有策略类的情况。
##  详细回答

- 策略模式主要由上下文、策略接口和具体策略类组成。**Context类**是用来操作策略的上下文环境类，持有抽象策略的引用，通过多态传进来不同的具体策略来调用不同策略的方法；**策略接口**定义了算法的接口或抽象类，声明所有具体策略必须实现的方法；具体的**策略类**实现了这个接口，实现具体算法的封装；
- 实现策略模式的步骤：
  1. 定义一个策略接口，声明算法的抽象方法。
  2. 创建具体的策略类，实现策略接口并封装具体的算法。
  3. 创建环境类，包含对策略接口的引用以及一个用于设置具体对象的方法。
  4. 在客户端中创建环境类的对象，并调用其方法来执行具体的算法。
##  代码示例

1. 下面是一个简单的策略模式的示例，假设有一个商场销售系统，根据不同的促销策略计算最终价格：

```
`// 策略接口
public interface PricingStrategy {
    double calculatePrice(double price);
}

// 具体策略类：无折扣
public class NoDiscountStrategy implements PricingStrategy {
    @Override
    public double calculatePrice(double price) {
        return price;
    }
}

// 具体策略类：打九折
public class DiscountStrategy implements PricingStrategy {
    @Override
    public double calculatePrice(double price) {
        return price * 0.9;
    }
}

// 具体策略类：满减
public class CashbackStrategy implements PricingStrategy {
    @Override
    public double calculatePrice(double price) {
        if (price >= 200) {
            return price - 50;
        } else {
            return price;
        }
    }
}

// 环境类
public class ShoppingCart {
    private PricingStrategy pricingStrategy;

    public void setPricingStrategy(PricingStrategy pricingStrategy) {
        this.pricingStrategy = pricingStrategy;
    }

    public double checkout(double totalPrice) {
        return pricingStrategy.calculatePrice(totalPrice);
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();

        // 选择无折扣策略
        cart.setPricingStrategy(new NoDiscountStrategy());
        double price1 = cart.checkout(100);
        System.out.println("Total Price: " + price1); // 输出：Total Price: 100.0

        // 选择打九折策略
        cart.setPricingStrategy(new DiscountStrategy());
        double price2 = cart.checkout(100);
        System.out.println("Total Price: " + price2); // 输出：Total Price: 90.0

        // 选择满减策略
        cart.setPricingStrategy(new CashbackStrategy());
        double price3 = cart.checkout(200);
        System.out.println("Total Price: " + price3); // 输出：Total Price: 150.0
    }
}

`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68

##  知识图解

1. 策略模式结构图  使用场景

  1. 当需要动态切换算法时，可以使用策略模式。
  2. 如果代码中有多个条件语句，存在复杂条件判断时，可以使用策略模式简化代码。
  3. 使用策略模式可以避免与算法相关的数据结构暴露，只需使用抽象接口调用，无需知道实现细节。
  4. 如果有许多相关的类仅仅是行为不同，那么可以使用策略模式。
##  知识扩展

  1. 面试官可能追问：
  2. Q1：如何避免策略类过多的问题？
    3. 解决策略模式策略类过多的问题，核心有三层优化思路，首先最常用的是结合**简单工厂模式**，把所有策略对象的创建逻辑统一封装到策略工厂中，由工厂根据业务标识匹配并返回对应策略对象，客户端只需传入标识即可使用，无需感知所有策略类；其次进阶优化是在工厂中用**Map容器做策略缓存**，把策略标识和策略实例做键值对映射，项目启动时初始化存入 Map，获取策略时直接 O(1) 查询，彻底去掉 if-else/switch 分支判断，同时满足开闭原则；最后从根源上避免类膨胀，是对策略做**复用与参数化抽离**，很多策略类只是参数不同、核心逻辑一致，比如不同满减规则、不同计费标准，这种场景无需新建类，只需把可变参数抽离出来，通过构造器或方法传入策略类，一个策略类即可适配多套规则，从根本上减少策略类的数量。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Bean作用域｜高频面试题｜Java高频面试题｜Singleton、Prototype、Request、Session

#  Bean作用域

##  简要回答

- Spring Bean有6种作用域，核心常用前2种：**Singleton（默认单例，容器内唯一实例）、Prototype（多例，每次请求新建实例）**；Web 场景专属 4 种：**Request**（单次 HTTP 请求内有效）、**Session**（会话内有效）、**Application**（ServletContext之内有效）、**Websocket**（WebSocket 会话内有效），作用域决定 Bean 的生命周期和可见范围。
##  详细回答

- 在Spring中，**Bean**是组成应用程序的主体，由Spring IOC容器所管理的对象。**Bean的作用域**则定义了在应用程序中创建的Bean实例的生命周期和可见范围。Spring容器根据作用域管理Bean的实例，包括它们的创建、销毁，以及是否可以被共享。
1. **Singleton**（单例）：是默认的作用域，当一个Bean的作用域是Singleton时，Spring IOC**容器只存在一个共享的Bean实例**，并且对所有的Bean请求，只要id与该bean定义匹配，都会返回bean的同一实例。
2. **Prototype**（原型）：每次请求都会创建一个新的Bean实例。当Bean的作用域为prototype时，每次对该Bean请求都会得到一个新的Bean实例，适用于瞬时的对象。
3. **Request**（请求）：**每次HTTP请求都会产生一个新的Bean实例**，该Bean只在当前HTTP请求中有效。Request只适用于Web程序，请求结束后该对象的生命周期即结束。
4. **Session**（会话）：**每一次HTTP请求都会产生一个新的Bean实例**，该bean仅在当前HTTP Session内有效，Session同样只适用于Web程序。可以根据需求更改实例的内部状态，其他session无法看到特定session的状态变化。在该HTTP Session作用域内的bean随着session的结束而销毁。
5. **Application**：当前的**ServletContext中只存在一个Bean实例**，仅在Spring Web应用程序中有效，该Bean实例在整个ServletContext中共享。Application作用域只在支持Web的Spring ApplicationContext中有效。
6. **Websocket**：每次WebSocket会话会产生一个新的bean，该实例在WebSocket会话范围内共享。
7. **Global Session**（全局会话）：类似Session作用域，仅在基于集群的Web应用程序中有意义，指定了在整个集群中共享的Bean实例。Portlet规范了全局Session的概念，它**被所有构成某个portlet的web应用的不同portlet所共享**，在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。
8. **Custom scopes**（自定义作用域）：Spring允许开发者自定义作用域，可以通过实现Scope接口来创建新的Bean作用域。
##  知识图解

- 单例作用域和原型作用域的区别示意图  代码示例


```
`//1. 在Spring配置文件中通过<bean>标签的scope属性指定Bean的作用域
<bean id="myBean" class="com.example.MyBean" scope="singleton"/>

//2. 使用@Scope注解指定Bean的作用域
@Bean
@Scope("prototype")
public class MyBean {
    return new MyBean();
}
`
```
 1
2
3
4
5
6
7
8
9

##  知识扩展

  1. 面试官可能追问
  - Q1：Singleton单例Bean是线程安全的吗？为什么？如何解决？
    - **默认不安全**，因为**单例Bean是全局共享的**，若存在可修改的成员变量，多线程并发修改会导致数据错乱；应该避免Bean存储可变状态，可以用局部变量替代成员变量、使用ThreadLocal隔离线程数据、加锁（synchronized/ReentrantLock）或采用无状态设计。
  - Q2：Singleton和Prototype的Bean生命周期有什么区别？容器会管理Prototype Bean的销毁吗？
    - **Singleton Bean生命周期与容器一致**（容器启动创建，容器关闭销毁），容器**全程管理**；**Prototype Bean**每次请求创建新实例后，容器仅负责实例化和依赖注入，**创建后不再管理**，销毁由 JVM 垃圾回收，也可以手动释放资源，如在@PreDestroy中处理，但容器不会主动调用）。
  - Q3：如何在Singleton Bean中注入Prototype Bean？直接用@Autowired会有什么问题？
    - 直接注入会导致Prototype Bean变成 “**伪多例**”。当Singleton Bean初始化时只注入一次Prototype实例，后续每次使用都是同一个实例。可以用@Lookup注解，即Spring 动态生成代理，每次调用返回新实例、还可以通过ApplicationContext手动获取（context.getBean(PrototypeBean.class)）或者使用 Scoped Proxy（作用域代理）。
  - Q4：如何自定义 Bean 的作用域？
    - 先实现Scope接口：重写get（获取 Bean）、remove（移除 Bean）和registerDestructionCallback（注册销毁回调）等方法；再通过CustomScopeConfigurer将自定义 Scope 注册到 Spring 容器；在 Bean 上用@Scope("自定义作用域名称")标注使用。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 面向对象三大特性｜高频面试题｜Java高频面试题｜封装、继承、多态、组合优于继承

#  面向对象三大特性

##  简要回答

1. **封装（Encapsulation）**：
  2. **概念**：是指将**数据**（属性）和**操作数据的方法**（行为）**捆绑**在一起形成一个**独立的单元**（类/对象），并**隐藏其内部实现细节**，而是仅通过公共方法对外提供访问接口。
  3. **好处**：数据被保护在封装体内部，对外隐藏实现细节。
4. **继承（Inheritance）**：
  5. **概念**：是指让类与类之间产生**父子关系**。它允许一个类（子类）从另一个已存在的类（父类）中继承属性和方法，从而实现代码的复用，并建立类之间的层级关系。
  6. **好处**：提高代码复用性，类与类之间建立“is-a”的关系。
7. **多态（Polymorphism）**：
  8. **概念**：是指允许**父类引用指向子类对象**，并在运行时根据对象的实际类型调用相应的方法。它使得“一个接口，多种实现”成为现实。
  9. **好处**：提高代码的灵活性和可扩展性。

---

##  详细回答

###  1. 封装（Encapsulation）

- **定义**：
  - 封装是指将对象的**数据**（属性）和**操作数据的方法**（行为）**捆绑**在一起形成一个**独立的单元**（类/对象）。
  - 同时，它通过控制访问权限来 **隐藏对象的内部实现细节**，而只对外提供**有限的、受控的**公共接口进行交互。
- **JavaBean**：JavaBean 是封装的典型应用，符合JavaBean标准的类，必须是具体，公共的；并且不但具有空参构造，通常也需要写出它的带参构造；成员变量全部用 private关键字 修饰，并且要提供用来操作这些成员变量的 setter和getter方法。
- **关键实现**：
  - **访问修饰符**：主要通过 `private` 关键字来实现数据和内部方法的隐藏。`public` 关键字则用于暴露对外接口。
  - **`this` 关键字**：在类的方法内部，`this` 关键字用于引用当前对象的实例，常用于解决“成员变量和形参重名”的问题，以及在构造器中调用其他构造器。
  - **与抽象的关系**：封装与抽象紧密相关，**抽象**是识别事物的共同特征和行为的过程，而封装则将这些抽象出来的特征和行为及其实现细节捆绑并隐藏起了。
- **优点**：
  - **数据安全性高**：数据被保护在封装体内部，外部不能随意访问和修改，保证了数据完整性。
  - **降低耦合度**：只要接口不变，类内部实现的变化就不会影响外部调用者。
  - **提高代码复用性**：封装好的组件可以更容易地在不同项目中复用。
###  2. 继承（Inheritance）

- **定义**：
  - 继承是面向对象中实现 **代码复用** 和 建立**类之间层级关系（“is-a”关系）** 的机制。它允许一个类（**子类/派生类**）从另一个已存在的类（**父类/基类/超类**）中继承其属性和方法。
  - 子类可以在继承的基础上，添加新的属性和方法，或者重写（Override）父类的方法，以实现特有的行为。
  - 子类不能直接访问父类的私有（`private`）成员，但可以通过父类提供的 `public` 或 `protected` 方法间接访问。
- **特点**：
  - **单继承**：Java 类只支持单继承，即**一个类只能直接继承一个父类**。
  - **多层继承**：但是 Java 支持多层继承，即一个子类可以有自己的子类。
  - **传递性**：子类不但可以继承直接父类的非私有成员，还可以继承直接父类的父类的非私有成员。
  - **构造器不被继承**：子类不继承父类的构造器，但子类构造器在执行前会**隐式或显式地**调用父类的构造器。
- **关键实现**：
  - **`extends` 关键字**：Java中如果使用**继承**，需要用到 **extends关键字**，在定义子类时，子类类名后跟上“extends + 要继承的父类类名”，即可。
  - **`super` 关键字**：用于在子类中访问父类的成员（属性和方法），特别是调用父类的构造器（如`super()`）来完成属性的初始化。
  - **方法重写（Override）**：子类对父类中已有的方法进行重新实现，以适应子类的特定行为。方法重写是实现多态的基础。
- **优点**：
  - **提高可维护性**：修改父类中的公共逻辑，可以影响所有子类。
  - **模拟现实世界**：继承关系可以体现现实世界中的层级结构 和 “is-a”的关系。
- **缺点**：
  - **高耦合性**：子类与父类之间形成强耦合关系，不符合“**低耦合，高内聚**”的程序设计要求。并且子类暴露了父类的实现细节（“白盒复用”）。
  - **灵活性限制**：Java 的单继承机制限制了类的功能扩展性。
###  3. 多态（Polymorphism）

- **定义**：
  - 是指“**一个接口，多种实现**”。在 Java 中，多态主要体现在——允许**父类引用指向子类对象**，并在运行时根据对象的实际类型调用相应的方法，即动态绑定机制。
- **特点**：
  - **运行时绑定（动态绑定）**：这是多态的核心机制，编译器在编译时只知道引用变量的类型（静态类型），但在程序运行时，JVM 会根据对象的实际类型（动态类型）来查找并调用相应的方法。
  - **提高可扩展性**：增加新的子类时，无需修改现有代码，只需让新子类重写父类方法。
- **实现多态的三个必要条件**：
  1. **继承或实现关系**：必须存在子父类继承关系或接口实现关系。
  2. **方法重写（Override）**：子类必须重写父类的方法（或实现接口方法）。
  3. **父类引用指向子类对象**。
- **优点**：
  - **可维护性高**：修改具体实现不影响调用方。
  - **可扩展性强**：新增子类只需实现父类接口或重写方法，无需修改调用方代码。
- **缺点**：
  - **父类引用不能直接使用子类的特有成员**：这是多态的一个限制。例如，`Animal animal = new Dog();`，`animal` 引用对象无法直接调用 `Dog` 类特有的方法，除非进行强制类型转换。

---

##  知识拓展

1. **三大特性——封装**，示意图如下：
  →

### 评论
 

验证登录状态...

## SpringBoot常用注解｜大厂面试题｜Java高频面试题｜SpringBoot、自动配置、条件装配、全局异常

#  SpringBoot常用注解

##  简要回答

- SpringBoot的注解以Spring的注解为基础，做了轻量化、自动化和场景化的改进，在Spring注解的基础上做了封装与扩展，实现**约定大于配置**，减少开发者的配置工作。
- 自动配置核心注解 **@SpringBootApplication** ，整合了@SpringBootConfiguration、@EnableAutoConfiguration和@ComponentScan。
- 配置读取可使用 **@ConfigurationProperties** 绑定配置文件属性， **@Value** 读取单个配置值。
- Web开发在Spring注解基础上扩展了 **@GetMapping/@PostMapping** 等，新增@RestControllerAdvice处理全局异常。
- **条件装配**可通过@ConditionalOnClass、@ConditionalOnMissingBean等控制Bean的创建时机。
- 启动相关有 **@SpringBootApplication** 标记启动类， **@MapperScan** 扫描MyBatis映射接口。
##  详细回答

- @**SpringBootApplication**：SpringBoot是应用的核心启动注解，是@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan 三个注解的组合：
  - **@SpringBootConfiguration**：本质是 @Configuration，标记当前类为配置类；
  - **@EnableAutoConfiguration**：开启自动配置，SpringBoot 会根据类路径下的依赖自动配置 Bean（如引入 spring-boot-starter-web 则自动配置 Tomcat、DispatcherServlet）；
  - **@ComponentScan**：默认扫描当前类所在包及子包下的 @Component、@Service 等注解组件。
- **@ConfigurationProperties**：能够批量绑定配置文件中的属性到Bean中，支持前缀指定，例如@ConfigurationProperties(prefix = "app.datasource")可绑定app.datasource.url等配置到类的成员变量。
- **@Value**：可以从属性文件或配置中读取单个配置值并注入到成员变量，例如@Value("${server.port:8080}")读取端口配置，默认值 8080。
- **@RestControllerAdvice**：负责全局异常处理、数据绑定、响应增强的注解，结合 @ExceptionHandler 可统一处理 Controller 层异常，替代传统的 @ControllerAdvice+@ResponseBody 组合。
- **@ExceptionHandler**：在 @RestControllerAdvice 标注的类中使用，指定处理特定类型的异常，例如使用@ExceptionHandler(NullPointerException.class)处理空指针异常。
- **@ConditionalOnClass**：条件装配注解，当类路径下存在指定类时，才加载当前配置类Bean；反之 @ConditionalOnMissingClass 则是不存在指定类时生效。
- **@ConditionalOnBean**：当Spring容器中存在指定Bean时，才创建当前Bean；@ConditionalOnMissingBean 则是容器中不存在指定Bean时生效，常用于自定义Bean覆盖默认配置。
- **@MapperScan**：指定MyBatis映射接口的扫描包路径，替代在每个Mapper接口上标注@Mapper，例如@MapperScan("com.example.mapper")，让容器识别接口并创建代理实现类。
- **@EnableTransactionManagement**：开启Spring的事务管理功能，结合 @Transactional 注解实现声明式事务（SpringBoot 中引入 spring-boot-starter-jdbc/orm后会自动开启，无需手动标注）。
- **@Async**：能够标记方法为异步执行，需配合@EnableAsync 注解开启异步功能，Spring会为异步方法创建独立线程执行。
- **@EnableCaching**：开启缓存功能，结合@Cacheable、@CachePut、@CacheEvict等注解实现数据缓存。
- **@PathVariable**：从URL路径中提取参数，例如@GetMapping("/user/{id}")中通过@PathVariable("id") Long id获取路径中的 id 值。
- **@RequestParam**：获取请求参数（URL 拼接或表单提交的参数），支持指定参数名、是否必传、默认值，例如@RequestParam(value = "name", required = false, defaultValue = "guest") String name。
##  知识图解

1. Spring框架与Spring Boot的关系  知识扩展

  1. 面试官可能追问：
  2. @SpringBootApplication的扫描范围是什么？可以修改吗？
    3. **@SpringBootApplication注解默认扫描当前包及其所有子包下的组件**。
    4. 可以通过@ComponentScan注解或者使用@SpringBootApplication的scanBasePackages属性指定扫描包。@ComponentScan的excludeFilters属性可以排除特定组件。
  5. SpringBoot的@Transactional注解什么时候会失效？
    6. 如果事务方法被private/static/final修饰，无法被AOP代理时，事务方法将不会被事务管理。
    7. 同类中非事务方法调用事务方法，注解无效。
    8. 如果异常被catch捕获但没有抛出或者异常类型不是运行时异常时，事务方法将失效。
  9. @Async注解的方法为什么不能是private或者static？
    10. **@Async注解基于Spring AOP实现**，而AOP无法代理private/static方法。
  11. @RestControllerAdvice和@ControllerAdvice的区别是什么？
    12. **@RestControllerAdvice是@ControllerAdvice和@ResponseBody的组合**，内部的@ExceptionHandler方法返回的数据会以json格式返回给前端。而@ControllerAdvice返回的是视图页面，手动添加@ResponseBody注解后才能返回json数据。
  13. 怎么通过@RestControllerAdvice实现全局异常处理？
    14. 先定义异常枚举的错误码和错误信息，然后编写统一返回结果类(code/msg/data)，在@RestControllerAdvice类中使用@ExceptionHandler标注方法，捕获不同类型异常，封装为统一的返回结果返回。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 代理模式｜高频面试题｜Java高频面试题｜静态代理、动态代理、JDK、CGLIB

#  代理模式

##  简要回答

- 代理模式(Proxy Pattern)是一种结构型设计模式，其主要目的是在访问某个对象时引入一种代理对象，通过代理对象控制对原始对象的访问。代理模式可以用于实现懒加载、控制访问、监控对象等场景。
- 优点：代理模式不修改目标对象的原有代码，通过代理对象实现在核心方法前后添加额外的业务逻辑，符合设计模式的开闭原则和单一职责原则。
- 缺点：静态代理会增加类的数量，一个真实对象对应一个代理对象，业务复杂时代理类数量会很多。
##  详细回答

- 代理模式可以由抽象主题(Subject)、具体主题(RealSubject)、代理(Proxy)三部分组成。**抽象主题**定义了代理对象和真实对象的共同接口，使得代理对象能够替代真实对象，客户端只通过该接口与对象交互；**真实主题**则是实际执行业务逻辑的对象，是代理模式中被代理的对象，实现抽象主题的接口，关注核心业务代码编写；而**代理**持有一个指向真实主题的引用，也会实现抽象主题的接口，负责执行增强逻辑，可以控制对真实主题的访问，并在需要时负责创建或删除真实主题的实例。
- 代理模式的**执行流程**如下：首先由客户端发起业务方法调用，调用代理对象的对应方法；代理对象会执行权限校验、日志打印等前置增强逻辑，然后通过持有的引用调用真实对象的核心业务方法，将执行结果返回给代理对象后，代理对象执行后置增强逻辑，将最终结果返回给客户端，使用代理模式时客户端只和代理对象交互。
- 代理模式可以分为静态代理和动态代理两类，**静态代理**在编译期就编写好了代理类的代码，代理类与真实类一一对应，编译完成后代理类就是一个固定的class文件。实现简单，适合业务比较固定的场景。如果抽象主题新增方法，真实类和代理类都需要同步修改，扩展性差。**动态代理**则是在运行期由JVM通过反射机制动态生成代理类的字节码，无需手动编写代理类代码，一个动态代理类可以为任意多个真实类提供代理服务，能够解决静态代理的扩展性问题。Java中有基于接口的JDK动态代理和基于继承实现的CGLIB动态代理。
- 代理模式有四种常见的应用类型：
  1. **远程代理**(Remote Proxy)会对请求及其参数进行编码，并向不同地址空间中的实体发送已编码的请求，为远程对象提供本地访问。
  2. **虚拟代理**(Virtual Proxy)可以缓存实体的附加信息以便延迟对它的访问，当创建开销很大的对象时使用。
  3. **保护代理**(Protection Proxy)控制对原始对象的访问，检查调用者是否具有实现一个请求所必需的访问权限。
  4. **智能引用**(Smart Reference)是调用真实的对象时，执行附加操作。
##  知识图解

1. 代理模式结构图  示例代码


```
`// 1. 抽象主题：统一业务接口
interface Service {
    void doWork();
}

// 2. 真实主题：只负责核心业务
class RealService implements Service {
    @Override
    public void doWork() {
        System.out.println("执行真实核心业务");
    }
}

// 3. 代理主题：代理+增强逻辑，核心！
class ProxyService implements Service {
    private Service realService;
    // 传入真实对象
    public ProxyService(Service realService) {
        this.realService = realService;
    }

    @Override
    public void doWork() {
        // 前置增强
        System.out.println("代理前置：权限校验/日志记录/懒加载...");
        // 调用真实业务
        realService.doWork();
        // 后置增强
        System.out.println("代理后置：缓存结果/资源释放/计数统计...");
    }
}

// 4. 客户端调用
public class Client {
    public static void main(String[] args) {
        Service real = new RealService();
        Service proxy = new ProxyService(real);
        proxy.doWork(); // 只调用代理
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40

##  知识扩展

  1. 面试官可能追问：
  2. Q1：代理模式和装饰者模式有什么区别？
    3. 代理模式和装饰者模式的设计目的与增强方式不同。
    4. 代理模式的核心是控制访问，代理对象会决定是否调用真实对象方法，真实对象的核心业务逻辑不变，增强逻辑则是为了辅助核心业务，如权限校验、日志记录、缓存等。
    5. 装饰者模式的核心是增加功能，装饰者对象会对真实对象的核心功能进行扩展，丰富核心业务。
  6. Q2：静态代理和动态代理有什么区别？为什么实际开发选择动态代理？
    7. 静态代理和动态代理的核心区别是代理类的创建时机和扩展性不同，静态代理在编译期创建，扩展性差；动态代理则是运行期反射生成，扩展性极强。在实际开发中业务会频繁迭代，新增方法，使用动态代理在新增真实对象时无需修改任何代码，能大幅降低维护成本。
  8. Q3：你在项目中使用过代理模式吗？
    9. 使用过。我在项目中实现接口的统一日志和权限校验时，使用动态代理为所有业务生成代理对象，在调用核心方法前会先对用户token、接口访问权限等进行判断。这样所有的业务类只需要写核心业务代码，无需关心日志与权限，代码整洁且可扩展性强。
  10. Q4：Spring中哪里使用了代理模式？
    11. Spring的声明式事务是基于代理模式实现的，在业务方法上添加@Transactional注解，Spring容器会为这个类生成一个代理对象，代理对象会拦截对这个类的方法调用，并在调用之前执行事务管理逻辑，帮助我们完成事务的开启、提交和回滚。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## HashMap实现原理｜大厂面试题｜Java高频面试题｜底层结构、哈希冲突、红黑树

#  HashMap实现原理

##  简要回答

- 在 **JDK 1.8** 之前，HashMap的底层是一个 **数组** 结构，在 **JDK 1.8** 后，为了优化哈希冲突的性能，演变成了 “**数组 + 链表 / 红黑树**” 的复合结构。
- 当我们 **`put`** 一个键值对时，会先用 **Key的哈希值** 计算出它在**数组中的位置**（也叫**桶**）。
- 如果这个位置是空的，就直接放入。如果这个位置已经有数据了（哈希冲突），在 JDK 1.7 及以前会以 **链表** 形式存储。从JDK 1.8开始，当这个链表的长度超过一个阈值（默认为8）时，它就会被转换成**红黑树**，以此将严重哈希冲突下的查询时间复杂度从 O(n) 优化到 O(log n)。

---

##  详细回答

###  JDK 1.7 及之前：数组 + 链表

1. **底层结构**：
  2. 在 HashMap 内部，有一个 `Entry[]` 类型的数组，名为`table`。`Entry`是 HashMap 的一个内部类，它包含了`key`, `value`, `hash`以及一个指向下一个`Entry`的`next`指针。
3. **`put`流程**：
  4. 当执行`put(key, value)`时，首先计算`key`的**哈希值**。
  5. 然后通过这个**哈希值**与数组长度进行运算，定位到`table`数组中的一个索引位置。
  6. 如果该位置没有元素，就创建一个新的`Entry`对象放进去。如果该位置已经有元素了（即发生**哈希冲突**），就在这个位置形成一个链表。新加入的元素会采用 “**头插法**” 插入到链表的头部，这是因为设计者认为后插入的元素更可能被先访问。
7. **缺点**：
  8. 这种结构的主要问题是，一旦哈希函数设计不佳或者数据本身就容易产生大量冲突，就会导致某个桶后面的链表变得非常长。在这种极端情况下，`get()`操作需要遍历整个长链表，其时间复杂度会从理想的O(1)退化到**O(n)**，导致性能急剧下降。
###  JDK 1.8 及之后：数组 + 链表 / 红黑树

1. **底层结构**：
  2. HashMap 的**底层数据结构**变成了 **数组 + 链表 或 红黑树** 。数组的类型从`Entry[]`变成了 **`Node[]`** ，这个Node类型是HashMap的内部类，它又实现了Map接口的内部接口Entry，`Node`是`Entry`的替代者。`Node`还有一个子类`TreeNode`，用于表示红黑树的节点。
3. **`put`流程**：
  4. `put`的大体流程与之前版本的JDK源码相比不变，依然是**先计算哈希，再定位数组桶**。
  5. 当发生哈希冲突时，如果该桶内元素的组织形式是**链表**，那么新元素会采用 “**尾插法**” 加入到链表的末尾。
  6. 在插入链表后，会检查该链表的长度。如果长度**大于等于树化阈值（TREEIFY_THRESHOLD，默认为8）**，HashMap并不会立即树化，它还会再检查当前数组的长度，如果数组长度小于一个**最小树化容量（MIN_TREEIFY_CAPACITY，默认为64）**，它会优先选择扩容而不是树化。但如果数组长度足够，这条链表就会被重构为一棵**红黑树**。
  7. 如果该桶内已经是一棵红黑树了，那么新元素会按照红黑树的规则插入，时间复杂度为**O(log n)**。
8. **优势**：
  9. 通过引入红黑树，即便在最坏的情况下（即所有key都映射到同一个桶），查询一个元素的时间复杂度也从 O(n) 降低到了 **O(log n)** ，极大地提升了HashMap在恶劣情况下的性能稳定性。

---

##  知识图解

1. **HashMap的底层数据结构**，示意图如下：
  知识拓展

  1. **面试官可能的追问1：你提到了`hashCode`和`equals`方法，能谈谈它们在HashMap中的作用和它们之间的关系吗？**
    2. **`hashCode()` 的作用**：它是性能的关键。HashMap用它来快速计算出`key`应该被放入哪个桶，目的是将元素尽可能地散列开，减少冲突。它决定了“**去哪里找**”。
    3. **`equals()` 的作用**：它是正确性的保证。当`hashCode()`定位到某个桶后，如果桶内有多个元素（即当前通的结构是链表或红黑树），就需要用`equals()`方法去逐一比较，以确定哪个才是我们真正要找的`key`。它解决了“**是不是**”的问题。
    4. **两者关系**：有这么一个约定——如果两个对象通过`equals()`方法比较为`true`，那么它们的`hashCode()`值必须相等。反之则不要求。如果只重写`equals()`而不重写`hashCode()`，就会导致两个我们认为相等的对象，在HashMap中被定位到不同的桶，从而发生“存得进去，取不出来”的诡异情况。
  5. **面试官可能的追问2：为什么选择在链表长度为8的时候进行树化？这个数字有什么讲究吗？**
    6. 这是一个在**时间**和**空间**之间权衡的结果，在HashMap的源码注释中有详细说明。
    7. **理想情况**：在一个设计良好的哈希函数下，桶中元素的分布遵循**泊松分布**。计算可知，一个桶中出现8个元素的概率已经小于千万分之一，是极小概率事件。所以再大部分情况下都不会触发树化。
    8. **成本权衡**：红黑树节点的体积（`TreeNode`）大约是普通链表节点（`Node`）的两倍，所以从空间上看，我们不希望过早树化。从时间上看，当链表较短时（比如长度为6或7），其遍历成本和红黑树的查找成本相差无几，甚至可能更快。
    9. 所以选择8作为阈值，是在“极小概率发生”和“发生后查询性能（O(log 8)）显著优于链表（O(8)）”之间的一个折中点，同时考虑了空间和时间成本。

---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 线程创建方式｜大厂面试题｜Java高频面试题｜Thread、Runnable、Callable、线程池

#  线程创建方式

##  简要回答


在Java中,创建线程有四种方式:

- 继承Thread类,重写run()方法。
- 实现Runnable接口，实现run()方法。
- 使用线程池(Executor框架)。
- 实现Callable接口（配合Future/FutureTask），实现call()方法。
##  详细回答

1. **继承Thread类**：
- 用户可以通过继承Thread类创建一个新的线程，并且重写run()方法，该方法包含线程的执行代码。
- 创建该类的实例，调用start()方法启动线程。

```
`class MyThread extends Thread{
  @Override
  public void run(){
    // 线程执行的逻辑代码
    System.out.println("MyThread is running...");
  }
}

//创建并启动线程
MyThread myThread = new MyThread();
myThread.start();
`
```
 1
2
3
4
5
6
7
8
9
10
11

- 优点：代码简单，如果需要访问当前线程，无需使用Thread.currentThread()方法，直接使用this即可获取当前线程。
- 缺点：Java只支持单继承，线程类不能再继承其他的父类
1. **实现Runnable接口：**
- 用户可以实现Runnable接口，实现run()方法。
- 创建一个Thread对象，将实现类Runnable接口的类的实例作为参数传递给Thread构造函数，调用Thread对象的start()方法启动线程。

```
`class MyRunnable implements Runnable{
  @Override
  public void run(){
    //线程执行的逻辑代码
    System.out.println("MyRunnable is running...");
  }
}

// 创建并启动线程
Thread myThread = new Thread(new Runnable());
myThread.start();
`
```
 1
2
3
4
5
6
7
8
9
10
11

- 优点：可以实现多个线程共享同一个目标对象，方便实现资源的并发控制，且线程的创建和业务逻辑分离。
- 缺点：如果需要访问当前线程，需要使用Thread.currentThread()方法，run()方法没有返回值。
1. **使用Executer框架：**
- Executor框架是Java并发编程中的高级工具，通过Executer，可以将任务提交给线程池，由线程池来管理线程的生命周期和执行。

```
`import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

class MyTask implements Runnable{
  @Override
  public void run(){
    //线程执行的逻辑代码
    System.out.println("MyTask is running...");
  }
}

// 创建线程池并提交任务
Executor executor = Executors.newFixedThreadPool(3);
// 提交任务
executor.execute(new MyTask());
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15

- 优点：可以避免频繁创建和销毁线程的开销，可以控制线程并发数量，监控线程状态，可以接受多种任务类型（Runnable和Callable）。
- 缺点：核心线程占用系统资源，线程池配置比较复杂，容易造成资源浪费。
1. **实现Callable接口:**
- Callable的call()方法可以有返回值并且可以抛出异常。
- Callable的返回值在子线程中产生，需要配合Future/FutureTask用于获取线程执行的结果。
- 可以使用**线程池配合future**，将MyCallable实例在线程池执行。或者由于Thread只能执行Runnable任务，可以使用**FutureTask包装Callable任务**。

```
`// 使用Future和线程池
class MyCallable implements Callable<Integer>{
  publc Integer call() throws Exeception{
    //线程执行的代码，返回一个整形结果
    return 42;
  }
}

ExecutorService excutor = Executors.newSingleThreadExecutor();
Future<Integer> future = excutor.submit(new MyCallable());

try{
  //获取线程执行结果
  Integer result = future.get();
}catch(InterruptedExeception | ExecutionException e){
  //处理异常
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17

```
`// 使用FutureTask包装
// MyCallable代码同上

MyCallable task = new MyCallable();
FutureTask<Integer> futureTask = new FutureTask<>(task);
Thread myThread = new Thread(futureTask);
myThread.start();

try{
  //获取线程执行结果
  Integer result = futureTask.get();
}catch(InterruptedExeception | ExecutionException e){
  //处理异常
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14

- 优点：Callable接口的方法有返回值，还可以抛出异常
- 缺点：代码复杂，访问当前线程需要调用Thread.currentThread()方法。
##  使用场景
 创建方法 适用场景 继承Thread类 简单场景 实现Runnable接口 多线程共享资源场景 实现Callable接口 需要任务返回结果或处理异常的场景 使用线程池（Executor） 生产环境，高并发，需要复用线程场景
##  知识图解


**1. 创建线程方法示意图**


 知识扩展

1. 扩展：
- 线程池的核心参数： 线程池的构造函数有**七个参数**
  - corePoolSize 核心线程数: 线程池中**长期存活**的线程数
    - 如果线程池中线程数量少于corePoolSize,这些线程处于空闲状态也不会被销毁
  - maximumPoolSize 最大线程数: 线程池允许创建的**最大线程数**(包括核心线程和非核心线程)
    - 当核心线程已满,队列已满时,如果当前线程小于最大线程数,就会创建新的线程执行此任务,否则触发**拒绝策略**
  - keepAliveTime 空闲线程存活时间: 当线程数大于corePoolSize时,如果某个线程的**空闲时间超过了keepAliveTime,那么这个线程就会被销毁**
  - TimeUnit 与keepAliveTime一起使用,指定keepAliveTime的**时间单位**
  - workQueue 线程池任务队列: 线程池存放任务的队列,没有空闲线程执行新任务时,用来**存储线程池的所有待执行任务**
  - ThreadFactory 创建线程的工厂: 线程池**创建线程时调用的工厂方法**,通过此方法可以设置线程的优先级,线程命名规则以及线程类型(用户线程还是守护线程)
  - RejectedExecutionHandler **拒绝策略**: 当线程池的任务超出线程池队列可以存储的最大值之后,执行的策略
1. 面试官可能追问:
- Q1：调用start()方法和直接调用run()方法有什么区别？
  - **start()本质是native方法**，会启动一个新线程，由JVM调用run()方法，实现真正的并发执行。而直接调用**run()是仅作为普通方法**在当前线程执行，不会创建新线程。
- Q2：线程池创建线程和手动创建线程有什么不同？
  - 手动创建线程是**即用即建**，线程池能够实现**预先创建和线程复用**，能够减少线程创建/销毁开销。
- Q3：Future的get()方法阻塞怎么避免主线程的长时间等待？
  - 使用**get(long timeout, TimeUnit unit)** 设置超时时间，避免无限阻塞。
  - 用**isDone()** 先判断任务是否完成，再调用get()。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## 工厂模式｜高频面试题｜Java高频面试题｜设计模式、创建型模式、简单工厂、工厂方法、抽象工厂

#  工厂模式

##  简要回答

- 工厂模式是常用的**创建型设计模式**，会定义创建对象的接口，但由子类决定实例化哪一个类，避免向客户端暴露对象的创建逻辑，实现封装和管理对象的创建过程。可以分为**简单工厂模式**、**工厂方法模式**和**抽象工厂模式**三种形式。
- 工厂模式的优点是**将对象的创建和使用进行分离**，客户端只需要知道接口而不需要了解具体实现细节；如果对象的创建过程比较复杂，工厂模式可以**提高代码的复用性**；工厂模式还可以方便地进行扩展，增加新的产品类而不需要修改已有代码，**符合开闭原则**；将对象的创建封装在工厂类中也是**符合单一职责原则**的。
##  详细回答

1. **简单工厂模式（静态工厂模式）**：工厂类通过静态方法，根据传入的参数判断并创建对应的具体产品对象，返回给调用者。
  2. 简单工厂模式有三类角色，分别是工厂、抽象产品和具体产品。**工厂类**是工厂模式的核心，负责实现所有产品的内部逻辑，可以直接被外界调用；**抽象产品**是工厂类所创建的所有对象的父类，封装了产品对象的公共方法；**具体产品**则是简单工厂模式的创建目标，实现了抽象产品中声明的抽象方法。
  3. 缺点是使用简单工厂模式需要新增产品时，需要修改原工厂类，违反了开闭原则，且不够灵活。
  4. 在JDK中DateFormate、Calendar类都有使用简单工厂模式，通过不同参数返回需要的对象。
5. **工厂方法模式**：将简单工厂中的工厂类变为抽象接口，工厂接口定义创建产品的方法，具体工厂类实现该方法，每个具体工厂对应一个具体产品。调用者通过实例化具体工厂获取产品，无需传入参数。
  6. JDK的Collection接口中Iterator的实现使用了工厂方法模式，每个具体的集合类都有一个对应的迭代器类，迭代器类实现了Iterator接口，负责遍历集合中的元素。
7. **抽象工厂模式**：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们的具体类。抽象工厂模式通常涉及多个抽象产品、多个具体产品和多个具体工厂。
  8. 抽象工厂模式的优点是可以创建一组相关的产品，保证产品族的一致性；
  9. 对抽象工厂模式的业务进行扩展时需注意，对产品族扩展是符合开闭原则的，无需修改原有代码；如果需要对产品等级扩展，将会导致所有代码变动，所以需要新增产品等级的业务不能使用抽象工厂模式。
##  知识图解

1. 简单工厂模式和工厂方法模式结构图  代码示例

  1.

**简单工厂模式**：一个工厂类、一个产品接口、多个具体产品类


```
`// 产品接口
public interface Car {
    void drive();
}

// 具体产品1
public class CarA implements Car {
    @Override
    public void drive() {
        System.out.println("驾驶A车");
    }
}

// 具体产品2
public class CarB implements Car {
    @Override
    public void drive() {
        System.out.println("驾驶B车");
    }
}

// 简单工厂类
public class CarFactory {
    public static Car createCar(String type) {
        if ("CarA".equals(type)) {
            return new CarA();
        } else if ("CarB".equals(type)) {
            return new CarB();
        }
        return null;
    }
}

// 调用者
public class Client {
    public static void main(String[] args) {
        Car car = CarFactory.createCar("CarA");
        car.drive(); // 输出：驾驶A车
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40

  2.

工厂方法模式：一个工厂接口、多个具体工厂类、一个产品接口、多个具体产品类


```
`// 产品接口（同简单工厂）
public interface Car {
    void drive();
}

// 具体产品1（同简单工厂）
public class CarA implements Car {
    @Override
    public void drive() {
        System.out.println("驾驶A车");
    }
}

// 具体产品2（同简单工厂）
public class CarB implements Car {
    @Override
    public void drive() {
        System.out.println("驾驶B车");
    }
}

// 工厂接口
public interface CarFactory {
    Car createCar();
}

// 具体工厂1：创建A车
public class CarAFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new CarA();
    }
}

// 具体工厂2：创建B车
public class CarBFactory implements CarFactory {
    @Override
    public Car createCar() {
        return new CarB();
    }
}

// 调用者
public class Client {
    public static void main(String[] args) {
        CarFactory factory = new CarBFactory();
        Car car = factory.createCar();
        car.drive(); // 输出：驾驶B车
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50

  3.

抽象工厂模式：一个工厂接口、多个具体工厂类、多个产品接口、多个具体产品族（每个具体工厂对应一个具体产品族，每个产品族包含多个产品）


```
`// 产品接口1：汽车
public interface Car {
    void drive();
}

// 产品接口2：发动机
public interface Engine {
    void run();
}

// 具体产品族1：A系列
public class CarA implements Car {
    @Override
    public void drive() {
        System.out.println("驾驶A车");
    }
}

public class AEngine implements Engine {
    @Override
    public void run() {
        System.out.println("A发动机运转");
    }
}

// 具体产品族2：B系列
public class CarB implements Car {
    @Override
    public void drive() {
        System.out.println("驾驶B车");
    }
}

public class BEngine implements Engine {
    @Override
    public void run() {
        System.out.println("B发动机运转");
    }
}

// 抽象工厂接口：创建汽车和发动机
public interface CarAbstractFactory {
    Car createCar();
    Engine createEngine();
}

// 具体工厂1：创建A产品族
public class AFactory implements CarAbstractFactory {
    @Override
    public Car createCar() {
        return new CarA();
    }

    @Override
    public Engine createEngine() {
        return new AEngine();
    }
}

// 具体工厂2：创建B产品族
public class BFactory implements CarAbstractFactory {
    @Override
    public Car createCar() {
        return new CarB();
    }

    @Override
    public Engine createEngine() {
        return new BEngine();
    }
}

// 调用者
public class Client {
    public static void main(String[] args) {
        CarAbstractFactory factory = new AFactory();
        Car car = factory.createCar();
        Engine engine = factory.createEngine();
        car.drive(); // 输出：驾驶A车
        engine.run(); // 输出：A发动机运转
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82

##  使用场景

  2. 工厂设计模式适合在以下场景中使用：
    3. 当一个类不知道它所需要创建的对象的类时，即调用类只需知道产品的接口，而不需要知道具体的实现类。
    4. 当一个类希望通过其子类来指定它所创建的对象时，即工厂模式将类的实例化延迟到子类，父类只需要定义通用的业务逻辑，由子类实现对象的创建逻辑。
  5. 工厂模式的本质是根据不同的需要创建不同的对象，例如根据不同的数据库类型创建不同的数据库连接对象、根据配置文件选择不同的日志记录器、根据文件类型选择不同解析器等等。
###  三种工厂模式对比：

  6. 简单工厂模式适合产品类型少且固定场景，调用者只需知道传入参数即可创建对象
  7. 工厂方法模式适合产品类型较多且需要新增产品的场景，希望将对象实例化延迟到子类实现
  8. 抽象工厂模式适合需要创建一组相关产品的场景，创建不同的产品族，系统与产品族交互。
##  知识扩展

  1. 面试官可能追问：
  9. Q1：工厂模式和单例模式、原型模式有什么区别和联系？
    10. 工厂、单例、原型均为创建型设计模式，都封装对象创建逻辑、优化对象创建方式、解耦创建与使用；
    11. 区别在于核心设计目标不同：**工厂模式**是按需创建不同类型的全新对象，核心做对象的统一生产，封装对象的创建细节；**单例模式**是确保一个类仅创建唯一实例，核心做对象的全局复用，避免重复创建消耗资源；**原型模式**是基于已有对象拷贝生成新对象，核心做对象的高效克隆。
  12. Q2：Spring框架中哪里使用到了工厂模式？
    13. SpringIoC的根接口**BeanFactory**就是简单工厂模式和工厂方法模式的结合体，所有Spring容器都是它的实现类。统一封装了所有Bean对象的创建、实例化、初始化。依赖注入等操作，因此我们可以在不了解Bean的具体创建的情况下获取Bean对象。
    14. Spring框架的扩展接口**FactoryBean**也使用了工厂方法模式，FactoryBean接口是自定义Bean工厂，负责创建某一个特定的、创建逻辑复杂的Bean实例。在FactoryBean定义的getObject()方法中实现具体的Bean创建逻辑，Spring容器调用这个方法获取Bean实例。
  15. Q3：什么时候不适合使用工厂模式？
    16. 当对象的创建逻辑极其简单（仅一行new关键字即可完成，无任何初始化、配置等复杂逻辑）、业务中产品类的数量极少且永远不会扩展、项目是简单的小工具/小型业务模块，或是为了极简化代码、降低系统冗余度时，就不适合使用工厂模式；
    17. 此时使用工厂模式，不仅发挥不出其优势，反而会凭空增加多余的工厂类、抽象接口，让代码结构变复杂、类的数量冗余、阅读和维护成本上升，违背了简单设计的原则。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## JVM内存结构｜大厂面试题｜Java高频面试题｜运行时数据区、堆、栈、方法区

#  JVM内存结构

##  简要回答

- JVM内存结构（运行时数据区）可以分为**Java虚拟机栈、堆、方法区、本地方法栈和程序计数器**五个部分。在JDK8之前，方法区通过永久代实现，JDK8及以后，永久代被元空间取代。
1. **Java虚拟机栈**：线程私有，存储方法栈帧。
2. **堆**：线程共享，存储对象实例，是GC主要区域。
3. **方法区**：线程共享，存储类信息、常量、静态变量等，JDK8后由元空间实现。
4. **本地方法栈**：线程私有，为本地方法服务。
5. **程序计数器**：线程私有，记录当前线程执行的字节码指令地址。
##  详细回答

1.

**Java虚拟机栈(Java Virtual Machine Stacks)** ：每个线程都有一个私有的虚拟机栈，和线程同时创建。每个方法在执行时会创建一个**栈帧，用于存储局部变量、操作数栈、动态链接、方法出口**等信息。栈帧在方法调用时入栈，方法返回时出栈。

  2. 栈的**生命周期和线程一致**，由操作系统管理。
  3. **虚拟机栈的大小**可以是**动态的或者是固定不变的**。如果是固定大小，每个线程的虚拟机栈容量在线程创建时指定，线程请求分配的栈容量大于虚拟机栈允许的最大容量时，会抛出StackOverflowError异常。如果是动态扩展的，在没有足够内存时可能会抛出一个OutOfMemoryError异常。
4.

**Java堆(Java Heap)** ：是Java虚拟机中最大的一块内存区域，**存储对象实例和数组**。堆被所有线程共享，在虚拟机启动时创建。默认初始堆内存大小是电脑内存大小的1/64，最大堆内存大小是电脑内存大小的1/4。

  5. **新生代**：可以分为**Eden区**和**Survivor区**,大多数新创建的对象首先存放在Eden区，当Eden区满时会触发一次Minor GC（新生代垃圾回收）。Survivor区分为两个大小相等的区域（**S0和S1**），每次Minor GC后存活下来的对象被移动到S0或S1中，继续对象的生命周期。Eden和S0，S1的比例是8:1:1。
  6. **TLAB**：在Eden空间内，JVM为每个线程分配的私有缓存区域，内存占Eden空间的1%。多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时可以提升内存分配的吞吐量。
  7. **老年代**：存放一次或多次Minor GC后仍存活的对象，这些对象生命周期较长，Major GC（老年代垃圾回收）频率低，但是执行时间长于Minor GC。新生代和老年代的比例大概在1:2，老年代大于新生代，存储更多的长期存活对象。
  8. **大对象区**：有的JVM实现会为需要大量连续内存空间的对象分配专门区域，这些大对象会直接分配在老年代，避免新生代频繁GC导致内存碎片化。
  9. **字符串常量池**：JDK7之后出现，JVM为了提升性能和减少内存消耗，对字符串（String类）专门开辟的一块区域，可以避免字符串的重复创建。
  10. **永久代**：JDK7及之前存在堆中，是方法区的实现，存储类的元数据，运行时常量池，已加载的类信息，静态变量和常量。
  11. **元空间**：JDK8及之后，取代永久代，存储在本地内存中，存储类的元数据信息（字段、方法信息），能够解决永久代容易出现的内存溢出问题。
12.

**方法区(Method Area/Non-Heap)** ：存储已经被虚拟机加载的类信息、常量、静态变量以及即时编译器编译后的代码。大小和堆空间一样，可以固定或者可扩展，能够决定系统可以放多少个类，JVM关闭时方法区被释放。

  13. **运行时常量池**：JVM会为每个已加载的类型维护一个常量池。是方法区的一部分，在类加载后，存放编译器生成的各种字面量和符号引用。与class文件中的常量池不同，运行时常量池可以动态改变。
14.

**本地方法栈(Native Method Stack)** ：管理本地方法的调用，是线程私有的。本地方法是使用其他编程语言实现的，通过JNI(Java Native Interface)与Java代码进行交互。

  15. 与虚拟机栈相同，允许固定或者可动态分配的内存空间。
  16. 本地方法栈会登记本地方法，生成本地方法接口，可以根据本地方法接口加载本地方法库中对应的实现方法，本地方法栈为本地方法提供内存支持。
17.

**程序计数器(Program Counter Register)** ：当前线程所执行的字节码的行号指示器，在多线程环境下，每个线程有自己独立的程序计数器，当线程执行Java方法时，程序计数器记录正在执行的虚拟机字节码指令的地址。

  18. 是一个很小的内存空间，运行速度快。
  19. 是线程私有的，生命周期和线程一致。
20.

**直接内存(Direct Memory)** ：直接内存与JVM的内存管理有关。Java的NIO库允许直接分配堆外内存，这些内存不受Java堆大小的限制，也不受垃圾回收器管理。通常通过ByteBuffer类来使用。

##  知识图解


jvm运行时数据区组成示意图  知识扩展

1. 扩展：
- **栈帧(Stack Frame)** ：每个栈帧会存储方法的局部变量表、操作数栈、动态链接和方法返回地址等。
  1. **局部变量表**：存储方法参数和定义在方法中的局部变量。其中的变量只在当前方法中有效。局部变量表需要的容量大小是编译器确定的。被局部变量表中直接引用或间接引用的变量都不会被回收。
  2. **操作数栈（表达式栈）**：保存计算过程的中间结果，在方法执行过程中会根据字节码指令往操作数栈中写入数据或提取数据。
  3. **动态链接**：指向运行时常量池的方法引用，可以将符合引用转换成调用方法的直接引用。
  4. **方法返回地址**：存放调用该方法的PC寄存器的值。当方法返回时，将返回值压入调用者栈帧的操作数栈，设置寄存器值，让调用者方法继续执行。如果是因异常退出方法，不会给调用者返回值。
1. 面试官可能追问：
- Q1：为什么需要程序计数器记录当前线程的执行地址？
  - CPU需要不停切换各个线程，JVM的字节码解释器需要通过改变程序计数器的值来明确下一条应该执行什么字节码指令。
- Q2：为什么会使用本地方法？
  - Java是一次编写、到处运行的语言，**无法直接操作操作系统的核心资源**，本地方法可以直接与操作系统交互，比如java.io.File类通过本地方法调用操作系统的文件系统API。线程管理也依赖本地方法调用操作系统的线程调度接口。
  - 使用本地方法可以**复用非Java的代码库**。
- Q3：为什么使用元空间取代永久代？
  - **永久代受堆大小限制**，元空间使用的是本地内存，而且有动态扩容机制，能够解决永久代容易内存溢出的问题。
  - 永久代很难设置合适的内存大小，并且**调优困难**。
- Q4：给对象的分配内存过程是什么？
  - 新创建的对象会计算其所需内存是否为大对象，如果是则直接分配在大对象区（老年代）。
  - 非大对象会优先在新生代的Eden区的TLAB中分配，如果TLAB已满，则分配在新生代Eden区。
  - 如果Eden区已满，JVM会进行一次Minor GC，将存活对象移动To Survivor区（S0，S1交替），再将新对象放在Eden区。
  - 如果S0或S1中的对象经历一定回收次数（默认15次），会被移动到老年代。
  - 如果老年代内存不足，会触发Major GC，若仍旧内存不足则会触发OOM异常。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Java中什么是自动装箱和拆箱？有什么坑？

#  Java中什么是自动装箱和拆箱？有什么坑？

##  简要回答

- 自动装箱（Autoboxing）是指：Java编译器把**基本类型**自动转换为对应的**包装类型**，如 `int -> Integer`。
- 自动拆箱（Unboxing）是指：Java编译器把**包装类型**自动转换为对应的**基本类型**，如 `Integer -> int`。
- 装箱/拆箱提升了代码的可读性和集合使用便利性，但是使用时需要注意：`==` 比较陷阱、`NullPointerException`、性能开销、三目运算符类型提升等问题。
##  详细回答

1. 什么是自动装箱/拆箱
  2. 自动装箱：当需要对象的地方传入基本类型时，编译器会自动调用包装类的 `valueOf()`。
  3. 自动拆箱：当需要基本类型的地方传入包装类型时，编译器会自动调用如 `intValue()`、`longValue()`。
  4. 典型映射关系：
    5. `int <-> Integer`
    6. `long <-> Long`
    7. `double <-> Double`
    8. `boolean <-> Boolean`
    9. `char <-> Character`
    10. `float <-> Float`
    11. `byte <-> Byte`
    12. `short <-> Short`
13. 为什么会有这个机制
  14. Java集合（如 `List`、`Map`）只能存对象，不能直接存基本类型。
  15. 泛型和很多框架API都以对象类型为核心，自动装箱/拆箱让编码更简洁。
16. 编译器背后做了什么
  17. 示例1：`Integer x = 10;` 本质接近 `Integer x = Integer.valueOf(10);`把基本类型 10 赋值给包装类变量 x，Java 编译器会自动帮你完成「基本类型 → 包装类对象」的转换，底层就是调用 Integer.valueOf(10)。
  18. 示例2：`int y = x;` 本质接近 `int y = x.intValue();`把包装类对象 x 赋值给基本类型变量 y，Java 编译器会自动帮你完成「包装类对象 → 基本类型」的转换，底层就是调用 x.intValue()，也就是说，这是编译阶段的语法糖。
##  常见坑点

1. `==` 比较包装类型，结果可能不符合预期
  2. `Integer` 在 `[-128, 127]` 范围通常会走缓存，`==` 可能是 `true`。
  3. 超出缓存范围一般是不同对象，`==` 往往是 `false`。
  4. 结论：包装类型比较值请用 `equals()`（并注意空指针）。
5. 拆箱时对象为 `null`，会触发 `NullPointerException`
  6. `Integer a = null; int b = a;` 会在拆箱时NPE。
  7. 常见于 `Map.get()` 返回 `null` 后直接参与算术运算或比较。
8. 循环中频繁装箱/拆箱导致性能和内存开销上升
  9. 大量创建包装对象会增加GC压力，热点路径性能下降。
  10. 高性能场景优先使用基本类型，必要时使用原生数组或专用集合。
11. `equals()` 的跨类型比较容易踩坑
  12. `Long.valueOf(1).equals(Integer.valueOf(1))` 是 `false`，因为类型不同。
  13. 如果是数值语义比较，先统一类型再比较。
14. 三目运算符混用包装类型与基本类型会隐式拆箱
  15. 如 `Integer a = null; int r = flag ? a : 0;` 可能因 `a` 拆箱导致NPE。
16. 误用包装类型做锁对象
  17. `Integer` 存在缓存复用，`synchronized(Integer.valueOf(1))` 可能锁住同一个共享对象，带来并发隐患。
##  代码示例


```
`public class BoxingDemo {
    public static void main(String[] args) {
        // 1) == 陷阱（缓存区间）
        Integer a = 127;
        Integer b = 127;
        Integer c = 128;
        Integer d = 128;
        System.out.println(a == b); // true
        System.out.println(c == d); // false
        System.out.println(c.equals(d)); // true

        // 2) null 拆箱 NPE
        Integer x = null;
        // int y = x; // 会抛出 NullPointerException

        // 3) Map.get 返回 null 后拆箱
        java.util.Map<String, Integer> map = new java.util.HashMap<>();
        // int count = map.get("k"); // 也会 NPE

        // 4) 跨类型 equals
        Long l = 1L;
        Integer i = 1;
        System.out.println(l.equals(i)); // false
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25

##  知识图解

1. 常见的数据类型及其包装类  知识扩展

  1. 扩展：
    2. `Integer.valueOf()` 有缓存机制（默认 `-128~127`），而 `new Integer()` 每次都创建新对象，已不推荐使用。
    3. 在集合中删除元素时要区分重载：`list.remove(1)` 是按索引删；`list.remove(Integer.valueOf(1))` 是按值删。
  4. 面试官可能追问：
  2. Q1：为什么建议包装类型用 `equals()`，不用 `==`？
    3. `==` 比较的是引用地址；`equals()` 比较的是值语义（实现正确时）。缓存导致 `==` 结果不稳定，容易出错。
  4. Q2：自动拆箱引发NPE最常见在哪？
    5. `Map.get()`、数据库查询字段为空、RPC反序列化字段缺失后参与数值计算或条件判断时最常见。
  6. Q3：如何规避装箱/拆箱的性能问题？
    7. 热点代码优先用基本类型；避免无意义的包装对象创建；必要时做对象池或批处理降低开销。
  8. Q4：包装类型和基本类型该怎么选？
    9. 需要 `null` 语义、集合/泛型/反射交互时用包装类型；追求性能、并且不需要空值表达时优先基本类型。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## 观察者模式｜高频面试题｜设计模式高频面试题｜行为型模式、发布订阅、事件驱动

#  观察者模式

##  简要回答

- **观察者模式**是一种**行为型设计模式**，核心思想是定义一对多的对象依赖关系，让多个观察者对象同时监听某一个主题对象，这个主题对象在状态变化时会通知所有的观察者对象，触发响应行为；观察者能够动态订阅/取消订阅主题，实现了发布者与观察者的解耦。
- 观察者模式优点：除了能够**动态扩展**观察者外，还能实现观察者对象和主题对象之间的**解耦**，让耦合的双方都依赖于抽象，不会相互影响；同时**封装**了主题状态变更的通知逻辑，不用手动遍历调用观察者。
- 但是观察者模式的**通知顺序是无序**的，还有**内存泄漏**风险，若观察者订阅后没有及时取消订阅，会导致观察者对象无法被GC回收；如果观察者更新时触发主题再次变更，可能会引发**循环通知**而导致程序卡死。
##  详细回答

- 观察者模式有四个核心角色：抽象主题、具体主题、抽象观察者、具体观察者。
  1. **抽象主题**(Subject)：定义主题的核心行为规范，提供了添加/删除观察者的方法和统一通知所有观察者的方法。
  2. **具体主题**(ConcreteSubject)：继承抽象主题，是被观察的对象，维护自身的业务状态，存储所有已订阅的观察者对象集合；在自身状态发生变化时主动调用通知方法，通知所有已订阅的观察者对象。
  3. **抽象观察者**(Observer)：定义观察者的核心行为规范，声明响应主题状态变化的更新方法。
  4. **具体观察者**(ConcreteObserver)：是订阅消息的对象，实现抽象观察者所声明的更新方法，在自身状态发生变化时，调用该方法，将自身状态作为参数传递给观察者。
- 实现观察者模式时，需要先**定义主题接口**，包括注册、移除和通知观察者的方法；然后**定义观察者接口**，包括更新方法；再**定义具体的主题类**，维护一组观察者对象，实现在主题状态改变时，调用观察者的更新方法；最后**定义具体的观察者类**，实现更新方法。客户端负责创建主题对象和观察者对象，将观察者注册到主题中，实现主题改变状态时观察者得到通知并进行更新。
##  知识图解

1. 观察者模式结构图  代码示例

  1.

以Java代码为例，实现一个简单的观察者模式如下：


```
`import java.util.ArrayList;
import java.util.List;

// 观察者接口
interface Observer {
    void update(String message);
}

// 具体观察者
class ConcreteObserver implements Observer {
    private String name;

    public ConcreteObserver(String name) {
        this.name = name;
    }

    @Override
    public void update(String message) {
        System.out.println("观察者 [" + name + "] 收到通知: " + message);
    }
}

// 被观察者接口（主题）
interface Subject {
    void attach(Observer observer);   // 添加观察者
    void detach(Observer observer);   // 移除观察者
    void notifyObservers(String message); // 通知所有观察者
}

// 具体被观察者（具体主题）
class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();

    @Override
    public void attach(Observer observer) {
        if (observer == null) {
            throw new IllegalArgumentException("观察者不能为 null");
        }
        if (!observers.contains(observer)) {
            observers.add(observer);
        }
    }

    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(String message) {
        if (message == null) {
            message = "";
        }
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}

// 测试类
public class ObserverPatternDemo {
    public static void main(String[] args) {
        // 创建被观察者
        ConcreteSubject subject = new ConcreteSubject();

        // 创建观察者
        Observer obs1 = new ConcreteObserver("张三");
        Observer obs2 = new ConcreteObserver("李四");
        Observer obs3 = new ConcreteObserver("王五");

        // 注册观察者
        subject.attach(obs1);
        subject.attach(obs2);
        subject.attach(obs3);

        // 发送通知
        subject.notifyObservers("第一次更新：系统即将维护");

        // 移除一个观察者
        subject.detach(obs2);

        // 再次发送通知
        subject.notifyObservers("第二次更新：维护完成");
    }
}

/* 代码执行结果：
观察者 [张三] 收到通知: 第一次更新：系统即将维护
观察者 [李四] 收到通知: 第一次更新：系统即将维护
观察者 [王五] 收到通知: 第一次更新：系统即将维护
观察者 [张三] 收到通知: 第二次更新：维护完成
观察者 [王五] 收到通知: 第二次更新：维护完成
*/

`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94

  2.

Java中内置了观察者模式的核心API，核心类是java.util.Observable和java.util.Observer。Observable类是被观察者（抽象主题），内置了观察者集合和增删/通知方法；Observer类是观察者（抽象观察者），定义了更新方法。

    3. 使用JDK原生API时，需要自定义主题类和观察者类并分别继承Observable和Observer类，当被观察者的状态发生改变后，需要调用setChanged()标记状态变更，然后调用notifyObservers()通知所有观察者。
##  使用场景

  2. 观察者模式适用于**当一个对象状态发生改变，需要通知多个其他对象进行更新**的场景，尤其是不知道具体有多少个观察者对象的情况下，希望这些对象不是紧耦合的情况。
  1. **事件处理**场景中，当进行GUI界面开发时需要使用观察者模式监听鼠标点击、键盘输入等事件；
  2. 实现**发布-订阅模型**时，使用观察者模式实现有一个主题负责发送通知，多个观察者负责监听并响应通知，如微服务中的事件驱动架构或消息队列系统场景；
  3. 观察者模式还可以用于**实现通知业务**，如短信推送、邮件通知等场景。
  4. 框架中也经常使用观察者模式，如**Spring框架**中的ApplicationContext事件监听机制，其中ApplicationListener接口定义了监听器（观察者），ApplicationEvent接口定义事件（主题）；**Redis框架**中的Pub/Sub机制，也是基于观察者模式实现的。
##  知识扩展

  1. 面试官可能追问：
  3. Q1：观察者模式和发布-订阅模式有什么区别？
    4. 观察者模式没有中间层，主题直接持有观察者的引用，可以直接实现通知和调用其更新方法；适合单机且轻量级的场景。
    5. 发布-订阅模式引入了中间层实现完全的解耦，发布者负责发布消息，由中间层负责将消息分发给订阅者，发布者和订阅者互相不知道对方的存在。适合分布式、跨进程的大型系统，比如微服务、消息队列等。
  6. Q2：使用观察者模式的缺点有哪些，怎么解决？
    7. 使用观察者模式时可以**给观察者设置优先级**，按照优先级排序通知，解决通知无序问题。
    8. 对于内存泄漏问题，可以**使用弱引用存储观察者**，还可以在观察者生命周期结束时主动调用detach方法移除观察者引用。
    9. 在update方法中**增加状态变更标记**，避免重复触发通知逻辑，可以解决循环通知问题。
    10. 当观察者模式使用同步通知遇到性能瓶颈时，可以**使用线程池异步通知**，主题状态变化时异步调用观察者的update方法。
  11. Q3：观察者模式中有哪些设计原则？
    12. **依赖倒置原则**：观察者模式的主题和观察者都依赖抽象而不依赖具体实现；
    13. **开放闭合原则**：新增观察者时无需修改主题代码，对扩展开放、对修改闭合；
    14. **单一职责原则**：观察者模式的主题只负责维护状态和通知，观察者只负责处理通知。  Last Updated: 3/10/2026, 6:08:48 PM

 ← 

### 评论
 

验证登录状态...

## equals与==｜高频面试题｜Java高频面试题｜equals方法、==运算符、hashCode

#  equals与==

##  简要回答

- **使用"=="进行比较**：
  1. ==是一个**比较运算符**，既可以判断基本类型，又可以判断引用类型。
  2. 如果**判断基本类型**，判断的是二者的**值**是否相等(eg : 判断1 == 1，结果为true；判断1 == 3，结果为false)；
  3. 如果**判断引用类型**，判断的是二者的**地址**是否相同，即判定是否为同一对象(eg : Student student1 = new Student();，Student student2 = student1；判断student2 == student1，结果为true)。
- **使用equals()方法进行比较**：
  1.

equals()方法是顶层父类Object类中的方法，**equals方法**本身在Object类中的**源码如下** :


```
`public boolean equals(Object obj) {
	return (this == obj);
}
`
```
 1
2
3

  2.

可以看到，Object类中的 equals 方法用来检测两个对象是否相等，即**默认情况下比较的是两个对象的引用**(地址)。这一点和 == 用于判断引用类型时一致。

  3.

**equals的特点**在于，它是Object类中的方法，因此，equals方法**往往在子类中被重写**，例如在String类中，equals方法被重写去判断两个字符串的内容是否相等。并且，在我们自己创建的类中，equals方法也常常被重写，去判断两个对象的指定的具体内容是否一致。

  4.

还有一点要注意，**“==”的运行速度通常比“equals方法”更快**；因为==比较引用类型时，仅比较地址；而equals方法的性能要取决于具体实现。


---

##  详细回答

###  使用"=="进行比较

-

**基本类型比较**：直接比较两个变量的**值**是否相等。


```
`int temp_a = 10;
int temp_b = 10;
System.out.println(temp_a == temp_b); // true
`
```
 1
2
3

-

**引用类型比较**：比较两个对象的**内存地址**是否相同（即是否为同一对象）。


```
`Object obj1 = new Object();
Object obj2 = obj1;
System.out.println(obj1 == obj2); // true（同一对象）
System.out.println(new Object() == new Object()); // false（不同对象）
`
```
 1
2
3
4

###  使用equals()方法进行比较

-

**默认行为**： Object类中的 equals()方法 默认比较对象的**地址**，与 == 在进行引用类型比较时行为一致，**equals方法**本身在Object类中的**源码如下** :


```
`public boolean equals(Object obj) {
	return (this == obj);
}
`
```
 1
2
3

-

**子类重写**： 大多数类会 **重写 equals()** 以比较对象的**内容**而非地址。eg：

  1.

**String类**：比较字符串的字符序列。


```
`String s1 = new String("Hello");
String s2 = new String("Hello");
System.out.println(s1 == s2);       // false（地址不同）
System.out.println(s1.equals(s2));  // true（地址不同但内容相同）
`
```
 1
2
3
4

  2.

**Integer类**：比较整数值。

  3.

**自定义类**：按照实际业务逻辑手动重写 equals()方法，比较指定内容。


---

##  知识拓展

###  == 和 equals 的jvm示意图

1. **使用`==`进行比较的 内存图解**：
  重写 equals()的注意事项

  1.

**遵守 equals() 契约**：

    2. 自反性：`a.equals(a)` 必须为 `true`。
    3. 对称性：若 `a.equals(b)` 为 `true`，则 `b.equals(a)` 必须为 `true`。
    4. 传递性：若 `a.equals(b)` 和 `b.equals(c)` 为 `true`，则 `a.equals(c)` 必须为 `true`。
    5. 一致性：多次调用 `a.equals(b)` 结果应一致（除非对象被修改）。
    6. 非空性：`a.equals(null)` 必须为 `false`。
  7.

**必须同时重写hashCode()** ： 若两个对象通过 equals()方法 比较为 **true**，则它们的 **hashCode()** 必须相同。代码演示如下：


```
`@Override
public int hashCode() {
    return Objects.hash(name, age); // 使用相同字段生成哈希值
}
`
```
 1
2
3
4

  8.

**正确处理 null 和 对象类型**： 在 equals()方法中 需检查参数是否为 **null** 或对象类型是否匹配。代码演示如下：


```
`@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    Person person = (Person) obj; // 强制类型转换
    return age == person.age && Objects.equals(name, person.name);
}
`
```
 1
2
3
4
5
6
7

  9.

`Objects.equals()`方法源码如下：
   →

### 评论
 

验证登录状态...

## JVM常见参数｜高频面试题｜Java高频面试题｜内存配置、垃圾回收、GC日志、OOM处理

#  JVM常见参数

##  简要回答

- **JVM参数**为JVM运行提供配置依据，可以对内存分配、垃圾回收、日志等方面进行配置，为不同的业务场景选择合适的参数。
- 进行内存配置时可以使用 **-Xms**和 **-Xmx**设置堆内存的初始值和最大值； **-Xmn**设置新生代大小， **-XX:NewRatio**设置老年代与新年代比例；
- 进行垃圾回收时可以使用 **-XX:+UseParallelGC、-XX:+UseConcMarkSweepGC、-XX:+UseG1GC**等参数进行垃圾回收器的选择。
- 关于日志可以使用 **-XX:PrintGCDetails**打印GC详细信息，使用 **-XX:PrintGCDateStamps**打印GC日期和时间，还可以使用 **-Xloggc:/path/to/gc-%t.log**打印GC日志到指定文件。%t表示当前时间。
- JVM参数也可以处理OOM异常，使用 **-XX:+HeapDumpOnOutOfMemoryError**可以在OOM时自动生成堆快照，使用 **-XX:HeapDumpPath=/path/to/heapdump.hprof**可以指定堆快照文件路径。
##  详细回答

1. **内存配置**
  2. **-Xms**：设置JVM初始堆内存大小，例如-Xms1024m，需要设置数值和单位。
  3. **-Xmx**：设置JVM最大堆内存大小，一般与-Xms参数设置相同，避免动态调整。
  4. **-Xmn**：固定新生代内存大小，一般是堆内存的三分之一。
  5. **-XX:NewSize**：设置新生代初始空间大小。默认情况下新生代大小为1310MB。
  6. **-XX:MaxNewSize**：设置新生代最大空间大小。
  7. **-XX:NewRatio**：设置老年代和新生代的内存大小比例。默认值为2，即老年代和新生代占比为2:1。
  8. **-XX:PermSize**：设置永久代初始空间大小。
  9. **-XX:MaxPermSize**：设置永久代的最大大小，超出则会抛出OOM异常。
  10. **-XX:MetaspaceSize**：元空间阈值，当元空间使用量达到该阈值时，会触发Full GC，之后JVM会动态调整该阈值
  11. **-XX:MaxMetaspaceSize**：设置元空间增长上限，默认值4096MB。
12. **垃圾回收配置**
  13. **-XX:+UseSerialGC**：使用串行垃圾收集器，单线程执行GC，适用于客户端模式或者单核CPU环境。
  14. **-XX:+UseParallelGC**：使用新生代并行收集器，不影响老年代垃圾回收，多线程执行新生代GC。
  15. **-XX:+UseParallelOldGC**：使用老年代并行收集器，多线程执行老年代GC。
  16. **-XX:+UseConcMarkSweepGC**：使用CMS垃圾收集器,以获取最短回收停顿时间为目标，大部分GC阶段可与用户线程并发执行。
  17. **-XX:+UseG1GC**：使用G1收集器进行垃圾回收。
  18. **-XX:ParallelGCThreads**：设置GC线程数，默认值为CPU核心数。
  19. **-XX:+UseZGC**：使用ZGC进行垃圾回收，目标是将GC停顿时间控制在几号秒甚至更短时间内，适用于超大堆场景。
20. **类加载配置**
  21. **-verbose:class**打印类加载信息
22. **日志相关参数**
  23. **-XX:+PrintGCDetails**：打印GC详细信息。
  24. **-XX:+PrintGCTimeStamps**：打印GC时间戳。
  25. **-XX:PrintGCDateStamps**：打印GC日期和时间。
  26. **-Xloggc:/path/to/gc-%t.log**：指定GC日志文件路径。%t表示当前时间。
27. 处理OOM
  28. **-XX:+HeapDumpOnOutOfMemoryError**：出现OOM时自动生成堆快照。
  29. **-XX:HeapDumpPath=/path/to/heapdump.hprof**：指定OOM时生成的堆快照文件路径。
##  知识图解

1. **运行时数据区示意图**  知识扩展

  1. 扩展：
    2. **GC调优策略**：
      1. 因为新生代垃圾回收的成本低，所以需要尽可能让新创建的对象在新生代分配内存并被回收，避免频繁Full GC。可以通过分析GC日志判断新生代空间分配是否合理。
    3. **内存泄漏**：
      4. 程序在运行过程中不再使用的对象仍然被引用，无法被垃圾回收，导致可用内存减少。
      5. 常见原因：
        1. **静态集合**：使用静态数据结构存储对象，没有进行清理。
        2. **事件监听**：未取消对事件源的监听，导致对象持续被引用。
        3. **线程**：未停止的线程可能持有对象引用，无法被回收。
    6. **内存溢出**(OOM)：
      7. JVM申请内存时无法找到足够的内存，引发OutOfMemoryError。
      8. 出现原因：
        1. **堆内存溢出**：代码中出现**大对象分配**或者是**内存泄漏**时，多次GC也没有足够的内存空间。
        2. **栈溢出**：代码中出现**递归调用**，压栈过深或者无法扩展栈空间时出现。
        3. **元空间溢出**：系统的**代码过多**或加载的类文件过多，导致元空间内存占用大。
  9. 面试官可能追问：
  2. Q1：秒杀高并发场景下你怎么进行参数配置？
    1. 秒杀高并发场景下需要**避免GC停顿和内存浪费**，采取固定堆大小，放大新生代和压缩老年代的方式。
    2. 将 **-Xms和-Xmx设置为相同值**，可以避免JVM在-Xms<-Xmx时进行动态扩容而触发Full GC，导致额外停顿，确保内存分配高效。
    3. 将 **-Xmn设置为-Xms的1/2到2/3之间**，秒杀会创建大量的临时对象，这些对象生命周期短，应该优先在新生代回收，防止对象过早的进入老年代，进而触发Full GC。还应该对新生代采用复制算法，可以容纳峰值时的短期对象。
    4. 可以**压缩老年代的空间**，因为秒杀场景下长期存活的对象少，尽量让对象在新生代回收可以减少停顿。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## Java常见集合类｜高频面试题｜Java高频面试题｜Collection、Map、List、Set、HashMap

#  Java常见集合类

##  简要回答

1. **`Collection` 接口（单列集合）**：
  2. 单列集合用于存储单个元素。
  3. 主要的子接口包括：
 ① **`List`** ：它的特点是“**有序、可重复**”。List接口的常见实现类有：`ArrayList`, `Vector` 和 `LinkedList`。
 ② **`Set`** ：Set集合的特点是“**无序、不可重复**”。Set接口的常见实现类有：`HashSet`, `TreeSet`。
 ③ **`Queue`** ：队列，它的特点是先进先出（FIFO）。Queue接口的常见实现类有：`PriorityQueue`。
4. **`Map` 接口（双列集合）**：
  5. 用于存储**键值对**（**Key-Value Pair**）。特点是“**键（Key）唯一，值（Value）可重复**”。
  6. Map接口的常见实现类有：：`HashMap`, `Hashtable`, `TreeMap`。

---

##  详细回答

###  `Collection` 接口（单列集合）

1. **定义**：`Collection` 是**所有单列集合的根接口**，用于存储一系列不重复或重复的单个元素。它定义了所有单列集合的通用行为，如添加、删除、判断元素是否存在等。
2. **分类**：
  3. **`List` 接口**：
 Δ **特点**：元素**有序**（按插入顺序），元素**可重复**。可以通过索引访问元素。
 Δ **常见的实现类**：
 ① **`ArrayList`** ：底层基于**动态数组**实现。特点是**查询（随机访问）速度快**，因为可以通过索引直接定位；但**增删元素（特别是中间位置）效率相对较低**，因为可能会涉及到大量元素的移动。ArrayList类是**非线程安全的**。
 ② **`LinkedList`** ：底层基于**双向链表**实现。特点是**增删元素（特别是首尾和中间位置）效率高**，因为只需修改结点指针；但**查询（随机访问）效率较低**，因为需要从头或尾遍历。LinkedList也是**非线程安全的**。
 ③ **`Vector`** ：Vector类与 `ArrayList` 类似，但它是**线程安全**的（方法都加了 **`synchronized`** 关键字）。但由于**同步开销**，其性能通常低于 `ArrayList`。
  4. **`Set` 接口**：
 Δ **特点**：元素**无序**（不保证插入顺序），元素**不可重复**（通过 `hashCode()` 和 `equals()` 方法判断）。
 Δ **常见的实现类**：
 ① **`HashSet`** ：底层其实是HashMap。特点是**增删改查效率高**，但元素**无序**。HashSet是**非线程安全的**。
 ② **`LinkedHashSet`** ：继承自 `HashSet`，底层基于**哈希表和链表**实现。它在 `HashSet` 的基础上，通过链表维护了元素的**插入顺序**。LinkedHashSet也是**非线程安全的**。
 ③ **`TreeSet`** ：底层基于**红黑树**实现。TreeSet类的特点是元素**有序**（按自然排序或自定义比较器排序），增删改查效率相对 `HashSet` 略低（对数时间复杂度）。TreeSet也是**非线程安全的**。
  5. **`Queue` 接口**：
 Δ **特点** ：遵循 **先进先出（FIFO）** 原则，常用于模拟队列数据结构。
 Δ **常见的实现类** ：
 ① **`PriorityQueue`**：**优先级队列**，元素根据其自然排序或自定义比较器进行排序，每次取出的是优先级最高的元素。PriorityQueue是**非线程安全的**。
###  `Map` 接口（双列集合）

1. **特点**：
  2. **键（Key）是唯一的**，用于快速查找对应的值。
  3. **值（Value）可以重复**。
  4. **一个键最多映射一个值**。
5. **常见的实现类**：
  6. **`HashMap`** ：底层基于**哈希表**实现。它的特点是**查询、增删效率高**，但元素（键值对）**无序**。HashMap是**非线程安全的**。
  7. **`LinkedHashMap`** ：继承自 `HashMap`，底层基于**哈希表 和 双向链表**实现。它在 `HashMap` 的基础上，通过链表维护了键值对的**插入顺序**。LinkedHashMap是**非线程安全的**。
  8. **`TreeMap`** ：底层基于**红黑树**实现。它的特点是键值对**有序**（按键的自然排序或自定义比较器排序），增删改查效率相对 `HashMap` 略低。TreeMap是**非线程安全的**。
  9. **`Hashtable`** ：与 `HashMap` 类似，但它是**线程安全**的（方法都加了 `synchronized` 关键字），性能较低，是 Java 早期版本提供的集合类。**Hashtable的键和值都不能为 `null`** 。
  10. **`ConcurrentHashMap`** ：在 `JDK1.5` 之后提供的**线程安全且高性能**的双列集合实现。它通过分段锁（JDK7）或 `CAS + synchronized`（JDK8）等机制，提供了比 `Hashtable` 更好的并发性能。

---

##  知识拓展

###  图解

1. **Java集合类的选择策略**示意图如下：
  集合与数组的区别：

  1. **元素类型**：
    2. **集合**：只能存储**引用类型**。当存储基本类型时，会自动进行**装箱**（Autoboxing）转换为对应的包装类。
    3. **数组**：既可以存储**基本类型**，也可以存储**引用类型**。但一个数组中**不能混用**基本类型和引用类型（即数组的元素类型是固定的）。
  4. **元素个数（长度）**：
    5. **集合**：**不固定**。集合的容量可以根据需要动态地进行扩容或缩减，可以随时添加或删除元素。
    6. **数组**：**固定**。数组的长度一旦指定，在创建后就不能再更改。如果需要改变数组长度，只能创建新数组并复制旧数组的元素。
###  集合的好处：

  1. **动态容量**：集合不受容器大小限制，可以随时添加或删除元素，可以动态地保存任意多个对象，无需预先指定大小。这使得集合在处理不确定数量数据时非常灵活。
  2. **丰富操作**：集合提供了大量操作元素的方法（例如 `add()`, `remove()`, `contains()`, `size()`, `isEmpty()` 等），以及各种遍历方式（迭代器、增强for循环、Stream API），使得数据的操作和管理更加高效。

---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Spring AOP｜高频面试题｜Java高频面试题｜AOP、动态代理、切面、通知

#  Spring AOP

##  简要回答

- AOP是**面向切面**的编程，允许开发者**在不改动业务代码的情况下横向切入添加新的功能**，比如日志。切面包含很多类型和对象，AOP以切面为单位对它们进行模块化管理，可以减少系统重复代码，降低模块间的耦合度。
- AOP使用Java的**动态代理机制**，能够在运行时动态的创建代理对象，开发者在不修改源码的情况下增强方法功能，动态代理包含基于**JDK**的动态代理和基于CGLIB的动态代理。
##  详细回答

###  AOP概念

- AOP(Aspect Oriented Programming)，面向切面的编程，是面对对象编程的完善。**面向对象编程引入封装，继承和多态的概念建立对象层次，但是无法定义横向关系。**以日志功能为例，日志代码会横向散布在所有对象层次中，对应对象的核心功能毫无关系，这些代码就是横切，在面对对象设计中，会产生大量代码的重复，不利于各个模块的重用。
- AOP技术可以将日志、安全性、异常处理这些影响了多个类的公共行为封装到一个可重用模块中，命名为**切面(Aspect)**。切面将这些**与业务无关，却为业务模块所共同调用的逻辑或责任**封装起来，便于减少系统的重复代码，降低模块之间的耦合度，有利于未来的可操作性和可维护性。
- 使用"横切"技术， AOP把软件系统分为两个部分:核心关注点和横切关注点。业务处理的主要流程是核 心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。 **AOP 的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。**
###  AOP重要术语

1. **横切关注点**: 对哪些方法进行拦截，拦截后怎么处理。
2. **切面**( aspect):类是对物体特征的抽象，切面就是对横切关注点的抽象，切面对横切关注点进行封装。
3. **连接点**( joinpoint ):程序执行中被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring中连接点指的就是被拦截到的方法调用或者执行时的特定时间，实际上连接点还可以是字段或者构造器。
4. **切入点**( pointcut ):对连接点进行拦截的定义，即筛选规则。
5. **通知**( advice ):指的是拦截到连接点之后要执行的代码，通知分为前置(Before)、后置(After)、返回(AfterReturning)、异常(AfterThrowing)、环绕通知(Around)五类。前四种通知在目标方法前后执行，环绕通知可以控制目标方法执行过程。 通知类型 执行时机 前置通知(Before) 目标方法执行**之前**执行 后置通知(After) 目标方法执行**之后**执行(无论是否抛出异常) 返回通知(AfterReturning) 目标方法**正常执行完毕并返回结果**后执行 异常通知(AfterThrowing) 目标方法**执行过程中抛出异常**时执行 环绕通知(Around) 包裹目标方法，在目标方法执行前后均可执行逻辑，还能控制目标方法是否执行、修改返回值等，功能最强
1. **目标对象**:代理的目标对象。
2. **代理对象**:代理对象会执行切面的通知逻辑，再调用目标对象的原始方法。
3. **织入**( weave ):将切面应用到目标对象最后生成代理对象的过程。
4. **引入**( introduction)在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段。
###  Spring中的AOP

-

Spring中的AOP代理由 Spring 的 **IOC 容器**负责生成、管理，其依赖关系也由IOC容器负责管理。因此，AOP代理可以直接使用容器中的其它 bean 实例作为目标，这种关系可由 IOC 容器的依赖注入提供。

-

spring 创建代理的规则为:

  1. 默认使用 JDK动态代理来创建AOP代理，这样就可以为任何接口实例创建代理了
  2. 当需要代理的类不是代理接口的时候， Spring 会切换为使用CGLIB代理，也可强制使用CGLIB
-

AOP编程实现步骤：

  1. 定义普通业务组件。
  2. 定义切入点，一个切入点可能横切多个业务组件
  3. 定义增强处理，增强处理就是在 AOP框架为普通业务组件织入的处理动作
  4. 一旦定义了合适的**切入点**和**增强处理**， AOP框架将自动生成 AOP代理，即: 代理对象的方法=增强处理+被代理对象的方法。
###  动态代理的方式

1.

**基于JDK**的动态代理：只提供接口的代理，不支持类的代理。核心接口是InvocationHandler和Proxy类。

  2. InvocationHandler通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起。Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。
3.

**基于CGLIB**的动态代理：代理类没有实现InvocationHandler接口时，SpringAOP使用CGLIB动态代理目标类。

  4. CGLIB是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，覆盖其中特定方法并添加增强代码以实现AOP。
  5. CGLIB通过**继承**方式做动态代理，如果某个类被标记为final则无法使用CGLIB。
##  知识图解

1. AOP不同类型通知时机  知识扩展

  1. 面试官可能追问？
    2. 动态代理和静态代理有什么区别？
      3. 代理是一种常用的设计模式，为其他对象提供一个代理以控制对某个对象的访问，解耦两个类的关系。
      4. **静态代理**是由程序员创建或由特定工具创建，在代码编译时就确定了被代理的类。
      5. **动态代理**是在代码运行期间，运用反射机制动态创建生成的，动态代理的是一个接口下的多个实现类。
    6. AOP有哪些注解？
      7. @ **Aspect**：定义切面，标注在切面类上。
      8. @ **Pointcut**：定义切点，标注在方法上，用于指定连接点。
      9. @ **Before**：在方法被调用之前执行通知。
      10. @ **After**：在方法被调用后之后执行通知。
      11. @ **Around**：任意的在方法调用前后执行。
      12. @ **AfterReturning**：在方法执行返回结果后执行通知。
      13. @ **AfterThrowing**：在方法抛出异常后执行通知。
      14. @ **Advice**：通用的通知类型，替代@ Before、@ After。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Java异常类型｜高频面试题｜Java高频面试题｜Error、Exception、编译期异常、运行期异常

#  Java异常类型

##  简要回答

- Java 中的异常都继承自 `java.lang.Throwable` 类，它分为两大类：`Error` 和 `Exception`。
  1. **`Error`(错误)** ：
 表示 **JVM 内部** 或 **系统级别**的问题，通常是很严重的，程序无法恢复。例如：`OutOfMemoryError` (内存溢出)、`StackOverflowError` (栈溢出)。
  2. **`Exception`(异常)** ：
 表示程序在 **编译时** 或 **运行时** 可能发生的问题，通常是**可捕获和处理**的。`Exception` 又分为两大子类：
 ① **编译期异常 (Checked Exception)** ：是指编译器**强制检查**的异常，必须进行 `try-catch` 捕获处理 或者 `throws` 声明处理，常见的编译期异常有 `IOException` (输入输出异常)、`SQLException` (数据库操作异常)。
 ② **运行期异常 (Runtime Exception / Unchecked Exception)** ：是指编译器**不强制检查**的异常，通常是程序逻辑错误导致的，可选择性处理，常见的运行期异常有 `NullPointerException` (空指针异常)、`ArrayIndexOutOfBoundsException` (数组越界异常)。

---

##  详细回答

- Java 中所有的异常都源于 `java.lang.Throwable` 类，它有两个主要的直接子类：`Error` 和 `Exception`。
1. **`Error`(错误)** ：
  2. **定义**：`Error` 类及其子类表示了在 **程序运行期间发生的**、**JVM** 或 **系统级别的** 严重问题，这些问题通常是应用程序**无法预料和恢复**的。例如，JVM 内存耗尽、JVM 错误、线程死锁等。
  3. **特点**：`Error` 是**非受检查异常 (Unchecked Exception) 的一种特殊形式**，编译器不会强制捕获或声明。由于它们通常是致命的，即使捕获了也难以恢复，因此在开发中，通常不会去捕获 `Error`，而是让程序终止，以便系统管理员介入处理。
  4. **常见类型**：
 ① `java.lang.OutOfMemoryError`：是指当JVM**内存不足**，无法分配对象时抛出。
 ② `java.lang.StackOverflowError`：是指当方法调用**栈溢出**时抛出，通常是由于无限递归导致。
 ③ `java.lang.NoClassDefFoundError`：当JVM试图加载某个类，但在**运行时找不到该类的定义**时抛出。
 ④ `java.lang.VirtualMachineError`：**虚拟机**在运行时发生的**内部错误**。
5. **`Exception`(异常)** ：
  6. **定义**：`Exception` 类及其子类表示程序在运行过程中可能遇到的、**可以被捕获和处理**的异常情况。这些异常通常是由于**外部因素**（如文件不存在、网络中断）或**程序逻辑错误**导致的。
  7. **特点**：`Exception` 是我们日常开发中主要关注和处理的异常类型。它进一步分为两大类：编译期异常 和 运行期异常。
  8. **编译期异常 (Checked Exception / 受检查异常)** ：
    9. 这类异常是指所有**直接或间接继承自 `java.lang.Exception`，但不是 `java.lang.RuntimeException` 及其子类**的异常。它们之所以被称为“编译期异常”或“受检查异常”，是因为Java编译器会**强制检查**你是否对这类异常进行了处理（例如 使用 `try-catch` 块捕获，或 使用 `throws` 关键字抛出），如果未处理或声明，代码将无法编译通过。
    10. 编译期异常通常表示**程序外部环境**可能出现的问题，**是可预见的、可恢复的**。所以开发者在编写代码时，需要明确地考虑这些异常，并提供相应的处理逻辑。
    11. **常见类型**：
 ① `java.io.IOException`：是指所有 **I/O操作**（如文件读写、网络通信）可能抛出的异常的基类。
 ② `java.io.FileNotFoundException`：是指尝试**打开一个不存在的文件**时抛出的异常。
 ③ `java.sql.SQLException`：在**数据库访问操作**中发生的异常。
 ④ `java.lang.ClassNotFoundException`：是指当应用程序试图**通过类的字符串名称加载类**，但找不到该类定义时抛出。
 ⑤ `java.lang.InterruptedException`：当**一个线程**在等待、休眠或占用途中，**被另一个线程中断**时抛出。
  12. **运行期异常 (RuntimeException / Unchecked Exception / 非受检查异常)** ：
    13. 这类异常是 **`RuntimeException` 类及其所有子类**。它们被称为“运行期异常”或“非受检查异常”，是因为编译器**不会强制检查**你是否捕获或声明它们，因为即使不处理，代码也能编译通过。
    14. 运行期异常通常表示**程序内部**的逻辑错误、编程缺陷或不合法的参数。这类异常通常是由于程序员的疏忽导致的，例如，尝试访问空对象的成员、数组越界等。
    15. **常见类型**：
 ① `java.lang.NullPointerException`：是指尝试**对一个 `null` 对象进行操作**时抛出的异常。
 ② `java.lang.ArrayIndexOutOfBoundsException`：是指**数组索引**超出其有效范围时抛出的异常。
 ③`java.lang.ClassCastException`：是指在进行**强制类型转换**时，如果对象不是目标类型的实例，则抛出。
 ④ `java.lang.IllegalArgumentException`：是指向方法传递了一个**不合法或不正确的参数**时抛出的异常。
 ⑤ `java.lang.NumberFormatException`：是指尝试将一个**不符合数字格式**的字符串转换为数字时抛出的异常。
 ⑥ `java.lang.ArithmeticException`：是指发生**不合法的算术运算**时抛出的异常，例如除以零。
 ⑦ `java.lang.IndexOutOfBoundsException`：是指**所有索引超出范围**异常的基类（包括数组、字符串、集合等）。

---

##  知识拓展

1. **Java异常体系图**，如下所示：
   →

### 评论
 

验证登录状态...

## HashMap的put方法｜高频面试题｜Java高频面试题｜putVal、哈希冲突、扩容、红黑树

#  HashMap的put方法

##  简要回答

1. **计算哈希值和索引：**
  2. 对传入的 `key` 计算哈希值，并通过扰动函数和位运算确定其在底层数组中的**存储位置**（索引）。
3. **判断位置是否为空：**
  4. 如果该索引位置**为空**，直接创建新结点并放入。
  5. 如果该索引位置**不为空**（发生**哈希冲突**），则需要进一步处理。
6. **处理哈希冲突：**
  7. **Key已存在：** 如果链表/红黑树中已存在相同的 `key`，则放弃添加元素，更新其对应的 `value`，并返回旧值。
  8. **Key不存在：** 在链表末尾添加新结点。
  9. **链表转红黑树：** 如果链表长度达到阈值（默认为 8），会尝试将链表转换为红黑树，以提高查找效率。
10. **扩容检查：**
  11. 元素添加成功后，会检查 `HashMap` 的当前元素数量 `size` 是否超过了扩容阈值 `threshold`。如果超过，则进行扩容操作，将底层数组的容量翻倍，并重新计算所有元素的索引位置。

---

##  详细回答（结合 JDK 17.0 源码）

###  HashMap源码解读

1.

`HashMap` 的 `put` 方法内部调用了 `putVal` 方法。 `putVal` 方法的源码流程如下：


```
`final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i; // 定义局部变量：tab(当前哈希表数组), p(当前索引位置的第一个结点), n(数组长度), i(计算出的索引)

    // 1. 检查并初始化/扩容哈希表数组
    // 如果当前哈希表为空或长度为0，说明是第一次put或者之前被清空了，需要进行初始化或扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length; // 调用 resize() 方法进行初始化或扩容，并获取新的数组长度

    // 2. 计算索引位置并尝试直接插入
    // (n - 1) & hash 是计算索引位置的核心逻辑，利用位运算替代取模，效率更高。
    // 如果计算出的索引位置 tab[i] 为空，则直接创建新结点并放入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null); // null 表示没有下一个结点
    else {
        // 3. 处理哈希冲突（索引位置不为空）
        Node<K,V> e; K k; // e 用于存储找到的匹配结点，k 用于存储当前结点的key

        // 3.1 检查链表/红黑树的第一个结点是否就是目标key
        // 如果第一个结点的哈希值和key都匹配（key相同或equals），则 e 指向该结点
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 3.2 如果第一个结点是红黑树结点（说明该桶已树化）
        else if (p instanceof TreeNode)
            // 调用红黑树的 putTreeVal 方法进行插入或更新
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 3.3 如果是链表（非红黑树）
        else {
            // 遍历链表，查找是否存在相同的key
            for (int binCount = 0; ; ++binCount) { // binCount 统计链表长度
                if ((e = p.next) == null) { // 如果遍历到链表末尾，没有找到相同key
                    p.next = newNode(hash, key, value, null); // 在链表末尾添加新结点
                    // 检查链表长度是否达到树化阈值（TREEIFY_THRESHOLD - 1），如果达到，则尝试树化
                    // -1 是因为 binCount 是从0开始计数，且当前结点是第 binCount+1 个
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash); // 尝试将链表转换为红黑树
                    break; // 插入完成，跳出循环
                }
                // 如果找到相同key的结点
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break; // 找到，跳出循环，e 指向该结点
                p = e; // p 移动到下一个结点，继续遍历
            }
        }

        // 4. 更新已存在的key的value
        // 如果 e 不为 null，说明在链表或红黑树中找到了与新key相同的旧key
        if (e != null) { // existing mapping for key
            V oldValue = e.value; // 保存旧值
            // 如果 onlyIfAbsent 为 false (put方法默认是false，表示总是更新)
            // 或者旧值为 null (表示旧key存在但value是null，也允许更新)
            if (!onlyIfAbsent || oldValue == null)
                e.value = value; // 更新value
            afterNodeAccess(e); // 钩子方法，用于LinkedHashMap等
            return oldValue; // 返回旧值
        }
    }

    // 5. 插入成功后的通用处理
    ++modCount; // 结构修改次数加1，用于迭代器快速失败
    if (++size > threshold) // 插入后，如果当前元素数量 size 超过了扩容阈值 threshold
        resize(); // 调用 resize() 方法进行扩容
    afterNodeInsertion(evict); // 钩子方法，用于LinkedHashMap等
    return null; // 返回null表示没有旧值被替换
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64

2.

**`resize()` 方法的流程（扩容）：** 当 `size` 超过 `threshold` 或者 `HashMap` 首次初始化时，会调用 `resize()` 方法。


```
`final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table; // 旧的哈希表数组
    int oldCap = (oldTab == null) ? 0 : oldTab.length; // 旧的容量
    int oldThr = threshold; // 旧的扩容阈值
    int newCap, newThr = 0; // 新的容量和新的扩容阈值

    // 1. 计算新的容量和阈值
    if (oldCap > 0) { // 如果旧容量大于0（非首次初始化）
        if (oldCap >= MAXIMUM_CAPACITY) { // 如果旧容量已达到最大容量
            threshold = Integer.MAX_VALUE; // 阈值设为最大值，不再扩容
            return oldTab; // 返回旧表
        }
        // 新容量为旧容量的两倍，且不超过最大容量
        // 新阈值为旧阈值的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 首次初始化，但指定了初始容量（initial capacity was placed in threshold）
    else if (oldThr > 0)
        newCap = oldThr;
    // 首次初始化，使用默认值
    else {
        newCap = DEFAULT_INITIAL_CAPACITY; // 默认初始容量 16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 默认加载因子 0.75 * 16 = 12
    }

    // 2. 确保新阈值被正确计算
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr; // 更新 HashMap 的扩容阈值

    // 3. 创建新的哈希表数组
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab; // 将 HashMap 的内部数组指向新创建的数组

    // 4. 将旧表中的元素转移到新表
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) { // 遍历旧表的每个桶
            Node<K,V> e;
            if ((e = oldTab[j]) != null) { // 如果当前桶不为空
                oldTab[j] = null; // 清空旧桶，帮助GC
                if (e.next == null) // 如果桶中只有一个结点
                    newTab[e.hash & (newCap - 1)] = e; // 直接将其放到新表中对应的位置
                else if (e instanceof TreeNode) // 如果是红黑树
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); // 调用红黑树的split方法进行分裂
                else { // 如果是链表（非红黑树）
                    // 链表拆分优化：将链表中的结点分成两部分
                    // 基于 (e.hash & oldCap) == 0 判断，将结点分配到原索引或新索引 (原索引 + oldCap)
                    Node<K,V> loHead = null, loTail = null; // 存储在新索引 j 的链表头尾
                    Node<K,V> hiHead = null, hiTail = null; // 存储在新索引 j + oldCap 的链表头尾
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 关键优化：判断结点的哈希值在旧容量二进制表示下，最高位是否为0
                        // 如果为0，则在新数组中的索引位置不变 (j)
                        // 如果为1，则在新数组中的索引位置变为 j + oldCap
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null); // 继续遍历链表
                    // 将拆分后的两个链表分别挂载到新数组的对应位置
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab; // 返回新的哈希表数组
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89

###  `putVal` 的核心流程：

1.

**计算所添加元素的哈希值和索引值**

  2.

当调用 `put(key, value)` 时，首先会对 `key` 进行哈希计算。`HashMap` 内部会调用 `key.hashCode()` 获取原始哈希值，然后通过一个**扰动函数**对其进行处理，以减少哈希冲突的概率，使哈希值分布更均匀。最终，这个处理后的哈希值（`int hash`）会被传递给 `putVal` 方法。

  3.

如果 `HashMap` 尚未初始化（`table` 为 `null`）或者其容量为 0，会首先调用 `resize()` 方法进行初始化。`resize()` 方法会创建一个默认容量（16）的数组，或根据构造函数指定的容量进行初始化。

  4.

接着，通过位运算确定 `key` 在数组中的 **索引位置 `i`** ，代码如下：


```
`// 计算索引位置
if ((p = tab[i = (n - 1) & hash]) == null) // (n - 1) & hash 是核心的索引计算逻辑
    tab[i] = newNode(hash, key, value, null); // 如果该位置为空，直接插入新结点
`
```
 1
2
3

  5.

这里的 `(n - 1) & hash` 是一个高效的位运算，等价于 `hash % n`，但仅适用于 `n` 是 2 的幂次方的情况。它将哈希值映射到数组的有效索引范围内。如果计算出的索引位置 `tab[i]` 当前为 `null`，说明该桶是空的，没有发生哈希冲突，此时会直接创建一个新的 `Node` 并将其放入该位置。

6.

**发生哈希冲突时的处理情况**

  7. 如果 `tab[i]` 不为 `null`，则表示发生了**哈希冲突**，需要进一步处理。
  8. **检查首结点：** 此时，HashMap 会首先判断 **`tab[i]`** 的**第一个结点** `p` 是否与待插入的 `key` **相同**（通过哈希值和 `equals()` 方法判断）。如果相同，则同为Node<K,V>类型的 `e` 会指向 `p`，后续会更新它的 `value`。
  9. **红黑树处理：** 如果 `p` 是 `TreeNode` 类型（表示该桶已经转换为**红黑树**），则会调用 `TreeNode` 的 `putTreeVal` 方法，以红黑树的逻辑进行插入或更新操作，保证 **O(logN)** 的查找效率。
  10. **链表处理：** 如果是**普通链表**，则会遍历链表。在遍历过程中，如果找到与待插入 `key` 相同的结点，则 `e` 指向该结点，并在后续更新它的 `value`，用新值替换旧值。如果遍历到链表末尾仍未找到相同 `key`，则在链表的末尾添加新的 `Node`。
  11. **树化检查：** 在链表末尾添加新结点后，会立即检查当前链表的长度（`binCount`）。如果链表长度达到 `TREEIFY_THRESHOLD` (默认为 8)，并且数组容量也满足树化条件（默认为 64），则会调用 `treeifyBin` 方法，将整个链表转换为红黑树，以优化后续的查找性能。
  12. **关于value更新的决策：** 对于有相同key的情况，HashMap 根据 `onlyIfAbsent` 参数的设置，决定是否更新 `key` 对应的 `value`。默认情况下 (`onlyIfAbsent` 为 `false`)，旧值会被新值覆盖。
13.

**扩容检查**

  14.

无论是否发生哈希冲突，只要成功插入了一个新的键值对（即 `e` 为 `null`，表示没有替换旧值），`HashMap` 的 `size` 都会增加。所以在 `putVal` 方法的最后，HashMap 会进行扩容检查。代码如下：


```
`// 插入成功后的通用处理
++modCount; // 结构修改次数加1
if (++size > threshold) // 如果当前元素数量超过扩容阈值
    resize(); // 调用 resize() 方法进行扩容
// ... (afterNodeInsertion 钩子方法)
return null; // 返回null表示没有旧值被替换
`
```
 1
2
3
4
5
6

  15.

`modCount` 会增加，用于 `Iterator` 的快速失败机制。

  16.

`size` 也会增加。但如果 `size` 超过了 `threshold`（扩容阈值，通常是 `容量 * 加载因子`），就会触发 `resize()` 方法。


---

##  知识图解

1. HashMap解决哈希冲突的方案，如下图所示：
  知识拓展

  1. **面试官可能的追问 1：在 `resize()` 方法中，为什么链表在扩容时，元素只会分到两个位置（原位置 `j` 或 `j + oldCap`），而不是完全重新计算哈希值？这种优化是如何实现的？**
    2. 这种优化是基于哈希值和扩容特性实现的。当容量从 `oldCap` 翻倍到 `newCap` (`2 * oldCap`) 时，一个元素在新数组中的索引位置只可能保持不变或变为 `原索引 + oldCap`。
    3. 这是因为 `newCap` 总是 `oldCap` 的两倍，且两者都是 2 的幂。通过检查元素的 `hash` 值在 `oldCap` 位上的二进制位是 0 还是 1 (`(e.hash & oldCap) == 0`)，就可以直接判断它应该去新数组的哪个位置，避免了重新计算 `(newCap - 1) & hash`，从而大幅提高了扩容时链表元素重新分配的效率。
  4. **面试官可能的追问 2：`putVal` 方法中，`onlyIfAbsent` 和 `evict` 这两个参数的作用是什么？`afterNodeAccess` 和 `afterNodeInsertion` 钩子方法又是用来做什么的？**
    5. **`onlyIfAbsent`：** 这是一个布尔参数，控制是否只在键不存在时才插入。如果为 `true`，则当 `key` 已存在时，不会更新其 `value`；如果为 `false` (默认值)，则会更新 `value`。
    6. **`evict`：** 这是一个布尔参数，通常用于 `LinkedHashMap`。它指示在插入结点后是否需要执行逐出策略（例如删除最不常用的结点）。对于 `HashMap` 本身，这个参数通常不发挥作用。
    7. **`afterNodeAccess` 和 `afterNodeInsertion`：** 这些是**钩子方法 (hook methods)** ，它们是 `HashMap` 提供给其子类（如 `LinkedHashMap`）进行扩展的。 **`afterNodeAccess`** 是在**结点被访问**（如 `get` 或 `put` 更新值）后调用，`LinkedHashMap` 利用它来调整结点的访问顺序，实现 LRU (Least Recently Used) 缓存策略。 **`afterNodeInsertion`** 则是在**结点被成功插入后**调用，`LinkedHashMap` 利用它来维护其双向链表结构。 这些钩子方法使得 `LinkedHashMap` 可以在不修改 `HashMap` 核心逻辑的情况下，实现额外的功能。

---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 垃圾回收器｜大厂面试题｜Java高频面试题｜GC、CMS、G1、回收算法

#  垃圾回收器

##  简要回答

- 新生代收集器：Serial、ParNew、Parallel Scavenge
- 老年代收集器：Serial Old、Parallel Old、CMS
- 整堆收集器（新生代和老年代）：G1
- 我们需要根据不同的场景选择适合的垃圾回收器，在JDK8中默认的垃圾回收器是Parallel Scavenge和Parallel Old，在JDK9之后是G1。
##  详细回答

-

新生代垃圾收集器：

  1. **Serial** 串行收集器-复制算法：新生代的单线程收集器，简单高效，没有线程交互的开销，在垃圾收集时必须暂停其他所有的工作线程直到收集完成。Serial收集器是虚拟机运行在Client模式下默认新生代收集器。
  2. **ParNew** 并行收集器-复制算法：是新生代的并行收集器，Serial的多线程版本，包括Serial收集器的所有控制参数、算法、对象分配规则、回收策略等。唯一支持与CMS并行的新生代收集器。
  3. **Parallel Scavenge** 并行收集器-复制算法：是新生代的并行收集器，追求高吞吐量和高效利用CPU，该收集器的目标是达到一个可控制的吞吐量。可以自适应调节新生代大小等参数，无需手动调优。
-

老年代垃圾收集器：

  1. **Serial Old** 串行收集器-标记整理算法：是Serial收集器的老年代版本，用于Client模式下的虚拟机。如果在Server模式下可以用在JDK1.5及之前和Parallel Scavenge收集器搭配使用，也可以作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用。
  2. **Parallel Old** 并行收集器-标记整理算法：是Parallel Scavenge收集器的老年代版本，在JDK1.6及之后开始提供。
  3. **CMS(Concurrent Mark Sweep)** 收集器-标记清除算法：追求最短的回收停顿时间，适合重视服务器响应速度的应用，高并发且低停顿，用户体验好。收集过程可以分为初始标记、并发标记、重新标记和并发清除。初始标记和重新标记需要暂停其他工作线程。
    4. **初始标记**会短暂停顿，标记直接与root相连的对象。
    5. **并发标记**会同时开始GC和用户线程，记录可达对象，不需要停顿。
    6. **重新标记**能够修正并发标记期间因为用户程序继续运行而导致标记变动的对象，需要停顿。
    7. **并发清除**会开启用户线程同时GC线程对未标记的区域清扫，不需要停顿。清除阶段新产生的垃圾会等到下一次GC处理。
    8. CMS对CPU资源敏感，并发阶段需要多个线程和用户线程并行工作，会占用部分CPU资源，在CPU核心较少情况下可能会导致用户线程执行效率下降；基于标记清除算法，回收后会产生内存碎片；清除阶段老年代空间不足会导致Concurrent Mode Failure，JVM会启用Serial Old进行回收；为了避免上述情况，会限制对内存的使用，内存利用率降低。
-

新生代和老年代垃圾收集器：**G1 收集器**

  1. G1收集器在JDK1.7被引入，后面取代CMS为默认收集器。
  2. 将**堆内存分为多个区域(Region)** ，维护一个优先列表，根据允许的收集时间选择回收价值最大的Region，尽可能提高收集效率；可以充分利用CPU资源缩短停顿时间；筛选回收阶段的复制算法可以避免内存碎片，尽可能满足GC停顿时间要求的同时具备高吞吐量性能特征。
  3. G1收集器回收过程：分为初始标记，并发标记，最终标记和筛选回收四个步骤。
    4. **初始标记**会短暂停顿，标记所有直接可达的对象，**并发标记**会标记所有可达对象，**最终标记**会短暂停顿，处理并发标记阶段中未处理的引用变更，**筛选回收**会停顿，根据标记结果，优先选择回收价值高的区域，复制存活对象到新区域，回收旧区域内存。
##  知识图解

1. CMS回收示意图  使用场景
 垃圾收集器 适用场景 回收类型 使用算法 作用域 特点 Serial 单核CPU下的Client模式 串行回收 复制算法 新生代 响应速度优先 Serial Old 单核CPU下的Client模式 串行回收 标记整理算法 老年代 响应速度优先 ParNew 多核CPU环境中Server模式下与CMS配合使用 并行回收 复制算法 新生代 响应速度优先 Parallel Scavenge 适用于后台运算，交互少的场景 并行回收 复制算法 新生代 吞吐量优先 Parallel Old 适用于后台运算，交互少的场景 并行回收 标记整理算法 老年代 吞吐量优先 CMS 适用于B/S业务，交互多场景 并发回收 标记清除算法 老年代 响应速度优先 G1 面向服务端应用 并发，并行回收 标记整理算法+复制算法 堆内存 响应速度优先
##  知识扩展

  1. 扩展
  2. **吞吐量**：CPU用于运行用户代码的时间与CPU总消耗时间的比值，吞吐量=用户代码运行时间/（用户代码运行时间+垃圾收集时间）。停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验，**高吞吐量可高效利用CPU时间**，加快完成程序运算任务。
  3. 并发：用户线程和垃圾收集线程同时执行。
  4. 并行：多条垃圾收集线程并行工作，用户线程处于等待状态。
  1. 面试官可能追问：
  5. Q1：G1的筛选回收阶段怎么实现“可控停顿”？参数设置过小会怎么样？
    6. G1可以**根据预期停顿时间动态调整回收范围**，G1为每个Region维护垃圾比例与耗时，根据参数 **-XX:MaxGCPauseMillis** 选择高收益的Region进行回收，保证单次STW时间不超过预期值。
    7. 如果参数设置过小，G1只能选择少数Region进行回收，释放内存小于新对象的分配速度，可能会更频繁触发GC，降低用户代码执行效率。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 单例模式｜大厂面试题｜Java高频面试题｜懒汉式、饿汉式、双重检查锁、静态内部类

#  单例模式

##  简要回答

- 单例设计模式是一种确保一个类在运行期间只有一个实例，提供一个**全局访问点**来访问该实例的创建型模式。核心特点如下：
  1. 私有化构造方法，防止外部通过new关键字直接实例化对象。
  2. 类内部自行创建一个私有的静态变量，保存该类的唯一实例。
  3. 提供公共静态方法，给使用者提供调用方法，返回唯一实例。
- 单例模式适用于资源独占、全局统一管理的场景，可以避免一个全局使用的类被频繁创建与销毁，耗费系统资源。
##  详细回答

-

实现单例模式有6种常用的方法：

  1.

**懒汉式（线程不安全）**：等待该类第一次被调用时再创建实例。

    2. 优点是能够**延迟实例化**，只在需要时创建实例，避免资源浪费。
    3. 缺点是线程不安全，在多线程环境下，如果多个线程同时进入了懒汉式单例的创建方法，就可能会有多个线程执行uniqueInstance = new Singleton();导致实例化多个实例。

```
`public class Singleton {
    private static Singleton instance;
    private Singleton (){}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

`
```
 1
2
3
4
5
6
7
8
9
10
11
12

  4.

**饿汉式（线程安全）**：在类加载时就创建实例，需要使用时可以直接调用方法使用。

    5. 优点是可以**提前实例化**好一个实例，避免了线程不安全的问题。
    6. 缺点是直接实例化，没有延迟加载的效果，可能系统没有使用该实例而使得操作系统的资源浪费。

```
`public class Singleton {
    private static Singleton instance;
    private Singleton (){}
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

`
```
 1
2
3
4
5
6
7
8
9
10
11


3.**懒汉类（线程安全）**：在懒汉类的基础上实现线程安全，本质上是给getInstance()方法加锁，只有拿到锁的线程能够进入该方法。

  - 优点是能够延迟实例化、节约资源并且实现线程安全。
  - 缺点是性能降低，因为实例在实例化后依然存在加锁操作，只有拿到锁的线程才能进入会导致线程阻塞。

```
`public class Singleton {
    private static Singleton uniqueInstance;
    private static singleton() {}
    private static synchronized Sing leton getUinqueInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10

  1.

**双重检查锁（线程安全）**：使用双重检查锁实现单例模式能够在多线程环境下保证线程安全并具有高性能，通过**synchronized**保证多线程下的原子性，**volatile**解决指令重排问题。

    2. 优点是双重检查锁可以在延迟实例化的基础上实现线程安全，并且性能高于线程安全的懒汉模式。
    3. 缺点是创建实例的开销可能会高一些，因为使用了volatile关键字，并且需要进行两次检查。

```
`public class DCLSingleton {
    // volatile禁止指令重排，避免多线程下获取到"半初始化"实例
    private static volatile DCLSingleton instance;

    private DCLSingleton() {}

    public static DCLSingleton getInstance() {
        // 第一次检查：避免每次调用都加锁
        if (instance == null) {
            synchronized (DCLSingleton.class) {
                // 第二次检查：防止多线程同时进入第一个if，重复创建实例
                if (instance == null) {
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19

  4.

**静态内部类（线程安全）**：当外部类被加载时，静态内部类不会被加载，调用getInstance()方法时，静态内部类才会被加载，并创建单例实例。此时静态内部类才会被加载进内存，并且初始化INSTANCE实例，通过JVM保证只被实例化一次。

    5. 优点是能够延迟实例化的同时实现线程安全，并且节约资源，性能较高。

```
`public class Singleton {
    private Singleton() {}

    // 静态内部类持有实例
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    // 公共静态方法，返回实例
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}

`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14

  6.

**枚举类实现（线程安全）**：枚举方式实现单例由JVM保证唯一，且枚举类的构造器默认是私有，能够防止反射创建实例，支持序列化机制，防止反序列化重新创建新对象。


```
`public enum Singleton {
    INSTANCE;

    // 可以添加其他方法和属性
    public void doSomething() {
        // 实现
    }
}
`
```
 1
2
3
4
5
6
7
8

##  知识图解

1. 单例模式结构图  使用场景

  2. 单例模式适用于类只能有一个实例且客户可以从一个众所周知的访问点访问它的场景，频繁创建与销毁的对象可以通过单例模式提高性能，经常使用且实例化复杂的对象可以通过单例模式避免重复创建，需要控制共享资源时可以使用单例模式方便通信。
  3. 常见的应用场景如下：
    1. **资源密集型对象的创建**：创建数据库连接池、线程池时可以使用单例模式，避免重复创建浪费系统资源，单例模式可以保证全局仅有一个池实例，进行统一管理连接、线程。
    2. **系统的全局配置**：数据库地址、APIkey等配置资源可以通过单例模式封装，能够保证所有模块获取的配置资源一致，避免重复读取。
    3. **工具类**：例如日志管理工具类和缓存工具类，需要全局统一的入口操作资源，使用单例模式可以避免多实例导致的日志混乱、缓存不一致等问题。
    4. **硬件操作**：硬件资源唯一，单例模式可以保证同一时间只有一个实例操作硬件，避免冲突。
##  知识扩展

  1. 面试官可能追问：
  4. Q1：怎么选择单例模式的实现方式？
    5. 根据业务的实际需求选择，如果不需要实现懒加载，可以使用饿汉式简单实现，如果需要防止反射和序列化则使用枚举方式实现；
    6. 需要懒加载实现单例模式时，对懒汉式加锁可以以性能较低的方式实现，还可以使用静态内部类实现；如果需要传参初始化，可以使用双重检查锁(DCL)实现单例模式。
  7. Q2：为什么双重检查锁需要加 volatile？
    8. instance=new DCLSingleton()并非原子操作，会被拆分为3步：①分配内存；②初始化实例；③给 instance赋值。JVM中可能发生指令重排（如先执行③再执行②），导致多线程下某个线程获取到 “半初始化” 的实例（instance 不为 null，但对象未初始化完成）。volatile关键字可禁止指令重排，保证实例创建的原子性。
  9. Q3：反射 / 序列化会破坏单例吗？如何解决？
    10. 反射破坏：通过Constructor.setAccessible(true) 可强制调用私有构造方法，创建多个实例；使用饿汉式 / 静态内部类可在构造方法中加判断（若 instance 不为 null 则抛异常）；枚举天然防反射，如果反射创建枚举会抛 IllegalArgumentException异常。
    11. 序列化破坏：序列化单例对象后反序列化，会生成新实例；可以通过重写readResolve() 方法解决，返回已有的单例实例；枚举天然防序列化破坏。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Hash冲突解决方案｜高频面试题｜Java高频面试题｜HashMap、红黑树、链地址法

#  Hash冲突解决方案

##  简要回答

###  哈希冲突的通用解决方式

1. **开放地址法**：是指当发生冲突时，在数组中寻找下一个可用的空位。常见的探测方式有**线性探测**、**二次方探测**等。
2. **链地址法（拉链法）**：是指在冲突位置拉出一个数据结构（通常是链表），然后将所有冲突的元素都存放在这个数据结构中。
###  HashMap对于哈希冲突的解决方案

- HashMap采用的是**链地址法**。并且在 JDK 1.8 中进行了优化：当链表的长度超过树化阈值 **8** 时，会自动将链表转换为**红黑树**，以将极端冲突情况下的查询时间复杂度从 O(n) 降低到 O(log n)。

---

##  详细回答

###  哈希冲突的通用解决方式

1. **开放地址法 (Open Addressing)** ：
  2. **概念**：这种方法**不使用任何外部数据结构**，所有的元素都存储在哈希表数组本身。当发生冲突时，它会通过一个 **探测（Probing）算法** 在数组中寻找下一个可用的空槽位。根据探测序列生成方式的不同，又可以细分为下面几种算法：
  3. **线性探测 (Linear Probing)** ：如果位置 `i` 被占用，就依次检查 `i+1`, `i+2`, `i+3`... 对应的位置。这种方式实现简单，缓存友好，但容易导致“二次聚集”（或称堆积），即冲突的元素会堆积在一起，形成连续的占用块，影响后续的查找效率。
  4. **二次方探测 (Quadratic Probing)** ：如果位置 `i` 被占用，就依次检查 `i+1²`, `i-1²`, `i+2²`, `i-2²`...。它能有效地缓解线性探测的堆积问题，但并不能像线性探测法一样探测到散列表的全部存储单元。
  5. **双重哈希 (Double Hashing)** ：使用两个不同的哈希函数。第一个用于计算初始位置，第二个用于计算每次探测的步长。双重哈希能最大程度地减少聚集现象，让元素分布得更均匀。
6. **链地址法 (Separate Chaining)** ：
  7. **概念**：它的核心思想是在哈希表的每一个槽位（桶）上，都维护一个**独立的数据结构**来存储所有映射到该槽位的元素。
  8. **实现方式**：每个桶位通常是一个指向链表头部的指针。当 `put` 一个元素时，先计算出它属于哪个桶，然后将这个元素插入到该桶对应的链表中。
  9. **优点**：实现简单，逻辑清晰；对于哈希函数的选择不那么苛刻；由于冲突的元素被隔离在各自的链表中，不容易产生聚集现象；且哈希表的负载因子可以超过1。
  10. **缺点**：需要额外的空间来存储指针和链表节点；在冲突严重时，链表过长会导致查询性能下降。
###  HashMap对于哈希冲突的解决方案

1. **JDK 1.7及之前：数组 + 纯链表** ：
  2. 在这个版本中，当发生哈希冲突时，HashMap会创建一个新的`Entry`节点，并通过**头插法**将其插入到对应桶位的链表头部。
  3. 选择头插法是因为设计者认为新插入的元素更有可能被立即访问，放在头部可以提高效率。但是，这种纯链表的结构**在哈希冲突严重时，性能会线性下降**，时间复杂度退化为O(n)。
  4. 同时，在**多线程**环境下进行扩容时，头插法可能会导致链表成环，造成死循环。
5. **JDK 1.8及之后：数组 + 链表 / 红黑树** ：
  6. 在链表中插入新元素时，**从头插法改为了尾插法**。这解决了多线程扩容时可能出现的环形链表问题，并为后续的树化转换做好了铺垫。
  7. 引入了**红黑树**。当一个桶中的链表长度达到**树化阈值`TREEIFY_THRESHOLD`（默认为8）**，并且整个哈希表的容量也达到**最小树化容量`MIN_TREEIFY_CAPACITY`（默认为64）** 时，这条链表就会被重构为一棵**红黑树**。
  8. 红黑树查找、插入、删除操作的时间复杂度稳定在**O(log n)** ，保证了即使在哈希函数设计不佳或数据分布极不均匀的**最坏情况下**，其性能也能维持在**对数级别**，而不是像旧版一样退化为线性级别。

---

##  知识图解

1. **哈希冲突——开放地址法和链地址法的直观对比图**：
  知识拓展

  1. **面试官可能的追问1：为什么HashMap在JDK 1.8要从头插法改为尾插法？**
    2. 在JDK 1.7中，**多线程**环境下对HashMap进行扩容（resize）时，**头插法**在转移旧数据到新数组的过程中，多个线程可能会同时操作同一条链表，导致链表的`next`指针指向混乱，最终形成**环形链表**。这会导致后续`get`操作时陷入死循环，CPU占用率100%。改为**尾插法**后，在扩容迁移数据时能保持元素的相对顺序，从而**避免了环形链表的产生**。
    3. 尾插法能保持元素的插入顺序，这在将链表转换为红黑树或在红黑树上进行操作时，逻辑会更清晰、更易于处理结点。
  4. **面试官可能的追问2：既然开放地址法也能解决冲突，为什么HashMap不采用它，而是选择链地址法？**
    5. 链地址法在**删除元素时非常简单**，只需要将链表中的对应节点移除即可。而开放地址法在删除元素时会很麻烦，不能直接删除，因为这会“打断”探测链，导致后续的元素找不到。所以它通常需要一个特殊的“已删除”标记，这会使实现变得复杂，并且可能需要定期清理这些标记。
    6. 链地址法在冲突严重时，只会影响到特定桶的链表长度，**扩容的压力相对较小**。而开放地址法对负载因子更敏感，一旦数组变得拥挤，其性能会急剧下降，需要更频繁地进行扩容（Rehash），扩容成本更高。
    7. 链地址法**对哈希函数的均匀性要求相对较低**，即使有少量聚集，也只是让几条链表变长一些。而开放地址法对哈希函数的均匀性要求极高，一旦出现聚集，性能会受到严重影响。

---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 建造者模式｜大厂面试题｜Java高频面试题｜创建型设计模式、复杂对象构建、工厂模式区别

#  建造者模式

##  简要回答

1. 建造者模式是**创建型设计模式**，将复杂对象的构建过程与它的表示分离，把构建步骤拆分为固定的模版，通过不同的建造者实现这些步骤，使得**相同的构建过程能够得出不同的表示**。
2. 优点：解决了复杂对象多参数、多配置导致的构造器臃肿问题，让对象的创建过程更加灵活和可维护。
3. 缺点：会增加类的数量，如果构建步骤频繁变化时需要修改抽象建造者。
##  详细回答

- 建造者模式有四种角色：产品、抽象建造者、具体建造者和指挥者
  1. **产品(Product)** ：是被构建的复杂对象，包含多个部件。
  2. **抽象建造者(Builder)** ：是定义构建产品的固定步骤模版，为产品对象的各个部件指定抽象接口，声明获取最终产品的方法。
  3. **具体建造者(ConcreteBuilder)** ：实现抽象建造者的步骤方法，负责创建产品的具体部件。
  4. **指挥者(Director)** ：是使用Builder接口的对象。能够控制构建流程，调用建造者的步骤方法完成对象构建，客户端只和指挥者交互，无需关注具体的构建细节。
- 建造者模式的工作流程如下：由**客户端**创建具体建造者和指挥者，并将建造者传入指挥者；**指挥者**调用建造者的固定步骤方法，**具体建造者**会在每个步骤中创建产品的对应部件；指挥者调用建造者的方法**获取最终构建好的复杂对象**，客户端直接使用该产品，无需参与任何构建步骤。
##  知识图解

1. 建造者模式结构图  示例代码


```
`// 1. Product（产品类）：被构建的复杂对象
class Product {
    // 产品的核心部件
    private String partA;
    private String partB;

    // 设置部件的方法（供建造者调用）
    public void setPartA(String partA) {
        this.partA = partA;
    }

    public void setPartB(String partB) {
        this.partB = partB;
    }

    // 重写toString，方便查看构建结果
    @Override
    public String toString() {
        return "Product{" +
                "partA='" + partA + '\'' +
                ", partB='" + partB + '\'' +
                '}';
    }
}

// 2. Builder（抽象建造者）：定义构建产品的固定步骤模板
abstract class Builder {
    // 持有产品对象（所有具体建造者共用）
    protected Product product = new Product();

    // 固定构建步骤1：构建部件A
    public abstract void buildPartA();

    // 固定构建步骤2：构建部件B
    public abstract void buildPartB();

    // 获取最终构建好的产品
    public Product getProduct() {
        return product;
    }
}

// 3. ConcreteBuilder（具体建造者1）：实现抽象步骤，构建「类型1」产品
class ConcreteBuilder1 extends Builder {
    @Override
    public void buildPartA() {
        product.setPartA("部件A-类型1"); // 具体部件实现
    }

    @Override
    public void buildPartB() {
        product.setPartB("部件B-类型1"); // 具体部件实现
    }
}

// 3. ConcreteBuilder（具体建造者2）：实现抽象步骤，构建「类型2」产品
class ConcreteBuilder2 extends Builder {
    @Override
    public void buildPartA() {
        product.setPartA("部件A-类型2"); // 不同的部件实现
    }

    @Override
    public void buildPartB() {
        product.setPartB("部件B-类型2"); // 不同的部件实现
    }
}

// 4. Director（指挥者）：控制构建流程，调用建造者的步骤方法
class Director {
    // 核心方法：传入建造者，按固定顺序执行构建步骤
    public Product construct(Builder builder) {
        // 固定构建流程：先构建A，再构建B（指挥者定义流程）
        builder.buildPartA();
        builder.buildPartB();
        return builder.getProduct(); // 返回最终产品
    }
}

// 客户端测试代码
public class Client {
    public static void main(String[] args) {
        // 1. 创建指挥者（控制流程）
        Director director = new Director();

        // 2. 构建「类型1」产品（传入具体建造者1）
        Builder builder1 = new ConcreteBuilder1();
        Product product1 = director.construct(builder1);
        System.out.println("构建的类型1产品：" + product1);

        // 3. 构建「类型2」产品（仅替换具体建造者，流程不变）
        Builder builder2 = new ConcreteBuilder2();
        Product product2 = director.construct(builder2);
        System.out.println("构建的类型2产品：" + product2);
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96

##  使用场景

  1. 建造者模式适合在产品包含多个部件，且产品的构建顺序固定时使用，允许客户指定部件的配置方法，并构建出完整的产品。
##  知识扩展

###  面试官可能追问：

  1. 建造者模式和工厂模式有什么区别？
    1. 建造者模式和工厂模式都是创建型设计模式，用于创建对象。
    2. 工厂模式关注**快速创建对象**，客户端指定类型后工厂直接返回产品对象；而建造者模式关注**复杂对象的构建过程**，将有多个部件的对象分步构建，通过不同建造者实现不同部件，最终获取不同配置的产品。
  2. 在项目使用中，你使用过建造者模式吗？
    1. 使用过。我在项目中构建复杂的HTTP请求对象时，使用建造者模式将设置URL、请求方法、请求头等步骤拆分，客户端只需要传入建造者即可生成不同配置的请求；这样避免了构造器参数过多的问题，同时也提高了代码的可读性和维护性。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 线程安全集合类｜大厂面试题｜Java高频面试题｜ConcurrentHashMap、Vector、CopyOnWriteArrayList、Collections.synchronized

#  线程安全集合类

##  简要回答

1. **`Collection` 接口（单列集合）**：
  2. 单列集合用于存储单个元素。
  3. 主要子接口包括：
 ① **`List`**：它的特点是**有序、可重复**。List接口的常见实现类有：`ArrayList` (非线程安全), `Vector` (线程安全), `LinkedList` (非线程安全)。
 ② **`Set`**：Set集合的特点是**无序、不可重复**。Set接口的常见实现类有：`HashSet` (非线程安全), `TreeSet` (非线程安全)。
 ③ **`Queue`**：**先进先出**（FIFO）。Queue接口的常见实现类有：`PriorityQueue` (非线程安全)。
4. **`Map` 接口（双列集合）**：
  5. 双列集合用于存储**键值对**（**Key-Value Pair**）。键（Key）唯一，值（Value）可重复。
  6. Map接口的常见实现类有：`HashMap` (非线程安全), `Hashtable` (线程安全), `TreeMap` (非线程安全), `ConcurrentHashMap` (线程安全，高性能)。
##  详细回答

###  `Collection` 接口（单列集合）

1. **定义**：**`Collection`** 是**所有单列集合的根接口**，用于存储一系列不重复或重复的单个元素。它定义了所有单列集合的通用行为，如添加、删除、判断元素是否存在等。
2. **分类**：
  3. **`List` 接口**：
 Δ **特点**：元素**有序**（按插入顺序），元素**可重复**。可以通过索引访问元素。
 Δ **常见的实现类**：
 ① **`ArrayList`** ：底层基于**动态数组**实现。特点是**查询（随机访问）速度快**；但**增删元素（特别是中间位置）效率相对较低**。`ArrayList` 类是**非线程安全的**。
 ② **`LinkedList`** ：底层基于**双向链表**实现。特点是**增删元素（特别是首尾和中间位置）效率高**；但**查询（随机访问）效率较低**。`LinkedList` 也是**非线程安全的**。
 ③ **`Vector`** ：`Vector` 类与 `ArrayList` 类似，但它是**线程安全**的（方法都加了 **`synchronized`** 关键字）。但由于**同步开销**，其性能通常低于 `ArrayList`，是 Java 早期版本提供的集合类。
  4. **`Set` 接口**：
 Δ **特点**：元素**无序**（不保证插入顺序），元素**不可重复**（通过 `hashCode()` 和 `equals()` 方法判断）。
 Δ **常见的实现类**：
 ① **`HashSet`** ：底层其实是 `HashMap`。特点是**增删改查效率高**，但元素**无序**。`HashSet` 是**非线程安全的**。
 ② **`LinkedHashSet`** ：继承自 `HashSet`，底层基于**哈希表和链表**实现。它在 `HashSet` 的基础上，通过链表维护了元素的**插入顺序**。`LinkedHashSet` 也是**非线程安全的**。
 ③ **`TreeSet`** ：底层基于**红黑树**实现。`TreeSet` 类的特点是元素**有序**（按自然排序或自定义比较器排序），增删改查效率相对 `HashSet` 略低（对数时间复杂度）。`TreeSet` 也是**非线程安全的**。
  5. **`Queue` 接口**：
 Δ **特点** ：遵循 **先进先出（FIFO）** 原则，常用于模拟队列数据结构。
 Δ **常见的实现类** ：
 ① **`PriorityQueue`**：**优先级队列**，元素根据其自然排序或自定义比较器进行排序，每次取出的是优先级最高的元素。`PriorityQueue` 是**非线程安全的**。
###  `Map` 接口（双列集合）

1. **特点**：
  2. **键（Key）是唯一的**，用于快速查找对应的值。
  3. **值（Value）可以重复**。
  4. **一个键最多映射一个值**。
5. **常见的实现类**：
  6. **`HashMap`** ：底层基于**哈希表**实现。它的特点是**查询、增删效率高**，但元素（键值对）**无序**。`HashMap` 是**非线程安全的**。
  7. **`LinkedHashMap`** ：继承自 `HashMap`，底层基于**哈希表和双向链表**实现。它在 `HashMap` 的基础上，通过链表维护了键值对的**插入顺序**。`LinkedHashMap` 是**非线程安全的**。
  8. **`TreeMap`** ：底层基于**红黑树**实现。它的特点是键值对**有序**（按键的自然排序或自定义比较器排序），增删改查效率相对 `HashMap` 略低。`TreeMap` 是**非线程安全的**。
  9. **`Hashtable`** ：与 `HashMap` 类似，但它是**线程安全**的（方法都加了 `synchronized` 关键字），性能较低，是 Java 早期版本提供的集合类。`Hashtable` 的键和值都不能为 `null` 。
  10. **`ConcurrentHashMap`** ：在 `JDK1.5` 之后提供的**线程安全且高性能**的双列集合实现。它通过分段锁（JDK7）或 `CAS + synchronized`（JDK8）等机制，提供了比 `Hashtable` 更好的并发性能。
##  知识拓展

1. **Java集合体系图（简要版）** 如下图所示：
   →

### 评论
 

验证登录状态...

## Spring常见注解｜大厂面试题｜Java高频面试题｜依赖注入、Bean管理、Web开发

#  Spring常见注解

##  简要回答

- Spring核心注解可以进行**Bean管理、依赖注入、AOP、事务和Web**等场景。
- Bean注册时可以使用@Component或者使用@Service、@Repository、@Controller标记组件。
- 依赖注入时可以使用@Autowired按类型注入或者使用@Resource按名称注入，还可以使用@Qualifier指定名字。
- @Aspect可以标记切面，@Transactional可以声明事务。
- 与Web相关注解有@RequestMapping用于请求映射，@RequestBody接收请求体，@ResponseBody返回响应体。
##  详细回答

- Spring框架提供了很多注解，可以用于简化配置，管理Bean、处理事务和处理AOP等。
- @Component：Spring的组件注解，将一个类标识为Spring的组件，通过组件扫描可以向Spring注册Bean。
- @ **Bean**：也是向Spring声明Bean，在配置类中使用，Spring容器会根据配置类中@ Bean方法返回的实例来管理Bean。
- @ **Autowired**：用于自动注入依赖项，可以在构造器、Setter方法和字段上，Spring会自动查找匹配类型的Bean进行注入。
- @ **Qualifier**：与@ Autowired一起使用，指定注入时使用的Bean名称。
- @ **Primary**：在没有使用Qualifier注解，优先注入Primary的实例。
- @ **Value**：用于从属性文件或配置中读取值，将值注入到成员变量中。
- @ **Service**、@ **Repository和**@ **Controller服务层**、持久层和控制层的Bean，也是比较明确的Component。
- @ **Controller**：将类标记为控制器，用于处理HTTP请求。
- @ **RestController**：用于构建RESTful Web服务，类的所有处理器方法返回值会被自动序列化，写入HTTP响应体。
- @ **Configuration**：用于定义配置类，替代XML配置文件，其中定义的Bean会被Spring容器管理。
- @ **RequestMapping**：将HTTP请求路径映射到Controller的处理方法上，定义请求的URL路径、请求方法和参数。
- @ **RequestBody**：可以读取Request请求的Body部分，接受客户端传递的JSON、XML格式的数据并自动绑定到Java对象上，应用于RESTful接口的开发。
- @ **GetMapping**、@ **PutMapping**、@ **DeleteMapping**用于处理对应的HTTP请求，简化了RequestMapping。
- @ **PostMapping**：处理post请求，@ PostMapping 通常与 @ RequestBody 配合，用于接收 JSON 数据并映射为 Java 对象。
- @ **PathVariable**：可以从URL路径中提取参数。
##  知识图解


 知识扩展

1. 面试官可能追问：
-

@ Component和@ Bean注册的Bean有什么本质区别？

  - @ Component是类注解扫描，Spring会通过类路径扫描**自动实例化**，依赖无参构造；@ Bean可以进行**手动定义**，开发者可以自定义实例化的逻辑，根据@ Bean方法返回的实例来管理Bean。
  - 如果同一类型的Bean被两种方式注册，Bean会覆盖Component的，因为**Bean是显式配置，优先级更高**。
-

如果一个类既引用了@ Service又在@ Configuration中使用@ Bean定义了该类的实例，容器中会有几个Bean?分别是什么名称？

  - 使用Service没有指定名称时默认类名是首字母小写，而Bean的默认名称是方法名，如果产生了名称冲突，@ Bean注解定义的Bean会覆盖Service注解，容器中只有一个Bean，如果Bean注解指定了不同的名称那么容器中会有两个Bean。
-

@ Autowired和@ Resource有什么区别？Autowired什么时候会报错？

  - @ Autowired是Spring的原生注解，默认会按类型匹配，@ Resource是JDK注解，是优先按照名称匹配的。
  - 当Autowired没有找到匹配类型的Bean时会报错NoSuchDefinitionException，需要设置required=false，将变量赋值为null，但是后续代码需要判断该Bean是否为空。
-

@ RestController和@ Controller有什么区别？如果@ Controller想返回JSON可以怎么做？

  - RestController注解是Controller注解和ResponseBody注解的结合，类中的所有方法默认返回JSON；所以在有Controller注解的方法上添加@ ResponseBody注解可以返回JSON。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## Spring框架对比｜高频面试题｜Java高频面试题｜SpringBoot、SpringMVC、SpringCloud、IoC、AOP

#  Spring框架对比

##  简要回答

- **Spring**是最基础的应用程序开发框架，提供广泛功能包括依赖注入、AOP和事务管理等。SpringBoot,SpringMVC和SpringCloud都是基于Spring的IoC和AOP核心。
- **SpringBoot**是一个快速开发脚手架，延续了Spring框架的核心思想IOC和AOP，能够**简化**Spring应用的创建、运行、调试和部署过程，使用SpringBoot可以专注于应用开发无需过多关注XML配置。
- **SpringCloud**是基于SpringBoot的微服务治理框架，提供微服务架构中的组件。
- **SpringMVC**是Spring中一个重要模块，能够让Spring快速构建MVC架构的Web程序。MVC就是模型model，视图view和控制器Controller，核心思想是通过将业务逻辑、数据和显示分离，主要关注处理Web请求、管理用户会话。控制应用程序流程。
##  详细回答

1. **Spring是整个Spring生态的基础**，核心思想有DI（依赖注入）、AOP（面向切面编程）和IOC（控制反转）。
  2. DI解决依赖关系的硬编码问题；IoC容器管理对象的创建和依赖关系；AOP可以实现事务管理、安全日志等横切关注点的模块化，提高代码可维护性和可重用性；同时提供了一系列事务管理接口。
  3. 能够解决传统Java开发中对象耦合度高、代码重复的问题，提供一套统一编程规范。
4. **SpringMVC是Spring生态中专注于Web开发的MVC框架**，基于Spring核心功能构建。
  5. 用于实现MVC模式，分别是模型model、视图view和控制器controller，可以分离数据、显示和控制逻辑。
  6. 可以处理HTTP请求，通过DispatcherServlet，将HTTP请求映射到控制器方法,处理后返回视图或数据。规范请求处理流程。
  7. 提供参数绑定、拦截器和数据验证登Web开发必备功能。
8. **SpringBoot是一个基于Spring的快速开发工具**。
  9. 能够进行自动装配，根据引入的依赖配置相关组件，减少XML或注解配置。
  10. 内置Tomcat，Jetty等服务器，可直接打包为jar运行，无需单独部署容器。
  11. 能够简化Spring应用的搭建和部署，解决传统Spring应用配置繁琐、依赖管理复杂和部署麻烦的问题，让开发者专注于业务逻辑。
12. **SpringCloud是基于SpringBoot的微服务治理框架**，提供了微服务架构中的各个组件。
  13. 解决微服务架构的复杂问题，比如服务注册、负载均衡、服务熔断与降级、API网关等。
- Spring是Spring生态基础，SpringMVC负责Web层交互，SpringBoot简化整体开发流程，SpringCloud解决微服务架构的治理问题。上层技术依赖下层技术，形成Spring技术体系。
##  知识图解

1. Spring生态示意

 知识扩展

1. 扩展：
  2. Spring IOC
    1. 概念：控制反转是对程序执行流程的控制，原先由程序员控制整个程序的执行，使用IoC思想则是**将控制权交给了Spring**，由框架控制Bean的生命周期,负责对象的创建、初始化和销毁。
    2. 实现机制：
      3. 利用Java的**反射机制**动态地加载类、创建对象实例和调用对象方法，实现灵活的对象实例化和管理；再通过**依赖注入**，让容器管理应用程序组件之间的依赖关系。
      4. 采用**工厂模式**管理对象的创建和生命周期容器作为工厂实例化Bean和管理它们的生命周期。
  3. Spring AOP
    1. 概念：在Spring框架中实现面向切面的编程。切面包含很多种类型和对象，AOP就是以切面为单位对它们进行模块化的管理。AOP可以将核心业务和周边功能进行分离，将日志、事务管理和权限管理等周边功能封装，**减少系统重复代码，降低模块间的耦合度**。
    2. 实现：
      3. 利用Java的动态代理机制，在运行时动态地创建代理对象，开发则在不修改源码的情况下增强方法功能。有基于JDK的动态代理和基于CGLIB的动态代理。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## 类加载机制｜高频面试题｜Java高频面试题｜JVM、类加载器、双亲委派

#  类加载机制

##  简要回答

- 类加载过程可以分为**加载**、**链接**和**初始化**三个阶段，其中链接阶段可以划分为**验证、准备和解析**三个子阶段。
1. **加载(Loading)**
  2. 在加载阶段，类加载器负责查找类的字节码文件并将其加载到内存中。
3. **链接(Linking)**
  4. 包含三个子阶段：验证、准备、解析
  5. 验证(Verification)：在验证阶段，会确保加载的类文件格式正确，并且不包含不安全的构造。
  6. 准备(Preparation)：在准备阶段，在内存中为类的静态变量分配内存空间，并设置默认初始值。
  7. 解析(Resolution)：在解析阶段，会将类、接口、字段和方法的符号引用解析为直接引用，也就是内存地址。
8. **初始化(Initializing)**
  9. 在初始化阶段，执行类的静态初始化代码，包括静态字段的赋值和静态代码块的执行。静态初始化在类的首次使用时进行，可以是创建实例、访问静态字段或调用静态方法。
##  详细回答

1. 加载(Loading)
  2. 根据一个类的全限定名来获取字节码文件的二进制**字节流**，然后将这个字节流所代表的静态存储**结构转化**为方法区的运行时数据结构，随后在堆内存中生成一个代表这个类的java.lang.**Class对象**，作为方法区这个类的各种数据的访问接口。
3. 链接(Linking)
  4. 包含三个子阶段：验证、准备、解析
  5. 验证(Verification)：**确保Class文件中字节流包含的信息符合当前虚拟机要求且不会危害自身**，验证文件格式、元数据、字节码和符号引用。
    6. 文件格式校验是验证字节流是否符合class文件规范并且能够被当前版本的虚拟机处理。
    7. 元数据验证是对字节码描述的信息进行语义分析，保证其信息符合Java语言规范的要求。
    8. 字节码验证会进行数据流和控制流分析，保证类不会危害虚拟机安全。
    9. 符号引用验证是发生在解析阶段，保证解析阶段可以正常执行。
  10. 准备(Preparation)：在准备阶段，在内存中**为类的静态变量分配内存空间**，并设置默认初始值为0、null或者false。如果是静态常量，编译时会被存入调用类的常量池，准备阶段不分配内存。
  11. 解析(Resolution)：在解析阶段，虚拟机将常量池内的**符号引用替换为直接引用**，也就是得到类、字段、方法在内存中的指针或者偏移量。
12. 初始化(Initializing)
  13. 执行由编译器生成的初始化方法clinit，是类加载的最后一步。完成静态变量赋值和静态代码块的执行，如果类有父类先初始化父类。
##  代码示例


```
`public class ClassLoadingExample {
    public static void main(String[] args) {
        // 步骤1：加载
        MyClass myClass = new MyClass();
        // 步骤3：初始化
        System.out.println(MyClass.staticField);
        // 输出：Initialized static field
        // 使用类中的方法
        myClass.printMessage(); // 输出：Initialized static method
    }
}
class MyClass {
    // 步骤2：连接-准备
    public static String staticField = "Initialized static field";
    // 步骤3：初始化
    static {
        System.out.println("Initialized static method");
    }
    public void printMessage() {
        System.out.println("Instance method called");
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22

##  知识图解

1. JVM的组成-类加载器  知识扩展

  1. 扩展：
  2. **类加载器ClassLoader**：
    3. 启动类加载器(Bootstrap ClassLoader)：是最顶层的加载类，JVM启动时加载**核心类库**（rt.jar,resources.jar,charset.jar等jar包和类,被参数-Xbootclasspath指定路径下的类）。是由C/C++编写的，不能修改。
    4. 扩展类加载器(Extension ClassLoader)：是启动类加载器的子类，加载**扩展目录(lib/ext)中的类库**。
    5. 应用程序类加载器(Application ClassLoader)：是扩展类加载器的子类，加载**当前应用类路径下的所有jar包和类**。
  1. 面试官可能追问：
  6.

Q1：类的生命周期是怎么样的？

    7. 类的生命周期包括**加载、验证、准备、解析、初始化、使用和卸载**7个阶段。
  8.

Q2：准备阶段分配的变量包含实例变量吗？

    9. 不包含。实例变量会在对象实例化时和对象一起被分配在Java堆中，类加载则是发生在所有实例化操作之前。
  10.

Q3：解析阶段一定在准备阶段之后，初始化之前吗？

    11. 不一定。解析阶段在JVM中是按需延迟解析，可以在初始化之前解析，也可以在首次使用该符号引用时再解析，调用方法时才解析方法的符号引用，这样可以避免不必要的解析开销。
  12.

Q4：哪些场景会触发类的初始化？

    13. 类的初始化会在以下场景触发：
      14. 当虚拟机启动时，初始化用户指定的主类，也就是含main方法的类。
      15. 有反射调用时会触发类的初始化。
      16. 当访问类的静态变量或调用静态方法时，会触发类的初始化。
      17. 当创建类的实例时，会触发类的初始化。
      18. 初始化子类时，会触发父类的初始化。
  19.

Q5：类加载后会被卸载吗？什么时候会卸载？

    20. 当类的所有实例都被回收，加载该类的类加载器被回收以及该类的Class对象没有任何引用，无法通过反射等方式访问时JVM会卸载该类。
    21. 启动类加载器加载的核心类不会被卸载，因为该类加载器无法被回收。
  22.

Q6：类的卸载和实例的垃圾回收有什么区别？

    23. 类的卸载是**回收方法区中类的元数据**，比如字段和方法信息，类的生命周期结束；实例的垃圾回收是**回收堆中对象的内存**，是对象生命周期的结束。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 抽象类与接口｜高频面试题｜Java高频面试题｜接口默认方法、抽象方法、多实现

#  抽象类与接口

##  简要回答

###  抽象类 和 接口 的概念

1. **抽象类**：
 是一种**不能被实例化**的类，它**可能**包含抽象方法（没有方法体）和具体方法（有方法体）。抽象类通常是作为对应子类的基类，提供共同的属性和部分行为的实现，并强制子类实现某些特定行为。
2. **接口**：
 是一种**完全抽象**的类型，接口技术用于描述类 **应该具有什么功能，但并不给出具体实现**，等到当某个类要使用接口时，再去实现接口中的这些方法。类需要遵从接口中描述的统一规则进行定义，所以，接口是对外提供的**一组规则，标准**。
###  抽象类 和 接口 的区别

-

如下表所示：
 **维度** **抽象类** **接口** **成员变量** 可有各种访问修饰符的实例变量、静态变量和常量。 默认且必须是 `public static final`（常量），且必须在定义时就初始化。 **成员方法** 可有抽象方法（无实现）和具体方法（有实现）。抽象方法不能是 `private`, `static`, `final`。 Java 8之前：只能有 `public abstract` 方法。
Java 8及之后：可有默认方法、静态方法。
Java 9及之后：可有私有方法。 **构造方法** 可以有构造方法，用于子类初始化父类成员，但不能直接实例化。 不能有构造方法。 **创建对象** 不能直接实例化。只能通过子类（非抽象）实例化。 不能直接实例化。只能通过实现类（非抽象）实例化，并通常通过多态引用。 **继承关系** 使用 `extends` 关键字继承，只支持单继承。 使用 `implements` 关键字实现，支持多实现（一个类可实现多个接口）。
接口之间可多继承。 **适用场景** 表示 “is-a” 关系，作为类族共同的基类，提供模板方法模式。 表示 “can-do” 或 “has-a” 关系，定义行为规则/标准，实现多态，弥补单继承机制的不足。
##  详细回答

###  抽象类 和 接口 的概念

1. **抽象类**：
  2. 抽象类是Java中用于实现**抽象概念**的一种特殊类，它使用 `abstract` 关键字修饰，**不能被直接实例化**（即不能使用 `new` 关键字创建对象）。
  3. 抽象类通常是作为对应子类的**基类**，提供一个通用的模板或骨架，定义一组共同的属性和部分行为，并强制子类实现某些特定的、对于每个子类来说比较特别 的行为。
4. **接口**：
  5. 是一种**完全抽象**的类型，它使用 `interface` 关键字定义，与抽象类类似，也不能被直接实例化。
  6. 接口技术用于描述类 **应该具有什么功能，但并不给出具体实现**，等到当某个类要使用接口时，再去实现接口中的这些方法。类需要遵从接口中描述的统一规则进行定义，所以，接口是对外提供的**一组规则，标准**。
  7. 接口的**主要目的**是实现**多重继承**的效果，定义不同的类可以拥有的共同的 能力或行为，强调 “做什么” 而非 “怎么做”。
###  抽象类 和 接口 的区别

1. **成员变量**：
  2. **抽象类**：在抽象类中，可以定义各种类型的成员变量，包括**实例变量（非静态）、静态变量，以及静态和非静态的 常量**。这些变量可以使用**任何访问修饰符**（`public`, `protected`, `default`, `private`），并且可以 不需要被 `final` 修饰，也可以不进行初始化（因为对于实例变量，可以在构造方法中初始化）。
  3. **接口**：在接口中，所有成员变量都必须是 `public static final` 类型，即**接口中没有成员变量，只有 公有静态常量** 。由于它们是常量，因此**必须在定义时进行初始化**。而因为接口不包含构造方法或静态代码块来初始化这些常量，所以定义时就进行赋值是**唯一**的初始化途径。
4. **成员方法**：
  5. **抽象类**：
 Δ 抽象类可以包含 **抽象方法** （抽象方法没有方法体，用 `abstract` 关键字修饰）和 **具体方法** （具体方法有方法体）。
 Δ 抽象方法不能是 `private` 修饰的，因为私有方法无法被子类继承和实现，也不能是 `static` 修饰的，因为静态方法属于类，与实例无关，无法强制子类实现，还不能是 `final` 修饰的，因为 `final` 方法不能被重写，而抽象方法必须被重写，这就有矛盾。
 Δ 具体方法（非抽象方法）则可以有任何访问修饰符，并且也可以是 `static` 或 `final`修饰的。
  6. **接口**：
 Δ 在 **Java 8 之前**，接口中所有方法必须是 `public abstract`，即**公有的抽象方法**，不允许有方法体。事实上，接口中的方法**默认**就是**公有抽象方法**，因此在接口中定义抽象方法时，可以省略掉abstract关键字。
 Δ 从 **Java 8 开始**，接口引入了 **默认方法（default methods）** 和 **静态方法（static methods）** 。默认方法使用 `default` 关键字修饰，允许接口提供方法的默认实现，实现了接口的**向后兼容性**，因为默认方法是可以被实现类重写的。静态方法使用 `static` 关键字修饰，可以通过接口名直接调用，与类中的静态方法类似。
 Δ 从 **Java 9 开始**，接口进一步支持 **私有方法（private methods）** 。私有方法可以是实例方法或静态方法，它们主要用于在接口 **内部** 封装默认方法和静态方法的公共逻辑，提高代码复用性，但不能被外部或实现类直接访问。
7. **构造方法**：
  8. **抽象类**：抽象类可以拥有构造方法，并且支持构造方法的重载。尽管抽象类不能被直接实例化，但其构造方法会在子类实例化时被调用，用于初始化抽象类中定义的成员变量。这是因为子类的构造方法在执行时，会隐式或显式地调用其父类的构造方法。
  9. **接口**：接口不能有构造方法。接口只定义行为规范和常量，不涉及实例的状态初始化，因此不需要构造方法。
10. **创建对象**：
  11. **抽象类**：抽象类不能直接使用 `new` 关键字创建对象（即**不能被实例化**）。要使用抽象类，必须通过其**非抽象子类**来实例化。通常，我们会通过**多态**的方式，将子类对象 赋值给 抽象类类型的**引用**，从而调用抽象类中定义的方法（包括抽象方法和具体方法）。
  12. **接口**：接口也不能直接使用 `new` 关键字创建对象（即**不能被实例化**）。要使用接口，必须通过其**非抽象的实现类**来实例化。同样，通常会通过**多态**的方式，将实现类对象 赋值给 接口类型的**引用**，以实现接口定义的功能。接口的**实现类**可以是抽象类（抽象类允许部分实现接口方法）或普通类（普通类要求 必须实现接口中所有抽象方法）。
13. **继承关系**：
  14. **抽象类**：
 Δ 类与抽象类之间是**继承关系**，使用 `extends` 关键字。
 Δ Java只支持**单继承**，即一个类只能继承一个抽象类（或普通类）。这使得抽象类更适合表示 “is-a” 的**强关联**关系，例如“猫是一种动物”，“程序员是一种生物”。
  15. **接口**：
 Δ 类与接口之间 是**实现关系**，使用 `implements` 关键字。Java支持**多实现**，即**一个类可以实现多个接口**。这使得接口能够弥补Java单继承的不足，允许一个类拥有多种 不相关的“能力”或“行为”。
 Δ 接口与接口之间 是**继承关系**，使用 `extends` 关键字。接口支持**多继承**，即**一个接口可以同时继承多个接口**。
16. **适用场景**：
  17. **抽象类**：
 ① 适用于表示 “is-a” 关系，我们上面也提到了，例如，当一组类之间存在共同的属性和行为，并且其中一些行为是共同实现的，而另一些行为是必须由子类各自实现的（但具体实现方式不同），这时候，就可以将子类共同的属性和行为抽象出来，把它们封装到一个抽象类中。
 ② 抽象类常用于**模板方法模式**，就是定义一个操作中的算法骨架，而将一些步骤延迟到子类中去实现。
  18. **接口**：
 ① 适用于表示 “can-do”， “has-a” ，或者 “like a” 这样一种关系，例如，当需要定义 一组不相关的类 **都必须遵循**的规则或者标准时，就可以通过接口来解决。
 ② 接口经常用于实现**多态性**，使得不同类的对象能够以统一的方式被处理。
 ③ 有时候需要定义一个纯粹的规范，不涉及任何实现细节时（传统接口），也可以通过定义接口来解决。
 ④ 而且 从Java 8开始，有时候需要为 现有接口 添加新方法，同时还不去破坏已有的实现类时，就可以在接口中添加新的默认方法来解决。

---

##  知识拓展

1. **接口的特性示意图如下**：
   →

### 评论
 

验证登录状态...

## Map接口实现类｜高频面试题｜Java高频面试题｜HashMap、LinkedHashMap、TreeMap、ConcurrentHashMap

#  Map接口实现类

##  简要回答

1. **`HashMap`**: HashMap是基于哈希表实现的，可以提供快速的键值对存取，但是不保证迭代顺序。它允许 `null` 键和 `null` 值，但它是非线程安全的。
2. **`LinkedHashMap`**: LinkedHashMap继承自 `HashMap`，它通过内部维护的双向链表来维护元素的插入顺序或访问顺序。LinkedHashMap也允许 `null` 键和 `null` 值，但它也是非线程安全的。
3. **`TreeMap`**: TreeMap是基于红黑树实现的，它按照键的自然顺序 或者是 自定义的比较器进行排序。TreeMap并不允许 `null` 键，它也是非线程安全的。
4. **`ConcurrentHashMap`**: ConcurrentHashMap是目前常用的高性能的线程安全 `Map`。它通过分段锁（JDK 1.7）或 CAS + `synchronized`（JDK 1.8+）来实现并发控制。ConcurrentHashMap不允许 `null` 键。
5. **`Hashtable`**: Hashtable算是比较传统的线程安全 `Map`，它的所有操作都通过 `synchronized` 关键字同步，因此性能较低。Hashtable既不允许 `null` 键，也不允许 `null` 值。
6. **`Properties`**: 它是 `Hashtable` 的子类，主要用于读写键和值都是 `String` 类型的配置文件。

---

##  详细回答

1. **HashMap** :
  2. **底层数据结构：**
 ① 在 **JDK 1.8 之前**，`HashMap` 的底层就是**数组 + 链表**。
 ② 在 **JDK 1.8 及之后**，如果table数组中某一条链表的元素个数达到或超过了**TREEIFY_THRESHOLD**（默认是8），并且table数组的长度达到或超过了**MIN_TREEIFY_CAPACITY**（**默认是64**），底层就会对该链表进行树化，将其转化为一棵红黑树。当红黑树的节点数量减少到 6 时，则会又退化为链表。
  3. **核心特性：**
 ① **无序性：** HashMap不保证键值对的迭代顺序。
 ② **允许 `null` 键和 `null` 值：** HashMap允许有 `null` 键但最多只能有一个 `null` 键，`null` 值可以同时存在多个。
 ③ **非线程安全：** 在多线程环境下并发修改 `HashMap` 可能导致数据不一致 或者 出现 `ConcurrentModificationException`。
  4. **时间复杂度：**
 ① 在**理想的哈希分布**下，HashMap的 `put()`, `get()`, `remove()` 等操作的平均时间复杂度均为 **O(1)** 。
 ② 在**最坏**情况下（例如所有键的哈希值都相同，导致链表过长），这时时间复杂度会退化为 **O(N)** ，但 JDK 1.8 引入红黑树后，这种情况的概率大大降低。
  5. **适用场景：**
 ① **绝大多数需要存储键值对的场景。** 它是 Java 中使用最广泛的 `Map` 实现。
 ② 不需要保持元素插入顺序或对键进行排序的场景。
6. **LinkedHashMap** :
  7. **底层数据结构：**
 ① `LinkedHashMap` 继承自 `HashMap`，因此其底层也基于 **数组 + 链表/红黑树**（同 JDK 1.8+ 的 `HashMap`）。
 ② 在此基础上，`LinkedHashMap` 额外维护了一个**双向链表**，这个链表连接了 `Map` 中所有的键值对（Entry 或 Node），从而能够保持元素的特定顺序。
  8. **核心特性：**
 ① **保持插入顺序（默认）：** 默认情况下，迭代器会按照元素插入的顺序返回键值对。
 ② **保持访问顺序（LRU 策略）：** 可以通过构造函数设置 `accessOrder` 参数为 `true`，使其按照元素的最近访问顺序（即 LRU 策略，最近访问的元素会被移到链表末尾）进行排序。
 ③ **允许 `null` 键和 `null` 值：** 同 `HashMap`。
 ④ **非线程安全：** 同 `HashMap`。
  9. **时间复杂度：**
 ① `put()`, `get()`, `remove()` 等操作的平均时间复杂度为 **O(1)** 。链表的维护操作对性能影响很小。
  10. **适用场景：**
 ① **需要保持元素插入顺序的缓存或处理队列。**
 ② **实现 LRU (Least Recently Used) 缓存策略：** 通过重写 `removeEldestEntry` 方法，可以在 `Map` 达到指定容量上限时自动移除最不常用的元素，非常适合作为固定大小的缓存。
11. **TreeMap** :
  12. **底层数据结构：**
 ① `TreeMap` 的底层是基于**红黑树（Red-Black Tree）**实现的。红黑树是一种自平衡的二叉查找树，它能保证树的高度保持对数级别，从而提供稳定的性能。
  13. **核心特性：**
 ① **按键排序：** 键值对会根据键的自然顺序（即键类必须实现 `Comparable` 接口）或在构造时提供的 `Comparator` 进行排序。
 ② **不允许 `null` 键：** 因为需要对键进行比较操作来确定其在树中的位置，而 `null` 无法参与比较，所以 `TreeMap` 不允许 `null` 键。
 ③ **允许 `null` 值：** 可以有多个 `null` 值。
 ④ **非线程安全：** 同 `HashMap`。
  14. **时间复杂度：**
 ① `put()`, `get()`, `remove()` 等操作的时间复杂度为 **O(logN)** ，其中 N 是 `Map` 中元素的数量。这是因为红黑树的高度是 O(logN)。
  15. **适用场景：**
 ① **需要对键进行排序的场景。** 例如，按字母顺序存储用户，按日期存储事件，或者需要获取某个范围内的键值对。
 ② **需要执行范围查询（如 `subMap()`、`headMap()`、`tailMap()`）的场景。**
 ③ 需要高效地查找最小/最大键、或键的最近邻居的场景。
16. **ConcurrentHashMap** :
  17. **底层数据结构：**
 ① **JDK 1.7 及之前：** 采用**分段锁（Segment）**的机制。`ConcurrentHashMap` 内部维护一个 `Segment` 数组，每个 `Segment` 都是一个独立的 `ReentrantLock`，并管理着一个哈希表（类似于 `HashMap`）。通过对不同的 `Segment` 加锁，实现了对 `Map` 的分段并发控制，降低了锁的粒度。
 ② **JDK 1.8 及之后（包括 JDK 17.0）：** 放弃了 `Segment` 概念，改用 **CAS (Compare And Swap) + `synchronized` 关键字**（针对哈希桶的头节点）+ **数组 + 链表/红黑树**。在链表/红黑树的头节点上使用 `synchronized` 锁，进一步降低了锁的粒度，提高了并发性能。读操作通常不需要加锁，写操作只锁住当前操作的哈希桶。
  18. **核心特性：**
 ① **高性能线程安全：** 提供了比 `Hashtable` 更好的并发性能，读操作通常不需要加锁。
 ② **不允许 `null` 键：** 但允许 `null` 值（JDK 8+）。
 ③ **无序：** 不保证迭代顺序。
  19. **时间复杂度：**
 ① `put()`, `get()`, `remove()` 等操作的平均时间复杂度为 **O(1)** 。
  20. **适用场景：**
 ① **高并发环境下需要线程安全的键值对存储。** 例如，共享缓存、会话管理、计数器等。它是现代 Java 并发编程中 `Map` 的首选。
21. **Hashtable** :
  22. **底层数据结构：**
 ① `Hashtable` 的底层基于**数组 + 链表**，与 JDK 1.8 之前的 `HashMap` 类似。
  23. **核心特性：**
 ① **线程安全：** `Hashtable` 的所有公共方法都使用了 `synchronized` 关键字进行修饰，这意味着在任何时候只有一个线程可以访问 `Hashtable` 的方法。这种**粗粒度的全表锁**机制保证了线程安全。
 ② **不允许 `null` 键和 `null` 值：** 如果尝试插入 `null` 键或 `null` 值，会抛出 `NullPointerException`。
 ③ **无序性：** 不保证迭代顺序。
  24. **时间复杂度：**
 ① 在单线程环境下，`put()`, `get()`, `remove()` 等操作的平均时间复杂度为 **O(1)** 。
 ② 但在高并发环境下，由于其粗粒度的全表锁，多个线程的竞争会导致严重的性能瓶颈，实际性能会急剧下降。
  25. **适用场景：**
 ① **基本已被淘汰，不推荐在新代码中使用。** 主要出现在一些遗留系统中，现代应用应优先选择 `ConcurrentHashMap`。
26. **Properties** :
  27. **底层数据结构：**
 ① `Properties` 类继承自 `Hashtable`。
  28. **核心特性：**
 ① **键和值都必须是 `String` 类型：** 这是 `Properties` 最显著的特点，它专门设计用于存储字符串类型的键值对。
 ② **常用于与 I/O 流结合：** 提供了 `load()` 和 `store()` 方法，可以直接从输入流加载键值对或将键值对保存到输出流，通常用于读写 `.properties` 文件。
 ③ **线程安全：** 由于继承自 `Hashtable`，它也具备线程安全特性，但同样面临性能瓶颈。
  29. **时间复杂度：**
 ① 同 `Hashtable`，平均时间复杂度为 **O(1)** 。
  30. **适用场景：**
 ① **读写 `.properties` 配置文件：** 这是其最主要的用途，例如存储数据库连接信息、应用程序配置参数等。

---

##  知识图解

1. **Map类实现类的选择策略**，如下图所示 ；  知识拓展

  2. **`Collections.synchronizedMap(Map<K,V> m)`：**
    3. 这是一个工具方法，可以将任何非线程安全的 `Map`（如 `HashMap`、`LinkedHashMap`、`TreeMap`）包装成一个线程安全的 `Map`。
    4. 它的实现机制是**对传入的 `Map` 对象进行包装，并在每个方法调用时对整个 `Map` 对象加锁**。虽然能提供线程安全，但由于是粗粒度的锁，其并发性能通常不如 `ConcurrentHashMap`。在并发度要求不高的场景下可以使用。
  5. **`WeakHashMap`：**
    6. 其键是弱引用，当键不再被其他强引用指向时，即使 `WeakHashMap` 中还存有它的引用，垃圾回收器也会回收这个键值对。
    7. 适用于实现简单的缓存，当内存不足时，可以自动清理不再被引用的键值对，避免内存泄漏。

---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Bean生命周期｜大厂面试题｜Java高频面试题｜Spring、BeanPostProcessor、循环依赖、三级缓存

#  Bean生命周期

##  简要回答

- Bean的生命周期可以概括为**容器初始化、Bean实例化、依赖注入、初始化、使用和销毁阶段**，由Spring容器进行管理，通过注解、配置文件或接口进行配置管理。
##  详细回答

1. 在**容器初始化**阶段，Bean容器通过XML或注解扫描Spring Bean的定义，将其封装成BeanDefinition对象并注册到BeanFactory中。
2. 在**实例化**阶段中，容器会通过构造器反射创建Bean实例。
3. 在**依赖注入**阶段中，容器通过Autowired、Resource或者XML配置将Bean实例的属性值或者引用注入到其他Bean中。
4. 在**Bean初始化**阶段：
  5. 容器会检查**Aware相关的接口**：如果Bean实现了BeanNameAware接口，调用setBeanName方法，传入Bean的名字；如果Bean实现了BeanFactoryAware接口，调用setBeanFactory方法，传入BeanFactory实例；如果Bean实现了ApplicationContextAware接口，调用setApplicationContext方法，注入ApplicationContext实例。
  6. 执行**前置处理**：如果Bean实现了BeanPostProcessor接口，Spring会调用它们的postProcessBeforeInitialization()方法。
  7. 执行**自定义初始化方法**，调用Bean使用@PostConstruct或XML方式init-method声明的初始化方法，如果Bean实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法；
  8. 执行**后置处理**：如果Bean实现了BeanPostProcessor接口，Spring就调用他们的postProcessAfterInitialization()方法。
9. Bean在初始化后就可以被应用程序使用，使用结束后进行**销毁**，执行@PreDestroy标注或者在XML中使用destroy-method指定的方法，如果Bean实现了DisposableBean接口，Spring会调用destroy()方法。
##  知识图解

- Bean的生命周期示意图  知识扩展

  1. 面试官可能追问
  - Q1：Bean的作用域对生命周期有什么影响？
    - 作用域为 **单例(singleton)** 时，Bean由容器管理完整的生命周期，在初始化后存入容器缓存，容器关闭时销毁。
    - 作用域为**原型(prototype)** 时，容器仅负责实例化、依赖注入和初始化阶段，后续容器不会主动调用，销毁阶段由用户手动处理或JVM垃圾回收处理。
    - 其他作用域如request、session、global session等生命周期与作用域一致，随着请求创建，请求结束销毁。
  - Q2：@PostConstruct、InitializingBean、init-method有没有执行顺序，为什么？
    - **先执行@PostConstruct，再执行InitializingBean，然后执行init-method。**
    - @PostConstruct是由Spring的CommonAnnotationBeanPostProcessor解析，属于BeanPostProcessor前置处理后的回调；
    - InitializingBean是Spring内置接口，容器直接调用其方法，优先级高于自定义XML配置；
    - init-method是XML配置的自定义方法，优先级最低
  - Q3：Spring怎么解决单例Bean的循环依赖问题？为什么原型Bean无法解决？
    - Spring是通过**三级缓存**来解决循环依赖问题的。**一级缓存SingletonObjects**存储完全初始化好的Bean,**二级缓存EarlySingletonObjects**存储早期的Bean引用，是已经实例化但还未完全初始化的Bean，**三级缓存SingletonFactories**存储Bean的工厂，可以生成早期Bean引用。
    - 假设A和B存在循环依赖问题，A创建实例后未注入属性时会存放ObjectFactory对象到三级缓存中，开始给A注入属性时发现A依赖B，此时容器开始创建B，B实例化后也会被存放ObjectFactory对象到三级缓存中，Spring给B注入属性时发现B依赖A，容器在三级缓存中找到A的ObjectFactory对象，获取A的早期引用并放入二级缓存，并清理三级缓存。将A的早期引用注入到B中，完成B的初始化后进入一级缓存。回到A的属性注入环节，将就绪的B注入A，完成A的初始化后进入一级缓存，解决循环依赖问题。
    - **原型Bean无法解决循环依赖问题**，因为原型Bean每次获取都会创建新实例，会导致递归创建，Spring直接抛出BeanCurrentlyInCreationException异常。
  - Q4：如何监听Bean的生命周期？
    - 实现BeanPostProcessor接口，在postProcessBeforeInitialization和postProcessAfterInitialization方法中添加监听逻辑。
    - Bean实现Aware接口、InitializingBean接口、DisposableBean接口，在初始化、销毁前、销毁后添加监听逻辑。
    - 使用@EventListener注解监听容器事件，如ContextRefreshedEvent容器初始化完成、ContextClosedEvent容器关闭等。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## JVM组成部分｜高频面试题｜Java高频面试题｜类加载器、运行时数据区、执行引擎

#  JVM组成部分

##  简要回答

- JVM可以分为**类加载器**、**运行时数据区**、**执行引擎**、**本地方法接口**和**本地方法库**五个部分组成。
- JVM通过**类加载器将字节码（.class文件）加载到内存并生成class对象**，**运行时数据区作为JVM内存管理的区域**中，使用特定的**执行引擎将字节码翻译成底层系统指令**，再交给CPU执行，这个过程需要**调用其他语言的本地接口**实现整个程序的功能。
##  详细回答

1. **类加载器ClassLoader**：
  2. 启动类加载器(Bootstrap ClassLoader)：是最顶层的加载类，JVM启动时加载**核心类库**（rt.jar,resources.jar,charset.jar等jar包和类,被参数-Xbootclasspath指定路径下的类）。
  3. 扩展类加载器(Extension ClassLoader)：是启动类加载器的子类，加载**扩展目录(lib/ext)中的类库**。
  4. 应用程序类加载器(Application ClassLoader)：是扩展类加载器的子类，加载**当前应用类路径下的所有jar包和类**。
5. **运行时数据区Runtime Data Area**：是Java虚拟机在执行Java程序过程中管理内存的一种方式，将内存划分为不同的区域，分为程序计数器，Java虚拟机栈，本地方法栈，Java堆和方法区。每个区域都有特定的用途和生命周期。
  6. **程序计数器**：记录当前线程所执行的字节码指令的地址。生命周期与线程相同。
  7. **Java虚拟机栈**：方法执行时会创建一个栈帧，存储局部变量、操作数栈、动态链接等。每个线程有独立的Java虚拟机栈，生命周期与线程相同。
  8. **本地方法栈**：用于存储java中使用native标记的本地方法。是线程私有的，可以固定或动态扩展。
  9. **堆**：所有线程共享的内存区域，存放对象实例和数组。也是垃圾回收器执行垃圾回收的主要区域。
  10. **方法区（元空间）**：所有线程共享的内存区域，存储已被虚拟机加载的类信息、常量、静态变量即时编译后的代码等数据。在启动JVM时创建，关闭JVM后释放。
11. **执行引擎Execution Engine**：
  12. 解释器：将字节码指令翻译为对应平台的本地机器指令，由CPU执行，当一条指令执行完后，根据PC寄存器中记录的下一条指令执行解释操作。
  13. JIT编译器：将源代码直接编译成与本地平台有关的机器码，并且进行各种层次的优化。由中间代码生成器、代码优化器、目标代码生成器、分析器组成。
  14. 垃圾回收器：主要用于Java堆的管理，系统运行期间会产生的大量对象实例，GC回收无用的对象。
15. **本地方法接口Native Method Interface**：根据本地方法栈中有标记native的方法，生成一个对应的本地接口，启动执行引擎时，会根据本地方法接口去本地方法库中加载对应实现方法。
16. **本地方法库Native Method Library**：存储本地方法的实现方法。
##  知识图解

1. JVM内部结构示意图  知识扩展

  1. 扩展：
  2. 类加载过程：可以分为三个主要阶段：**加载，链接，初始化**。
    1. **加载阶段**：根据类的全限定名，获取字节码文件的二进制流。将字节流解析为方法区中的运行时数据结构，在堆内存生成Class对象，作为方法区数据的访问入口。
    2. **链接阶段**：
      3. 验证：确保字节码符合JVM规范，保证安全性。
      4. 准备：为类的静态变量分配内存并设置默认值。
      5. 解析：将常量池中的符号引用转换为直接引用。
    6. **初始化阶段**：执行类的构造器方法，完成静态变量赋值和静态代码块的执行，如果类有父类，则先初始化父类。
  3. **JVM（虚拟机）**：能够屏蔽操作系统平台相关信息，使Java程序只需要生成在Java虚拟机上运行的目标代码，即解释自己的字节码并映射到本地的CPU指令集和OS的系统调用，实现“**一次编译，到处运行**”。主要工作有：
    1. 加载、验证、执行代码。
    2. 为各种应用程序提供运行时环境，还提供了内存区域、寄存器集。
    3. 提供垃圾收集堆，会报告致命错误，提供类文件格式。
  4. **rt.jar**：代表RunTime，是Java基础类库，包含Java doc里所有的类的文件，如java.util.*，java.io.*，java.nio.*，java.lang.*，java.sql.*，java.math.*等常用内置库。
  1. 面试官可能追问：
  5. Q1：**解释器**和**JIT编译器**的区别是什么？
    6. 解释器会逐条执行字节码，没有编译开销，内存占用小，但是执行效率低。
    7. JIT编译器会将热点代码编译成本地机器码，执行效率高，但是需要一定的编译时间，同时占用CodeCache。
  8. Q2：为什么Java解释和编译都有？
    9. Java同时采用解释和编译执行，这样可以平衡启动速度和执行效率。
    10. 编译性：使用JIT（即时编译器）可以将频繁执行的代码（热点代码）提前编译为机器码并缓存，后续直接执行机器码。
    11. 解释性：使用JVM中的解释器，将字节码逐行翻译为机器码并立即执行，不用提前编译，启动速度快且有跨平台性。JVM中一个方法调用计数器，当累计计数大于一定值后，就使用JIT进行编译生成机器码文件。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## SpringBoot Starter｜高频面试题｜Java高频面试题｜自动配置、依赖管理、约定优于配置

#  SpringBoot Starter

##  简要回答

- SpringBoot Starter是**依赖整合包**，把某类功能相关的常用依赖打包整合。
- 应用可以通过**引入不同的Starter来实现模块化的开发**，**简化和加速项目的配置与依赖管理**。每个Starter都关注一个特定的功能领域，如数据库访问、消息队列、Web开发等。
- 开发者可以创建**自定义的Starter**，能够在项目中共享和重用特定功能的配置和依赖项。
##  详细回答

- SpringBoot Starter是一种预配置的模块，封装了特定功能的依赖项和配置，开发者只需引入相关的Starter依赖，就无需手动配置大量的参数和依赖项。**引入启动器后，SpringBoot会自动配置所需的组件和Bean，开发者只需关注业务逻辑。**
- Starter还管理了相关功能的依赖项，确保其他Starter和第三方库能够良好地协同工作，依赖版本由SpringBoot中的spring-boot-dependencies管理，**避免版本冲突和依赖问题**。
- 一个Starter由**自动配置类**、**依赖jar包**和**配置文件**组成。AutoConfiguration类用于自动配置应用程序，会根据依赖和配置自动向Spring容器中注入功能所需的Bean，是自动配置的实现载体；Starter中包含功能场景所需的依赖jar包；ConfigurationProperties类用于绑定配置文件中的属性，封装Starter的默认配置项，同时读取application.properties中的自定义配置。
###  常用的Starter

- **spring-boot-starter-web**：用于快速构建Web应用程序，包含Spring MVC、Tomcat嵌入式服务器等组件。
- **spring-boot-starter-security**：提供了Spring Security的基本配置，帮助开发者快速实现应用的安全性、认证和授权。
- **mybatis-spring-boot-starter**：简化在SpringBoot应用中集成MyBatis的过程，自动配置了MyBatis的相关组件，包括SqlSessionFactory、MapperScannerConfigurer等，开发者可以快速地开始使用MyBatis进行数据库操作。
- **spring-boot-starter-data-jpa**包含了Hibernate等JPA实现以及数据库连接池等必要的库，适合在使用JavaPersistence API进行数据库操作场景，需要在application.properties中进行配置；**spring-boot-starter-jdbc**包含了JDBC驱动和数据库连接池等组件，提供了基础的JDBC支持。
- **spring-boot-starter-data-redis**：集成了Redis缓存和数据存储服务，包含了与Redis交互所需的客户端（默认是Lettuce客户端），以及Spring Data Redis的支持，需要在配置文件中添加Redis的连接信息。
- **spring-boot-starter-test**：提供了单元测试和集成测试所需的库，包括JUnit、Hamcrest、Mockito等，便于进行测试驱动开发(TDD)。
###  自定义Starter

- 开发者可以创建自定义的Starter，能够在项目中共享和重用特定功能的配置和依赖项。当现有的Starter无法满足业务时，可以自定义Starter，封装专属依赖，**编写**XXXAutoConfiguration自动配置类，在META-INF/spring.factories中**配置**自动配置类，实现专属功能的快速集成。
- 创建自定义Starter：
  1. 需要创建Maven项目，在pom.xml中添加必要的依赖；
  2. 在src/main/resources/META-INF/spring.factories中配置自己的自动配置类后，即可创建自动配置类，需要@Configuration和@EnableAutoConfigurationProperties两个注解，@Configuration注解用于标识该类是一个配置类，@EnableAutoConfigurationProperties注解用于启用自动配置属性绑定；还需结合条件注解@ConditionalOnClass，确保仅在依赖满足时生效。
  3. 创建配置属性类，用@ConfigurationProperties注解绑定配置文件中的属性；
  4. 创建服务类、服务实现类和控制器规定自定义Starter的功能；
  5. 随后可以在Maven仓库中发布自己的Starter，供项目使用。
##  知识图解

- Starter结构示意图：  知识扩展

  1. 扩展
    2. 约定优于配置：
      3. SpringBoot遵循**约定优于配置**的原则，预设默认的配置和约定，开发者按照这些约定进行开发，可以大大减少配置文件的编写。
      4. SpringBoot提供**特定的项目结构**，将主应用程序类置于根包，将控制器类、服务类、数据访问类等分别放在相应子包中，使开发者更易理解项目结构与组织，新成员加入项目也能快速定位各功能代码的位置，提升协作效率。
      5. SpringBoot提供了大量**默认配置**，比如连接数据库、设置Web服务器、处理日志等，开发者无需配置日志级别、输出格式与位置。
      6. SpringBoot的**自动化配置**也是约定大于配置的体现，通过分析项目的依赖和环境，自动配置应用程序的行为。
  7. 面试官可能追问
  - Q1：引入多个Starter时，若出现依赖冲突怎么办？
    - 可以通过mvn dependency:tree分析依赖树，找到冲突的依赖版本；再通过< exclusions >排除Starter中冲突的依赖，手动引入兼容版本；或优先使用Spring Boot官方维护的Starter，其依赖版本已做兼容性适配。
  - Q2：自定义Starter时，如何确保自动配置类仅在特定条件下生效？
    - 通过Spring Boot的条件注解实现，例如@ConditionalOnClass实现类路径存在指定类时生效、@ConditionalOnMissingBean在容器中无指定 Bean 时生效、@ConditionalOnProperty在配置文件中存在指定属性时生效，结合这些注解实现 “按需配置”。
  - Q3：spring-boot-starter和spring-boot-starter-parent的区别是什么？
    - spring-boot-starter是基础 Starter，包含核心依赖；spring-boot-starter-parent是Maven父工程，用于统一管理依赖版本、编译插件等配置；项目通常继承spring-boot-starter-parent，再引入具体功能的 Starter。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## synchronized与Lock对比｜高频面试题｜Java高频面试题｜可重入锁、AQS、Condition、中断响应

#  synchronized与Lock对比

##  简要回答

- synchronized：是**基于JVM的内置锁**，只能**用于方法和代码块**，需要与**wait()** 和**notify()/notifyAll()** 方法一起使用，用于线程等待和通知。
- lock：是接口，Java提供的**显式锁机制**，需要**手动获取和释放**锁，更加**灵活**，支持响应中断、设置锁的公平性等，与Condition接口结合可以实现**更细粒度的线程等待和通知机制**。
- ReentrantLock：是Lock接口的一个**具体实现类**，拥有Lock的特性。
##  详细回答

- 相同点：
  - 他们都是可重入锁，同一个线程可以多次获取同一个锁。
- 不同点：
  - synchronized和lock（以ReentrantLock为例）
  1. **公平性**：synchronized是非公平锁，ReentrantLock支持公平性设置。
  2. **中断响应**：synchronized不支持响应中断，ReentrantLock支持响应中断，即线程在等待锁时可以中断。
  3. **等待与通知**：synchronized不**支持绑定多个条件**，只能与wait()和notify()/notifyAll()方法一起使用，实现线程等待与通知。ReentrantLock可以**与多个Condition对象结合**，实现复杂的线程同步机制。
  4. **实现原理**：synchronized通过**互斥锁**保证共享数据不会被访问，ReentrantLock通过**AQS**来实现。
  5. **操作锁方式**：synchronized是**隐式锁**，线程进入/退出代码时JVM自动获取释放锁，ReentrantLock是**显式锁**，必须手动调用lock()/unlock()方法。
  6. **性能**：ReentrantLock可以提供更细粒度的控制和灵活性，性能高于synchronized。
##  使用场景
 锁 适用场景 优势 synchronized 简单、低并发 无需手动管理 Lock 作为接口，不直接使用 定义锁的操作规范 ReentrantLock 复杂同步场景、高并发 支持中断、超时、公平性选择
##  代码示例

1. ReentrantLock中断特性

```
`// 线程1先获取锁并持有
Thread t1 = new Thread(() -> {
    lock.lock();
    try { Thread.sleep(5000); } // 持有锁5秒
    finally { lock.unlock(); }
});
t1.start();
Thread.sleep(100); // 确保t1先拿到锁

// 线程2尝试获取锁，1秒后被中断
Thread t2 = new Thread(() -> {
    try {
        // 可中断地获取锁
        lock.lockInterruptibly();
        System.out.println("t2获取到锁");
        lock.unlock();
    } catch (InterruptedException e) {
        System.out.println("t2被中断，放弃等待"); // 预期执行
    }
});
t2.start();
Thread.sleep(1000); // 等待1秒后中断t2
t2.interrupt();
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23

1. ReentrantLock超时特性

```
`// 线程1先获取锁并持有3秒
new Thread(() -> {
    lock.lock();
    try { Thread.sleep(3000); }
    finally { lock.unlock(); }
}).start();
Thread.sleep(100);

// 线程2尝试获取锁，设置最多等2秒
new Thread(() -> {
    try {
        // 超时获取锁（2秒）
        boolean success = lock.tryLock(2, TimeUnit.SECONDS);
        if (success) {
            System.out.println("t2获取到锁");
            lock.unlock();
        } else {
            System.out.println("t2等待超时，放弃"); // 预期执行
        }
    }catch(){...}
}).start();
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21

##  知识图解

1. AQS获取锁  知识扩展

  1. 扩展：
  2. AQS：
    3. 是一个用于构建锁和同步容器的框架，ReentrantLock、Semaphore、CountDownLatch等都基于AQS构建。
    4. 使用一个volatile int**变量state**表示同步状态，内部维护一个**CLH变体双向队列**（FIFO）管理等待线程。
      5. 当线程竞争同步状态失败时会被封装成**节点**加入阻塞队列，当同步状态释放时，会把节点中的线程唤醒，使其再次获取同步状态。
    6. AQS有**独占锁**和**共享锁**两种资源共享方式，AQS对于独占模式有一个独占线程标记，记录当前持有锁的线程，实现可重入。
  1. 面试官可能追问:
  7. Q1：CAS和AQS有什么关系？
    8. CAS：一种**乐观锁机制**，即尝试修改，若条件不符合则不做操作，满足原子性。
    9. **AQS内部会使用CAS操作更新state变量**实现线程安全的状态修改。线程获取资源时使用CAS操作更新值，线程释放资源时会使用CAS操作恢复state的值，保证状态更新的原子性。
  10. Q2：高并发下，synchronized 的重量级锁和 ReentrantLock 的性能差异主要来自哪里？
    11. synchronized重量级锁依赖Monitor结构，**线程阻塞/唤醒开销大**，ReentrantLock在高并发下通过**AQS的自旋和CLH队列减少内核态切换**。
  12. Q3：ReentrantLock的公平锁和非公平锁哪个性能好？
    13. 公平锁需要**按照CLH队列顺序**唤醒线程，可能会导致线程频繁阻塞/唤醒，非公平锁允许“**线程插队**”和**刚释放锁的线程直接重入**，避免一部分开销。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## String、StringBuffer与StringBuilder｜高频面试题｜Java高频面试题｜可变性、线程安全、性能对比

#  String、StringBuffer与StringBuilder

##  简要回答

1. **可变性（Mutability）** ：
  2. `String` 不可变，一旦创建了一个`String`对象，它的值就不能被修改；
  3. `StringBuffer` 和 `StringBuilder` 可变，修改在原有对象上进行。
4. **线程安全性（Thread Safety）** ：
  5. `String` 线程安全（因其不可变），
  6. `StringBuffer` 线程安全（方法同步），
  7. `StringBuilder` 非线程安全。
8. **性能（Performance）** ：
  9. `String` 性能最低（频繁创建对象），
  10. `StringBuffer` 性能居中（同步开销），
  11. `StringBuilder` 性能最高（无同步）。
12. **应用场景（Application Context）** ：
  13. `String` 适用于存储字符串常量、少量修改字符串的场景；
  14. `StringBuffer` 适用于多线程环境下需要频繁修改字符串的场景；
  15. `StringBuilder` 适用于单线程环境下需要频繁修改字符串的场景。

---

##  详细回答

1. **可变性（Mutability）** ：
  2. **String** ：
 ① **不可变**：一旦创建了一个 `String` 对象，它的值就不能被修改。 任何对 `String` 对象进行修改的操作都会创建一个新的 `String` 对象。
 ② **JVM 实现角度** ：**`String` 对象本身总是存储在 Java 堆内存中。JVM 中存在一个称为字符串常量池（String Pool）的特殊区域（也位于堆内存中），字符串常量池本身并不直接“存储”完整的 `String` 对象，而是存储对堆中那些唯一的、不可变的 `String` 对象实例的引用**， 可以把它看作一个“目录”。
 ③ **JDK 源码角度** ： 在 JDK 9 之前，`String` 内部使用 `char[]` 数组存储字符。 JDK 9 之后，改用 `byte[]` 数组存储字符，并使用 `coder` 字段来标识字符的编码方式（LATIN1 或 UTF16），字节数组 **value** 和编码字段 **coder** 均声明为 **private final** 类型。而且无论使用哪种方式存储，`String` 类都没有提供任何修改内部字符数组的方法。 所有的修改操作都会创建一个新的 `String` 对象。 例如，`String` 类的 `substring()` 方法会创建一个新的 `String` 对象，而不是修改原始字符串。
  3. **StringBuffer 和 StringBuilder** ：
 ① **可变**：`StringBuffer` 和 `StringBuilder` 类允许在不创建新对象的情况下修改字符串的内容。
 ② **JVM 实现角度**：`StringBuffer` 和 `StringBuilder` 对象在 JVM 中存储在堆内存中。 它们内部使用一个**可变的字符数组**来存储字符串。 当需要修改字符串时，可以直接修改内部的字符数组，而不需要创建新的对象。
 ③ **JDK 源码角度：** `StringBuffer` 和 `StringBuilder` 类都继承自 **AbstractStringBuilder** 类。 **AbstractStringBuilder** 类内部使用 `byte[] value` 数组存储字符。 `StringBuffer` 和 `StringBuilder` 类都提供了修改 `value` 数组的方法，例如 `append()`、`insert()`、`delete()` 等，这些方法可以直接修改 `value` 数组的内容，而不需要创建新的对象。`StringBuffer` 和 `StringBuilder` 内部还维护了一个 `count` 变量，用于记录字符串的长度。 当字符串的长度超过 `value` 数组的容量时，`StringBuffer` 和 `StringBuilder` 会自动扩容 `value` 数组。
4. **线程安全性（Thread Safety）** ：
  5. **String**：`String` 类是**线程安全**的，因为它的不可变性保证了多个线程可以安全地访问同一个 `String` 对象，而不会发生数据竞争。 任何线程都无法修改 `String` 对象的值，因此不存在线程安全问题。
  6. **StringBuffer**： `StringBuffer` 类是**线程安全**的。 它的方法都经过同步（使用 **synchronized** 关键字），可以保证在多线程环境下对字符串的修改是安全的。查看 `StringBuffer` 类的源码，可以看到它的 `append()`、`insert()`、`delete()` 等方法都使用了 **synchronized** 关键字进行同步。 这意味着在同一时刻，只有一个线程可以访问 `StringBuffer` 对象的这些方法。 这种同步机制保证了多线程环境下对 `StringBuffer` 对象的修改是线程安全的。
  7. **StringBuilder：** `StringBuilder` 类是**非线程安全**的。 它的方法没有进行同步，因此在多线程环境下使用可能会导致数据不一致的问题。查看 `StringBuilder` 类的源码，可以看到它的 `append()`、`insert()`、`delete()` 等方法都没有使用 **synchronized** 关键字进行同步。 这意味着多个线程可以同时访问 `StringBuilder` 对象的这些方法，从而导致数据竞争和线程安全问题。
8. **性能（Performance）** ：
  9. **String**：由于 `String` 的不可变性，每次修改都会创建新的对象，导致大量的内存分配和垃圾回收，性能较低。 尤其是在循环中频繁修改字符串时，性能问题会更加明显。
  10. **StringBuffer：** `StringBuffer` 在多线程环境下性能略低于 `StringBuilder`，因为它需要进行同步操作。 同步操作会带来一定的性能开销，例如线程上下文切换、锁竞争等。
  11. **StringBuilder**：`StringBuilder` 在单线程环境下性能最高，因为它没有同步开销。 所有的操作都是在同一个对象上进行的，避免了频繁创建对象的开销。
12. **应用场景（Application Context）** ：
  13. **String**：
 ① 存储字符串常量。
 ② 少量字符串修改操作。
 ③ 需要在多线程环境下共享字符串对象。
  14. **StringBuffer**：
 ① 在多线程环境下需要频繁修改字符串。
 ② 需要保证线程安全。
  15. **StringBuilder**：
 ① 在单线程环境下需要频繁修改字符串。
 ② 不需要保证线程安全。

---

##  知识拓展

1.

**String类对象的内存图解 示意图如下**：
   →

### 评论
 

验证登录状态...

## 线程池参数｜大厂面试题｜Java高频面试题｜核心线程数、最大线程数、任务队列、拒绝策略

#  线程池参数

##  简要回答

- 线程池的构造函数有七个参数，分别是核心线程数（**corePoolSize**）、最大线程数（**maximumPoolSize**）、空闲线程存活时间（**keepAliveTime**）、时间单位（**TimeUnit**）、线程池任务队列（**workQueue**）、线程工厂（**ThreadFactory**）和拒绝策略（**RejectedExecutionHandler**）。
- 线程池的参数决定了线程池的并发能力，资源占用和任务处理策略，需要根据实际场景进行合理配置。
##  详细回答

1. **corePoolSize**：核心线程数，线程池中长期存活的线程数
  2. 如果线程池中的线程数量小于核心线程数，这些线程处于空闲状态也不会被销毁
3. **maximumPoolSize**：最大线程数，线程池最多能创建的线程数
  4. 当核心线程已满，任务队列已满时，如果当前线程小于最大线程数，就会创建新的线程执行此任务，否则触发拒绝策略。
5. **keepAliveTime**：空闲线程存活时间。
  6. 当线程数大于核心线程数时，如果每个线程的空闲时间超过了keepAliveTime,这个线程就会被销毁，销毁线程数=当前线程数-核心线程数
7. **TimeUnit**：时间单位，keepAliveTime的时间单位。
8. **workQueue**：线程池的任务队列，线程池存放任务的队列，没有空闲线程执行新任务时，存储所有待执行任务。
9. **ThreadFactory**：创建线程的工厂，通过此方法设置线程的优先级、线程命名规则以及线程类型（用户线程/守护线程）。
10. **RejectedExecutionHandler**：拒绝策略，当线程池的任务超出线程池队列可以存储的最大值之后，执行的策略。
##  使用场景


线程池参数设置需要结合任务特性和系统资源综合判断，在**避免资源浪费的同时防止任务堆积或线程竞争过度**。

1. CPU密集型任务（计算、排序）
  2. 任务消耗CPU资源，线程执行时阻塞少
  3. corePoolSize = CPU核心数 + 1 ：减少线程切换开销。
4. IO密集型任务（数据库、文件IO）
  5. 任务大部分时间等待IO，CPU利用率低。
  6. corePoolSize = CPU核心数 * 2 ：利用空闲CPU处理更多任务，提高并发。
##  知识图解

1. 线程池工作原理  知识扩展

  1. 扩展：
  2. 线程池种类：
    1. **ScheduledThreadPoolExecutor**：定时执行任务，支持定时执行和周期执行。
    2. **FixedThreadPool**：固定大小线程池，线程数量固定,核心线程数和最大线程数相同，不会创建更多线程处理任务。
    3. **CachedThreadPool**：可缓存线程池，在于线程数是几乎可以无限增加（最大Integer.MAX_VALUE，2^31-1），线程闲置时会进行回收。
    4. **SingleThreadExecutor**：使用唯一的线程去执行任务，原理和FixedThreadPool一样，如果线程在执行任务时发生异常，会重新创建新线程执行后续任务，适合所有任务需要按顺序执行场景。
    5. **SingleThreadScheduledExecutor**：和ScheduledThreadPool线程池相似，属于其特例，内部只有一个线程。
  1. 面试官可能追问:
  3. Q1：可不可以设置核心线程数为0？
    4. **可以**，当核心线程数为0时，有新任务时会将任务添加到任务队列，如果此时工作的线程数为0，则创建一个线程来执行任务。
  5. Q2：线程池中的shutdown(),shutdownNow()这两个方法有什么作用？
    6. shutdown()使用后会**置状态为SHUTDOWN**,**正在执行的任务会继续**，没有被执行则中断，不会再往线程池中添加新的任务，否则抛出RejectedExecutionException。
    7. shutdownNow()使用后会**置状态为STOP**,**正在执行任务的线程会中断**，没有被执行的任务则返回。
    8. shutdownNow()终止线程调用的是Thread.interrupt()方法，如果线程中没有sleep,wait,Condition或者定时锁等应用，interrupt()方法是无法中断当前线程的，仍需等待正在执行的任务都执行完成了才能退出。
  9. Q3：提交给线程池中的任务可以被撤回吗？
    10. **可以**，当向线程池提交任务时，会得到一个Future对象，通过Future对象可以取消任务。
    11. 调用Future中的**cancel(boolean mayInterruptIfRunning)方法**，该方法会尝试取消执行的任务，参数mayInterruptIfRunning表示是否允许取消正在执行的任务。如果设置为true，则表示允许取消正在执行的任务，如果设置false，则表示不允许取消正在执行的任务。
  12. Q4：如果corePoolSize=5，maximumPoolSize=10，队列容量=20，同时提交30个任务，执行流程是什么？
    1. 前五个任务由核心线程直接执行
    2. 20个任务进入队列等待
    3. 剩余五个任务创建非核心线程执行
    4. 如果后面再提交任务，会执行拒绝策略  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 线程安全｜高频面试题｜Java高频面试题｜synchronized、Lock、CAS、原子类

#  线程安全

##  简要回答


实现Java线程安全，需要**控制多个线程对共享资源的并发访问**，避免出现**数据不一致**，**竞态条件**等问题


可以分为三种方式：

- 阻塞同步（使用锁实现）
- 非阻塞同步（基于CAS操作）
- 无同步方案（避免共享资源）
##  详细回答

1. 阻塞同步：通过“**加锁-释放锁**”控制线程对资源的访问，未获取锁的线程会阻塞等待。
- synchronized关键字（隐式锁）
  - 通过**JVM隐式管理锁**，对对象或者类加锁，确保同一时间只有一个线程执行同步代码。

```
`//同步方法
public synchronized void setA(){
  // 执行代码逻辑
}

//同步代码块
public void setB(){
  synchronized (this){
  // 执行代码逻辑
  }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11

- JUC显式锁（Lock接口）
  - 通过ReentrantLock**手动加锁和释放锁**，能够支持更复杂的同步场景，比如公平锁、超时释放、中断。
  - 通过**读写锁**，允许多个读取者同时访问共享资源，但只允许一个写入者。
1. 非阻塞同步
- 原子类（CAS操作）：通过**循环重试**保证变量的更新。
- 版本号机制：乐观锁，通过**版本号来判断数据是否被修改**，仅在提交时校验。
- 优缺点：
  - 避免线程阻塞，唤醒线程开销以及用户态内核态切换带来的**性能问题**。
  - **过度自旋会浪费CPU资源**。
  - 仅能**操作单个共享资源**，对于组合类型还是需要加锁处理，或者重新组合为一个共享资源。
  - ABA问题
1. 无同步方案：让共享资源尽可能的在同一个线程中执行。
- 线程本地存储：为线程**创建独立的资源副本**，无需同步
- 创建不可变对象
##  使用场景
 实现方式 适用场景 阻塞方式 共享资源操作复杂、资源竞争不激烈、需要严格保证资源独占性场景 非阻塞操作 高频/简单操作、重试成本低、不允许线程阻塞场景 无同步方案 数据仅线程私有、数据不可修改、业务逻辑不依赖状态
##  知识图解

1.

ABA问题示意图  知识扩展

  1. 扩展：
  2. ABA问题：
    3. 在CAS更新的过程中，**读取到的值是A**，准备赋值时是A，但是其实是因为A的值先变成B又被改成了A。
    4. ABA漏洞在大部分场景下不会影响最终结果，可以通过Java中的**AtomicStampedReference**来解决问题，该类会加入**预期标志**和**更新后标志**两个字段，更新时不光检查值，还要检查当前标志是否等于预期标志，相等则更新。
  1. 面试官可能追问:
  5. Q1：原子类的底层原理是什么？
    6. 底层是**CAS指令**+**volatile**+**自旋重试**的组合。CAS保证操作的原子性，volatile保证变量的可见性，自旋重试解决并发冲突。
  7. Q2：线程安全体现在哪些方面？
    8. **原子性**：提供互斥访问，同一时刻只有一个线程对数据进行操作。
    9. **可见性**：线程对主存修改可以及时被其他线程看到。
    10. **有序性**：线程中的指令执行有序。可以使用happens-before原则实现。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 多态｜大厂面试题｜Java高频面试题｜动态绑定、方法重写、向上转型

#  多态

##  简要回答

- **定义**：
  - 多态（Polymorphism）是指“**一个接口，多种实现**”。在 Java 中，多态主要体现在——允许**父类引用指向子类对象**，并在运行时根据对象的实际类型调用相应的方法，即动态绑定机制。
- **特点**：
  - **运行时绑定（动态绑定）**：这是多态的核心机制，编译器在编译时只知道引用变量的类型（静态类型），但在程序运行时，JVM 会根据对象的实际类型（动态类型）来查找并调用相应的方法。
  - **提高可扩展性**：增加新的子类时，无需修改现有代码，只需让新子类重写父类方法。
- **实现多态的三个必要条件**：
  1. **继承或实现关系**：必须存在子父类继承关系或接口实现关系。
  2. **方法重写（Override）**：子类必须重写父类的方法（或实现接口方法）。
  3. **父类引用指向子类对象**。
- **优点**：
  - **可维护性高**：修改具体实现不影响调用方。
  - **可扩展性强**：新增子类只需实现父类接口或重写方法，无需修改调用方代码。
- **缺点**：
  - **父类引用不能直接使用子类的特有成员**：这是多态的一个限制。例如，`Animal animal = new Dog();`，`animal` 引用对象无法直接调用 `Dog` 类特有的方法，除非进行强制类型转换。

---

##  详细回答

###  多态的概念

1. **为什么需要多态？**
  2. 在没有多态的情况下，如果我们需要处理多种不同类型的对象（但它们有共同的行为），就可能需要使用大量的 `if-else` 或 `switch-case` 语句来判断对象的具体类型，然后再分别调用对应的方法。这会导致代码冗余、结构复杂且难以维护和扩展。多态的出现正是为了解决这种问题，它允许我们**以统一的方式处理不同类型的对象**。
3. **什么是多态？**
  4. 多态（Polymorphism）在 Java 中，它表现为**父类引用指向子类对象**，并在程序**运行时**根据对象的**实际类型**来决定调用哪个方法。多态成为可能，即同一个方法调用，在不同对象上会产生不同的行为。
5. **多态的实现步骤（必要条件）**：
  1. **有继承或实现关系**：必须存在**父子类继承**关系，或者**类实现接口**的关系，这是多态的基础。
  2. **方法重写（Override）**：子类必须重写（覆盖）父类的方法，或者实现接口中定义的方法，这是多态行为差异的来源。
  3. **父类引用指向子类对象**：这是多态的语法形式，即 `父类类型 变量名 = new 子类类型();`。
###  多态的使用

1. **多态中成员方法的使用（“编译看左，运行看右”）**：
  2. **编译时看左边（引用类型）**：编译器在编译阶段，会检查引用变量的**声明类型**（即左边的父类类型），看它里面是否存在被调用的方法。如果父类中没有这个方法，编译就会报错。这决定了我们**能调用哪些方法**。
  3. **运行时看右边（实际对象类型）**：在程序运行时，JVM 会根据引用变量实际指向的**对象类型**（即右边的子类类型）来决定调用哪个方法。如果子类重写了该方法，则调用子类重写后的方法；如果子类没有重写，则向上查找并调用父类的方法。这决定了运行时**实际使用哪个方法**。
4. **多态中成员变量的使用（“编译看左，运行看左”）**：
  5. **编译时看左边（引用类型）**：编译器在编译阶段，会检查引用变量的声明类型中是否存在被访问的成员变量。
  6. **运行时看左边（引用类型）**：在多态关系中，成员变量**不涉及重写**。无论引用变量实际指向哪个子类对象，通过该引用访问的成员变量始终是**声明类型（左边）中定义的那个成员变量**。
###  多态的优缺点

1. **多态的优点**：
  2. **提高代码的灵活性和可扩展性**：通过多态机制可以编写更通用的代码来处理不同类型的对象，而无需为每种具体类型编写特定逻辑。当需要增加新的子类时，也无需修改现有代码。
  3. **降低耦合度**：调用者只需面向父类类型或接口编程，而无需关心具体的子类实现，从而实现了 **调用者** 与 **具体实现** 之间的解耦。
  4. **提高代码复用性**：通过父类引用或接口，可以统一处理多种不同类型的对象，以提高代码的复用性。
  5. **简化代码**：避免了大量的 `if-else` 或 `switch-case` 语句来判断对象的具体类型，使代码更加简洁。
6. **多态的缺点**：
  7. **父类引用不能直接使用子类的特有成员**：这是多态的一个主要限制。当父类引用指向子类对象时，它只能访问父类中声明的成员（包括方法和变量），而不能直接访问子类中特有的成员。如果需要访问子类特有成员，必须进行**向下转型**。
###  多态的类型转换

1. **向上转型（自动）**：
  2. **概念**：将子类对象或引用赋值给父类类型的引用变量。这是**自动发生**的，不需要强制类型转换。
  3. **格式**：`父类类型 变量名 = new 子类类型();`
  4. **特点**：向上转型后，引用变量只能访问父类中声明的成员。
5. **向下转型（强制）**：
  6. **概念**：将父类类型的引用变量强制转换为子类类型。这是**强制发生**的，需要显式地进行类型转换。
  7. **格式**：`子类类型 变量名 = (子类类型) 父类引用变量;`
  8. **特点**：向下转型后，可以访问子类特有的成员。
9. **注意事项**：
  10. **只有在继承关系的基础上才能进行类型转换**：否则，会抛出 `ClassCastException`（类型转换异常）。
  11. **对引用变量进行向下转型之前，必须要保证该引用变量指向的堆中真正的对象就是目标类型**：即只有当父类引用变量实际指向的对象就是**目标子类类型** 或 **目标子类类型的子类型**时，向下转型才能成功。例如，`Animal animal = new Cat();`，如果想将 `animal` 强制转型为 `Dog` 类型，就会失败，因为 `animal` 实际指向的是 `Cat` 对象，而不是 `Dog` 对象。
12. **`instanceof` 关键字**：
  13. 为了避免在向下转型时发生 `ClassCastException`，可以使用 **`instanceof`** 关键字在转型前进行类型判断。
  14. **格式**：`if (引用变量 instanceof 目标类型) { ... }`
  15. **作用**：判断引用变量所指向的实际对象是否是指定类型或其子类型。
###  多态的应用场景

1. **多态数组**：
  2. **概念**：定义一个**父类类型（或接口类型）的数组**，数组中可以**存放**该父类（或接口）的**各种子类（或实现类）的对象**，方便统一管理和操作不同类型的对象。
  3. **示例**：`Animal[] animals = new Animal[3];` `animals[0] = new Dog();` `animals[1] = new Cat();`
4. **多态参数**：
  5. **概念**：方法的**参数类型定义为父类类型（或接口类型）** ，在调用方法时可以**传入**任何该父类（或接口）的**子类（或实现类）的对象**，每传入一个子类对象，都相当于形成了一次多态（父类引用指向子类对象），多态参数可以提高方法的通用性，减少方法重载的数量。
  6. **示例**：`public void showAnimal(Animal animal) { //。。。 }`。调用时可以传入 `new Dog()` 或 `new Cat()`。

---

##  知识拓展

1. **多态的必要条件 与 基本形式**，如下图所示：
   →

### 评论
 

验证登录状态...

## Java反射的优缺点及应用场景是什么？

#  Java反射的优缺点及应用场景是什么？

##  简要回答

- Java反射是指程序在**运行时**动态获取类信息（字段、方法、构造器、注解等），并完成对象创建与方法调用的机制，核心是 `Class` 对象，即JVM为每个加载的类生成的元数据对象。
- **优点**：灵活性高、扩展性强、解耦效果好，是Spring（IOC/AOP）、MyBatis（ORM）等框架的核心基础（AOP底层动态代理依赖反射实现）。
- **缺点**：反射调用比直接调用慢，运行时会解析元数据、进行访问权限检查；编译期的静态类型检查被弱化，错误推迟到运行期暴露；过度使用 setAccessible(true) 会破坏封装、带来安全风险，还会降低代码可读性与可维护性。
- **应用场景**：反射常常被应用于依赖注入、注解扫描、动态代理、配置绑定、序列化/反序列化、BeanUtils（对象拷贝）、Validator（参数校验）等通用工具。
##  详细回答

1. 反射是什么
  2. 反射（Reflection）是Java提供的运行时元数据访问机制。程序可以在不知道具体实现类的前提下，动态地：**获取类结构信息**（字段、方法、构造器、注解、父类、接口）、**创建对象**（调用构造器）、**调用方法**、**读写字段**（包含私有成员）
  3. 常见API：
    4. 获取Class对象：`Class.forName()`、`对象.getClass()`、`类名.class`
    5. 获取成员：`getDeclaredFields()`、`getDeclaredMethods()`、`getDeclaredConstructor()`
    6. 动态调用：`newInstance()`、`invoke()`、`set()/get()`
7. Java反射的优点
  8. **动态扩展能力强**：可以在运行时决定加载哪个类，适合插件化、SPI扩展、策略动态切换。
  9. **降低耦合**：业务代码依赖接口或配置，不依赖具体实现类，便于模块解耦。
  10. **通用框架能力**：很多“自动化”能力都基于反射，例如IOC注入、ORM映射、注解驱动校验。
  11. **提高开发效率**：可抽象出通用工具，减少重复代码（如对象映射、参数绑定、通用导出）。
12. Java反射的缺点
  13. **性能开销更高**：反射调用需要额外的访问检查和动态分派，热点路径中频繁使用会拖慢性能。
  14. **类型安全变弱**：编译期无法完全校验，很多错误会在运行时才暴露（如方法名写错、参数不匹配）。
  15. **可读性与维护性下降**：反射代码不直观，调试和排错成本高。
  16. **可能破坏封装**：通过 `setAccessible(true)` 可访问私有成员，不当使用会带来安全和设计风险。
17. 使用反射的实践建议
  18. 在框架层、基础设施层使用反射，在高频业务逻辑中尽量避免。
  19. 缓存 `Class/Method/Field` 元数据，减少重复查找。
  20. 能用接口、多态、工厂模式解决的场景，优先不用反射。
  21. 对私有成员反射访问要受控，避免滥用 `setAccessible(true)`。
##  使用场景

1. Spring IOC/DI
  2. 容器扫描注解后，通过反射实例化Bean并完成属性注入。
3. Spring AOP与动态代理
  4. 通过JDK动态代理或CGLIB在运行时生成代理对象，拦截并增强方法调用。
5. ORM框架（如MyBatis）
  6. 将数据库结果集与Java对象字段动态映射，自动完成对象填充。
7. 插件化与SPI
  8. 根据配置动态加载实现类，支持按需扩展能力。
9. 通用工具能力
  10. 如对象拷贝、参数校验、配置绑定、JSON序列化/反序列化等。
##  代码示例


```
`import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ReflectionDemo {
    public static void main(String[] args) throws Exception {
        // 1. 获取Class对象
        Class<?> clazz = Class.forName("demo.User");

        // 2. 通过构造器创建对象
        Constructor<?> constructor = clazz.getDeclaredConstructor(String.class, int.class);
        Object user = constructor.newInstance("Alice", 20);

        // 3. 访问私有字段
        Field nameField = clazz.getDeclaredField("name");
        nameField.setAccessible(true);
        nameField.set(user, "Bob");

        // 4. 调用方法
        Method method = clazz.getDeclaredMethod("sayHello");
        method.invoke(user); // 输出：Hello, I am Bob
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23

##  知识图解

1. Java反射机制示意  知识扩展

  1. 扩展：
    2. 反射是“运行时动态能力”，泛型是“编译期类型约束”，两者关注点不同但经常配合使用。
    3. Java9之后模块化（JPMS）增强了封装边界，对非法深度反射访问限制更严格。
  4. 面试官可能追问：
  2. Q1：反射一定会触发类初始化吗？
    3. 不一定。仅获取 `Class` 元数据不一定触发初始化；执行静态方法、访问静态字段或反射创建实例通常会触发初始化。
  4. Q2：`Class.newInstance()` 和 `Constructor.newInstance()` 有什么区别？
    5. `Class.newInstance()` 要求无参构造且异常处理能力弱，已不推荐；更推荐 `Constructor.newInstance()`，控制更细。
  6. Q3：反射性能慢，线上怎么优化？
    7. 缓存反射元数据、减少热路径反射调用、提前初始化、必要时改为字节码增强或 `MethodHandle`。
  8. Q4：反射和动态代理有什么关系？
    9. 动态代理底层通常借助反射调用目标方法，代理负责“拦截与增强”，反射负责“动态调用能力”。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## 介绍一下Java的IO流？

#  介绍一下Java的IO流？

##  简要回答

- Java IO流是Java用于**数据输入/输出**的一套抽象机制，核心思想是“将程序与数据源的交互过程抽象为‘流’，像水流一样单向、顺序读写数据”。
- IO流可按多个维度分类：按数据单位分为**字节流**（8bit）和**字符流**（字节+编码转换）；按方向分为**输入流**（读数据到程序）和**输出流**（写数据到外部）；按功能分为**节点流**（直接连接数据源）和**处理流（包装流）**（增强其他流的能力）。
- 常见字节流基类是 `InputStream/OutputStream`，常见字符流基类是 `Reader/Writer`。
- 实际开发中：二进制文件优先字节流，文本优先字符流；使用 `Buffered` 包装流减少系统调用提升性能，并配合 `try-with-resources` 自动关闭资源（无需手动flush）。
##  详细回答

1. 什么是IO流
  2. IO是Input/Output的缩写，即输入和输出。
  3. Java把不同数据源（文件、内存、网络、控制台）统一抽象为“流”，程序通过流来读取或写入数据。
4. IO流的分类
  5. 按数据单位：
    6. **字节流**：以字节（8 bit）为单位，适合图片、视频、音频、压缩包等二进制数据。
    7. **字符流**：以字符为单位，内部会处理字符编码，适合文本数据。
  8. 按流向：
    9. **输入流**：把数据读入程序（`InputStream`、`Reader`）。
    10. **输出流**：把数据从程序写出（`OutputStream`、`Writer`）。
  11. 按角色：
    12. **节点流**：直接连接数据源，如 `FileInputStream`、`FileReader`。
    13. **处理流（包装流）**：对节点流增强，如 `BufferedInputStream`、`BufferedReader`、`DataInputStream`、`ObjectOutputStream`。
14. 常见IO类
  15. 字节流常用类：
    16. `FileInputStream/FileOutputStream`：文件字节读写
    17. `BufferedInputStream/BufferedOutputStream`：缓冲加速
    18. `DataInputStream/DataOutputStream`：按基本类型读写
    19. `ObjectInputStream/ObjectOutputStream`：对象序列化
  20. 字符流常用类：
    21. `FileReader/FileWriter`：文件字符读写
    22. `BufferedReader/BufferedWriter`：缓冲、按行读写
    23. `InputStreamReader/OutputStreamWriter`：字节流与字符流桥接，可指定编码
24. 读写流程：先创建流对象（必要时套上缓冲流），然后进行循环读取/写入（通常用缓冲区数组），最后需要刷新并关闭资源（推荐 `try-with-resources`）。
25. 使用注意点
  26. 文本处理要显式指定编码（如UTF-8），避免乱码。
  27. 不要逐字节频繁读写，尽量使用缓冲区提高效率。
  28. `flush()` 负责把缓冲区数据刷出，`close()` 会自动调用 `flush()` 并释放资源。
  29. 多线程并发写同一文件要做好同步控制，避免数据错乱。
##  代码示例


```
`import java.io.*;
import java.nio.charset.StandardCharsets;

public class IoDemo {
    public static void main(String[] args) {
        String src = "input.txt";
        String dst = "output.txt";

        // 使用字符流 + 缓冲流 + try-with-resources（自动close，无需手动flush）
        // FileOutputStream第二个参数为true时追加写入，false（默认）覆盖写入
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(new FileInputStream(src), StandardCharsets.UTF_8));
             BufferedWriter bw = new BufferedWriter(
                new OutputStreamWriter(new FileOutputStream(dst, false), StandardCharsets.UTF_8))) {

            String line;
            while ((line = br.readLine()) != null) {
                bw.write(line);
                bw.newLine(); // 写入换行符（跨平台兼容）
            }
            // 无需手动flush：try-with-resources会自动调用close()，BufferedWriter的close()包含flush()
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26

##  知识图解

1. JavaIO体系示意图  知识扩展

  1. 扩展：
    2. Java传统IO（`java.io`）是阻塞式、面向流；NIO（`java.nio`）是面向缓冲区/通道，支持非阻塞能力，适合高并发场景。
    3. 大文件拷贝可考虑 `Files.copy()` 或 `FileChannel`，通常比手写小缓冲循环更高效。
  4. 面试官可能追问：
  2. Q1：字节流和字符流怎么选？
    3. 二进制数据用字节流；文本数据用字符流，且注意字符集一致。
  4. Q2：为什么要用 `Buffered` 流？
    5. 减少系统调用次数，提升IO效率，尤其是频繁小数据读写场景。
  6. Q3：`flush()` 和 `close()` 有什么区别？
    7. `flush()` 仅刷新缓冲区，不释放资源；`close()` 会先刷新再关闭流并释放资源。
  8. Q4：`FileReader` 和 `InputStreamReader` 的区别？
    9. `FileReader` 使用平台默认编码，不够可控；`InputStreamReader` 可显式指定编码，更推荐。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## HashMap、HashSet、HashTable与ConcurrentHashMap｜高频面试题｜Java高频面试题｜线程安全、null值、哈希冲突、扩容机制

#  HashMap、HashSet、HashTable与ConcurrentHashMap

##  简要回答

1. HashMap：存键值对（键唯一、值可重，允许 null 键值），非同步（单线程快），底层哈希表，迭代器为 Iterator（并发修改可能抛异常），可自定义初始容量和加载因子，继承 AbstractMap。
2. HashTable：存键值对（键值均不允许 null），同步（方法加锁，多线程性能差），底层哈希表，迭代器为 Enumeration，初始容量和加载因子固定，继承 Dictionary。
3. HashSet：存唯一元素（无键值，仅元素操作，允许 1 个 null），非同步，底层基于 HashMap（仅用键位），迭代器为 Iterator，因只存键，性能略优于 HashMap。
4. ConcurrentHashMap：存键值对（键值均不允许 null），线程安全（分段锁 / Java8 后 CAS，高并发性能优），支持并发增删（迭代顺序不保证），可自定义容量、加载因子和并发级别。
##  详细回答

1. HashMap
  2. 用于存储键值对，键唯一，值可以重复，通过键可以获取对应值。底层通过哈希表实现。
  3. 不是同步的，不能保证线程安全，可以使用Collections.synchronizedMap()创建同步的HashMap。
  4. 在单线程环境中可能比较快，因为没有同步开销。
  5. 性能会受到键的哈希分布和哈希冲突的影响。
  6. 允许使用null键和null值。
  7. HashMap是AbstractMap的子类，实现了Map接口。
  8. 迭代器通过Iterator实现，迭代过程中如果有其他线程对其修改，可能抛出异常。
  9. HashMap允许通过构造方法设置初始容量和加载因子。
  10. 默认初始容量是16，每次扩容变成原来的两倍。
11. HashTable
  12. 是同步的，方法线程安全，通过在每个方法上添加同步关键字synchronized来实现。
  13. 多线程环境中可能性能较差。
  14. 不允许使用null键和null值。
  15. 迭代器通过Enumeration实现。
  16. HashTable是Dictionary的子类。
  17. HashTable的初始容量和加载因子是固定的。
  18. 默认初始容量是11，每次扩容变成原来的2n+1。
19. HashSet
  20. 不是同步的，线程不安全。
  21. 存储唯一元素，不能重复，只能通过元素本身进行操作。
  22. 基于HashMap实现，使用键的部分，将值部分设置为一个固定常量。
  23. 允许存储一个null元素。
  24. 迭代器通过Iterator实现。
  25. 性能会受到键的哈希分布和哈希冲突的影响，但是它只存储键，通常比HashMap的性能稍好。
26. ConcurrentHashMap
  27. 是线程安全的，使用分段锁机制保证数据安全。
  28. 不允许null键或null值。
  29. 允许迭代时进行并发插入和删除，不能保证迭代器顺序。
  30. 在Java8后引入了ConcurrentHashMap(int initialCapacity,float loadFactor,int concurrentLevel)构造方法，允许设置初始容量、负载因子和并发级别。
  31. 高并发场景下，ConcurrentHashMap的性能比HashMap高很多。
##  适用场景

1. HashMap：适用于单线程下键值对的存储场景，如业务数据临时缓存，通过唯一键快速查询对应值，且有存储null键值的场景。
2. HashTable：仅适用于低并发，需要线程安全且禁止null键值的老旧场景。如维护遗留系统，但是因为同步方式低效，性能差，基本被弃用。
3. HashSet：适用于单线程下需要存储唯一元素的场景，如需要快速判断元素是否存在和需要去重的场景。
4. ConcurrentHashMap：适用于高并发场景下的键值对存储场景，如多线程读写的缓存，秒杀活动中更新商品库存，能够保证线程安全同时兼顾性能。 集合类 适用场景 HashMap 单线程下键值对的存储场景 HashTable 低并发，需要线程安全老旧场景 HashSet 单线程下需要存储唯一元素的场景 ConcurrentHashMap 高并发场景下的键值对存储场景
##  知识图解

1. HashSet实现唯一存储示意图  知识扩展

  1. 面试官可能追问：
  2. Q1：高并发场景下，为什么优先选 ConcurrentHashMap 而不是 “Collections.synchronizedMap (HashMap)”？两者的同步粒度和性能损耗有什么差异？
    3. Collections.synchronizedMap (HashMap)是全表锁（对象锁），同一时间仅允许一个线程读写，但是ConcurrentHashMap采用了节点锁（局部锁），可以允许多个线程同时操作不同节点的数据。
  4. Q2：为什么 HashMap 允许 null 键和 null 值，而 HashTable、ConcurrentHashMap 却禁止？HashSet 允许 1 个 null 元素，是否和它底层依赖的 HashMap 特性直接相关？
    5. HashMap允许null是为了简化实现，降低额外判断开销。HashMap会直接将null键视为0，并且固定存储在哈希表中第0个桶中，对null值作为普通值处理。单线程下用户可以通过containsKey(null)主动判断,避免歧义。
    6. HashTable和ConcurrentHashMao是线程安全的，并发场景下，null会难以判断key是否存在，为了避免歧义，直接禁止存储null键/值。
    7. HashSet是基于HashMap实现的，所以允许有null元素存在，插入第二个时，底层HashMap会判断已经存在null键，会返回false，所以只能插入一个null元素。
  8. Q3：当存储数据量较大时，HashMap 和 HashTable 的扩容机制（如初始容量、加载因子、扩容后容量）有什么不同？这会如何影响它们的性能？
    9. 从哈希冲突频率来看，HashTable冲突频率高。HashMap的容量始终是2的幂次，配合扰动函数可以让元素重新分布后保持较好的均匀性，但是HashTable的容量默认是11，元素的均匀只依赖hashCode()的质量。
    10. 从扩容开销来看，HashTable扩容开销大。HashMap扩容容量翻倍，无需重复计算哈希值，HashTable扩容为原来的2n+1，需要对每个元素重新计算哈希值。
    11. 从初始化操作来看，HashMap可以自定义初始容量和加载因子，可以根据数据灵活调整，HashTable的初始容量默认为11，加载因子默认为0.75，不能自定义。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Checked与Unchecked异常｜高频面试题｜Java高频面试题｜编译期异常、运行期异常、try

#  Checked与Unchecked异常

##  简要回答

###  Checked Exception 和 Unchecked Exception 的概念

1. **Checked Exception (编译期异常)** ：指所有直接或间接继承自 `java.lang.Exception`，但不是 `java.lang.RuntimeException` 及其子类的异常。编译器会强制检查这类异常。
2. **Unchecked Exception (运行期异常)** ：指 `java.lang.RuntimeException` 类及其所有子类。编译器不强制检查这类异常。
###  Checked Exception 和 Unchecked Exception 的区别

-

**如下表所示**：
 区分维度 CheckedException(编译期异常) UncheckedException(运行期异常) **编译器检查** 强制检查，必须 `try-catch` 或 `throws` 声明。 不强制检查，可选择性处理。 **发生原因** 通常是外部因素或可预见的问题，如I/O错误。 通常是程序逻辑错误或编程缺陷，如空指针。 **可恢复性** 通常可恢复，程序可继续执行。 通常不可恢复，表示程序状态可能已经损坏。 **处理策略** 必须显式处理，提高代码健壮性。 通过改进代码逻辑来尽量避免，不强制要求捕获。 **API 设计** 适用于要求调用者必须处理的API 适用于表示编程错误，不侵入调用者代码。

---

##  详细回答

###  Checked Exception 和 Unchecked Exception 的概念

1. **Checked Exception (受检查异常，也叫编译期异常)** ：是指所有**直接或间接**继承自 `java.lang.Exception`，但不是属于 `java.lang.RuntimeException` 及其子类的异常。编译器会强制检查这类异常。
2. **Unchecked Exception (非受检查异常，也叫运行期异常)** ：是指属于 `java.lang.RuntimeException` 类及其所有子类。编译器不强制检查这类异常。
###  Checked Exception 和 Unchecked Exception 的区别

1. **编译器检查 (Compiler Check)** ：
  2. **Checked Exception**：
 Java**编译器会强制要求**开发者对这类异常进行**显式**处理，例如使用 **`try-catch` 块** 来捕获 或者 在方法签名中使用 **`throws` 关键字** 声明抛出。
 如果没有显式处理，代码将无法编译通过。这种强制性可以确保程序对**外部环境**可能出现的、可预见的错误具备**健壮性**。
  3. **Unchecked Exception**：
 编译器**不会强制要求**开发者处理或声明这类异常，即使代码可能抛出例如 `NullPointerException` 或者 `ArrayIndexOutOfBoundsException` 这类异常。
 因为即使开发者不进行任何处理，代码也能正常编译通过。这可以让开发者不必为所有潜在的**运行期异常**编写冗余的 `try-catch`块，而是更专注于业务逻辑。
4. **发生原因 (Cause of Occurrence)** ：
  5. **Checked Exception**：
 通常是由**外部因素**或**可预见但不可避免的问题**导致的。例如，文件不存在（`FileNotFoundException`）、网络连接中断（`IOException`）、数据库连接失败（`SQLException`）等。
  6. **Unchecked Exception**：
 通常是由 **程序内部的逻辑错误**、**编程缺陷** 或者 **不合法的参数** 导致的。这些错误本应该在开发阶段通过更严谨的编码、输入校验和测试来避免。例如，空指针引用（`NullPointerException`）、数组越界（`ArrayIndexOutOfBoundsException`）、类型转换错误（`ClassCastException`）、非法参数（`IllegalArgumentException`）等。
7. **可恢复性 (Recoverability)** ：
  8. **Checked Exception**：
 通常**可恢复**。这类异常发生后，程序可以通过捕获异常并执行备用逻辑 或者 向用户提供错误提示等方式，从错误中恢复并继续执行，而不会导致程序立即崩溃。
  9. **Unchecked Exception**：
 通常**不可恢复**。这类异常发生时，往往意味着程序内部状态已经损坏或逻辑存在根本性缺陷，继续执行可能会导致更严重的问题。因此，通常不建议尝试恢复，而是让程序终止，以便开发者发现并修复根本的编程错误。
10. **处理策略 (Handling Strategy)** ：
  11. **Checked Exception**：
 **必须显式处理**，比如我们前面提到的 `try-catch`块 和 `throws`关键字。
  12. **Unchecked Exception**：
 **应该通过改进代码逻辑来尽量避免**。
13. **API 设计 (API Design)** ：
  14. **Checked Exception**：
 适用于要求**调用者必须处理**的API。当一个方法可能因为外部环境问题而失败 并且调用者有能力处理这种失败时，应该声明抛出 Checked Exception。这是一种明确的**契约**，告诉API的使用者可能需要处理的风险。
  15. **Unchecked Exception**：
 适用于表示**编程错误**，不侵入调用者代码。当一个方法因为调用者传递了非法参数 或 违反了方法的前置条件而失败时，通常抛出 Unchecked Exception。这表明调用者应该在使用API前确保参数的合法性，而不是强制调用者捕获。

---

##  知识拓展

1. **Checked Exception 和 Unchecked Exception 的区别**，示意图如下：
   →

### 评论
 

验证登录状态...

## Java锁｜大厂面试题｜Java高频面试题｜乐观锁、悲观锁、可重入锁、synchronized

#  Java锁

##  简要回答


Java中有很多关于锁的概念，可以分类成下面几个方面理解：

1. 按照锁的获取机制（看待并发同步的角度）：**乐观锁**/**悲观锁**
2. 按照锁的竞争策略：**公平锁**/**非公平锁**
3. 按照锁控制的资源范围：**偏向锁**/**轻量级锁**/**重量级锁**/**分段锁**
4. 按照功能特性：**可重入锁**/**读写锁**/**自旋锁**/**互斥锁**
5. 按照持有方式：**独享锁**/**共享锁**
##  详细回答

1. 乐观锁：**假设线程的并发访问不会发生冲突**，操作时不加锁，在更新数据时采用尝试更新，如有冲突则重试。
  2. 乐观锁在Java中即**无锁编程**
  3. 例如原子类，通过CAS自旋实现原子操作的更新。
4. 悲观锁：假设线程并发一定会有冲突，**每次操作前必须先加锁**，阻止其他线程干扰。
5. 公平锁：线程**按照申请锁的顺序获取锁**，先等待先获得。
  6. 优缺点：公平,但是效率低，存在线程唤醒开销。
7. 非公平锁：**线程获取锁时不按顺序**，允许“插队”。
  8. 优缺点：效率高，吞吐量大，但是可能导致优先级反转或饥饿现象。
- **偏向锁、轻量级锁、重量级锁都是锁的状态**，是针对synchronized的概念，通过对象监视器在对象头中的字段来表明的。
1. 偏向锁：是JVM对synchronized的优化，如果只有一个线程访问同步资源，一旦**线程获取锁，后续无需重复加锁**。
2. 轻量级锁：当**偏向锁被多个线程竞争时升级**为轻量级锁，其他线程会通过自旋尝试获取锁，**不会阻塞**。
3. 重量级锁：轻量级锁**自旋失败一定次数后升级**为重量级锁，线程进入阻塞。重量级锁依赖操作系统的互斥量实现，**重量级锁会使其他申请的线程进入阻塞**，性能降低。
4. 分段锁：将大对象拆分为多个小段，对每个段单独加锁，细化锁的粒度，减少锁竞争。
  5. 分段锁是一种锁的设计，**不是具体的锁**。
  6. 以ConcurrentHashMap中put操作为例，不会对整个hashmap加锁，会先通过hashcode计算放入的分段，对分段加锁。**如果不是放在同一个分段中，可以实现并行插入**。
7. 可重入锁（递归锁）：**线程可以重复获取已持有的锁**，避免自己锁死自己。
  8. 实现方式：synchronized（隐式）ReentrantLock（显式）

```
`// 由于可重入锁的特性，setB可以正常执行
synchronized void setA() throws Exception{
  Thread.sleep(1000);
  setB();
}
synchronized void setB() throws Exception{
  Thread.sleep(1000);
}
`
```
 1
2
3
4
5
6
7
8

1. 读写锁：区分“读操作”、“写操作”，**允许多个读线程并发访问**，读和写互斥，写和写互斥。
  2. ReadWriteLock
3. 自旋锁：线程获取锁失败时不立即阻塞，而是**循环尝试获取锁**，循环有次数限制。
  4. 优缺点：减少线程上下文切换开销，但是循环会消耗CPU。
5. 互斥锁：通过**互斥机制保证同一时间只允许一个线程持有锁**。
  6. ReentrantLock
- 互斥锁/读写锁是独享锁/共享锁的具体体现。
1. 独享锁：**同一时间只能有一个线程持有锁**
  2. ReentrantLock 是独享锁
  3. ReadWriteLock 写锁是独享锁
4. 共享锁：同一时间允许多个线程同时持有锁，**线程间不互斥**。
  5. ReadWriteLock 读锁是共享锁，保证并发读高效，而读写、写读、写写的过程互斥。
##  使用场景
 锁 使用场景 乐观锁 读操作频繁，冲突概率低的场景 悲观锁 写操作频繁，冲突概率高 偏向锁 单线程反复访问同步块 轻量级锁 短时间、低冲突并发场景 重量级锁 长耗时、高冲突并发场景 分段锁 对大对象的并发访问 读写锁 读多写少

**Java中具体工具**
 工具 适用场景 synchronized 实现互斥锁。对共享资源的访问进行同步控制 ReentrantLock 实现可重入锁。可手动控制锁的获取和释放，支持公平锁，适合更高级别控制场景 ReadWriteLock 读写锁接口。适用于读多写少场景 StampedLock 乐观读写锁。并发性能更高，适用于读多写少场景。 AtomicInteger 基于CAS的原子操作类（无锁）。实现共享变量的原子更新
##  知识图解

1. 锁的使用

 知识扩展

1. 扩展：
- CAS操作（Compare and Swap）是一种无锁的原子操作机制，广泛应用于多线程编程中，实现高效的线程安全。
  - CAS有三个操作数：内存位置（V），预期原值（A），新值（B）。
  - 具体操作：
    1. 从内存位置V读取当前值。
    2. 比较当前值和预期原值A是否相等。
    3. 如果相等就将内存位置V的值更新为B。
    4. 如果不相等，则说明该内存位置的值已经被修改过了，则不进行更新操作，选择重试或者执行其他逻辑。
- 死锁
  - 两个线程分别对两个共享资源使用了两个互斥锁，造成**两个线程都在等待对方释放锁**。
  - 需要同时满足四个条件：
    1. 互斥条件
    2. 持有并等待条件
    3. 不可剥夺条件
    4. 环路等待条件
1. 面试官可能追问：
- Q1：如何避免死锁的产生？
  - 破坏死锁出现的四个条件之一就可以避免死锁产生。
  1. 破坏环路等待条件：强制所有线程按固定顺序申请资源。是最常用和有效的避免死锁产生措施。
  2. 破坏持有并等待条件：线程在执行任务前一次性申请所有需要的资源，如果无法申请其中某资源，则释放已申请的所有资源并等待。
  3. 破坏不可剥夺条件：允许资源被强制回收。
  4. 破坏互斥条件：将资源设计成共享资源，局限性高，不常使用。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## 面向对象与面向过程｜大厂面试题｜编程范式高频面试题｜封装、继承、多态、OOP、POP

#  面向对象与面向过程

##  简要回答

###  面向对象 和 面向过程 的概念

1. **面向过程 (Procedural Programming)** ： 一种编程范式，强调**“怎么做”**。面向过程编程以 **功能** 为中心，并通过一系列的 **步骤**（函数/过程）来解决问题，在此期间，数据与操作通常是分离的。
2. **面向对象 (Object-Oriented Programming, OOP)** ： 另一种编程范式，强调**“谁来做”**。面向对象以 **数据（对象）** 为中心，将数据和操作数据的方法**封装**在一起，并通过对象之间的相互配合来解决问题。
###  面向对象 和 面向过程 的对比

-

如下表所示 :
 维度 面向过程(POP) 面向对象(OOP) **核心思想/关注点** 更关注**步骤、流程**，解决“怎么做”。 更关注**对象、职责**，解决“谁来做”。 **程序结构** 函数/过程的集合，其中数据与函数分离。 类和对象的集合，其中数据与方法封装在一起。 **代码复用与扩展性** 主要通过函数调用来实现，复用性较低，扩展性差。 通过封装、继承、多态 以及 它们的组合来实现，复用性高，扩展性强。 **安全性与维护性** 数据暴露，易被误改；维护困难。 数据封装，安全性高；功能模块化，更易维护。 **适用场景** 简单、固定流程的小型项目。 复杂、需求多变、大型项目。

---

##  详细回答

###  面向对象 和 面向过程 的概念

1. **面向过程 (Procedural Programming)** ：
  2. 面向过程是一种以 **过程（函数）** 为中心的编程范式。它的核心思想是要把问题分解成**一系列的步骤或功能**，然后用函数（或者叫过程）来实现这些步骤。
  3. 在面向过程中编程中，数据和处理数据的函数通常是**分离的**。整个程序执行的流程是**线性的**，一步一步地完成任务。它强调的是 **“怎么做”** ，即通过详细的步骤来指导计算机完成任务。
4. **面向对象 (Object-Oriented Programming, OOP)** ：
  5. 面向对象是一种以 **对象** 为中心的编程范式。它的核心思想是要将现实世界中的实体**抽象**为程序中的对象，每个对象包含 **数据（也就是属性）和操作数据的方法（也就是行为）** 。
  6. 程序通过对象之间的**消息传递**和**协作**来完成任务。它强调的是 **“谁来做”** ，即通过定义具有特定职责的对象来解决问题。Java、C++、Python 这些都是典型的面向对象语言。
###  面向对象 和 面向过程 的对比

1. **核心思想/关注点**：
  2. **面向过程**：
 其核心思想是**“自顶向下，逐步求精”**。它关注的是解决问题的**步骤和流程**，往往将一个大问题分解为若干个子问题，每个子问题对应一个函数或过程。
  3. **面向对象**：
 其核心思想是**“万物皆对象”**。它关注的是构成问题的**实体（对象）及其对应的职责**，往往将问题分解为相互协作的不同对象。
4. **程序结构**：
  5. **面向过程**：
 程序结构表现为一系列**函数（或过程）的集合**。其中，数据和处理数据的函数通常是分离的。
  6. **面向对象**：
 程序结构表现为**类和对象的集合**，属性和行为都**封装**在对象的内部，形成一个独立的模块。程序通过创建对象，并让对象之间相互发送消息（调用方法）来完成任务。
7. **代码复用与扩展性**：
  8. **面向过程**：
 主要通过**函数调用**来实现代码复用。所以说，如果某个功能需要修改，就可能需要修改多个调用点，甚至影响到不相关的部分，导致代码**复用性较低，扩展性较差**。
  9. **面向对象**：
 通过**三大特性**和它们的组合来实现代码复用。每当需求变化时，通常只需修改或添加新的类，而无需修改现有类的代码（开闭原则），这使得代码的**复用性更高，扩展性更强**。
10. **安全性与维护性**：
  11. **面向过程**：
 数据通常是全局的或在函数间自由传递，**数据暴露**，容易被不相关的函数意外修改，可能导致数据不一致或程序错误，所以面向过程**安全性较低**。又因为功能与数据紧密耦合，所以**维护也相对困难**。
  12. **面向对象**：
 通过**封装**机制，将数据隐藏在对象内部，只通过公共方法对外提供访问接口，因而**数据安全性高**，又因为对象与对象之间耦合度较低，所以**维护起来也更加容易**。
13. **适用场景**：
  14. **面向过程**：
 适用于**简单、功能固定、流程明确**的小型项目或特定任务。例如，操作系统底层（如C语言编写的内核部分）、嵌入式系统、简单的脚本程序或一次性计算任务。
  15. **面向对象**：
 更适用于**复杂、需求多变、需要长期维护和迭代**的大型项目。例如，图形用户界面（GUI）应用、Web 应用、企业级系统、游戏开发等。

---

###  知识拓展

1. **面向对象 和 面向过程 的对比图**如下所示：
   →

### 评论
 

验证登录状态...

## ArrayList与数组的区别｜高频面试题｜Java高频面试题｜动态数组、固定长度、泛型、装箱拆箱

#  ArrayList与数组的区别

##  简要回答

###  ArrayList 和 普通数组 的概念

1. **普通数组（Array）**：
  2. **定义**：它是 Java 语言中一个内置的、**固定大小**的数据结构，用于存储一系列**相同类型**的元素。
  3. **优点**：**访问速度快**（通过索引/下标直接访问元素），可以存储基本数据类型，内存开销小。
  4. **缺点**：数组长度固定**不可变**，**增删元素不方便**，功能单一。
  5. **适用场景**：元素数量已知且固定，对性能要求比较高，或仅需存储基本数据类型。
6. **`ArrayList`** ：
  7. **定义**：它是 `java.util` 包下的一个集合类，是 `List` 接口的最常用的实现类，ArrayList 的底层是基于**动态数组**实现的。
  8. **优点**：长度可变（动态扩容），具有丰富的API操作，且支持泛型。
  9. **缺点**：**增删元素（特别是中间位置）效率相对较低**，存储基本类型时 存在装箱拆箱带来的开销。
  10. **适用场景**：元素数量不确定，需要频繁查询或在末尾增删，需要集合框架提供的API操作。
###  ArrayList 和 普通数组 的区别

- 如下表所示： 维度 普通数组（Array） `ArrayList` **长度/容量** **固定长度**，一旦创建，长度不可变。 **长度可变**，可根据需要自动扩容和缩容。 **元素类型** 可存储**基本数据类型**和**引用类型**，但不能混用。 只能存储**引用类型**（基本类型会自动装箱）。 **API/操作** 功能单一，通过 `[]` 运算符访问，无内置方法。 提供了丰富的 API 方法（`add()`, `remove()`, `get()` 等）。 **泛型支持** 不直接支持泛型，编译时类型检查不如 `ArrayList` 严格。 **原生支持泛型**，提供编译时类型安全。 **底层实现** 语言内置的底层数据结构。 基于**动态数组**实现，是集合框架的一部分。

---

##  详细回答

###  ArrayList 和 普通数组 的概念

1. **普通数组（Array）**：
  2. **定义**：数组是 Java 语言内置的一种最基本、最底层的数据结构。它的本质是一个**固定大小**的同类型元素的集合，数组的长度在创建时就已经确定，并且在程序运行过程中不可更改。
  3. **优点**：
 ① **访问速度快**：因为数组在内存中是**连续存储**的，所以我们可以直接通过 **索引/下标** 直接计算出元素的内存地址，实现 **O(1)** 时间复杂度的 **随机访问**。
 ② **内存开销小**：数组可以直接存储 `int`, `char`, `boolean` 等基本数据类型，避免了拆装箱带来的开销。
  4. **缺点**：
 ① **数组一旦被创建，长度就无法改变**。如果需要增加或减少元素，必须创建一个新数组并将旧数组的元素复制过去，效率较低。
 ② **增删元素不便**：在数组**中间位置插入或删除元素**需要移动大量后续元素，时间复杂度是 **O(n)** 。
  5. **适用场景**：
 ① 当**元素数量已知且固定**时；
 ② 对**性能要求极高**，且主要进行随机访问操作的场景；
 ③ 需要存储**多维数据**（比如说矩阵）时。
6. **`ArrayList`** ：
  7. **定义**：`ArrayList` 是 `java.util` 包中的一个集合类，底层是基于一个**动态数组**来实现的。它允许存储重复元素，并保持了元素的插入顺序。
  8. **优点**：
 ① **长度可变（根据实际存储情况动态扩容）**：当元素数量超过当前容量时，它会自动扩容（通常是当前容量的 1.5 倍）。
 ② 作为 Java 集合框架的一部分，`ArrayList` 提供了比方说是 `add()`, `remove()`, `get()`, `size()`, `contains()` 这些很方便操作元素的方法。
 ③ **原生支持泛型**：`ArrayList<E>` 原生支持泛型，**在编译时进行类型检查**，提供了类型安全，避免了运行时 `ClassCastException`。
  9. **缺点**：
 ① **存储基本类型时有装箱拆箱带来的开销**：因为 `ArrayList` 只能存储引用类型。如果存储 `int` 等基本数据类型，Java 会自动进行**装箱（Autoboxing）**转换为 `Integer` 等包装类，带来了额外的内存开销。
 ② **扩容带来的开销**：扩容操作本身（创建新数组和复制元素）也是有性能开销的。
  10. **适用场景**：
 ① 当需要存储的**元素数量不确定**时。
 ② 当需要利用 Java 集合框架提供的 **丰富功能** 和 **类型安全** 时。
###  ArrayList 和 普通数组 的区别

1. **长度/容量**：
  2. **普通数组**：具有**固定长度**，一旦数组被创建，数组的大小就无法更改。如果需要一个更大或更小的数组，就必须创建一个新数组，并将旧数组中的元素复制到新数组中。
  3. **`ArrayList`**：具有**动态可变长度**。它是一个可变大小的集合，当集合内的元素数量达到了内部数组的当前容量，并需要添加新元素时，`ArrayList` 会自动创建一个更大的新数组，并将所有现有元素复制到新数组中（通常是当前容量的 1.5 倍），然后丢弃旧数组。
4. **元素类型**：
  5. **普通数组**：可以直接存储**基本数据类型**（如 `int`, `char`, `boolean`等），也可以直接存储 **引用类型**（如 `String`, `Object`等）。但是不能混用，即一旦数组被声明为某种类型，它就只能存储该类型或其子类型的元素。
  6. **`ArrayList`**：只能存储**引用类型**，存储基本数据类型会自动执行**装箱（Autoboxing）** 操作，将其转换为对应的包装类（例如，`int` 会被转换为 `Integer` 对象）。
7. **API/操作**：
  8. **普通数组**：功能相对单一，主要通过 **`[]` 运算符** 来进行元素的存取，并没有内置的方法来执行常见的集合操作，如添加、删除、查找元素、获取当前大小等，这些操作需要我们手动实现。
  9. **`ArrayList`**：作为 Java 集合框架的一部分，内部封装了很多实用的 **API 方法**。例如，`add()` 用于添加元素，`remove()` 用于删除元素，`get()` 用于获取元素，`size()` 用于获取当前元素数量，`contains()` 用于判断元素是否存在等。
10. **泛型支持**：
  11. **普通数组**：泛型支持不如 `ArrayList` 严格。虽然我们可以去声明 `Object[]` 类型的数组来存储不同类型的对象，但它在编译时不会提供严格的类型检查，这可能会导致**运行时**出现 `ArrayStoreException`。例如，`String[]` 可以赋值给 `Object[]`（数组是**协变**的），但向 `Object[]` 中添加 `Integer` 会在运行时报错。
  12. **`ArrayList`**：**原生支持泛型**（例如 `ArrayList<String>`）。这意味着在编译时，编译器会强制检查 要添加或获取的元素，它的类型是否与泛型参数一致，保证了**编译时类型安全**，避免了运行时出现 `ClassCastException`。
13. **底层实现**：
  14. **普通数组**：是 Java 语言内置的**底层数据结构**。
  15. **`ArrayList`**：是 Java 集合框架中的一个**集合类**，其底层是基于**普通数组**来实现的。`ArrayList` 内部维护一个 `Object[]` 数组elementData，并在此基础上封装了动态扩容、增删改查一套逻辑。

---

##  知识拓展

1.

**ArrayList 和 普通数组 的区别**示意图如下：
   →

### 评论
 

验证登录状态...

## ArrayList与LinkedList的区别｜高频面试题｜Java高频面试题｜动态数组、双向链表、随机访问、增删效率

#  ArrayList与LinkedList的区别

##  简要回答

###  ArrayList 和 LinkedList 的概念

1. **`ArrayList`** ：
  2. **定义**：是`java.util` 包下的一个类，是 `List` 接口的实现，提供了可变长度的列表功能。
  3. **底层实现**：基于**动态数组**（`Object[]` 数组）实现。
4. **`LinkedList`** ：
  5. **定义**：是 `java.util` 包下的一个类，是 `List` 接口的实现，同时实现了 `Deque`（双端队列）接口。
  6. **底层实现**：基于**双向链表**实现。
###  ArrayList 和 LinkedList 的区别

-

如下表所示：
 维度 `ArrayList`(基于数组) `LinkedList`(基于双向链表) **底层数据结构** `Object[]` 数组 双向链表（每个节点存储数据、前驱和后继指针） **随机访问** **高效** (O(1))，通过索引直接定位。 **低效** (O(n))，需要从头或尾遍历。 **中间增删** **低效** (O(n))，需要移动大量元素。 **高效** (O(1)，找到位置后)，只需修改指针。 **首尾增删** 尾部**高效** (平摊O(1)，末尾操作)，但可能涉及扩容。 **高效** (O(1))，直接修改头尾指针。 **线程安全** **非线程安全**。 **非线程安全**。

---

##  详细回答

###  ArrayList 和 LinkedList 的概念

1. **`ArrayList`** ：
  2. **定义**：`ArrayList` 是属于 `java.util` 包下的一个集合类，实现了 **`List`** 接口，允许存储重复元素，并保持元素的插入顺序。
  3. **底层实现**：ArrayList类源码中维护了一个Object类型的数组elementData，用来存储ArrayList集合中的元素。
  4. **适用场景**：
 ① 需要存储的元素数量不确定，但 **查询操作** 比较频繁的场景。
 ② 需要在**列表末尾**频繁**增删**元素的场景。
5. **`LinkedList`** ：
  6. **定义**：`LinkedList` 也属于 `List` 接口下的实现类，但它同时还实现了 `Deque` 接口（双端队列），这意味着它既可以作为列表使用，也可以作为队列（Queue）和栈（Stack）使用。
  7. **底层实现**：`LinkedList` 的底层是基于**双向链表**实现的。`LinkedList` 内部维护着对 **头节点(`first`)** 和 **尾节点(`last`)** 的引用。
  8. **适用场景**：
 ① 需要频繁在**列表的中间、开头或结尾**进行**插入和删除**操作的场景。
 ② 需要实现**队列（FIFO）** 或 **栈(LIFO)** 数据结构的场景。
###  ArrayList 和 LinkedList 的区别

1. **底层数据结构**：
  2. **`ArrayList`**：底层是**`Object[]` 数组**。数组在内存中是连续存储的，因此可以通过索引直接访问任意位置的元素。
  3. **`LinkedList`**：底层是**双向链表**。元素在内存中不一定是连续存储的，每个节点通过指针连接到前一个和后一个节点。
4. **随机访问（`get(index)` / `set(index, E)`）效率**：
  5. **`ArrayList`**：**高效**。时间复杂度为 **O(1)**。由于底层是数组，可以通过索引直接计算出元素的内存地址，实现快速随机访问。
  6. **`LinkedList`**：**低效**。时间复杂度为 **O(n)**。要访问特定索引的元素，需要从链表的头部或尾部开始遍历，直到找到目标位置。
7. **中间位置插入和删除（`add(index, E)` / `remove(index)`）效率**：
  8. **`ArrayList`**：**低效**。时间复杂度为 **O(n)**。在数组的中间位置插入或删除元素时，需要将插入点之后的所有元素向后或向前移动一位，操作量与元素数量成正比。
  9. **`LinkedList`**：**高效**。时间复杂度为 **O(1)**（在找到插入/删除位置的前提下）。一旦找到要操作的节点，只需修改前后节点的指针即可，无需移动大量元素。然而，**寻找目标位置本身仍然是 O(n)**。
10. **首尾位置插入和删除（`add(E)` / `remove(E)` / `addFirst()` / `removeLast()`）效率**：
  11. **`ArrayList`**：在**末尾**添加和删除元素是**高效**的（平摊 O(1)），因为通常只需在数组末尾操作，但偶尔会涉及扩容开销。在**头部**添加和删除则为 O(n)。
  12. **`LinkedList`**：在**首尾**添加和删除元素是**高效**的（O(1)），因为它直接操作头尾节点的指针，无需遍历。
13. **线程安全性**：
  14. **`ArrayList`** 和 **`LinkedList`** 都是**非线程安全**的。在多线程环境下使用时，如果存在写操作，需要进行外部同步（例如使用 `Collections.synchronizedList()` 包装，或使用 `java.util.concurrent` 包下的并发集合）。

---

##  知识拓展

1. **ArrayList 和 LinkedList 的对比图**如下所示：
   →

### 评论
 

验证登录状态...

## ArrayList扩容机制｜高频面试题｜Java高频面试题｜ArrayList、grow、1.5倍扩容、源码分析

#  ArrayList扩容机制

##  简要回答

- `ArrayList` 的扩容机制：
  - 当其**内部存储元素的数组**，它的容量不足以容纳新元素时，`ArrayList` 会自动创建一个更大的新数组，并将原数组中的所有元素都复制到这个新数组中。
  - 这个过程发生在当内部数组的 `size` 等于 `elementData.length` 并且还调用 `add()` 方法时触发。
  - 默认的扩容策略是**将当前容量扩大 1.5 倍**（`newCapacity = oldCapacity + (oldCapacity >> 1)`）。虽然单次扩容涉及元素复制，时间复杂度为 O(N)，但由于容量是指数级增长的，因此向 `ArrayList` 中添加元素的**均摊时间复杂度为 O(1)** 。

---

##  详细回答 (基于 JDK 17.0 源码)

###  无参构造下扩容机制的底层实现

1.

**初始状态：** 内部的 **`elementData`** 数组会被初始化为 `DEFAULT_CAPACITY_EMPTY_ELEMENTDATA`，这是一个**共享的、静态的空数组**。此时 `size` 为 0。


```
`// ArrayList.java (JDK 17)
private static final Object[] DEFAULT_CAPACITY_EMPTY_ELEMENTDATA = {};
transient Object[] elementData; // non-private to simplify nested class access
public ArrayList() {
    this.elementData = DEFAULT_CAPACITY_EMPTY_ELEMENTDATA;
}
`
```
 1
2
3
4
5
6

2.

**第一次 `add()` 方法调用：**

  3.

在 **`add(E e)`** 方法内部，jvm会进一步调用一个 **私有的、重载的** `add` 方法—— **`add(e, elementData, size)`** 。


```
`  public boolean add(E e) {
      modCount++;
      add(e, elementData, size);
      return true;
  }

  private void add(E e, Object[] elementData, int s) {
      if (s == elementData.length)
          elementData = grow();
      elementData[s] = e;
      size = s + 1;
  }
`
```
 1
2
3
4
5
6
7
8
9
10
11
12

  4.

在 `private void add(E e, Object[] elementData, int s)` 方法中，形参`s` (大小等于当前 `size`，为 0) 等于 `elementData.length` (大小也为 0，因为是空数组 **`DEFAULT_CAPACITY_EMPTY_ELEMENTDATA`**)，条件 `s == elementData.length` 成立。

  5.

于是调用 `elementData = grow(s + 1)`，此时 `s + 1` 为 `1`。这个 `1` 就是 `minCapacity` (最小所需容量)。

  6.

接着进入 **`grow(int minCapacity)`** 方法：


```
`  private Object[] grow(int minCapacity) {
      int oldCapacity = elementData.length;
      if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
          int newCapacity = ArraysSupport.newLength(oldCapacity,
                  minCapacity - oldCapacity, /* minimum growth */
                  oldCapacity >> 1           /* preferred growth */);
          return elementData = Arrays.copyOf(elementData, newCapacity);
      } else {
          return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
      }
  }
`
```
 1
2
3
4
5
6
7
8
9
10
11

  7.

可以看到，对于无参构造器创建的 `ArrayList`，第一次添加元素时，`grow` 方法的 `else` 分支会被执行，`elementData` 会被初始化为一个大小为 `Math.max(DEFAULT_CAPACITY, minCapacity)` 的数组。由于 **`DEFAULT_CAPACITY`** 是 10，`minCapacity` 是 1，所以第一次扩容后新数组的容量是 **10**。

  8.

`grow` 方法返回这个新的、容量为 10 的数组，并将其赋值给 `this.elementData`。

  9.

回到 `add(E e, Object[] elementData, int s)` 方法，元素被放置到 `elementData[0]`，`size` 更新为 1。

10.

**后续 `add()` 方法调用（第二次扩容时）：**

  11.

当 `size` 达到 `elementData.length`（例如，已经添加了 10 个元素，`size` 就变成10了，`elementData.length` 也为10），再次调用 `add()` 时，内层 **`add(e, elementData, size)`** 方法中，`s == elementData.length` 再次成立，会触发第二次扩容。

  12.

调用 `elementData = grow(s + 1)`，此时 `s + 1` 为 11。

  13.

进入 `grow(int minCapacity)` 方法， `oldCapacity` 是 10，此时 `oldCapacity > 0` 条件成立，进入 `if` 分支。

  14.

if分支中会调用newLength方法，如下所示：


```
`int newCapacity = ArraysSupport.newLength(oldCapacity,
            minCapacity - oldCapacity, /* minimum growth */
            oldCapacity >> 1           /* preferred growth */);
`
```
 1
2
3

其中 `oldCapacity >> 1` 表示原来的数组长度对2取整，相当于传入了三个形参——①原来的数组长度；②新数组的自小增长量；③优先希望的增长量。newLength方法的源码如下：


```
`public static int newLength(int oldLength, int minGrowth, int prefGrowth) {
    // preconditions not checked because of inlining
    // assert oldLength >= 0
    // assert minGrowth > 0

    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // might overflow
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {
        return prefLength;
    } else {
        // put code cold in a separate method
        return hugeLength(oldLength, minGrowth);
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13

  15.

从 ArraysSupport.newLength 返回的 newCapacity 将是 15。

  16.

回到 grow 方法，elementData 会被 Arrays.copyOf(elementData, 15) 复制到一个新的、容量为 15 的数组。

  17.

此后，每次扩容都会按照类似的逻辑进行，即 ArraysSupport.newLength 会根据 1.5 倍的“首选增长量” (oldCapacity >> 1) 和“最小增长量” (minCapacity - oldCapacity) 的最大值来计算新的容量，直到达到 MAX_ARRAY_SIZE 的限制。

###  带参构造下扩容机制的底层实现

1.

**初始状态：**

  2.

如果 `initialCapacity > 0`：`elementData` 数组会直接被初始化为指定大小的数组。


```
`// ArrayList.java (JDK 17)
private static final Object[] EMPTY_ELEMENTDATA = {}; // 注意与 DEFAULT_CAPACITY_EMPTY_ELEMENTDATA 的区别
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA; // 初始容量为0时，使用另一个共享空数组
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11

  3.

如果 `initialCapacity == 0`：`elementData` 数组会被初始化为 `EMPTY_ELEMENTDATA`，这是另一个**共享的、静态的空数组**，与无参构造器的 `DEFAULT_CAPACITY_EMPTY_ELEMENTDATA` 不同。

4.

**`initialCapacity > 0` 时的 `add()` 行为：**

  5. 假设 `new ArrayList(3)`，此时 `elementData.length` 为 3。那么在添加前 3 个元素时，`s == elementData.length` 不成立，不会触发扩容。
  6. 添加第 4 个元素时 (`s` 为 3)， **`add(e, elementData, size)`** 中，`s == elementData.length` (即 `3 == 3`) 成立，会进行扩容。此时内层grow方法的形参 `minCapacity` 为 4。
  7. 内层grow方法中，`oldCapacity` 是 3。此时`oldCapacity > 0` 成立，进入 `if` 分支。ArraysSupport.newLength方法传入的三个形参就分别是——①oldCapacity = 3， ②minCapacity - oldCapacity = 1, ③oldCapacity >> 1 = 1;
  8. 新数组的长度就等于旧数组的长度加上minGrowth和prefGrowth中的最大值，也就是3 + 1 = 4.
  9. 最终，`elementData` 会被 `Arrays.copyOf(elementData, 4)` 复制到一个新的、容量为 **4** 的数组。
10.

**“线性增长”的效果：**

  11. 如果带参构造传入的 `initialCapacity` 小于4，那么在最初的几次扩容中，宏观上会呈现出“逐步加一”的效果。这就使得当oldCapacity等于0、1的情况下，prefGrowth都等于0；而oldCapacity等于2、3的情况下，prefGrowth都等于1。
  12. 直到size>=4之后，才会在宏观上更明显地看到1.5倍的扩容。

---

##  知识图解

1. **ArrayList初始状态和首次扩容**的示意图如下：
  知识拓展——Java 8 和 Java 17 `ArrayList` 源码上的区别？以及为什么要这么改进？

###  Java 8 `ArrayList` 扩容流程及源码特点


在 Java 8 中，`ArrayList` 的 `add()` 方法触发扩容的调用链相对直接：

  1.

**`public boolean add(E e)` 方法：**

    2.

直接调用 `ensureCapacityInternal(size + 1)` 来确保容量。


```
`// (Java 8)
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
`
```
 1
2
3
4
5
6

  3.

**`ensureCapacityInternal(int minCapacity)` 方法：**

    4.

这个方法不再直接包含容量初始化的逻辑，而是调用了一个新的私有静态方法 calculateCapacity 来计算实际所需的容量。

    5.

然后将计算出的容量传递给 ensureExplicitCapacity(calculatedCapacity)。


```
`//  (Java 8)
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
`
```
 1
2
3
4

  6.

**`private static int calculateCapacity(Object[] elementData, int minCapacity)` 方法** ：

    7. 这个方法专门用于计算在当前 elementData 状态下，所需的最小容量。
    8. 它处理了**无参构造器**创建的 ArrayList 在**第一次添加元素**时，将容量从 0 初始化为 DEFAULT_CAPACITY (10) 的逻辑。

```
`// (Java 8u40+)
// 这是一个辅助方法，用于计算实际所需的容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
`
```
 1
2
3
4
5
6
7
8

  9.

**`ensureExplicitCapacity(int minCapacity)` 方法** ：

    10.

检查 `modCount` 以支持快速失败（fail-fast）机制。

    11.

如果 `minCapacity` 大于当前数组长度 (`elementData.length`)，则调用 `grow(minCapacity)` 进行实际的扩容。


```
`// ArrayList.java (Java 8)
private void ensureExplicitCapacity(int minCapacity) {
    modCount++; // 记录修改次数，用于迭代器快速失败
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
`
```
 1
2
3
4
5
6
7

  12.

**`grow(int minCapacity)` 方法：**

    13.

这是实际执行扩容逻辑的方法。

    14.

**直接在方法内部计算新容量：** `int newCapacity = oldCapacity + (oldCapacity >> 1);` (即 1.5 倍)。

    15.

**直接在方法内部处理各种边界条件：** 包括 `newCapacity < minCapacity`（取 `minCapacity`）、`newCapacity > MAX_ARRAY_SIZE`（调用 `hugeCapacity`）。

    16.

**直接修改 `this.elementData` 字段：** `elementData = Arrays.copyOf(elementData, newCapacity);`


```
`// ArrayList.java (Java 8)
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); // 直接计算 1.5 倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity); // 直接赋值给 elementData
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12

###  Java 17 `ArrayList` 扩容流程及源码特点


在 Java 17 中，`ArrayList` 的 `add()` 方法的调用链和 `grow` 方法的实现方式发生了变化：

  1.

**`public boolean add(E e)` 方法：**

    2.

不再直接调用 `ensureCapacityInternal`，而是调用一个新的私有辅助方法 `add(E e, Object[] elementData, int s)`。新的辅助方法将 `elementData` 和 `size` 作为参数传入，这有助于 JIT 编译器进行优化。


```
`private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}

public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12

  3.

**`private Object[] grow(int minCapacity)` 方法：**

    4.

**容量计算相关代码位置的变化** 不再自己计算 1.5 倍，而是将容量计算逻辑委托给 `jdk.internal.util.ArraysSupport.newLength` 方法。

    5.

`ArraysSupport.newLength` 接收 `oldLength`、`minGrowth` (即 `minCapacity - oldCapacity`) 和 `prefGrowth` (即 `oldCapacity >> 1`) 作为参数，内部会处理 1.5 倍增长的逻辑。

    6.

`grow` 方法在计算出新容量并执行 `Arrays.copyOf` 后，会**将新数组赋值给 `this.elementData` 字段，同时也将这个新数组作为方法的返回值返回**。


```
`// ArrayList.java (JDK 17)
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULT_CAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                                                  minCapacity - oldCapacity, /* minimum growth */
                                                  oldCapacity >> 1 /* preferred growth */);
        return elementData = Arrays.copyOf(elementData, newCapacity); // 赋值并返回
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12

###  为什么要这么改进？

  1. **让代码变得更加模块化，提高代码复用性：**
    2. **`ArraysSupport.newLength` 的引入**是最大的变化。在 **Java 9** 之后，Oracle 将许多集合类（如 `ArrayList`, `Vector`, `HashMap` 等）中通用的数组/哈希表容量计算和增长逻辑抽象并封装到了 `jdk.internal.util.ArraysSupport` 类中。
    3. 这样做避免了在不同集合类中重复编写相似的扩容逻辑，提高了代码的**复用性**和**可维护性**。当需要修改扩容策略或处理边界条件时，只需修改 `ArraysSupport` 中的一处代码即可影响所有依赖它的集合类。
  4. **JIT 编译器优化（性能提升）：**
    5. **参数传递而非字段访问：** 在 `private void add(E e, Object[] elementData, int s)` 方法中，将 `elementData` 和 `size` 作为参数传入，而不是直接访问 `this.elementData` 和 `this.size` 字段。这为 JIT 编译器提供了更好的优化机会，例如**逃逸分析**。
    6. **`grow` 方法的返回值：** 尽管 `grow` 方法仍然修改了 `this.elementData`，但它同时返回新数组的设计，使得调用者能够明确接收并使用这个新数组。
  7. **代码清晰度与可读性：**
    8. 虽然引入了更多的内部方法，但从公共 API (`public boolean add(E e)`) 的角度看，代码整体的调用逻辑变得更加清晰明了。
    9. 将容量计算的复杂性隐藏在 `ArraysSupport` 之后，使得 `ArrayList` 自身的 `grow` 方法逻辑更清晰，专注于数组复制。

---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## HashMap扩容机制｜高频面试题｜Java高频面试题｜resize、加载因子、红黑树、线程安全

#  HashMap扩容机制

##  简要回答

- Java1.7及之前生成新数组后，会遍历老数组的每个链表上元素，获取每个元素的key并基于新数组长度，计算元素在新数组中的下标，将元素放入新数组中，元素转移完后将新数组赋值给HashMap对象的table属性。
- Java1.8及之后生成新数组后，则会遍历老数组中的每个链表/红黑树。计算每个元素在新数组中的下标，将元素放入新数组中，元素转移完后将新数组赋值给HashMap对象的table属性。
##  详细回答

- HashMap底层相关属性：
  1. 加载因子：默认0.75
  2. 阈值：容量*加载因子，**当元素数量超过阈值时触发扩容**
  3. 最大容量：2^30，超过最大容量，阈值设置为Integer.MAX_VALUE
- 扩容机制：当元素数量超过阈值时，触发扩容，新容量是旧容量的2倍，但是不能超过最大容量，会调用resize()方法。
- Java1.7之前扩容机制
  - 底层结构是**数组+单链表**
  - 调用 **resize()** 方法时，如果原容量没有没有达到最大，会建立新数组，再调用**transfer()** 方法将原数组的元素移动到新数组中，否则停止扩容。

```
`//resize()方法
void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        //如果原有容量已经达到了上限，停止扩容。
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
        // 创建新数组
        Entry[] newTable = new Entry[newCapacity];
        // 调用transfer方法，将数据迁移到新数组中
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16

- transfer()方法**遍历数组的每个Entity，重新计算其hash值**，找到新数组中的对应位置，以**头插法**的方式将元素放入新数组中。
- 头插法可能会导致**新旧链表元素转置**现象

```
`//transfer()方法
void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16

- Java1.8之后扩容机制
  - 底层结构是**数组+链表 / 红黑树**
    - 当**链表长度** 大于等于8时，判断**HashMap的size** 是否大于等于64，如果小于64，则进行扩容，否则将**链表转换为红黑树**。
    - 当**红黑树节点数量**小于等于6时，会将**红黑树转换为链表**。
  - 遍历原数组的每个桶，转移元素到新数组中，分三种情况处理：
    1. 桶中没有元素：跳过
    2. 桶中只有一个元素：直接放入。
    3. 桶中是链表或者红黑树：
      4. 链表：求新桶位置并放入
      5. 红黑树：调用split()方法将红黑树拆成两个链表，然后求新桶位置并放入
  - 在扩容时，不需要计算元素的hash值，用**原先位置key的hash值**与**旧数组的长度**进行“**与**”操作，如果结果是0，则新位置就是原位置，否则新位置就是原位置+旧数组长度。
  - 使用**尾插法**将元素插入新数组中。

```
`final Node<K,V>[] resize() {
        //变量初始化，获取旧数组长度、旧阈值
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //当原数组已经初始化
        if (oldCap > 0) {
            // 原容量已经达到最大值
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 原容量翻倍后没有达到最大值，正常扩容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // 阈值翻倍左移一位
        }
        // 原数组未初始化，但是已经设置阈值
        else if (oldThr > 0)
            newCap = oldThr;
        // 原数组未初始化，没有设置阈值
        else {
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 补全新阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 更新全局阈值
        threshold = newThr;
        // 创建数组并迁移元素
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 原数组非空
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 桶中只有一个元素
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 桶里是红黑树
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    // 桶里是链表
                    else {
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86

##  知识图解


 知识扩展

1. 面试官可能追问：
- Q1：HashMap线程安全吗？怎么解决线程安全问题？
  - **不是线程安全的**，在多线程环境下，可能会出现数据不一致的情况。
  - 解决方法：使用ConcurrentHashMap或者Collections.synchronizedMap()方法。
- Q2：HashMap的扩容条件是什么》
  - Java7中HashMap的扩容条件需要满足**当前数据存储的数量大小大于等于阈值，并且数据发生了hash冲突**。
  - Java8中HashMap的扩容条件需要满足**当前数据存储的数量大小大于等于阈值**。
- Q3：HashMap为什么使用的是红黑树而不是平衡二叉树？
  - **平衡二叉树**追求**完全平衡**状态，任何节点的左右子树的高度差不能超过1。但是，在HashMap中，节点的插入和删除操作会频繁，**导致节点的左右子树的高度差会频繁变化**，因此，使用平衡二叉树会比较麻烦。
  - **红黑树**追求的是“**弱平衡**”状态，整个树最长路径不会超过最短路径的两倍，所以在插入/删除操作时，**不会频繁调整树结构**。
- Q4：为什么HashMap数组长度是2的n次幂？
  - 为了**提高HashMap的性能**，HashMap的数组长度是2的n次幂。
  - 这是因为，当数组长度是2的n次幂时，**hash值与数组长度-1进行“与”操作，等价于hash值对数组长度取模**。
  - 而取模运算的效率要低于“与”操作，因此，使用2的n次幂作为数组长度可以提高HashMap的性能。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## ConcurrentHashMap线程安全｜高频面试题｜Java高频面试题｜分段锁、CAS、synchronized、锁升级

#  ConcurrentHashMap线程安全

##  简要回答

- ConcurrentHashMap在JDK1.7中使用**数组+链表**的结构，数组分为大数组**Segment**和小数组**HashEntry**。ConcurrentHashMap的线程安全主要通过**Segment加ReentrantLock**来实现的。
- 在JDK1.8中，ConcurrentHashMap使用**数组+链表+红黑树**的结构,通过**CAS操作或者synchronized**来保证线程安全，缩小了锁的粒度，从而提高并发性能。
##  详细回答

1. JDK1.7
  2. ConcurrentHashMap**分段锁技术**：将数据分成一段一段存储，每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问，实现并发访问。
  3. 对每个**Segment独立加锁（继承ReentrantLock）**，小数组HashEntry用于存储键值对数据。ConcurrentHashMap包含Segment数组，Segment包含HashEntry数组，HashEntry是链表结构的元素,**并发度为Segment的数量**。
  4. 读写操作：**读操作**无锁，volatile保证可见性，根据HashEntry的volatile字段保证读操作能获取最新值。**写操作**对目标加锁，完成后释放，同一Segment写操作互斥。
5. JDK1.8
  6. 通过**CAS/局部synchronized**实现线程安全。
  7. ConcurrentHashMap对**头结点加锁**保证线程安全，锁粒度变小，发生冲突和加锁频率降低，并发操作性能提高。
  8. 添加元素时：判断容器是否为空，如果为空利用CAS设置该节点；如果不为空则使用synchronized，遍历桶中的数据，替换或新增节点到桶中，最后再判断是否需要转为红黑树，这样可以保证并发访问时的线程安全。
  9. 读写操作：**读操作**无锁，通过volatile保证可见性，支持并发读。**写操作**采用上述方法实现线程安全。
##  适用场景
 版本 锁机制 优势 JDK1.7 分段锁Segment 实现简单，适合中等并发 JDK1.8 CAS+局部synchronized 锁粒度更高，高并发性能好
##  代码示例


```
`import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapDemo {
    public static void main(String[] args) throws InterruptedException {
        // 创建ConcurrentHashMap实例
        ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();

        // 添加元素
        concurrentMap.put("apple", 10);
        concurrentMap.put("banana", 20);

        // 并发场景：启动两个线程同时操作map
        Thread thread1 = new Thread(() -> {
            // 原子操作：如果key不存在则添加
            concurrentMap.putIfAbsent("orange", 15);
            // 原子操作：更新值（将banana的数量加5）
            concurrentMap.computeIfPresent("banana", (k, v) -> v + 5);
        });

        Thread thread2 = new Thread(() -> {
            // 原子操作：获取并移除元素
            Integer value = concurrentMap.remove("apple");
            System.out.println("Thread2 移除了 apple: " + value);
        });

        // 启动线程
        thread1.start();
        thread2.start();
        // 等待线程执行完毕
        thread1.join();
        thread2.join();

        // 输出最终结果
        System.out.println("最终映射: " + concurrentMap);
    }
}
// 结果：
// Thread1 移除了 apple: 10
// 最终映射: {orange=15, banana=25}
// 若使用HashMap,可能会出现异常或者丢失数据
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40

##  知识图解

1. JDK1.7分段锁机制示意  知识扩展

  1. 面试官可能追问：
  2. Q1：JDK1.7的分段锁为什么早在JDK1.8被弃用了？
    3. 分段锁的并发读度会受到**Segment数量限制**（默认为16）,高并发下仍有竞争。
    4. 每个Segment是独立哈希表，包含数组、链表等，**内存开销大**。
    5. **JDK1.8使用更细粒度的锁**，并发度理论上与数组长度相等，性能更好。
  6. Q2:JDK1.8中synchronized锁的是什么？为什么不直接使用ReentrantLock？
    7. 锁的是哈希桶数组中某个索引位置的**头节点**，仅影响当前冲突链，锁粒度极小。
    8. synchronized在JDK1.6新增了**锁升级机制**后，性能与ReentrantLock基本相同，且更轻量级，更适合这种局部短时间锁定场景。
  9. Q3:CAS操作具体会用在哪里？失败了会怎么样？
    10. 用于初始化哈希桶，数组索引为空时插入第一个节点，修改节点值，标记扩容状态等无锁场景。
    11. CAS失败时会自旋重试（有限次数），仍失败则退化为加锁，避免无限循环浪费CPU。
  12. Q4:扩容时怎么保证线程安全？会不会丢失数据？
    13. 扩容时通过volatile标记状态，多线程协同扩容时，每个线程负责迁移一部分哈希桶，通过**CAS标记已迁移的索引**，避免重复操作。
    14. **写入操作会先协助完成扩容**，再执行插入，确保数据不会丢失。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 双亲委派机制｜大厂面试题｜Java高频面试题｜类加载器、JVM、打破双亲委派

#  双亲委派机制

##  简要回答

- **双亲委派机制**是Java类加载器的一种类加载策略，该机制的核心思想是如果一个类加载器收到了类加载请求，默认先将该请求委托给父类加载器进行加载，如果父类加载器无法完成该加载任务，才会尝试自行加载。
##  详细回答

- JVM中有三类类加载器，分别是启动类加载器Bootstrap ClassLoader,扩展类加载器Extension ClassLoader和系统类加载器System ClassLoader。**启动类加载器**加载的是%JAVA_HOME%/jre/lib目录下的Java核心类库；**扩展类加载器**加载的是%JAVA_HOME%/jre/lib/ext目录下的Java扩展类库；**系统类加载器**加载的是应用程序类路径（classpath）下的类库。除此之外，用户也可以**自定义类加载器**，可以加载用户指定目录下的Class文件。
- 类加载器之间是父子关系，启动类加载器是最顶层的类加载器，扩展类加载器是启动类加载器的子加载器，系统类加载器又是扩展类加载器的子加载器。
- 当类加载器收到类加载的请求时，会先**检查该类是否已经被加载过**，如果已经加载则直接返回该类的引用；否则会**委托给父类加载器**加载该类，直到顶层的启动类加载器；父类加载器如果无法加载，当前类加载器才会**尝试自行加载**，也就是调用自己的findClass()方法。如果当前的类加载器也无法加载这个类，那么它会抛出一个ClassNotFoundException异常。
- 优点：
  1. 所有的加载请求都会传递到启动类加载器，可以避免不同类重复加载相同类的情况，可以**保证类的唯一性**。
  2. 核心类被顶层加载器加载一次所有的子加载器就能共享这个类，可以实现**类的复用**。
  3. 启动类加载器加载Java的核心类库，可以防止不可信的类假冒核心类，能够**增强程序安全性**。
  4. 实现了不同层次的类加载器服务于不同的类加载需求，例如启动类加载器加载核心类库，扩展类加载器加载扩展类库，应用程序类加载器加载用户代码，各个层级类加载器的职责清晰。
- 缺点：
  1. 类加载过程需要不断的委托给父类加载器，可能会导致实际应用中类加载的**灵活性降低**。
  2. 在类数量多或者层次比较深的情况下，类加载所需的时间可能会增加。
##  知识图解

1. 双亲委派机制示意图  知识扩展

  1. 扩展：
  2. 打破双亲委派机制
    3. 在自定义加载器时，需要继承ClassLoader类，并重写findClass方法，无法被父类加载的类会通过这个方法加载。但是如果希望打破双亲委派机制，则需要**重写loadClass()方法**，实现自己指定项目需求。
    4. 例如Tomcat服务器就自定义了类加载器WebAppClassLoader用以打破双亲委派机制，每个Web应用的WebAppClassLoader会优先加载Web应用目录下的类实现类，不会委托父类，避免不同Web应用的同名类冲突，实现类隔离。
  5. 启动类加载器
    6. 由C/C++语言实现的，属于JVM内核组件，没有对应的Java类对象，因此如果在代码中获取由启动类加载器加载的类时会返回null，无法返回启动类加载器实例。
  1. 面试官可能追问：
  7. Q1：如果两个不同的类加载器加载了同一个全限定名的类，这两个类在JVM中是一个类吗？
    8. 不是。**JVM判断两个类是否相同，需要全限定名和加载它们的类加载器都相同**，如果类加载器不同，JVM会视为两个独立的类，此时使用instanceof判断或强制转换会抛出ClassCastException异常。
  9. Q2：如果父类加载器加载的类依赖了由子类加载器加载的类，会发生什么？
    10. 会导致**类加载失败**，因为父类加载器无法委托子类加载器加载类，所以父类加载时会找不到类而抛出ClassNotFoundException异常。
    11. 可以使用**线程上下文类加载器**来解决，可以在父类加载器加载的类中获取子类的加载器并进行加载，从而绕过双亲委派从下而上的委托机制。
    12. 例如JDBC加载数据库驱动，DriverManager由启动类加载器加载，需要通过上述方法加载应用类路径下的驱动类。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## JDK8新特性｜高频面试题｜Java高频面试题｜Lambda表达式、Stream API、函数式接口、默认方法

#  JDK8新特性

##  简要回答

- Java8引入的核心新特性包括Lambda表达式，Stream API、函数式接口、默认方法、Optional类和新的日期与时间API。
- **Lambda表达式**把函数作为一个方法的参数，提供简洁的语法编写匿名函数。
- 引入的**函数式接口**注解@ FunctionalInterface，作为Lambda表达式的接口，比如Consumer、Supplier、Function和Predicate。
- **Stream API**是数据流处理工具，提供了一种声明式的方式来对集合进行操作，如过滤、映射、排序和去重等。
- Java8也引入了**java.time包**，提供了全新的日期和时间API，线程安全且易用，加强了对日期与时间的处理。
##  详细回答

1.

**Lambda表达式**：是Java8的核心特性，本质是匿名函数，即允许**函数作为方法参数传递**，简化代码编写。

  2. 语法结构是 **(参数列表) -> {方法体}** ，其中参数类型可以省略，由编译器类型判断，单参数可省略括号，单行逻辑可以省略大括号与return。
  3. lambda表达式替代匿名内部类，简化集合遍历、线程创建等场景的代码。依赖函数式接口实现，编译后会生成私有静态方法，通过invokedynamic指令调用，避免匿名内部类产生过多的class文件。
4.

**函数式接口**： 函数式接口是只有一个抽象方法的接口，JDK8开始，接口可以定义默认方法、静态方法，使用@FunctionalInterface注解声明为函数式接口。其中的核心内置接口有Consumer< T >、Function<T,R>、Predicate< T >、Supplier< T >等。

  5. **Consumer< T >** 是消费型接口，接收参数但无返回值(void accept(T t))，比如遍历集合场景。
  6. **Suppplier< T >** 是供给型接口，无参数有返回值(T get())，生成随机数场景。
  7. **Function<T,R>** 是函数型接口，接收参数并返回值(R apply(T t))，比如映射场景。
  8. **Predicate< T >** 是断言型接口，接收参数返回boolean值(boolean test(T t))，比如集合过滤场景。
9.

默认方法与静态方法：**JDK8开始，接口可以定义默认方法和静态方法**，默认方法使用default关键字，静态方法使用static关键字。

  10. 如果实现类同时实现了多个接口，接口中有同名的默认方法时，需要显示重写，通过接口名.super.方法名()指定调用哪个接口的默认方法。
11.

**Stream API**：JDK8开始提供，Stream是对集合数据的声明式处理流，不存储数据也不修改源数据，代码简洁，支持串行/并行处理。

  12. 使用Stream可以通过集合.stream()/集合.parallelStream()或者Stream.of()**创建Stream对象**；
  13. Stream支持中间操作与终止操作。**中间操作是链式调用返回新的Stream对象**，可以进行filter()过滤、map()映射、sorted()排序、distinct()去重等操作；**终止操作则是返回结果**，如forEach()遍历、collect()收集、count()计数和reduce()归约等。
14.

**Optional类**：可以**解决空指针异常问题**，封装可能为null的对象，提供安全的空值处理方式。

  15. 通过**Optional.of(T t)** 创建Optional对象，of()方法会检查参数是否为null，不为null则返回Optional对象，为null则抛出NullPointerException异常。
  16. 通过**Optional.ofNullable(T t)** 创建Optional对象，ofNullable()方法会检查参数是否为null，不为null则返回Optional对象，为null则返回Optional.empty()对象。
  17. 通过**Optional.orElse(T other)** 实现值为null时返回默认值。
  18. 通过**Optional.ifOfPresent(Consumer< ? super T > consumer)** 可以实现在值不为null时执行消费代码逻辑。
19.

新的日期与时间API

  20. Java8引入了新的日期与时间API，包括**LocalDate、LocalTime、LocalDateTime、Instant、DateTimeFormatter**等。替代了传统的Date、SimpleDateFormat，解决了之前线程不安全、设计混乱的问题。
  21. **LocalDate**：表示日期（年-月-日），如LocalDate.now()；
  22. **LocalTime**：表示时间（时:分:秒），如LocalTime.of(12, 30, 0)；
  23. **LocalDateTime**：表示日期和时间（年-月-日 时:分:秒），如LocalDateTime.now()；
  24. **Instant**：表示时间戳（从1970-01-01 00:00:00 UTC开始），如Instant.now()；
  25. **DateTimeFormatter**：用于格式化日期和时间，如DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")；
##  知识图解


 代码示例

1. Lambda表达式与函数式接口使用

```
`import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class LambdaDemo {
    public static void main(String[] args) {
        // 1. Consumer（消费型）：遍历输出
        Consumer<String> printConsumer = s -> System.out.println("消费：" + s);
        printConsumer.accept("Java8 Lambda");

        // 2. Supplier（供给型）：生成随机数
        Supplier<Integer> randomSupplier = () -> (int) (Math.random() * 100);
        System.out.println("生成随机数：" + randomSupplier.get());

        // 3. Function（函数型）：字符串转长度
        Function<String, Integer> lengthFunction = s -> s.length();
        System.out.println("字符串长度：" + lengthFunction.apply("Hello Java8"));

        // 4. Predicate（断言型）：判断数字是否大于10
        Predicate<Integer> gt10Predicate = num -> num > 10;
        System.out.println("15是否大于10：" + gt10Predicate.test(15));
    }
}

/*
运行结果
消费：Java8 Lambda
生成随机数：67
字符串长度：10
15是否大于10：true
*/
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32

1. StreamAPI使用示例

```
`import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class StreamDemo {
    public static void main(String[] args) {
        List<Integer> numList = Arrays.asList(1, 2, 3, 4, 5, 6, 6, 7);

        // 1. 过滤+去重+排序
        List<Integer> filterList = numList.stream()
                .filter(num -> num % 2 == 0) // 过滤偶数
                .distinct() // 去重
                .sorted((a, b) -> b - a) // 降序排序
                .collect(Collectors.toList());
        System.out.println("过滤去重排序：" + filterList); // [6,4,2]

        // 2. 映射+求和
        int sum = numList.stream()
                .map(num -> num * 2) // 每个数乘2
                .reduce(0, Integer::sum); // 求和
        System.out.println("映射求和：" + sum); // (2+4+6+8+10+12+12+14)=68

        // 3. 分组
        Map<Boolean, List<Integer>> groupMap = numList.stream()
                .collect(Collectors.groupingBy(num -> num > 3)); // 按是否大于3分组
        System.out.println("分组结果：" + groupMap); // {false=[1,2,3], true=[4,5,6,6,7]}
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29

1. 新的日期与时间API

```
`import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class DateTimeDemo {
    public static void main(String[] args) {
        // 1. 获取当前时间
        LocalDateTime now = LocalDateTime.now();
        System.out.println("当前时间：" + now); // 2025-12-21T15:30:20.123

        // 2. 格式化
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formatStr = now.format(formatter);
        System.out.println("格式化后：" + formatStr); // 2025-12-21 15:30:20
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16

##  知识扩展

1. 面试官可能追问
- Q1：**Lambda表达式**和**匿名内部类**有什么区别？
  - 匿名内部类会生成独立的class文件，而Lambda表达式会生成私有静态方法和invokedynamic指令，节省资源。
  - 匿名内部类需要显式声明final，Lambda表达式可以直接访问外部final变量。
- Q2：**Stream并行流**有什么特点？
  - Java8的Stream并行流**基于Fork/Join框架**实现**多核并行处理**，默认使用公共ForkJoinPool，会将数据源拆分给多个线程并行计算再合并结果，核心特点是能提升CPU密集型、大数据量场景的处理效率。
  - Stream默认无序以保证性能，也支持通过ordered()强制有序，但会牺牲部分效率；同时有**延迟执行**特性，仅在终止操作时触发计算。
  - 并行流**存在线程安全风险**，不能直接操作非线程安全的外部集合或变量，优先用collect/reduce等归约操作；
  - 适合**CPU密集型**任务，IO密集型任务使用反而会因线程阻塞降低效率，还可以通过自定义ForkJoinPool避免公共线程池的资源竞争问题。
- Q3：函数式接口为什么只能有一个抽象方法？
  - 函数式接口要求抽象方法只能有一个，用于和Lambda表达式绑定，因为**Lambda表达式本质是匿名函数，只能实现一个方法逻辑**，如果有多个抽象方法，编译器无法确定要调用哪个方法，会**编译报错**。
  - 但是函数式接口可以包含多个默认方法和静态方法，不会影响接口的函数式特性。
- Q4：延迟执行和并行流冲突吗？
  - 延迟执行是指Stream的中间操作仅记录操作逻辑而不实际执行，只有触发终止操作后才会一次性执行所有中间操作。
  - 并行流是将数据源拆分为多个片段，由多线程并行处理后合并结果。
  - **延迟执行决定了执行时机，而并行流是执行方式**，无论是串行还是并行，都是在终止操作触发时选择单线程执行或者Fork/Join框架多线程执行，中间操作的延迟特性对两种流类型都生效。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 线程池｜高频面试题｜Java高频面试题｜线程复用、资源管理、拒绝策略

#  线程池

##  简要回答

- 采用多线程编程时，如果**线程过多会造成系统资源的大量占用**，降低系统效率。如果有些线程存活时间短但是不得不创建很多时也会**造成资源浪费**，线程池可以决定由哪个空闲且存活的线程来执行，当**线程池中线程不够时会适当创建一部分线程，线程冗余时会销毁一部分线程**。可以**提高线程利用率**，降低系统资源损耗。
##  详细回答

- **资源管理**: 在多线程应用中线程池提供了**对线程的统一管理**（如任务队列、线程生命周期监控、拒绝策略等），方便开发者跟踪任务执行状态、调整资源分配。
- **提高性能**:线程的创建和销毁开销相对较大。线程池可以**复用线程**，**避免频繁地创建和销毁线程**从而提高系统性能。
- **任务排队**:线程池可以用于**排队任务**，确保任务按照一定的顺序执行，避免线程竞争和冲突
- **避免线程过多**:如果不使用线程池，程序员可能会手动创建大量线程来处理任务，每个线程都需要占用内存和CPU资源，如果不加限制地创建线程，会导致系统资源耗尽，可能引发系统崩溃。线程池通过**限制线程数量**来避免这种情况。
##  使用场景

- 线程池适用于需要**频繁创建和销毁线程场景**，如：处理大量数据，执行大量计算任务，处理大量IO任务。
- 线程池适用于需要**控制并发规模的场景**，如：Web服务器、数据库连接池、消息队列等。
- 线程池适用于需要**提升任务调度与管理场景**，线程池的getActiveCount()监控可以实时活跃线程数。
##  代码示例

1. 固定线程池使用示例

```
`import java.util.concurrent.Executorservice;
import java.util.concurrent.Executors;

public class ResourceManagementExample{
    public static void main(string[] args){
    //创建一个固定大小的线程池，包含3个线程
    ExecutorService executor=Executors.newFixedThreadPool(3);

    // 提交任务给线程池
    for(int i=0;i<5;i++){
        final int taskId = i;
        executor.submit(()->{
            System.out.printIn("Task "+ taskId +" is running on thread "+ Thread.currentThread().getName());});
    }
// 关闭线程池
executor.shutdown();
    }
}
// 运行结果:
Task 1 is running on thread pool-1-thread-2
Task 0 is running on thread pool-1-thread-1
Task 2 is running on thread pool-1-thread-3
Task 3 is running on thread pool-1-thread-2
Task 4 is running on thread pool-1-thread-1
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24

1. 可缓存线程池使用示例

```
`import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ImprovedPerformanceExample{
    public static void main(string[] args){
        ExecutorService executor =Executors.newCachedThreadPool();
        for(int i=0;i<5;i++){
            final int taskedId=i;
            executor.submit(()->{
                System.out.println("Task "+ taskId +" is running on thread "+ Thread.currentThread().getName());
        });
        }
        executor.shutdown();
    }
}
//运行结果:
Task 4 is running on thread pool-1-thread-5
Task 1 is running on thread pool-1-thread-2
Task 2 is running on thread pool-1-thread-3
Task 3 is running on thread pool-1-thread-4
Task 0 is running on thread pool-1-thread-1
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21

1. 单线程线程池使用场景

```
`import java.util.concurrent.Executorservice;import java.util.concurrent.Executors;
public class TaskQueueExample {
    public static void main(string[]args){
        //创建一个单线程线程池，任务会按顺序执行
        ExecutorService executor =Executors.newSingleThreadExecutor();
        //提交多个任务给线程池
        for(inti=0;i<5;i++){
            final int taskId = i;
            executor.submit(()->{
                System.out.println("Task "+ taskId + " is running on thread "+ Thread.currentThread().getName());
            });
        }
        // 关闭线程池
        executor.shutdown();
    }
}
// 运行结果:
Task 0 is running on thread pool-1-thread-1
Task 1 is running on thread pool-1-thread-1
Task 2 is running on thread pool-1-thread-1
Task 3 is running on thread pool-1-thread-1
Task 4 is running on thread pool-1-thread-1

`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23

##  知识图解

1. 线程池工作原理  知识扩展

  1. 扩展：
  2. 线程池种类：
    1. **ScheduledThreadPoolExecutor**：定时执行任务，支持定时执行和周期执行。
    2. **FixedThreadPool**：固定大小线程池，线程数量固定,核心线程数和最大线程数相同，不会创建更多线程处理任务。
    3. **CachedThreadPool**：可缓存线程池，在于线程数是几乎可以无限增加（最大Integer.MAX_VALUE，2^31-1），线程闲置时会进行回收。
    4. **SingleThreadExecutor**：使用唯一的线程去执行任务，原理和FixedThreadPool一样，如果线程在执行任务时发生异常，会重新创建新线程执行后续任务，适合所有任务需要按顺序执行场景。
    5. **SingleThreadScheduledExecutor**：和ScheduledThreadPool线程池相似，属于其特例，内部只有一个线程。
  1. 面试官可能追问:
  3. Q1：线程池越多越好吗？一个线程池有大量线程会怎样？
    1. 资源竞争加剧，性能下降。每个线程池也会占用一定内存空间，过多的**线程池同样会导致内存占用**。
    2. 过多的线程池会**增加代码维护成本**，难以统一各线程池状态。
    3. 如果多个线程池之间存在资源依赖，可能会导致**死锁**，同时过多线程池可能**耗尽系统的线程资源**。
    4. 大量线程竞争CPU时间片，会**增加上下文切换的频率**，降低系统整体效率。
    5. 操作系统调度线程池中的所有线程，线程数量越多调度器负担越重，可能会导致**关键任务延迟执行**。
  4. Q2：线程池工作队列满了有哪些拒接策略？
    5. 如果线程池的任务队列满了，线程池会执行指定的拒绝策略来应对，常用的四种拒绝策略包括：
      1. **CallerRunsPolicy**：使用线程池调用者所在的线程去执行被拒绝的任务，除非线程池被停止或者线程池的任务队列已有空缺。
      2. **AbortPolicy**：直接抛出一个任务被线程池拒绝的异常。
      3. **DiscardPolicy**：不做任何处理，然后执行该任务。
      4. **DiscardOldestPolicy**：抛弃最老的任务，然后执行该任务。
      5. 自定义拒绝策略，通过实现**RejectedExecutionHandler**接口可以自定义任务的拒绝处理逻辑。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Spring设计模式｜大厂面试题｜Java高频面试题｜工厂模式、代理模式、单例模式、观察者模式

#  Spring设计模式

##  简要回答

- Spring中使用的设计模式有**工厂设计模式、代理设计模式、单例模式、模版方法模式、包装器设计模式、观察者模式和适配器模式**等。
- 工厂模式：Spring通过BeanFactory和ApplicationContext等容器来创建和管理Bean对象，解耦对象的创建和使用。
- 单例模式：Spring中的Bean默认是**单例**的，Bean对象在Spring容器中唯一。
- 模版方法模式：Spring中如jdbcTemplate等以template结尾的类使用了**模版方法模式**，与Callback模式配合操作数据库。
- 观察者模式：Spring中的事件驱动模型使用了**观察者模式**，在一个事件触发时，会触发监听器，执行对应的方法。
- 代理模式：Spring AOP是基于**代理模式**的，SpringAOP的增强或通知需要使用适配器模式，Spring MVC中也使用**适配器模式**适配不同类型的Controller。
##  详细回答

1.

**工厂模式**

  2. 工厂模式能够定义专门的工厂类来封装对象的创建逻辑，分离对象的创建与使用，从而降低耦合度。
  3. Spring可以通过**BeanFactory**创建Bean对象，能够实现最基本的依赖注入支持，采用懒加载机制，即在获取Beans时候才创建对象，占用内存较少，程序启动速度快。
  4. Spring可以使用**ApplicationContext**创建Bean对象，扩展了BeanFactory，支持更多的上下文特性，采用预加载机制，会在容器启动时一次性创建所有的Bean对象。ApplicationContext有三个实现类，分别是ClassPathXmlApplication、FileSystemXmlApplication和XmlWebApplicationContext。
    5. ClassPathXmlApplicationContext从类路径下加载XML配置文件初始化容器
    6. FileSystemXmlApplicationContext从文件系统路径加载XML文件初始化容器
    7. XmlWebApplicationContext可以从Web应用中的XML文件加载上下文
8.

**代理模式**

  9. 代理模式为其他对象提供代理，控制对这个对象的访问，可以在不修改目标对象代码的前提下动态增强功能。
  10. Spring对代理模式的使用体现在**SpringAOP**中，SpringAOP将与业务无关，但多个业务模块所共同调用的逻辑封装，减少系统的重复代码。SpringAOP基于动态代理，有JDK和CGLIB两种代理方式，CGLIB代理基于继承，JDK代理基于接口。
  11. 客户端调用Subject接口的方法，实际调用的是代理类，代理类完成额外操作后调用目标类方法。
12.

**单例模式**

  13. 单例模式保证一个类只有一个实例，并提供一个全局访问点，可以避免重复创建消耗资源。
  14. Spring使用单例模式管理Bean对象，使用三级缓存机制保证Bean对象在Spring容器中的唯一性，Spring中的Bean默认是单例的。
15.

**模版方法模式**

  16. 模版方法定义了操作中的算法骨架，将算法的某些步骤延迟到子类中，模版方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。
  17. Spring中jdbcTemplate、hibernateTemplate等以Template结尾的类都使用了模版方法模式，Spring将固定的算法封装，将可变逻辑通过Callback接口暴露给用户，简化开发。
18.

**装饰者模式**

  19. 项目需要连接多个数据库，或者不同客户在每次访问中根据需要访问不同的数据库时，装饰者模式可以动态切换数据源。
20.

**观察者模式**

  21. 观察者模式可以定义对象之间一对多的依赖关系，实现当一个对象状态变化时，所有依赖它的对象会得到通知并自动更新，实现解耦通信。
  22. **Spring的事件驱动模型**是观察者模式的经典应用，包含三个核心组件：**事件**(ApplicationEvent)、**监听器**(ApplicationListener)、**发布者**(ApplicationEventPublisher)。定义事件时，实现ApplicationEvent接口，定义监听器时，实现ApplicationListener接口，事件发布者可以通过ApplicationEventPublisher的publishEvent()方法发布事件。
23.

**适配器模式**

  24. 适配器模式将一个类的接口转换成用户希望的另一个接口，让不同的类协同工作。
  25. Spring AOP的增强或通知需要使用适配器模式，AOP中每个类型的通知都有对应的拦截器，AOP执行时统一调用MethodInterceptor的invoke()方法，适配器会将具体的通知适配到MethodInterceptor中。
  26. Spring MVC中也使用适配器模式适配不同类型的Controller。SpringMVC中的HandlerAdapter**将不同类型处理器的调用接口转换成DispatcherServlet能识别的统一接口**，调用处理器的核心业务逻辑后返回ModelAndView结果。
##  知识图解

1. Spring中代理模式示意图  知识扩展

  1. 面试官可能追问
  2. Q1：Spring中的单例模式和传统单例模式有什么区别？
    3. 传统的单例模式是通过私有构造器和静态方法保证的，创建逻辑固定，不支持依赖注入。
    4. Spring中的单例模式是通过BeanFactory或ApplicationContext创建Bean对象，多个容器可存在多个实例，由容器管理创建、依赖注入、销毁，支持懒加载和循环依赖解决。
  5. Q2：Spring的动态代理选择逻辑是什么？
    6. Spring在目标类实现接口时**优先使用JDK代理**，目标类未实现接口时使用CGLIB代理，也可以通过spring.aop.proxy-target-class属性指定使用CGLIB代理。
    7. 基于JDK的动态代理，代理类需要实现InvocationHandler接口，通过invoke()方法反射来调用目标方法，将横切逻辑和业务编织在一起。Proxy类利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。
    8. 基于CGLIB的动态代理，在代理类没有实现接口时使用，是由代码生成的类库，在运行时动态生成指定类的一个子类对象，覆盖其中特定方法并添加增强代码实现AOP。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 装饰器模式｜高频面试题｜设计模式高频面试题｜结构型设计模式、动态扩展、开闭原则

#  装饰器模式

##  简要回答

1. 装饰器模式是**结构型设计模式**，不修改原始类接口的基础上，通过包装的方式给对象动态地添加功能或责任。通过创建一个装饰类包裹原始类的实例，保持原始类接口不变的情况下提供额外的功能，比生成子类更灵活。
2. 优点：解决了继承导致的类膨胀问题，符合开闭原则和单一职责原则。
3. 缺点：多层装饰器嵌套使用时调试难度增加，会出现少量代码冗余。
##  详细回答

1. 使用装饰器模式时，先由客户端创建具体组件，然后使用装饰器包装具体组件，生成增强对象。客户端调用最终装饰后的对象方法，会将所有装饰器的增强逻辑与基础逻辑叠加执行，最终返回组合结果。
2. 装饰器模式的主要角色有：组件接口、具体组件、装饰器和具体装饰器：
  1. **组件接口**定义了具体组件和装饰器共同的接口，确保它们可以互相替换，是所有组件的基础规范。
  2. **具体组件**实现了组件接口，是被装饰的具体对象，只包含核心基础功能。
  3. **装饰器**实现组件接口，同时持有组件对象的引用（指向具体组件或其他装饰器），为具体装饰器提供统一的父类，可以定义通用的装饰逻辑。
  4. **具体装饰器**继承自抽象装饰器，实现了具体的装饰逻辑，向组件添加职责，会调用父类的方法以保持接口一致。
##  知识图解

1. 装饰者模式结构图  示例代码

  1. 以咖啡订单为例，展示装饰器模式的实现：

```
`// 1. Component类：定义咖啡的核心接口
interface Coffee {
    // 获取咖啡描述
    String getDescription();
    // 获取价格
    double getPrice();
}

// 2. ConcreteComponent类：基础咖啡
class PlainCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "原味咖啡";
    }

    @Override
    public double getPrice() {
        return 10.0; // 基础价格
    }
}

// 3. Decorator类：咖啡装饰器（持有咖啡引用，实现咖啡接口）
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee; // 持有咖啡引用

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// 4. ConcreteDecorationA类：加牛奶（单一扩展功能）
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        // 叠加装饰描述
        return coffee.getDescription() + " + 牛奶";
    }

    @Override
    public double getPrice() {
        // 叠加装饰价格
        return coffee.getPrice() + 2.0;
    }
}

// 4. ConcreteDecorationB类：加糖（单一扩展功能）
class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return coffee.getDescription() + " + 糖";
    }

    @Override
    public double getPrice() {
        return coffee.getPrice() + 1.0;
    }
}

// 5. 客户端调用：自由组合装饰器
public class Client {
    public static void main(String[] args) {
        // 基础咖啡：原味
        Coffee coffee = new PlainCoffee();
        System.out.println(coffee.getDescription() + " → 价格：" + coffee.getPrice());

        // 装饰1：加牛奶
        coffee = new MilkDecorator(coffee);
        System.out.println(coffee.getDescription() + " → 价格：" + coffee.getPrice());

        // 装饰2：再加糖（嵌套装饰）
        coffee = new SugarDecorator(coffee);
        System.out.println(coffee.getDescription() + " → 价格：" + coffee.getPrice());
    }
}
/*
原味咖啡 → 价格：10.0
原味咖啡 + 牛奶 → 价格：12.0
原味咖啡 + 牛奶 + 糖 → 价格：13.0
*/
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87

##  使用场景

  1.

装饰器模式可以**避免继承爆炸**。


当一个对象有多种可选的扩展功能，且功能可组合时，使用继承会产生2^n个子类（n为扩展功能的数量），装饰器模式使用组合替代继承，n个装饰器类就可以实现所有组合。同理，装饰器模式也可以将复杂的功能拆分成多个独立的小功能。

  2.

需要在运行时为对象动态地添加功能时，且可随时移除该功能时，使用装饰器模式。

  3.

在很多框架中也使用了装饰器模式，JavaIO的InputStream和OutputStream使用了装饰器模式，FileInputStream是基础组件，BufferedInputStream是缓冲装饰器，DataInputStream是数据处理装饰器；Spring框架中的BeanWrapper、DecoratingProxy等类是通过装饰器模式动态扩展Bean功能的。

##  知识扩展

###  面试官可能追问：

  1. 装饰器模式和代理模式的区别是什么？
    2. 两者核心区别是设计的目的不同。装饰器模式主要作用是功能叠加，用于为对象新增核心业务功能，由客户端主动选择装饰器组合；而代理模式则是控制对象的访问，比如权限校验和日志记录等，客户端不需要知道代理的存在，目标是决定对象能不能做。
  3. JavaIO流为什么使用装饰器模式实现？
    4. JavaIO流有多种基础流和多种扩展功能，使用装饰器模式可以将基础流和扩展功能进行组合，如果使用继承实现会产生大量子类无法维护，而装饰器模式可以在运行时动态组合不同的流功能，灵活且易于扩展。
  5. 你在项目中使用过装饰器模式吗？
    6. 我在项目中的接口请求模块中使用了装饰器模式，基础组件是原生的HTTP请求，然后通过装饰器模式定义了超时重试的装饰器、数据加密装饰器和请求日志装饰器；客户可以根据实际场景进行组合使用，比如普通场景使用基础请求加超时重试和日志，敏感数据场景添加数据加密装饰器，无需修改原有请求代码，符合开闭原则且比继承更灵活。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## Spring IOC｜大厂面试题｜Java高频面试题｜控制反转、IOC容器、依赖注入、Bean生命周期

#  Spring IOC

##  简要回答

- SpringIOC是**控制反转的设计思想**，原先由程序员控制整个程序的执行，使用IOC思想后将控制权交给了IOC容器，由Spring **IOC容器来控制Bean的生命周期**，**对象的创建、初始化和销毁**。在程序中不手动new对象，而是通过@ Component或@ Autowired注解来配置，容器会自动实例化Bean，注入依赖，降低代码耦合，简化开发。
##  详细回答

###  IOC定义

- IOC是Inversion of Control的缩写，即控制反转，是一种设计思想。传统的开发中需要开发者手动new对象，维护依赖关系；在IOC模式下，它将设计好的对象交给IOC容器控制，由容器统一管理Bean的生命周期（对象的创建、初始化和销毁过程）及其依赖关系。
###  IOC作用

- IOC容器管理对象之间的相互依赖关系和对象的注入，可以简化应用的开发，让资源容易管理。同时降低对象之间的耦合度，在创建对象时只需写配置文件或注解，无需编写创建过程的代码。
- 例如：在项目中的一个Service类中，可能会有多个类作为其底层，这些类之间可能会有依赖关系，IOC容器可以自动管理这些依赖关系，将依赖的对象注入到Service类中，无需清楚所有底层类的构造函数。
###  IOC容器的实现与Bean注册/注入方式

- IOC容器会利用**反射机制**动态加载类，创建对象实例和调用对象方法；通过**依赖注入**让容器管理应用程序组件之间的依赖关系；通常使用**BeanFactory或ApplicationContext**来创建IOC容器，并管理Bean实例。
- 在Spring中，传统使用**配置注册**，即使用XML进行配置，将Bean注册到IOC容器中，并指定Bean的属性。现在，SpringBoot**注解注册**已经成为主流，开发者可以使用@Component、@Service、@Controller、@Repository等注解来实现自动扫描配置Bean，@Configuration和@Bean可以手动注册第三方组件，无需编写XML文件。
- IOC**依赖注入**的方式有注解注入、构造器注入和Setter方法注入三中。使用@Autowired可以按类型注入、@Qualifier可以按名称注入、@Resource可以按名称或类型注入；在构造器注入中，Bean构造器参数标注@Autowired，在Spring4.3后，已经变成默认的注入方式；Setter方法注入中，Bean的Setter方法标注@Autowired。
###  IOC容器启动流程

1. 先**创建SpringIOC容器**，也就是ApplicationContext或BeanFactory。
2. 容器通过ResourceLoader**加载**配置文件，ResourceResolver**解析**资源并生成Resource对象。
3. 将生成的Resource对象，通过BeanDefinitionReader进行解析，将Bean定义信息**生成BeanDefinition对象**。
4. 将BeanDefinition对象通过BeanRegistry**注册到容器**中,此时容器的初始化完成。
###  IOC容器管理Bean

1. 容器会加载配置（注解/XML）文件，扫描需要注册的Bean；随后通过反射**创建Bean实例**，并完成**依赖注入**；
2. 调用@PostConstruct注解的方法或者自定义方法进行**初始化**；
3. 将**Bean存入容器**，开发者可以通过getBean()或注解注入使用；在容器关闭时调用@PreDestroy注解的方法或者自定义方法进行销毁；
##  知识图解

1. IOC容器功能示意图  知识扩展

  1. 面试官可能追问？
  2. Q1：BeanFactory和ApplicationContext的区别是什么？
    3. **BeanFactory**是Spring最原始的IOC容器，**ApplicationContext**是BeanFactory的增强版，提供了更多的功能，如资源加载、国际化、事件发布等。
  4. Q2：如果两个Beann之间存在**循环依赖**，SpringIOC怎么处理？
    5. Spring通过**三级缓存**可以解决构造器注入之外的循环依赖问题。三级缓存分别是singletonObjects成品Bean缓存、earlySingletonObjects临时Bean缓存（半成品缓存）和singletonFactories工厂缓存，用于生成半成品Bean。通过在Bean实例化后，依赖注入前将其早期引用存入缓存，让依赖它的Bean能够提前获取引用，打破循环阻塞。
    6. 如果是构造器注入的循环依赖，Spring会抛出BeanCurrentlyInCreationException异常，因为构造器注入需要实例化依赖Bean，导致循环阻塞。将构造器注入改成字段注入或Setter注入，或者使用@Lazy注解延迟其中一个Bean的实例化可以解决。
  7. Q3：ApplicationContext加载配置时，Bean是立即实例化还是延迟实例化？可以控制吗？
    8. 默认情况下，**ApplicationContext会在容器启动时实例化所有单例Bean**，如果想实现延迟实例化，需要在类或@Bean方法上使用 **@Lazy注解** ，此时singleton Bean会延迟到第一次被使用时实例化。
    9. @Lazy对单例Bean有效，prototype Bean本身就是懒加载。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 堆与栈｜大厂面试题｜Java高频面试题｜内存管理、JVM、垃圾回收

#  堆与栈

##  简要回答

- 堆和栈都是内存管理单元，两者在**用途**、**生命周期**、**存取速度**、**存储空间**和**可见性**方面存在不同。
- 堆内存是由JVM管理的，栈内存由操作系统管理。
- 堆的生命周期是不确定的，栈则会在函数调用结束后被回收。
- 堆在分配速度上比栈慢，分配栈内存只需要移动一个指针。
- 堆是一块一块的内存，内存大小在运行时确定，栈则采用数据结构中的栈实现，具有先后出的顺序特点，内存大小则是在编译时确定的。
- 堆是线程共享的，栈是线程私有的。
##  详细回答

- 用途：
  1. 栈用于存储局部变量、方法调用的参数，方法返回地址以及一些**临时数据**。每个线程在创建时会创建一个栈，这个栈随着方法执行增长和缩小。每当一个方法执行时会创建一个栈帧，用于存储该方法的信息，当方法执行完毕，栈帧就会从栈中弹出。
  2. 堆用于**存储对象实例**（类的实例和数组）。使用new关键字创建的对象实例会在堆上分配空间。
- 生命周期：
  1. **栈**中的数据具有**确定的生命周期**，当方法调用结束时其对应的栈帧会被销毁，栈中的局部变量也会消失。当线程结束执行时，栈内存也会被自动回收。
  2. **堆**中的**对象生命周期不确定**，对象会在垃圾回收机制检测到对象不再被引用时进行回收。
- 存取速度：
  1. **栈的存取速度快**，因为栈遵循先进后出原则，操作简单快速。
  2. **堆存取速度较慢**，对象在堆上的分配和回收需要更多的时间，而且垃圾回收机制的运行也会影响性能。
- 存储空间：
  1. **栈**的**空间相对较小且固定**，由操作系统管理。栈溢出时可能是递归过深或者局部变量过大。
  2. **堆**的**空间可以动态分配**，具有可扩展性，可能会导致内存碎片化问题，分配和回收由JVM管理。堆溢出通常是创建了太多的大对象，或者未能及时回收不再使用的对象。
- 可见性：
  1. **栈**中数据对**线程私有**，每个线程有自己的栈空间，天生线程安全。
  2. **堆**的数据对**线程共享**，所有线程都可以访问堆上的对象。
##  知识图解

1. **栈内存和堆内存存储示意**  知识扩展

  1. 扩展：
  2. 堆的内部细分：JVM会把堆分成**新生代**和**老年代**，可以**提高GC的回收效率**，对不同区域采用不同回收算法。
    3. **新生代**：存储刚创建的对象，特点是“**对象生命周期短、创建和销毁频繁**”。进一步分为Eden区（新对象优先分配这里）和两个Survivor区（From/To，用于存放GC后存活的对象）。采用**“复制算法”**：将Eden和From区存活的对象复制到To区，然后清空Eden和From区，效率极高。
    4. **老年代**：存储在新生代多次GC后仍存活的对象（如长期使用的缓存对象、单例对象），特点是“**生命周期长**”。采用**“标记-清除”或“标记-整理”算法**：不需要频繁回收，重点是减少内存碎片。
  1. 面试官可能追问：
  5.

Q1：为什么局部变量存在栈中，而对象要放在堆里？

    6. 因为对局部变量和对象生命周期的要求符合栈和堆的生命周期。
    7. **局部变量的生命周期与方法的执行周期严格一致**，方法结束后就可以立即释放，栈的“自动管理”特性正好匹配这种短生命周期、大小固定的需求，效率极高。
    8. **对象的生命周期往往更长且不确定**（可能被多个引用持有，跨方法、跨线程使用），其大小也常是动态的（如集合的容量可动态变化）。堆的“动态分配+GC回收”模式，能灵活应对这种长生命周期、动态大小的存储需求，同时实现线程间的数据共享。
  9.

Q2：栈内存溢出（StackOverflowError）和堆内存溢出（OutOfMemoryError）的原因有什么不同？

    10. **堆内存溢出**：通常是因为**创建的对象过多且无法被GC回收**，比如循环创建大量对象并长期持有引用（如将对象不断添加到一个静态集合中且不清理）。堆的容量也有上限，当新对象无法分配到内存时，就会触发 OutOfMemoryError 。
    11. **栈内存溢出**：通常是因为方法调用栈过深，比如无限递归

```
`public void test(){
  test();
}
// 每个方法调用都会创建一个栈帧压入栈
栈的容量有限
递归过深会导致栈帧堆满
触发 StackOverflowError 。
`
```
 1
2
3
4
5
6
7
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Java中有哪些集合使用红黑树？红黑树有什么特点？

#  Java中有哪些集合使用红黑树？红黑树有什么特点？

##  简要回答

1. **直接以红黑树作为核心结构的集合**主要有：`TreeMap` 和 `TreeSet`。其中TreeSet 本质上是基于TreeMap实现的。
2. **在特定条件下会用到红黑树的集合**有：`HashMap`、`LinkedHashMap`、`HashSet`、`LinkedHashSet`，以及 `ConcurrentHashMap`（JDK 1.8+）。它们并不是整体就是红黑树，而是在**哈希冲突严重**时，会把桶中的链表树化成红黑树。
3. **红黑树**是一种**自平衡二叉查找树**。不追求绝对平衡，但能保证树的高度始终维持在对数级别，因此查找、插入、删除的时间复杂度都能稳定在 **O(logN)** 。
##  详细回答

###  哪些集合使用红黑树

1. **`TreeMap`** ：
  2. TreeMap的底层就是**红黑树**。
  3. 它会按照key的**自然顺序**或**自定义比较器**进行排序。
  4. 因为底层是红黑树，所以 `put()`、`get()`、`remove()` 的时间复杂度都是 **O(logN)** 。
  5. 它适合**需要排序、范围查询、获取最小值/最大值**的场景，比如 `firstKey()`、`lastKey()`、`subMap()`。
6. **`TreeSet`** ：
  7. TreeSet的底层本质上是TreeMap，因此它同样依赖**红黑树**。
  8. 它的元素默认按照**自然顺序**排序，也可以在创建时传入Comparator自定义排序规则。
  9. TreeSet适合**去重 + 排序**同时存在的场景。
10. **`HashMap`（JDK 1.8+）** ：
  11. HashMap的底层是**数组 + 链表 / 红黑树**。
  12. 正常情况下，桶中元素是链表；当链表长度达到**树化阈值 `TREEIFY_THRESHOLD`（默认 8）**，并且数组容量达到**最小树化容量 `MIN_TREEIFY_CAPACITY`（默认 64）** 时，链表会被转换为**红黑树**。
  13. 当树中节点数量减少到 **6** 时，又会退化回链表。
14. **`LinkedHashMap`（JDK 1.8+）** ：
  15. LinkedHashMap继承自HashMap，底层同样可能出现**链表树化**。
  16. 它比HashMap多维护了一条**双向链表**，用于保持**插入顺序**或**访问顺序**。它既可能在桶中使用红黑树，又能在整体上保持有序迭代。
17. **`HashSet` 和 `LinkedHashSet`** ：
  18. HashSet的底层是HashMap，所以在JDK 1.8+中，发生严重哈希冲突时，桶中也可能树化为红黑树。
  19. LinkedHashSet的底层可以看作LinkedHashMap，同样也可能在桶内使用红黑树。
  20. 需要注意，它们使用红黑树只是**底层实现细节**，并不代表集合整体具有排序能力。
21. **`ConcurrentHashMap`（JDK 1.8+）** ：
  22. ConcurrentHashMap在JDK 1.8+中的底层也是**数组 + 链表/红黑树**。
  23. 当某个桶中的链表过长时，也会树化，以降低极端冲突下的查询成本。
  24. 它和HashMap的区别在于，ConcurrentHashMap还要同时解决**并发安全**问题。
###  红黑树的特点

1. **二叉查找树**：
  2. 左子树节点值小于当前节点，右子树节点值大于当前节点。因此红黑树天然支持**有序遍历**、**范围查找**、**最值查找**。
3. **自平衡树**：
  4. 红黑树会通过**旋转 + 变色**来维持平衡。
  5. 它并不要求每一层都完全平衡，但能保证不会退化成一条长链表。
6. **“弱平衡”结构**：
  7. 红黑树要求从任意节点到所有叶子节点的每条路径上，**黑色节点数量相同**。同时，**红色节点不能连续出现**。
  8. 这使得红黑树的**最长路径不会超过最短路径的 2 倍**，所以整体高度依然是 **O(logN)** 。
9. **查找、插入、删除性能稳定**：
  10. 查找时间复杂度是 **O(logN)** 。
  11. 插入时间复杂度是 **O(logN)** 。
  12. 删除时间复杂度也是 **O(logN)** 。
  13. 这也是 `TreeMap`、`TreeSet` 和 `HashMap` 树化后能保持稳定性能的原因。
14. **插入和删除时调整成本比 AVL 树更低**：
  15. 红黑树不像 AVL 树那样追求“绝对平衡”，所以在频繁插入、删除时，旋转次数通常更少。这也是 Java 集合框架更偏向选择**红黑树**而不是 **AVL 树**的重要原因。
16. **查询极致性能不如更严格平衡的树**：
  17. 因为红黑树是“弱平衡”，所以它的查询效率理论上会略逊于 AVL 树。但综合插入、删除、维护成本后，它在工程上通常是更划算的选择。
##  知识图解

1. 红黑树结构  知识拓展

###  面试官可能的追问

  1. **为什么TreeMap和 TreeSet直接使用红黑树，而HashMap只是冲突严重时才树化？**
    2. 因为 `TreeMap` 和 `TreeSet` 的目标本来就是**保持有序**，所以适合使用红黑树。
    3. `HashMap` 的核心目标则是**通过哈希快速定位桶**，理想情况下平均复杂度是 **O(1)** 。只有在冲突严重、链表过长时，才需要用红黑树兜底，把最坏情况从 **O(N)** 优化到 **O(logN)** 。
  4. **为什么 Java 集合里更常用红黑树，而不是 AVL 树？**
    5. AVL 树平衡更严格，查询会更快一点，但插入和删除时需要更频繁地旋转，维护成本更高。
    6. 红黑树虽然不是绝对平衡，但已经足够保证 **O(logN)** 的性能，而且插入、删除调整更少，更适合像 `TreeMap`、`TreeSet` 这种需要频繁修改的数据结构。
  7. **`HashMap` 树化后是不是就变成有序集合了？**
    8. 不是。`HashMap` 中的红黑树只存在于**单个桶内部**，目的是优化冲突桶的查找效率。
    9. 整个 `HashMap` 依然是**无序的**，它不会像 `TreeMap` 或 `TreeSet` 那样提供全局排序能力。  Last Updated: 5/25/2026, 3:50:35 PM

 ←   →

### 评论
 

验证登录状态...

## Java引用类型｜高频面试题｜Java高频面试题｜强引用、软引用、弱引用、虚引用

#  Java引用类型

##  简要回答

- 强引用、软引用、弱引用和虚引用四种引用的主要区别在**垃圾回收过程中的行为不同**，分别是：
  - **强引用**：不会被垃圾回收器回收，新创建的对象会被强引用指向。
  - **软引用**：在系统内存不足时会被垃圾回收器回收，描述有用但是非必需的对象。
  - **弱引用**：在下一次进行垃圾回收时会被垃圾回收器回收，用于实现对象缓存。
  - **虚引用/幻影引用**：在对象被回收之前会被放入引用队列中，用于跟踪对象被垃圾回收的时机。
##  详细回答

- 在垃圾回收过程中可以根据**对象被哪些引用类型指向**来**判断对象是否可以被回收**，对对象的生命周期进行更精细地控制与监控。
1. **强引用(Strong Reference)** ：
  2. 如果一个对象有强引用指向它，那么垃圾回收器不会回收该对象。
  3. 进行引用赋值时（Object obj = new Object();），对象会被强引用指向。
4. **软引用(Soft Reference)** ：
  5. 如果一个对象只有软引用指向它，**系统内存不足时**垃圾回收器会尝试回收只有软引用指向的对象。
  6. 描述有用但是非必需的对象，用于实现内存敏感的缓存，可以在内存不足时释放缓存中的对象。
7. **弱引用(Weak Reference)** ：
  8. 如果一个对象只有弱引用指向它，则在下一次进行垃圾回收时，该对象会被回收。
  9. 用于实现对象缓存，不希望缓存的对象影响垃圾回收情况。
  10. 弱引用和软引用的区别是软引用只会在内存不足时进行回收，**弱引用则不管内存是否充足都会进行回收**。
11. **虚引用/幻影引用(Phantom Reference)** ：
  12. 如果一个对象只有虚引用指向它，那么它什么时候都可能被回收。对象在被回收之前会被放入引用队列中，程序员进行处理。
  13. **必须和引用队列一起使用**，用于跟踪对象被垃圾回收的时机，进行必要的清理或记录。
##  代码示例


```
`// 强引用
Object obj = new Object();

// 软引用
SoftReference<Object> softRef = new SoftReference<>(new Object());

// 弱引用
WeakReference<Object> weakRef = new WeakReference<>(new Object());

// 虚引用
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantomRef = new PhantomReference<>(new Object(), queue);
`
```
 1
2
3
4
5
6
7
8
9
10
11
12

##  知识图解

1. **四种引用比较示意图**  知识扩展

  1. 扩展：
  2. 判断对象是否可以被回收有引用计数法和可达性分析算法两种方法。其中**可达性分析算法比较常用**，能够避免出现循环引用导致无法回收的问题。
    3. **引用计数法**：
      4. 给对象中添加**引用计数器**，有地方引用该对象计数器加一，引用失效时计数器减一。如果**计数器为0则说明对象不再可用**。
      5. 如果两个对象互相引用，会导致计数器无法为0，无法进行垃圾回收。
    6. **可达性分析**算法：
      7. 设置**GC Roots对象**，如果一个对象到GC Roots没有任何引用链相连，则该对象不可用，需要被回收。可以避免使用引用计数器产生的循环引用对象不可回收的问题。
      8. **GC Roots对象是垃圾回收器可直接访问的起点，且对象本身不能被回收。** 可以作为GC Roots的对象包括**虚拟机栈中引用的对象，方法区中类静态属性引用的对象，方法区中常量引用的对象和被同步锁(synchronized)持有的对象以及JNI中引用的对象**等。
  9. **内存泄漏**：
    10. 程序在运行过程中不再使用的对象仍然被引用，无法被垃圾回收，导致可用内存减少。
    11. 常见原因：
      1. **静态集合**：使用静态数据结构存储对象，没有进行清理。
      2. **事件监听**：未取消对事件源的监听，导致对象持续被引用。
      3. **线程**：未停止的线程可能持有对象引用，无法被回收。
  12. **内存溢出**(OOM)：
    13. JVM申请内存时无法找到足够的内存，引发OutOfMemoryError。
    14. 出现原因：
      1. **堆内存溢出**：代码中出现**大对象分配**或者是**内存泄漏**时，多次GC也没有足够的内存空间。
      2. **栈溢出**：代码中出现**递归调用**，压栈过深或者无法扩展栈空间时出现。
      3. **元空间溢出**：系统的**代码过多**或加载的类文件过多，导致元空间内存占用大。
  1. 面试官可能追问：
  15. Q1：强引用绝对不会被垃圾回收吗？
    16. 如果对象有强引用指向它，及时出现OOM也不会被回收。但是**如果强引用被显式置为null，对象可能变为可回收**。
  17. Q2：在缓存场景中应该用什么引用？为什么？
    18. 缓存场景中应该用**弱引用或软引用**，因为缓存的对象是有用但是非必需的，当内存不足时可以被回收。
    19. **软引用可以尽可能保留缓存对象**，减少重新加载的开销。还可以在内存不足时释放内存，自动避免OOM。
    20. **弱引用可以存临时关联数据**，如WeakHashMap中键为弱引用，值为强引用，键对象被回收后，对应的键值对会被自动移除。但是**GC的触发时机不确定**，缓存命中率低。
  21. Q3：为什么虚引用必须和引用队列一起使用？
    22. **虚引用不会影响对象的生命周期**，虚引用的作用就是在对象被回收之前将引用放入引用队列中，程序可以通过检查引用队列来确定对象是否被回收。
  23. Q4：如果不处理引用队列中的对象，会不会导致内存泄漏？
    24. 不会，引用队列本身不持有强引用，而内存泄漏的核心是不必要的强引用。
    25. 引用队列存储的是已经失去目标对象的引用对象（SoftReference，WeakReference，PhantomReference的实例），他们的目标对象被GC回收后，其会实例被加入引用队列。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## 方法重载与方法重写｜高频面试题｜Java高频面试题｜重载、重写、多态、方法签名

#  方法重载与方法重写

##  简要回答

###  方法重载 和 方法重写 的概念

- **方法重载（Overload）**：在同一个类中定义多个**同名**方法，**但参数列表（类型、数量或顺序）不同**。
- **方法重写（Override）**：在子类中重新定义父类的方法，要求**方法签名（方法名、参数列表、返回类型）完全相同**。
###  方法重载 和 方法重写 的区别

-

如下表所示 :
 **维度** **方法重载** **方法重写** **作用范围** 同一类中 子类与父类之间（继承关系） **参数列表** 必须不同（类型、数量、顺序至少一个不同） 必须相同 **返回类型** 可以不同 必须相同或兼容（子类可返回父类返回类型的子类） **访问修饰符** 无限制 子类方法权限不能比父类更严格（访问权限：子类≥父类） **异常处理** 可抛出不同异常 不能抛出比父类方法更宽泛的检查异常 **静态/非静态** 允许静态方法重载 静态方法不能被重写 **多态性** 编译时多态（静态绑定） 运行时多态（动态绑定）

---

##  详细回答

###  方法重载 和 方法重写 的概念

- **方法重载（Overload）** ：在同一个类中定义多个**同名**方法，**但参数列表（类型、数量或顺序）不同**。方法重载可以有效地增强代码灵活性，例如，一个工具类可能提供多种参数类型的 `add()` 方法。
- **方法重写（Override）** ：子类通过**完全一致的方法签名**重新定义父类方法，以实现多态性。例如，子类自定义的 `toString()` 方法覆盖 **Object** 类的默认实现。
###  方法重载 和 方法重写 的区别

1. **作用范围**：
  2. 重载仅发生在**同一类**中。
  3. 重写需要**继承关系**，由子类覆盖父类的方法。
4. **参数列表**：
  5. 重载必须修改参数列表（参数的类型、数量或顺序**至少需要一个不同点**）。
  6. 重写参数列表**必须完全一加粗致**。
7. **返回类型**：
  8. 重载**允许返回类型不同**。
  9. 重写**要求返回类型相同或兼容**（协变返回类型）。例如，父类返回 **Object** 类型，子类可返回 **String** 类型。
10. **访问修饰符**：
  11. 重载方法可以使用任意访问修饰符（**public**、**protected**、**private**）。
  12. 重写方法的访问权限不能比父类更严格（如父类为 **public**，子类不能为 **protected**）。
13. **异常处理**：
  14. 重载可以抛出**不同**的异常。
  15. 重写要求子类方法抛出的检查异常（Checked Exception）**不能比父类更宽泛**。例如，父类抛出 **IOException**，子类可抛出 **FileNotFoundException**，但不能抛出 **Exception**。
16. **静态方法**：
  17. 静态方法可以重载（例如 `public static void log(String username)` 和 `public static void log(int identitycode)`）。
  18. 静态方法不能被重写。若**子类定义同名静态方法**，属于“**方法隐藏**”（Method Hiding），而非重写。
19. **多态性**：
  20. 重载在**编译时**决定调用哪个方法（静态绑定）。
  21. 重写在**运行时**根据对象实际类型决定调用哪个方法（动态绑定）。
###  方法重载 和 方法重写 的代码演示

-

**JDK 源码中的方法重载示例**：

  -

如 **java.io.PrintStream** 类中的 `println()` 方法：


```
`public class PrintStream {
	public void println() { ... }          // 参数个数不同
	public void println(boolean x) { ... } // 参数类型不同
	public void println(int x) { ... }     // 参数类型不同
	public void println(char[] x) { ... }  // 参数类型不同
}
`
```
 1
2
3
4
5
6

-

**JDK 源码中的方法重写示例**

  -

如 **java.util.AbstractList** 类中的 `equals()` 方法在 **ArrayList** 类中的重写：


```
`public class ArrayList<E> extends AbstractList<E> {
	// 重写 AbstractList 的 equals 方法
	@Override
	public boolean equals(Object o) {
		if (o == this) return true;
		if (!(o instanceof List)) return false;
		// 自定义比较逻辑...
	}
}
`
```
 1
2
3
4
5
6
7
8
9


---

##  知识拓展

- **方法重载 的示意图**：
   →

### 评论
 

验证登录状态...

## BIO、NIO与AIO｜大厂面试题｜Java高频面试题｜IO模型、多路复用、Selector、AIO回调

#  BIO、NIO与AIO

##  简要回答

- BIO、NIO、AIO是Java中用于**处理I/O操作的几种不同的模型**。
- BIO是**同步阻塞**模型，适合连接数较少且固定的情况
- NIO是**同步非阻塞**模型，适合连接数较多且连接时间较短的情况
- AIO是**异步非阻塞**模型，适合连接数较多且连接时间较长的情况。
##  详细回答

1. BIO(Blocking I/O)
  2. BIO指的是同步阻塞模型。
  3. 当执行到IO操作的代码时，**线程会一直阻塞**（无论是在等待操作还是进行读写操作），直到IO操作完成后才会执行后续的代码。如果有大量的并发连接，会导致大量线程阻塞，造成资源浪费。
  4. 传统的java.io包，**基于流模型**实现的。
  5. 优点：代码比较简单直观，由于阻塞IO操作的结果可靠
  6. 缺点：阻塞等待会导致资源浪费，并且每个连接都需要一个独立线程并发能力有限，系统整体性能下降。
7. NIO(Non-Blocking I/O)
  8. NIO是同步非阻塞模型。
  9. NIO**引入了Selector、Channel和Buffer**，Channel是双向的，可以读也可以写，而Buffer是负责传输的缓冲区。使用NIO不再阻塞，而是**采用轮询的方式进行IO操作**，一个线程可以进行多个IO操作，可以利用while循环在里面不断的询问多个IO是否准备好了，某一个IO准备好了就进行IO操作，如果没有就询问下一个IO。
  10. Java1.4引入的java.nio包，提供了更接近操作系统底层高性能的数据操作方式。
  11. 优点：可以提供更高的并发性和可扩展性，使用更少的线程处理相同数量的连接，节省了系统资源。
  12. 缺点：可靠性较低，可能出现部分读写或者错误数据。
13. AIO(Asynchronous I/O)
  14. AIO是异步非阻塞模型。
  15. AIO是在NIO的基础上进一步发展的，使用AIO时，该线程会开启另一个线程，并将IO操作交给另一个线程执行，然后执行后续的代码，当另一个线程执行完IO操作后会通知该线程，然后**使用回调函数**的方式执行AIO。
  16. Java1.7后引入的包，异步IO是**基于事件和回调机制**实现的。
  17. 优点：不会阻塞线程，更加高效。
  18. 缺点：每个操作都要创建一个回调函数，可能会消耗更多的系统资源。
##  适用场景

1. BIO：适合**连接数较小且固定**的情况，但在高并发环境中，性能可能会受到限制。
2. NIO：适合**连接数较多且连接时间较短**的场景，能够通过单一线程管理多个连接，提高了系统的扩展性和并发性。
3. AIO：适用于**处理大量并发连接，且连接时间较长**的场景，能够在I/O操作完成后异步通知应用程序。
- 在实际开发中，NIO比BIO更常用，而AIO的应用相对较少。
##  知识图解

1. BIO流程示意  知识扩展

  1. 面试官可能追问：
  2. Q1：NIO的Selector是如何实现“多路复用” 的？底层依赖操作系统的什么机制？
    1. Linux依赖**epoll机制**，Windows依赖**IOCP机制**。
    2. Selector的工作流程：先将Channel**注册**到Selector中，操作系统内核会**监听**IO事件，当事件发生时，内核通知Selector，**唤醒**线程处理事件，这样可以**避免线程阻塞**在单个IO操作上，实现**多路复用**。
  3. Q2：AIO的 “异步” 与NIO的 “非阻塞” 常被混淆，二者的核心差异是什么？AIO 在哪些场景下优势更明显？
    1. NIO的非阻塞是**线程需要主动调用select()轮询**就绪事件，线程需要参与IO的就绪检查。
    2. AIO的异步是线程发起IO操作后立即返回，由操作系统完成整个IO操作，完成后通过回调通知线程，**线程无需参与中间过程**。
  4. Q3：Netty 作为高性能网络框架，为何基于 NIO 而非 AIO？日常开发中应如何根据场景选择 BIO、NIO、AIO？
    1. NIO的多路复用在**高并发场景下性能稳定**，而且Netty通过优化线程模型可以改善其效率。
    2. AIO在操作系统层面实现还不够完善。
  5. Q4：三种 IO 模型在处理 “读数据” 时的流程有何不同？分别体现了什么设计思想？
    1. BIO：线程**调用read()后阻塞**，直到数据从内核缓冲区拷贝到用户缓冲区。
    2. NIO：线程通过**Selector检查Channel是否非阻塞**，然后**调用read()方法**，数据从内核缓冲区拷贝到用户缓冲区（时间短于BIO）。
    3. AIO：线程**调用read()并注册回调**，返回继续处理其他任务。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## volatile与synchronized对比｜高频面试题｜Java高频面试题｜并发编程、线程安全、单例模式

#  volatile与synchronized对比

##  简要回答

- synchronized是确保**数据的一致性**和**线程安全**，解决多个线程之间访问资源的同步性。而volatile是确保变量在多个线程的**可见性**和**有序性**。
- volatile不需要获取/释放锁，性能较高且轻量级。
- synchronized可以修饰**方法以及代码块**，volatile只能修饰**变量**。
##  详细回答

1. 机制和用途：
  2. synchronized用于提供**线程间的同步机制**，当一个线程进入一个由synchronized修饰的代码块或方法时，它会获取一个**监视器锁**（monitor lock），这保证了同一时间只有一个线程可以执行这段关系代码。
  3. volatile用于**修饰变量**，当一个线程修改了一个volatile变量的值，其他线程能够**立即看到修改**，还可以**防止指令重排序**。
4. 原子性
  5. synchronized可以保证被修饰的代码块的**原子性**，这段代码在执行过程中不会被其他线程打断。
  6. volatile只能保证单个读写操作的原子性，对于复合操作（自增、减）**不能保证原子性**。
7. 互斥性
  8. synchronized提供了**互斥性**，同一时间只有一个线程可以执行被其修饰的代码块和方法。
  9. volatile**没有提供互斥性**，它只能保证可见性和有序性。
10. 性能
  11. synchronized的性能相对较低，因为它需要获取和释放锁。
  12. volatile的性能较高，因为它不需要获取和释放锁，更加轻量级，不过提供的同步级别较低。
13. 使用范围
  14. synchronized可以修饰方法以及代码块。
  15. volatile只能修饰变量。
##  使用场景
 需求 选volatile 选synchronized 操作类型 简单无复合 复合、多步操作 原子性 无 有 性能 追求极致性能（轻量） 允许性能损耗（强同步保证） 线程协作 仅状态传递 有协作（如等待/唤醒机制）
##  代码示例

1. volatile状态标记

```
`private volatile boolean isRunning = true;

// 线程A：修改状态（一次写）
public void stop() { isRunning = false; }

// 线程B/C：读取状态（多次读）
public void work() { while (isRunning) { ... } }
`
```
 1
2
3
4
5
6
7

1. volatile禁止指令重排序

```
`private static volatile Singleton instance;
public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton(); // volatile 禁止此处重排序
            }
        }
    }
    return instance;
}
`
```
 1
2
3
4
5
6
7
8
9
10
11

1. synchronized保证复合操作原子性

```
`private int count = 0;

// 多线程同时调用，需保证count++的原子性
public synchronized void increment() { count++; }
`
```
 1
2
3
4

1. synchronized多变量

```
`private int total;
private int used;

// 保证total和used的同步更新
public synchronized void allocate(int amount) {
    if (used + amount <= total) {
        used += amount;
        // 其他依赖used和total的操作...
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10

1. volatile和synchronized实现单例模式

```
`public class Singleton {
    // 1. volatile修饰单例引用，禁止指令重排序
    private static volatile Singleton instance;

    // 2. 私有构造器，防止外部实例化
    private Singleton() {}

    // 3. 双重检查锁定获取实例
    public static Singleton getInstance() {
        // 第一次检查：避免不必要的同步（提高性能）
        if (instance == null) {
            // 同步块：保证只有一个线程进入初始化流程
            synchronized (Singleton.class) {
                // 第二次检查：防止多线程并发时重复初始化
                if (instance == null) {
                    // 若不加volatile，可能发生指令重排序导致的问题
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
`
```
 1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23

##  知识图解

1. volatile关键字的作用示意图

 知识扩展

1. 面试官可能追问：
- Q1：volatile的可见性是通过内存屏障实现的，具体会插入哪些内存屏障？这些内存屏障如何阻止指令重排序？
  1. 对于**写操作**，在写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障。
  2. 对于**读操作**，在读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障。
- Q2：当volatile变量和synchronized代码块同时修饰同一变量时，会有冲突吗？执行优先级如何？
  1. 当volatile变量和synchronized代码块同时作用于同一变量时，**不会产生冲突**，两者的作用是互补而非对立的。它们的执行逻辑遵循 Java 内存模型（JMM）的规范，存在明确的协作关系而非 “优先级” 之分。
- Q3：synchronized的wait()/notify()机制为什么必须在同步块中使用？volatile能否实现类似的线程间通信？
  1. wait()/notify()是synchronized机制中用于线程间协作的核心方法，必须在同步块中使用。
  2. wait()会**释放锁并且阻塞当前线程**，notify()则会**唤醒一个等待该锁的线程**，二者都基于当前线程有锁的前提。如果不在同步块中调用可能会抛出异常。
  3. volatile可以实现简单的线程状态传递，无法替代synchronized的线程协作机制，**没有精确唤醒能力以及原子性保证**。
- Q4：双重检锁已经有了synchronized，为什么还要使用volatile？
  1. 第一行代码不在同步代码块之内，可能会出现对象地址不为空，内容为空情况。
  2. 代码在编译运行时可能会出现重排序，在多线程环境下可能会出现报错。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## HashMap线程安全｜高频面试题｜Java高频面试题｜ConcurrentHashMap、synchronizedMap、死循环

#  HashMap线程安全

##  简要回答

1. HashMap为什么是线程不安全的？
  2. HashMap的**操作不是原子的**，多个线程操作一个HashMap对象时，可能会导致数据不一致,并发修改异常。
3. 如何实现线程安全？
  4. 使用**Collections.synchronizedMap()** 方法，返回一个线程安全的Map对象。
  5. 使用**ConcurrentHashMap**类，它是线程安全的。
  6. 使用**锁机制**，在HashMap的操作中加显式锁。
##  详细回答

1. HashMap存在的问题
  2. JDK1.7 采用数组+链表的数据结构，在多线程背景下，**数组扩容时可能会导致Entry链死循环**。
  3. HashMap并发执行**put()操作时会出现数据覆盖问题**，因为put()方法没有加锁，多线程环境可能会出现数据覆盖问题。
  4. HashMap的迭代器是快速失败迭代器，**并发修改会破坏迭代器的遍历逻辑**，导致数据不一致。
5. 实现HashMap线程安全的方法
- 使用**Collections.synchronizedMap()方法**，通过该方法创建一个线程安全的HashMap对象，返回一个同步的Map包装器，使所有对Map的操作都同步执行。

```
`Map<String,String> synchronizedMap = Collections.synchronizedMap(new HashMap<>());
`
```
 1

- 使用**ConcurrentHashMap**类，它专门设计用于多线程环境的哈希表实现。它使用分段锁机制，允许多个线程同时进行读操作，提高并发性能。

```
`Map<String,String> concurrentHashMap = new ConcurrentHashMap<>();
`
```
 1

- 使用**锁机制**，在HashMap的操作中加显式锁（如ReentrantLock）来保证线程安全。

```
`Map<String,String> map = new HashMap<>();
ReentrantLock lock = new ReentrantLock();

// 在需要线程安全的操作中使用锁
lock.lock();
try {
    // 进行线程安全的操作
} finally {
    lock.unlock();
}
`
```
 1
2
3
4
5
6
7
8
9
10

##  知识图解

1. HashMap死循环形成图解  知识扩展

  1. 扩展
    2. 并发编程的3个核心特性：**原子性、可见性、有序性**。
    3. 迭代器设计模式：
      4. 快速失败：迭代器实时检查modCount，一旦发现并发修改就抛出异常，实现强一致性，牺牲并发性能。
      5. 弱一致性：允许并发修改，迭代器可能不是最新的，支持高并发。
  6. 面试官可能追问：
  2. Q1：你能从并发编程的角度解释一下HashMap为什么是线程不安全的吗？
    1. **原子性**：HashMap的put()方法不是原子操作，并发时会被中断，导致数据覆盖。
    2. **可见性**：HashMap的modCount数组元素等共享变量未使用volatile修饰，线程A修改后，线程B可能看到旧值，导致迭代器判断错误。
    3. **有序性**：HashMap红黑树的插入/删除有复杂的指针操作，并发时指令重排序可能会破坏树结构。
  3. Q2：如果需要“强一致性”的线程安全Map，应该使用什么？
    1. 选Collections.synchronizedMap()方法。
    2. 在ConcurrentHashMap的迭代前后手动加锁。
  4. Q3："读多写少"的场景下，适合使用哪种线程安全的容器存储键值对？
    5. 适合使用ConcurrentHashMap，其**读操作无锁，写操作加桶级锁**，“读多写少”的场景下，ConcurrentHashMap几乎无竞争，性能好。
  6. Q4：为什么ConcurrentHashMap不支持键为null？但是HashMap支持？
    7. 在单线程场景下，null的hash值为0可以正常存储，但ConcurrentHashMap是并发容器，若允许null，调用get()方法时返回null，**无法区分键不存在还是键存在但是值为null**，导致并发场景下的逻辑错误。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## final关键字｜高频面试题｜Java高频面试题｜final修饰类、final修饰方法、final修饰变量

#  final关键字

##  简要回答

- **final**作为Java中的一个关键字**可以用来修饰 类 ， 方法 ，和 变量**。（但**final不能修饰构造器**）
  1. **修饰类**：
 被final修饰的类**不能被继承，但该类可以去继承别的 (没有被final修饰的)类**，例如String类和System类，它们被final修饰，是不可以被继承的，但是它们有自己的父类——即顶层父类Object类。
 被final修饰的类虽然不能被继承，但 **可以被实例化**。
  2. **修饰方法**：
 被final修饰的方法**不能被子类重写，但可以被子类继承并使用（在满足访问权限规则的前提下）**。当修饰方法时， **final关键字不能与abstract关键字共存**；因为abstract修饰的方法是必须被非抽象子类重写的。
  3. **修饰变量**：
 被final修饰的变量称为**最终变量，即常量**——根据被修饰变量的定义位置又可分为**成员常量和局部常量**。**常量只能赋值一次，不能被二次更改**。

---

##  详细回答

- **final**作为Java中的一个关键字**可以用来修饰 类 ， 方法 ，和 变量**。（但**final不能修饰构造器**）
  1. **修饰类**：
 ① 核心特点：
 被final修饰的类**不能被继承，但该类可以去继承别的 (没有被final修饰的)类**，例如String类和System类，它们被final修饰，是不可以被继承的，但是它们有自己的父类——即顶层父类Object类。
 被final修饰的类虽然不能被继承，但 **可以被实例化**。例如，**Java中所有的包装类都被final关键字修饰了**。也就是说，所有的包装类都不能被继承，但都可以被实例化。
 一般地，如果**一个类已经被final关键字**修饰，那么从逻辑上讲，**该类中的方法是没有必要再次用final修饰的**。这是因为用final修饰方法的目的就是为了不让该方法被子类重写；而final修饰的类本身就已经不能被继承了，也就不可能被重写。
 ② 设计意图：
 保证类的**不可变性**（如 **String** 类防止子类破坏字符串内容）。 确保**安全性**（如 **System** 类防止核心 API 被篡改）。
  2. **修饰方法**：
 ① 核心特点：
 被final修饰的方法**不能被子类重写，但可以被子类继承并使用（在满足访问权限规则的前提下）**。当修饰方法时， **final关键字不能与abstract关键字共存**；因为abstract修饰的方法是必须被非抽象子类重写的。
 ② 设计意图：
 **防止核心逻辑被修改**（如模板方法模式中的关键步骤）。
  3. **修饰变量**：
 ① 核心特点：
 被final修饰的变量称为**最终变量，即常量**——根据被修饰变量的定义位置又可分为**成员常量和局部常量**。**常量只能赋值一次，不能被二次更改**。
 **成员常量**——成员常量必须进行初始化，可以在定义成员常量时就对它赋初值来显式初始化，若在定义成员常量时没有赋初值——需要在构造器 或者 代码块中进行初始化。
 **局部常量**——局部常量如果未被使用，可以不赋初值；但如果局部常量被调用了，就必须赋初值。
 **静态常量**——对于静态常量来说，只能通过在定义时为其赋初值；或者先定义，然后在静态代码块中为其赋值，这样来初始化。但是不可以在构造器中进行静态常量的初始化。
 ② 设计意图：
 确保变量引用或值的**不可变性**（提升线程安全性与代码可读性）。

---

##  知识拓展

1. **final 修饰类**的示意图如下：
   →

### 评论
 

验证登录状态...

## 垃圾回收｜大厂面试题｜Java高频面试题｜GC算法、分代收集、内存泄漏

#  垃圾回收

##  简要回答

- 垃圾回收算法包括标记-清除算法、复制算法、标记-压缩算法、分代收集算法等。
  - **标记清除算法**：标记所有存活对象后清除未标记的垃圾对象。
  - **复制算法**：将内存分为两块等大区域（如 From 和 To），每次仅使用一块；GC 时将该区域的存活对象复制到另一块空区域，之后清空原区域并互换两块区域的角色，实现循环复用。
  - **标记压缩算法**：标记清除算法的改进，标记存活对象后将其压缩到内存一端，清除边界外的垃圾，避免内存碎片问题。
- 垃圾回收机制可以解决内存管理问题，**自动检测和回收不再使用的对象**，释放它们所占用的内存空间，避免内存泄漏和预防内存溢出。
##  详细回答

- 垃圾回收算法：
  1. 标记清除算法
    2. 将垃圾回收分为标记阶段和清除阶段。在标记阶段通过根节点(GC Roots)**标记所有从根节点开始的对象**，没有被标记的对象就是未被引用的垃圾对象。在清除阶段**清除所有未被标记对象**。
    3. 优点：无需移动对象，仅清除垃圾，适合**存活对象较多**的情况，如老年代（对象存活久，移动开销大）。
    4. 缺点：容易产生内存碎片，有大对象时可能会提前触发垃圾回收；扫描了两次空间（第一次标记，第二次清理）。
  5. 复制算法
    6. 从根集合节点进行扫描，**标记出所有的存活对象并将其复制到一块新的内存上**，之后将原来的一块内存回收。新生代的Survivor区就是复制算法的体现。
    7. 优点：适合**存活对象少**的情况，如新生代。只需扫描空间一次，标记并进行复制。
    8. 缺点：需要一块空的内存空间，如果存活的对象较多，会导致复制操作的效率较低。
  9. 标记整理算法（标记压缩算法）
    10. 在标记清除算法基础上进行优化，对可达对象做标记后，将其**移动到内存的一端实现空间压缩**，清理其他空间。
    11. 优点：能够避免碎片产生，不需要两块相同内存空间。
  12. 分代收集算法
    13. 将内存空间分为不同的年代，不同年代使用不同的算法。
    14. 新生代存活率低，可以使用复制算法；老年代存活率高，没有额外空间对它进行分配，使用标记清除/压缩算法。
- 内存分配与垃圾回收时机：
  - JVM将堆内存分为新生代和老年代，**新生代**用于保存新创建的对象，大对象以及GC后仍存活的对象会被保存在**老年代**中。新生代有Eden区和Survivor区（Survivor分为From Survivor和To Survivor）。
  1. 新产生的对象会被优先分配在堆内存新生代的Eden区（如果对象是大对象会直接进入老年代）
  2. 当Eden区满或者放不下时会触发一次Minor GC（新生代垃圾回收），每次GC后新生代里存活的对象会被移动到Survivor区（S0或S1），如果Survivor区满或放不下时会将新生代存活对象移入老年代。
  3. 默认情况下，有对象被复制了15次时会进入老年代。
  4. 当老年代满或放不下时会发生Major GC（老年代垃圾回收），老年代中对象存活率较高，所以每次回收可能需要更长时间。
##  知识图解

1. **标记阶段根据引用链判断对象是否可达示意图**  知识扩展

  1. 扩展：
  2. Minor GC/Major GC/Full GC：
    3. **Minor GC**针对新生代进行回收，因为新生代的对象生命周期较短，存活对象较少，回收效率高，Minor GC发生的比较频繁。
    4. **Major GC**针对老年代进行回收，老年代中对象存活率较高，所以每次回收可能需要更长时间，Major GC发生的比较少。
    5. **Full GC**是对整个堆内存进行回收，通常发生在内存不足的时候（老年代满、永久代满、显示调用System.gc()）。需要停止所有的工作线程并遍历整个堆内存来查找和回收不再使用的对象，耗时严重。
  6. **内存泄漏**：
    7. 程序在运行过程中不再使用的对象仍然被引用，无法被垃圾回收，导致可用内存减少。
    8. 常见原因：
      1. **静态集合**：使用静态数据结构存储对象，没有进行清理。
      2. **事件监听**：未取消对事件源的监听，导致对象持续被引用。
      3. **线程**：未停止的线程可能持有对象引用，无法被回收。
  9. **内存溢出**(OOM)：
    10. JVM申请内存时无法找到足够的内存，引发OutOfMemoryError。
    11. 出现原因：
      1. **堆内存溢出**：代码中出现**大对象分配**或者是**内存泄漏**时，多次GC也没有足够的内存空间。
      2. **栈溢出**：代码中出现**递归调用**，压栈过深或者无法扩展栈空间时出现。
      3. **元空间溢出**：系统的**代码过多**或加载的类文件过多，导致元空间内存占用大。
  1. 面试官可能追问：
  12. Q1：分代收集算法中对象从新生代晋升到老年代默认的“年龄”是15次，可以通过什么进行调整吗？会对GC有什么影响？
    13. 可以通过参数(-XX:MaxTenuringThreshold)进行调整。
    14. 阈值过小会使对象更早进入老年代，老年代对象增多，增加Major GC频率。
    15. 阈值过大会使对象在新生代停留更久，Minor GC时存活对象累积，增加复制算法开销。空间不足时对象提前晋升，加大老年代压力。  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Java泛型｜高频面试题｜Java高频面试题｜泛型概念、类型擦除、通配符

#  Java泛型

##  简要回答

1. **泛型的概念**：
  2. 泛型（**Generics**）是 Java 的一种**参数化类型**机制，允许在定义类、接口或方法时声明类型参数，并在使用时指定具体类型，实现类型安全的代码复用。
3. **泛型的作用**：
  4. **类型安全**：泛型强制在编译时检查类型匹配，避免运行时出现 ClassCastException。
  5. **消除强制转换**：泛型自动处理类型转换，无需手动进行强制类型转换。
  6. **代码复用**：适用于编写通用算法和数据结构（如 `List<T>`、`Map<K,V>`）。
7. **泛型的应用**：
  8. **自定义泛型类**：泛型最后代表的**数据类型**是**在创建对象时确定的**。
  9. **自定义泛型接口**：泛型最终代表的数据类型是在**继承该接口**或者**实现该接口**时确定的。
  10. **自定义泛型方法**：**泛型最终代表的数据类型是在调用方法时确定的**，**每次调用泛型方法，都可以指定不同的泛型类型**。
  11. 通过使用**通配符**（`?`）和**边界**（`<? extends T>`、`<? super T>`），可以实现灵活的类型匹配。

---

##  详细回答

1. **泛型的概念**：
  2. **定义**：
 **泛型（Generics），又称参数化类型(Parameterized Types)**，是一种可以“表示其他数据类型”的数据类型。泛型是**JDK5.0中出现的新特性**，解决数据类型的安全性问题，在类声明或实例化时只要指定好具体的类型即可。
  3. **语法**：
 ① 在指定泛型时，**必须要求** 最终确定的数据类型为**引用类型** ，而不可以是基本数据类型。
 ② 若在定义类时使用了泛型，实例化该类时却什么都没有传入，**默认使用Object类型。**
  4. **特性**：
 ① **类型参数化**：在定义类、接口或方法时声明 类型参数，实际使用时再指定 具体类型。
 ② **类型擦除（Type Erasure）**：泛型仅在**编译期**存在，**运行时 类型参数会被擦除**。
 ③ **编译时类型检查**：编译器会确保泛型代码的 类型一致性，防止非法赋值或操作。
5. **泛型的作用**：
  6. **类型安全**：
 泛型强制在编译时检查类型匹配，避免运行时出现 ClassCastException。
  7. **消除强制转换**：
 泛型自动处理类型转换，无需手动进行强制类型转换。
  8. **代码复用**： 通过泛型抽象化类型，可以实现 通用算法和数据结构。例如，`Collections.sort()` 可排序任何实现了 `Comparable<T>` 接口的类型。
9. **泛型的应用**：
  10. **自定义泛型类**：
 ① 泛型最后代表的**数据类型**是**在创建对象时确定的**；如果在创建泛型类对象时没有给出指定类型，**默认会以Object替代**。
 ② 自定义泛型类中，类的**非静态**成员**可以**使用泛型（属性，方法，构造器等）；**静态成员**则**不可以**使用类的泛型。
 ③ **在自定义泛型类中，使用了泛型的数组只可以被定义，不可以被初始化**。
  11. **自定义泛型接口**：
 ① 泛型最终代表的数据类型是在**继承该接口**或者**实现该接口**时确定的；若在使用时没有给出具体泛型，**默认使用Object类型替代**。
 ② 同自定义泛型类一样，自定义泛型接口的**静态成员**也**不能使用泛型**。
  12. **自定义泛型方法**：
 ① 泛型最终代表的数据类型是在**调用方法时确定的**，每次调用泛型方法，都可以**指定不同的泛型类型**。
 ② 自定义泛型方法，既可以定义在**普通类**中，也可以定义在**泛型类中**。
 ③ **注意区分自定义泛型方法 和 泛型在方法上的应用。**，如下所示：

```
`//以下代码仅作为演示，无实际意义
class<T, U> Watermelon {
	public<K> void taste(T t, U u, K k) {
		System.out.println(" T 和 U 代表泛型在方法上的应用；而 K 则是自定义泛型方法的使用。");
	}
}
`
```
 1
2
3
4
5
6

  13. **通配符与边界**： Java 泛型默认是**不变的（Invariant）**，即 `List<String>` 并不是 `List<Object>` 的子类型。为了增加泛型的灵活性和类型兼容性，引入了通配符和边界。
 ① **无界通配符 `<?>`** ：单独使用表示未知类型，支持任意泛型类型。例如 `List<?>`，它通常是可读不可写的（除了 `null`）。
 ② **上界通配符 `<? extends T>`** ：表示支持 `T` 类以及 `T` 类的子类，规定了泛型的上限。它实现了**协变（Covariance）**，主要用于 **生产者（Producer）** 场景，即从集合中**读取**数据。
 ③ **下界通配符 `<? super T>`** ：表示支持 `T` 类以及 `T` 类的父类，规定了泛型的下限。它实现了**逆变（Contravariance）**，主要用于 **消费者（Consumer）** 场景，即向集合中**写入**数据。

---

##  知识拓展

###  泛型的图解

1. **引入泛型前 和 引入泛型后 的对比图** 如下：  类型擦除（Type Erasure）

  1.

**类型擦除的本质**：类型擦除的核心规则是：

    2. **泛型类型参数在编译后会被擦除**，替换为**原始类型**（Raw Type）或**边界类型**（Bound）。
    3. **泛型仅在编译期存在**，运行时无法获取类型参数的具体信息（如 `List<String>` 运行时仅为 `List`）。
  4.

**擦除规则**：

    5.

**无界类型参数（`<T>`）**：替换为 `Object`。


```
`// 编译前
public class Box<T> { private T data; }
// 编译后（字节码等效）
public class Box { private Object data; }
`
```
 1
2
3
4

    6.

**有界类型参数（`<T extends Number>`）**：替换为边界类型（如 `Number`）。


```
`// 编译前
public class NumericBox<T extends Number> { private T data; }
// 编译后
public class NumericBox { private Number data; }
`
```
 1
2
3
4

  7.

**类型擦除的影响与限制**：

    8.

**运行时类型信息丢失**：


```
`List<String> strList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();
// 以下结果为 true，因为两者运行时类型均为 ArrayList
System.out.println(strList.getClass() == intList.getClass());
`
```
 1
2
3
4

    9.

**无法实例化类型参数**：


```
`public class Box<T> {
	// 编译错误：Cannot instantiate the type T
	private T instance = new T();
}
`
```
 1
2
3
4

    10.

**无法检查泛型类型**：


```
`if (list instanceof List<String>) {} // 编译错误
`
```
 1

    11.

**重载冲突**： 类型擦除后，以下方法会因签名相同导致编译错误：


```
`void print(List<String> list) {}
void print(List<Integer> list) {} // 编译报错
`
```
 1
2


---
  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...

## Spring循环依赖｜大厂面试题｜Java高频面试题｜三级缓存、提前暴露、AOP代理、单例Bean

#  Spring循环依赖

##  简要回答

- Spring通过**三级缓存**和**提前暴露未完全初始化的对象引用**机制来解决单例作用域Bean的循环依赖问题。
- 假设A和B存在循环依赖问题，A创建实例后未注入属性时会存放ObjectFactory对象到三级缓存中，开始给A注入属性时发现A依赖B，此时容器开始创建B，B实例化后也会被存放ObjectFactory对象到三级缓存中，Spring给B注入属性时发现B依赖A，容器在三级缓存中找到A的ObjectFactory对象，获取A的早期引用并放入二级缓存，并清理三级缓存。将A的早期引用注入到B中，完成B的初始化后进入一级缓存。回到A的属性注入环节，将就绪的B注入A，完成A的初始化后进入一级缓存，解决循环依赖问题。
##  详细回答

###  循环依赖

- 循环依赖就是循环引用，有两个或多个Bean之间相互引用，形成闭环，导致容器无法完成Bean的实例化和依赖注入，会抛出循环依赖异常。
- Spring中存在两种循环依赖，分别是**构造器依赖**和**field属性**的循环依赖。构造器的循环依赖问题无法解决，只能抛出BeanCurrentlyInCreationException异常，使用提前暴露对象的方法可以解决属性循环依赖。
###  检测循环依赖问题

- 在Bean创建的时候给该Bean打标，当递归回来发现正在创建中的话说明产生了循环依赖。
###  三级缓存解决循环依赖

- 三级缓存是Spring在DefaultSingletonBeanRegistry中维护的三个重要的缓存(Map)
  1. **一级缓存(singletonObjects)** 存放完全创建好的Bean实例，可以直接使用，Bean已经实例化，依赖注入、初始化，有AOP代理也已经生成。
  2. **二级缓存(earlySingletonObjects)** 存放提前暴露的Bean的原始对象引用或者早期代理对象引用，专门用来处理循环依赖。此时Bean已经实例化但是还没有完成依赖注入与初始化。
  3. **三级缓存(singletonFactories)** 存放Bean的ObjectFactory工厂对象，当Bean被实例化后Spring会创建一个工厂对象放入三级缓存。当其他Bean需要获取三级缓存中的Bean时，三级缓存可以动态返回半成品Bean，原始对象或代理对象。
- 当两个Bean存在相互依赖时，**Spring的处理过程**如下：
  1. 调用A的构造函数进行**实例化**，在属性注入之前在第三级缓存(singletonFactories)中创建一个工厂对象，并放入三级缓存中。
  2. Spring给A注入属性时发现A依赖B，缓存中没有B于是对B进行**实例化**，实例化过程中同样创建一个工厂对象，并放入三级缓存中。
  3. Spring给B注入属性时发现B依赖A，同时在三级缓存中找到A的工厂对象，调用该工厂的getObject()方法。该方法在A需要AOP代理时动态生成代理对象，否则返回原始对象，**得到的A早期引用或代理对象**后存入第二级缓存，同时清除三级缓存中的A工厂对象，使用A的早期引用完成B的依赖注入。
  4. 随后B正常完成初始化方法并转化为完整可用的Bean，存入一级缓存，同时清除二级缓存或三级缓存中B相关的内容。
  5. 回到A的依赖注入阶段，此时会直接从二级缓存中获取A的早期引用/代理，在一级缓存中获取B实例完成依赖注入，执行剩余方法A转换为完整可用的Bean，存入一级缓存，同时清除二级缓存或三级缓存中A相关的内容。
##  知识图解

- 循环依赖示意图  知识扩展

  1. 面试官可能追问
  - Q1：一定要三级缓存才能解决问题吗？
    - 只有三级缓存才能解决循环依赖问题，因为需要正确处理AOP代理的Bean，只使用二级缓存会导致注入对象的形态错误，破坏单例原则。
    - 当A和B两个Bean之间存在循环依赖时，如果A需要被动态代理，B创建时会拿到A的原始对象，而不是A的代理对象。导致B持有原始对象A，Spring容器存储的是代理对象A，同一个Bean出现了两个不同的实例，违反了单例的原则。
  - Q2：为什么构造器循环依赖不能解决？
    - 构造器循环依赖要求在对象实例化的同时就注入依赖，而循环依赖导致这个过程无法启动。而Setter注入和字段注入则是在实例化之后才进行依赖注入，这就为Spring提供了提前暴露Bean实例、解决循环依赖的机会。
  - Q3：Spring循环依赖，提前暴露对象的前提是什么？
    - Bean是单例作用域且循环依赖是字段属性依赖。
    - 其他作用域会造成递归创建，构造器依赖不能提前暴露。
  - Q4：Spring循环依赖，实际开发中怎么避免？
    - 将产生循环依赖的共同逻辑抽成第三方服务
    - 使用@Lazy注解标记依赖，延迟到首次使用时注入
    - 使用属性注入替代构造器注入  Last Updated: 3/10/2026, 6:08:48 PM

 ←   →

### 评论
 

验证登录状态...
