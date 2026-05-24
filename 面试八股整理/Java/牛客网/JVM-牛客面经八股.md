# JVM-牛客面经八股

> 来源：牛客网  |  共 10 题

## 1. 说说你了解的JVM内存模型
### 

### 一句话总结
 JVM内存模型主要包含堆（存放对象实例）、方法区（存储类信息、常量）、虚拟机栈（线程私有方法调用）、本地方法栈（Native方法）和程序计数器（线程执行位置）。堆和方法区线程共享，栈和程序计数器线程私有，直接内存通过堆外分配管理。
 
### 详细解析
 Java 虚拟机（JVM）的内存区域分为多个部分，每个部分负责不同的任务。 
---
 
#### **1. 程序计数器（Program Counter Register）作用**：记录当前线程执行的位置（字节码指令的地址），确保线程切换后能恢复到正确的位置。 **特点**： **线程私有**：每个线程独立存储，互不影响。 **唯一无 OOM 的区域**：不会抛出OutOfMemoryError。 **Native 方法时值为空**：执行本地方法（如 C/C++ 代码）时，计数器值为undefined。 
---
 
#### **2. 虚拟机栈（Java Virtual Machine Stacks）作用**：存储方法调用的栈帧（Stack Frame），每个方法从调用到完成对应一个栈帧的入栈到出栈。 **结构**： **局部变量表**：存放基本数据类型（int,boolean等）、对象引用（指针）和返回地址。 **操作数栈**：执行字节码指令的临时数据存储区（如算术运算的中间结果）。 **动态链接**：指向运行时常量池的方法引用。 **方法出口**：记录方法返回的地址（正常返回或异常退出）。 **异常**： **StackOverflowError**：栈深度超过限制（如无限递归）。 **OutOfMemoryError**：扩展栈时无法申请足够内存（较少见）。 **线程私有**：每个线程的栈独立分配。 
---
 
#### **3. 本地方法栈（Native Method Stack）作用**：为 Native 方法（非 Java 代码，如 JNI 调用）提供栈空间。 **特点**： 与虚拟机栈类似，但服务于本地方法。 HotSpot 虚拟机将两者合并。 **异常**：同虚拟机栈（StackOverflowError和OutOfMemoryError）。 
---
 
#### **4. 堆（Heap）作用**：存放对象实例和数组（所有线程共享的主内存区域）。 **结构**（分代模型）： **新生代（Young Generation）**： **Eden 区**：对象首次分配的区域。 **Survivor 区**（From/To）：存活对象在 Minor GC 后暂存。 **老年代（Old Generation）**：长期存活的对象（经过多次 GC 后晋升）。 **元数据区（JDK8+）**：替代永久代，存放类元信息。 **异常**：OutOfMemoryError（堆内存不足）。 **关键参数**： -Xms：初始堆大小。 -Xmx：最大堆大小。 -XX:NewRatio：新生代与老年代比例。 
 
---
 
#### **5. 方法区（Method Area）作用**：存储类信息（类名、方法、字段）、常量、静态变量、即时编译器代码。 **演变**： **JDK7 及之前**：称为永久代（PermGen），受-XX:PermSize和-XX:MaxPermSize控制。 **JDK8+**：改为元空间（Metaspace），使用本地内存，由-XX:MetaspaceSize和-XX:MaxMetaspaceSize配置。 **异常**：OutOfMemoryError（类元数据过多）。 
---
 
#### **6. 运行时常量池（Runtime Constant Pool）位置**：方法区的一部分。 **内容**： 编译期生成的字面量（如字符串常量"abc"）。 符号引用（类、方法、字段的全限定名）。 **动态性**：运行期间可添加新常量（如String.intern()）。 **异常**：OutOfMemoryError（常量池溢出）。 
---
 
#### **7. 直接内存（Direct Memory）作用**：通过ByteBuffer.allocateDirect()分配的堆外内存，提升 NIO 性能。 **特点**： 不受 JVM 堆限制，但受系统内存影响。 由-XX:MaxDirectMemorySize控制大小。 **异常**：OutOfMemoryError（直接内存耗尽）。 
---
 
#### **8. 内存区域对比表**
 
| 区域 | 线程共享 | 存储内容 | 异常 | 配置参数 |
| --- | --- | --- | --- | --- |
| 程序计数器 | 私有 | 当前指令地址 | 无 | 无 |
| 虚拟机栈 | 私有 | 方法栈帧（局部变量、操作数栈） | StackOverflowError/OOM | -Xss（栈大小） |
| 本地方法栈 | 私有 | Native 方法栈帧 | StackOverflowError/OOM | 无 |
| 堆 | 共享 | 对象实例、数组 | OOM: Java heap space | -Xms,-Xmx,-XX:NewRatio |
| 方法区（元空间） | 共享 | 类元数据、常量、静态变量 | OOM: Metaspace | -XX:MetaspaceSize,-XX:MaxMetaspaceSize |
| 直接内存 | 共享 | 堆外缓冲数据 | OOM: Direct buffer memory | -XX:MaxDirectMemorySize |
 
---
 
#### **9. 常见问题示例堆内存溢出**： 
 
```java
List<Object> list = new ArrayList<>();
while (true) {
 list.add(new Object()); // 不断创建对象，最终抛出 OOM
}
```
 
 **解决**：增大-Xmx或优化代码减少对象创建。 
 
 **元空间溢出**： 
 
```java
// 动态生成大量类（如使用 CGLib）
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(MyClass.class);
enhancer.setCallback(...);
enhancer.create(); // 重复执行导致元空间 OOM
```
 
 **解决**：增大-XX:MaxMetaspaceSize或限制动态类生成。 
 
 **栈溢出**： 
 
```java
public void recursiveMethod() {
 recursiveMethod(); // 无限递归导致 StackOverflowError
}
```
 
 **解决**：检查递归终止条件或增大-Xss（谨慎使用）。 
 
####

## 4. 说说JVM的垃圾回收算法。
### 

### 一句话总结
 JVM主要垃圾回收算法包括：标记-清除（产生内存碎片）、复制算法（适合新生代）、标记-整理（适合老年代）。现代JVM多采用分代收集策略，结合不同算法管理新生代（复制）和老年代（标记-清除/整理）。还有增量、并发算法（如G1）减少停顿时间。
 
### 详细解析
 JVM的垃圾回收（GC）算法是自动内存管理的核心，其设计目标是在不同场景下平衡吞吐量、延迟和内存利用率。以下是主流算法及其特点的总结： **一、基础算法分类** 
| 算法 | 原理 | 优点 | 缺点 | 适用场景 |
| --- | --- | --- | --- | --- |
| 标记-清除 | 分标记（遍历对象图）和清除（回收未标记对象）两阶段 | 实现简单 | 内存碎片化，可能触发Full GC | 老年代（CMS收集器） |
| 复制算法 | 将堆分为两块，存活对象复制到另一块后清空原区域 | 无碎片，效率高 | 内存利用率低（仅用50%） | 年轻代（Serial/Parallel） |
| 标记-整理 | 标记后整理存活对象至内存一端，清理边界外空间 | 避免碎片，内存利用率高 | 整理耗时，可能引发STW | 老年代（Serial Old） |
| 分代收集 | 按对象生命周期划分新生代（复制算法）和老年代（标记-清除/整理） | 针对性优化效率 | 需协调多代策略 | 通用方案（G1/Parallel GC） |
 
---
 
 **二、进阶算法与优化策略** 
 
| 算法 | 原理 | 优点 | 缺点 | 适用场景 |
| --- | --- | --- | --- | --- |
| 增量收集 | 将GC任务拆分为小步骤，与应用线程交替执行 | 减少单次STW时间 | 总GC时间增加，可能碎片化 | 早期CMS部分阶段 |
| CMS | 并发标记清除：初始标记（STW）→并发标记→重新标记（STW）→并发清除 | 低延迟（仅两次短暂STW） | 无法处理浮动垃圾，可能触发Full GC | 互联网服务端（低延迟需求） |
| G1 | 分区堆（Region），优先回收垃圾最多的Region，混合标记-清除与整理 | 可预测停顿时间，支持大堆 | 内存占用较高 | 大内存、低延迟（JDK 9+默认） |
| ZGC | 染色指针+读屏障，全堆并发标记，STW极短（<10ms） | 超低延迟，支持TB级堆 | 内存占用高，JDK 15+可用 | 超大规模内存应用 |
| Shenandoah | 类似ZGC，但通过Brooks指针实现并发压缩 | 低延迟，减少碎片 | 需额外内存维护指针 | 高吞吐与低延迟平衡场景 |
 
---
 
 **三、算法对比与选型** 
 
| 算法 | 吞吐量 | 延迟 | 内存利用率 | 适用代 | 典型收集器 |
| --- | --- | --- | --- | --- | --- |
| 标记-清除 | 中 | 高 | 低 | 老年代 | CMS |
| 复制算法 | 高 | 低 | 中 | 年轻代 | Serial/Parallel |
| 标记-整理 | 中 | 中 | 高 | 老年代 | Serial Old |
| 分代收集 | 高 | 中 | 中 | 年轻代+老年代 | G1/Parallel GC |
| ZGC | 高 | 极低 | 高 | 全堆 | ZGC |
 
---
 
 **四、总结** 
 
 JVM垃圾回收算法通过分代策略和混合回收实现高效内存管理： 
 
 **分代收集是主流**：结合不同区域对象特性选择高效算法，平衡吞吐量与延迟。 
 
 **演进趋势**：现代收集器（如 G1、ZGC）通过分区和并发处理进一步优化，减少 STW 时间。 
 
 **调优关键**：根据应用特性（如高吞吐或低延迟）选择合适的 GC 算法和收集器。

## 7. 说说GC的可达性分析。
### 

### 一句话总结
 GC可达性分析是判断对象存活的算法，从GC Roots（如栈局部变量、静态变量等）出发，遍历所有引用链。无法被Roots触及的对象视为垃圾，会被回收。此方法避免了循环引用问题，是主流JVM垃圾回收的基础机制。
 
### 详细解析
 
 垃圾回收（GC）中的 **可达性分析（Reachability Analysis）** 是一种用于判断对象是否存活的算法。其核心思想是：通过一系列称为 **GC Roots** 的根对象作为起点，遍历所有能被访问到的对象，将这些对象标记为“存活”，而无法被访问到的对象则判定为“不可达”（即垃圾对象），随后被回收。 
 
---
 
#### **1. 基本原理GC Roots** 是可达性分析的起点，通常是以下几种对象： **虚拟机栈中的局部变量**（例如当前执行方法中的局部变量、参数）。 **方法区中的静态变量**（类的静态字段）。 **本地方法栈中的对象**（JNI 引用的 Native 方法对象）。 **被锁（synchronized）持有的对象**。 **Java 虚拟机内部引用**（如基本类型对应的 Class 对象、常驻异常对象等）。 **可达对象**：从 GC Roots 出发，通过引用链（Reference Chain）能访问到的所有对象。 **不可达对象**：无法通过任何引用链连接到 GC Roots 的对象。 
---
 
#### **2. 分析过程标记阶段**： 
 从所有 GC Roots 出发，递归遍历对象图（Object Graph），标记所有可达对象。 例如：若对象 A 被 GC Root 直接引用，而对象 B 被 A 引用，则 A 和 B 均会被标记为存活。 
 **回收阶段**： 
 遍历堆中所有对象，回收未被标记的对象（不可达对象）。 某些垃圾回收器会在此阶段进行内存整理（如 CMS 的并发标记清除，G1 的分区整理）。 
---
 
#### **3. 对象的“复活”机会**
 对象被标记为不可达后，并非立即回收，而是会进入“缓刑”阶段： 第一次标记：对象被判定为不可达。 **finalize()方法**：若对象重写了finalize()方法，且未被调用过，则会被放入F-Queue队列，由低优先级的 Finalizer 线程执行finalize()。 第二次标记：若在finalize()中对象重新与 GC Roots 建立引用链（例如将自己赋值给某个静态变量），则对象“复活”；否则被回收。 **注意**：finalize()方法不推荐使用，因为其执行时机不确定，且可能引发性能问题。 
---
 
#### **4. 引用类型的影响**
 
 可达性分析还与 Java 的引用类型密切相关： 
 **强引用（Strong Reference）**：对象直接与 GC Roots 关联，不会被回收。 **软引用（Soft Reference）**：内存不足时，软引用关联的对象会被回收。 **弱引用（Weak Reference）**：无论内存是否足够，弱引用关联的对象在下一次 GC 时被回收。 **虚引用（Phantom Reference）**：无法通过虚引用访问对象，仅用于追踪对象被回收的状态。 
---
 
#### **5. 三色标记法（Tri-color Marking）**
 
 现代垃圾回收器（如 G1、ZGC）使用 **三色标记法** 优化可达性分析： 
 **白色**：未被访问的对象（初始状态）。 **灰色**：对象已被访问，但其引用的子对象未被访问。 **黑色**：对象及其子对象均被访问。 **过程**：从灰色对象出发，逐步将灰色对象转为黑色，直到所有可达对象被标记。 
---
 
#### **6. 关键注意事项循环引用**：即使对象之间形成循环引用，只要它们无法通过 GC Roots 访问到，仍会被回收。 **Stop-The-World（STW）**：可达性分析通常需要暂停用户线程（如 Serial、Parallel GC），但 ZGC、Shenandoah 等低延迟回收器通过并发标记减少停顿。

## 5. 请你讲下CMS垃圾回收器。
### 

### 一句话总结
 CMS（Concurrent Mark Sweep）是Java的并发低停顿垃圾回收器，通过“初始标记-并发标记-重新标记-并发清除”四阶段实现，减少停顿时间。缺点包括内存碎片、浮动垃圾问题，且无法处理并发失败。已在JDK9被标记废弃，JDK14正式移除。
 
### 详细解析
 
 **CMS（Concurrent Mark Sweep）垃圾回收器** 是 JVM 中一种以 **低延迟** 为目标的垃圾回收器，主要用于老年代的垃圾回收。它通过 **并发标记-清除算法** 减少垃圾回收时的停顿时间（STW），适合对响应速度敏感的应用（如 Web 服务、实时系统）。以下是其核心原理、工作流程及优缺点的详细说明： 
 
---
 
#### **一、CMS 的核心设计目标最小化 STW（Stop-The-World）时间**：通过并发执行大部分回收阶段，减少用户线程的停顿。 **适用场景**：适合内存较大、对延迟敏感的老年代回收（JDK 9 后已不推荐使用，逐渐被 G1、ZGC 等替代）。 
---
 
#### **二、CMS 的工作流程**
 
 CMS 的回收过程分为 **四个阶段**，其中 **初始标记** 和 **重新标记** 需要 STW，其他阶段与用户线程并发执行： 
 
| 阶段 | 行为 | 是否 STW |
| --- | --- | --- |
| 1. 初始标记（Initial Mark） | 标记 GC Roots 直接关联 的对象（速度极快）。 | 是 |
| 2. 并发标记（Concurrent Mark） | 从 GC Roots 出发，遍历老年代 所有存活对象（与用户线程并发执行）。 | 否 |
| 3. 重新标记（Remark） | 修正并发标记阶段 因用户线程运行导致的标记变动（增量更新或原始快照）。 | 是 |
| 4. 并发清除（Concurrent Sweep） | 清除未被标记的垃圾对象（与用户线程并发执行）。 | 否 |
 
---
 
#### **三、CMS 的优缺点优点低延迟**：大部分阶段与用户线程并发执行，STW 时间短。 **适用大内存**：适合老年代较大的堆内存（如 4GB 以上）。 **缺点内存碎片**：使用标记-清除算法，不整理内存，可能导致 Full GC 时触发压缩（STW 时间长）。 **CPU 敏感**：并发阶段占用 CPU 资源，可能影响应用吞吐量。 **浮动垃圾**：并发清理阶段用户线程可能产生新垃圾，需预留空间（通过-XX:CMSInitiatingOccupancyFraction设置触发阈值）。 **并发失败（Concurrent Mode Failure）**：若回收速度跟不上对象分配速度，会退化为 Serial Old 收集器（Full GC）。 
---
 
#### **四、CMS 的关键参数**
 
| 参数 | 作用 |
| --- | --- |
| -XX:+UseConcMarkSweepGC | 启用 CMS 收集器。 |
| -XX:CMSInitiatingOccupancyFraction=N | 设置老年代使用率达到 N% 时触发 CMS 回收（默认 68%）。 |
| -XX:+UseCMSCompactAtFullCollection | Full GC 时压缩内存碎片（默认启用）。 |
| -XX:CMSFullGCsBeforeCompaction=N | 执行 N 次 Full GC 后触发内存压缩（默认 0，每次 Full GC 都压缩）。 |
| -XX:+CMSClassUnloadingEnabled | 启用类卸载（默认禁用，需显式开启）。 |
 
---
 
#### **五、CMS 的适用场景与局限性适用场景** 老年代回收，且应用对延迟敏感（如响应时间要求 < 1s）。 JDK 1.8 及之前版本（JDK 9 后标记为废弃，JDK 14 中移除）。 **局限性内存碎片问题**：需配置压缩参数，但会增加 Full GC 时间。 **并发失败风险**：需合理设置触发阈值和堆大小。 **替代方案推荐**：在 JDK 8+ 中，优先使用 G1 或 ZGC（更低延迟、无碎片）。 
---
 
#### **六、CMS 的完整回收流程图**
 
```plaintext
 [CMS 开始]
 ↓
 → 初始标记（STW） ← 依赖 Young GC 触发
 ↓
 并发标记（并发执行）
 ↓
 → 重新标记（STW） ← 使用增量更新或原始快照
 ↓
 并发清除（并发执行）
 ↓
 [CMS 结束]
```
 
---
 
#### **七、CMS 的替代方案**
 
| 收集器 | 特点 |
| --- | --- |
| G1 | 分区回收、可预测停顿、兼顾吞吐与延迟（JDK 9+ 默认）。 |
| ZGC | 亚毫秒级延迟、支持 TB 级堆内存（JDK 15+ 生产可用）。 |
| Shenandoah | 低延迟、与 ZGC 竞争（需特定 JDK 版本支持）。 |
 
####

## 6. 介绍下G1收集器？
### 

### 一句话总结
 G1（Garbage-First）是Java HotSpot虚拟机的低延迟垃圾收集器，采用分Region堆内存布局，兼顾年轻代和老年代回收。它通过并发标记和优先回收垃圾最多的Region（Garbage-First原则），实现可控的停顿时间预测（通过-XX:MaxGCPauseMillis参数）。适用于大内存、低延迟场景，替代了传统的CMS收集器，支持压缩整理避免内存碎片。
 
### 详细解析
 
 **一、G1收集器的核心特点1. 设计目标低延迟与可预测停顿**：G1 以可控的停顿时间（默认 200ms）为核心目标，通过分区回收和启发式算法实现，适合大内存堆（如几十 GB 至 TB 级）。 **全堆管理**：无需区分新生代与老年代的物理边界，所有堆内存被划分为多个等大小的 **Region**（默认 2048 个），每个 Region 可动态扮演 Eden、Survivor 或老年代角色。 **2. 核心机制分区回收**：优先回收垃圾比例高的 Region（Garbage-First 原则），避免全堆扫描，减少回收范围。 **Remembered Set（RSet）**：记录跨 Region 的引用关系，避免全堆扫描。例如，YGC 时仅需扫描年轻代 Region 的 RSet，无需遍历老年代。 **SATB（Snapshot-At-The-Beginning）**：通过快照解决并发标记阶段的漏标问题，确保标记准确性。 **3. 工作流程** 
 G1 的回收过程分为四个阶段： 
 **初始标记（STW）**：标记 GC Roots 直接关联的对象，耗时极短。 **并发标记**：与用户线程并发标记存活对象。 **最终标记（STW）**：合并并发标记期间的引用变更（通过 RSet 和 SATB 快照修正）。 **筛选回收**：根据 Region 的回收价值排序，按用户设定的停顿时间选择回收区域，存活对象复制到空闲 Region（局部整理）。 **4. 适用场景** 大内存堆且对延迟敏感的应用（如实时数据处理）。 需要避免长时间 Full GC 的场景（如高并发服务）。 
---
 
#### **二、G1 与 CMS 的核心区别**
 
| 对比维度 | G1 收集器 | CMS 收集器 |
| --- | --- | --- |
| 适用范围 | 全堆（新生代 + 老年代），无需搭配其他收集器 | 仅老年代，需配合 ParNew 等新生代收集器 |
| 内存模型 | 逻辑分代，物理分区（Region），动态调整区域角色 | 物理分代（固定 Eden、Survivor、老年代），连续内存块 |
| 回收算法 | 标记-整理（整体） + 复制（局部），减少碎片 | 标记-清除，产生内存碎片，需 Full GC 时整理 |
| 停顿时间控制 | 可预测（通过-XX:MaxGCPauseMillis设定），混合回收（Mixed GC）降低影响 | 无法预测，依赖并发标记，可能因浮动垃圾触发 Full GC（Concurrent Mode Failure） |
| 处理大对象 | 大对象（>50% Region 大小）可跨多个 Region 存放，避免直接晋升老年代 | 大对象直接进入老年代，可能加剧碎片问题 |
| 内存碎片 | 局部复制整理，碎片较少 | 标记-清除导致碎片，频繁 Full GC 时需压缩 |
| 并发阶段 | 仅初始标记和最终标记需 STW，并发标记和筛选回收与用户线程并行 | 初始标记和重新标记需 STW，并发标记和清理与用户线程并行 |
| CPU 资源消耗 | 较高（RSet 维护和并发标记开销），但可通过参数调优平衡 | 高（并发标记和清理占用 CPU，默认线程数(CPU核数+3)/4） |
| 漏标处理 | 原始快照（SATB），保留标记开始时的引用快照 | 增量更新，重新标记阶段修正新增引用 |
| JDK 版本支持 | JDK 7 引入，JDK 9+ 默认 | JDK 5 引入，JDK 9 后废弃，JDK 14 移除 |
 
---
 
#### **三、关键问题浮动垃圾处理CMS**：并发清理阶段用户线程产生的垃圾需下次 GC 回收，易引发 Concurrent Mode Failure，退化为 Serial Old 收集器。 **G1**：通过 Mixed GC 动态回收部分老年代 Region，结合 SATB 减少漏标，避免 Full GC。 
 **内存碎片影响CMS**：频繁 Full GC 时需压缩碎片（-XX:+UseCMSCompactAtFullCollection），增加停顿时间。 **G1**：每次回收均整理部分 Region，整体碎片率低，适合长期运行的系统。 
 **三色标记与漏标CMS** 使用增量更新（记录新增引用），重新标记阶段需遍历变更对象。 **G1** 使用 SATB（记录删除的引用），通过快照保留标记开始时的对象图，减少重新标记工作量。

## 2. 说说类加载机制
### 

### 一句话总结
 类加载机制是JVM动态加载类的过程，包含加载、验证、准备、解析和初始化五个阶段。加载阶段读取.class文件生成Class对象；验证确保字节码合法；准备为静态变量分配内存并赋默认值；解析将符号引用转为直接引用；初始化执行静态代码块和变量赋值。各阶段顺序执行，确保类正确加载且符合安全规范。
 
### 详细解析
 
 Java的类加载过程可以分为以下几个阶段，每个阶段都有其特定的任务和顺序： 
 
#### **1. 加载（Loading）任务**：查找并加载类的二进制字节流（如.class文件），将类的静态存储结构转化为方法区的运行时数据结构，并在堆中生成一个代表该类的java.lang.Class对象。 **触发条件**：当程序首次主动使用类时（如实例化对象、访问静态字段等）。 **类加载器**： **启动类加载器（Bootstrap ClassLoader）**：加载JAVA_HOME/lib下的核心类库（如rt.jar）。 **扩展类加载器（Extension ClassLoader）**：加载JAVA_HOME/lib/ext下的扩展类。 **应用程序类加载器（Application ClassLoader）**：加载用户类路径（ClassPath）的类。 **自定义类加载器**：用户可继承ClassLoader实现自定义加载逻辑（如热部署、模块化加载）。 **双亲委派模型**：子类加载器优先委派父类加载器加载类，确保核心类库的安全性和唯一性。 
---
 
#### **2. 验证（Verification）任务**：确保字节码符合JVM规范，防止恶意代码破坏虚拟机。 **验证内容**： **文件格式验证**：检查魔数（0xCAFEBABE）、版本号、常量池等。 **元数据验证**：检查类继承、字段/方法访问权限、抽象类实现等是否符合语义。 **字节码验证**：验证方法体中的指令逻辑（如操作数栈类型、跳转指令合法性）。 **符号引用验证**：检查符号引用能否正确解析（如类、方法、字段是否存在）。 
---
 
#### **3. 准备（Preparation）任务**：为类的静态变量分配内存并设置初始值（默认值，非代码中显式赋予的值）。 **示例**： 
```java
public static int value = 123; // 准备阶段 value = 0，初始化阶段 value = 123
```
 **注意**：若字段被final修饰且为编译期常量（如public static final int VALUE = 123），则直接赋值到常量池，不经过准备阶段。 
---
 
#### **4. 解析（Resolution）任务**：将常量池中的符号引用（Symbolic References）转换为直接引用（Direct References）。 **符号引用**：以符号（如全限定名）描述引用的目标。 **直接引用**：指向目标的指针、偏移量或句柄。 **解析目标**： 类/接口解析 字段解析 方法解析 接口方法解析 **特点**：解析阶段可能发生在初始化之后（支持动态绑定）。 
---
 
#### **5. 初始化（Initialization）任务**：执行类构造器<clinit>()方法，完成静态变量赋值和静态代码块的执行。 **触发条件**：主动引用类的五种场景： new实例化对象、访问类的静态字段（非final）或静态方法。 反射调用类（如Class.forName("com.example.Test")）。 初始化子类时，父类未初始化会先触发父类初始化。 虚拟机启动时指定的主类（包含main()方法的类）。 JDK7+的动态语言支持（如MethodHandle解析结果触发初始化）。 **线程安全**：JVM保证<clinit>()方法在多线程环境下被正确加锁同步。 
---
 
#### **6. 类加载示例**
 
```java
class Parent {
 static int a = 1;
 static { System.out.println("Parent init"); }
}

class Child extends Parent {
 static int b = 2;
 static { System.out.println("Child init"); }
}

public class Main {
 public static void main(String[] args) {
 System.out.println(Child.b); // 输出：Parent init → Child init → 2
 }
}
```
 **执行过程**： 访问Child.b触发Child类的初始化。 由于Child的父类Parent未初始化，先初始化Parent。 父类Parent初始化完成后，再初始化子类Child。

## 3. 介绍下双亲委派模型，如何打破它？
### 

### 一句话总结
 双亲委派模型是Java类加载机制，子类加载器先委托父类加载器尝试加载类，避免重复加载并保证核心类安全。打破方式包括：1. 重写ClassLoader的loadClass()方法，自定义加载逻辑；2. 使用线程上下文类加载器，强制指定类加载器层级；3. SPI等场景通过反向委派（如JDBC驱动加载）绕过默认机制。
 
### 详细解析
 
#### 一、双亲委派模型（Parent Delegation Model）
 1. 基本概念 
 双亲委派模型是Java类加载器（ClassLoader）的一种**层次化委托机制**，用于确保类加载的**唯一性、安全性和一致性**。其核心思想是：**类加载器加载类时，先委托给父类加载器尝试加载，父类无法加载时才由自身加载**。 
 2. 类加载器的层次结构 
 Java的类加载器按层次分为以下几类（从顶层到底层）： 
 
| 类加载器 | 职责 | 实现语言 |
| --- | --- | --- |
| 引导类加载器（Bootstrap ClassLoader） | 加载JVM核心类（如java.lang.*、java.util.*），路径通常是$JAVA_HOME/lib | C++（JVM实现） |
| 扩展类加载器（Extension ClassLoader） | 加载JVM扩展类（如javax.*），路径通常是$JAVA_HOME/lib/ext | Java |
| 应用类加载器（Application ClassLoader） | 加载用户项目中的类（如classpath下的类），是ClassLoader.getSystemClassLoader()的返回值 | Java |
| 自定义类加载器（Custom ClassLoader） | 用户自定义的类加载器（如加载网络、数据库或加密的类文件） | Java |
 
 **父子关系**：
 自定义类加载器的父加载器默认是应用类加载器，应用类加载器的父是扩展类加载器，扩展类加载器的父是引导类加载器（引导类加载器在Java层不可见，通常表示为null）。 
 3. 工作流程 
 双亲委派的核心逻辑体现在ClassLoader的loadClass方法中，流程如下： 
 **检查已加载**：当前类加载器先检查目标类是否已加载（通过findLoadedClass方法），若已加载则直接返回。 **向上委托父类**：若未加载，委托给父类加载器尝试加载（递归此过程，直到引导类加载器）。 **父类无法加载**：若父类加载器均无法加载（如不在其加载路径），则当前类加载器自己尝试加载（通过findClass方法）。 **加载并解析**：加载成功后，调用resolveClass方法解析类（可选）。 
 **示例流程**：加载com.example.User类时： 
 应用类加载器先委托给扩展类加载器 → 扩展类加载器委托给引导类加载器。 引导类加载器检查java.lang等核心包，发现无com.example.User，返回失败。 扩展类加载器同理，返回失败。 应用类加载器最终自己加载classpath下的User.class。 
 4. 设计目的 **避免类重复加载**：父类加载器已加载的类，子类无需重复加载，确保类在JVM中唯一。 **保证核心类安全**：防止用户自定义同名核心类（如java.lang.String）覆盖JDK原生类，避免安全漏洞。例如，用户自定义的java.lang.String会被双亲委派机制委托给引导类加载器，而引导类加载器已加载JDK的String，因此用户的类不会被加载。 
#### 二、如何打破双亲委派模型？
 
 虽然双亲委派模型是Java的默认机制，但在某些场景下需要打破它，例如： 
 1. 打破的典型场景 **父类加载器需要调用子类加载器的类**（如JDK的SPI机制，如JDBC、JNDI）：
 JDK提供接口（如java.sql.Driver）由引导类加载器加载，但接口的实现类（如MySQL的com.mysql.cj.jdbc.Driver）由应用类加载器加载。此时父类加载器（引导类）需要访问子类加载器（应用类）的类，双亲委派无法实现。 **热部署/热替换**（如OSGi、Spring Boot DevTools）：
 需要动态加载、卸载类的不同版本（如模块升级），要求同一个类的不同版本由不同加载器加载。 **自定义类加载需求**（如加密类加载、非标准路径加载）：
 需要从网络、数据库或加密文件中加载类，且不希望父类优先加载。 2. 打破的核心方法 
 打破双亲委派的关键是**重写类加载器的loadClass方法**，改变“先委托父类”的默认逻辑。 
 3. 具体实现方式 （1）通过线程上下文类加载器（Thread Context ClassLoader） 
 JDK的SPI机制（如JDBC）通过**线程上下文类加载器**打破双亲委派。
 **原理**：允许父类加载器（如引导类加载器）使用子类加载器（如应用类加载器）来加载类。 
 
 **示例（JDBC）**： 
 java.sql.DriverManager由引导类加载器加载，它需要加载应用类加载器路径下的Driver实现类（如MySQL的Driver）。 DriverManager通过Thread.currentThread().getContextClassLoader()获取线程上下文类加载器（默认是应用类加载器），并用它加载Driver实现类，从而绕过了双亲委派的单向委托。 （2）重写loadClass方法（自定义类加载器） 
 自定义类加载器时，覆盖loadClass方法，改变委托逻辑（如不先委托父类，直接自己加载）。 
 
 **示例代码**： 
 
```java
public class CustomClassLoader extends ClassLoader {
 @Override
 protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
 synchronized (getClassLoadingLock(name)) {
 // 1. 检查是否已加载
 Class<?> c = findLoadedClass(name);
 if (c == null) {
 // 2. 不委托父类，直接自己加载（打破双亲委派）
 try {
 c = findClass(name); // 自定义findClass逻辑（如从网络加载）
 } catch (ClassNotFoundException e) {
 // 3. 自己加载失败时，再委托父类（可选）
 return super.loadClass(name, resolve);
 }
 }
 if (resolve) {
 resolveClass(c);
 }
 return c;
 }
 }

 @Override
 protected Class<?> findClass(String name) throws ClassNotFoundException {
 // 自定义加载逻辑（如从网络下载字节码）
 byte[] classBytes = downloadClassFromNetwork(name);
 return defineClass(name, classBytes, 0, classBytes.length);
 }
}
```
 （3）OSGi的动态模块加载 
 OSGi（动态模块化框架）通过**Bundle类加载器**打破双亲委派，每个模块（Bundle）有独立的类加载器。加载类时： 
 先检查当前Bundle的缓存。 再根据Import-Package委托给导出该包的Bundle的类加载器（而非全局父类加载器）。 最后才委托给父类加载器。
 这种机制支持模块的热部署（动态卸载/加载Bundle）。 
#### 三、总结
 
 双亲委派模型通过层次化委托保证了类的唯一性和核心类的安全，而打破它通常用于解决父类加载器需要子类资源、动态部署等场景。核心手段是重写loadClass方法或使用线程上下文类加载器，但需权衡安全性和复杂度。

## 8. 出现full gc的情况都有哪些？
### 

### 一句话总结

 出现Full GC的常见情况包括：老年代空间不足（对象晋升失败或大对象分配）、显式调用System.gc()、方法区（元空间）内存耗尽、空间分配担保失败（Minor GC前老年代剩余空间不足），以及垃圾回收器自身机制触发（如CMS并发处理失败或G1回收周期阈值达到）。

### 详细解析

 当 JVM 触发 **Full GC（全局垃圾回收）** 时，意味着对整个堆内存（Young、Old 代）以及方法区（Metaspace）进行全面回收。以下是常见的触发 Full GC 的场景及其原理：

---

#### **1. 老年代空间不足**

 - **触发条件**：对象从 Young 代晋升到 Old 代时，Old 代剩余空间不足以容纳这些对象。

 - **典型场景**： **长期存活对象积累**：长期存在的对象（如缓存）逐渐填满 Old 代。

 - **内存泄漏**：对象因代码逻辑错误（如静态集合持有无用对象）无法被回收。

 - **示例**： ```java static List<Object> cache = new ArrayList<>(); public void addToCache() { while (true) { cache.add(new Object()); // 持续向静态集合添加对象，导致 Old 代被填满 } } ```

---

#### **2. 显式调用System.gc()**

 - **触发条件**：代码中直接调用System.gc()或Runtime.getRuntime().gc()。

 - **注意**： System.gc()只是建议 JVM 执行 Full GC，实际是否执行由 JVM 决定。

 - 频繁调用可能导致不必要的性能损耗。

 - **解决方案**：通过 JVM 参数-XX:+DisableExplicitGC禁用显式 GC。

---

#### **3. Metaspace（方法区）空间不足**

 - **触发条件**：加载的类、方法、常量等元数据占满 Metaspace。

 - **常见原因**： 动态生成大量类（如反射、CGLib、动态代理）。

 - 未配置 Metaspace 大小限制，默认值过小。

 - **诊断**：通过 GC 日志中的Metaspace使用量监控。

 - **优化**：增大 Metaspace 空间（-XX:MaxMetaspaceSize=256m）。

---

#### **4. Young 代晋升失败（Promotion Failed）**

 - **触发条件**：Young 代 GC 后存活对象需要晋升到 Old 代，但 Old 代空间不足。

 - **直接后果**：JVM 会先尝试 Full GC 释放 Old 代空间，若仍不足则抛出OutOfMemoryError。

 - **典型场景**： Young 代过小，导致存活对象过早晋升。

 - 存在大量“朝生夕死”的大对象（如大数组），直接进入 Old 代。

---

#### **5. 大对象分配失败**

 - **触发条件**：尝试分配一个大对象（如超大数组），且 Young 代中没有足够的连续空间。

 - **注意**：某些垃圾回收器（如 G1）会优先尝试 Full GC 来整理内存碎片，腾出连续空间。

---

#### **6. 并发模式失败（CMS、G1 特有）**

 - **CMS 回收器**： 在并发标记阶段，Old 代空间被快速填满，导致并发回收失败，触发 Full GC。

 - 可通过-XX:CMSInitiatingOccupancyFraction调整触发 CMS 回收的阈值。

 - **G1 回收器**： 当 Mixed GC（混合回收）速度跟不上对象分配速度时，触发 Full GC。

---

#### **7. 内存碎片化**

 - **触发条件**：Old 代内存碎片过多，无法找到连续空间分配大对象。

 - **解决方案**： 使用-XX:+UseCMSCompactAtFullCollection（CMS 回收器在 Full GC 时压缩内存）。

 - 切换到 G1 回收器（自动处理内存碎片）。

---

#### **8. 如何诊断 Full GC 原因？**

 - **开启 GC 日志**： ```bash -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:gc.log ```

 - **分析日志关键字段**： [Full GC (Allocation Failure)：分配失败触发。

 - [Full GC (Metadata GC Threshold)：Metaspace 不足触发。

 - [Full GC (System.gc())：显式调用触发。

 - **使用工具监控**： **jstat**：实时查看堆内存分布。 ```bash jstat -gcutil <pid> 1000 ```

 - **VisualVM** / **MAT**：分析堆内存快照，定位内存泄漏。

---

#### **9. 优化建议**

 - **调整堆大小**： 合理设置-Xmx（最大堆）和-Xms（初始堆），避免动态扩展。

 - 增大 Old 代比例（如-XX:NewRatio=2，Young:Old=1:2）。

 - **选择合适的垃圾回收器**： 低延迟场景：G1（-XX:+UseG1GC）或 ZGC（-XX:+UseZGC）。

 - 高吞吐场景：Parallel（-XX:+UseParallelGC）。

 - **避免代码问题**： 减少大对象分配，优化缓存策略。

 - 避免内存泄漏（如无效的静态集合引用）。

## 9. 常见的jvm参数都有哪些？
### 

### 一句话总结

 常见JVM参数包括：

内存设置：-Xms（初始堆）、-Xmx（最大堆）、-Xss（线程栈）

垃圾回收器：-XX:+UseG1GC（G1回收器）、-XX:+UseParallelGC（并行回收）

监控调试：-XX:+HeapDumpOnOutOfMemoryError（OOM生成堆转储）

性能优化：-XX:MaxMetaspaceSize（元空间上限）

日志参数：-Xloggc（GC日志路径）

### 详细解析

 以下是常见的 JVM 参数分类整理，覆盖内存管理、垃圾回收、性能优化、诊断监控等场景，结合示例说明其用途：

---

#### **一、内存相关参数**

| 参数 | 说明 | 示例 |
| --- | --- | --- |
| -Xms<size> | 初始堆大小（JVM 启动时分配的堆内存） | -Xms2g（初始堆 2GB） |
| -Xmx<size> | 最大堆大小（堆内存上限） | -Xmx4g（堆最大 4GB） |
| -Xmn<size> | 年轻代（Young Generation）大小 | -Xmn1g（年轻代 1GB） |
| -XX:NewRatio=<n> | 老年代与年轻代的比例（默认值 2，即老年代:年轻代=2:1） | -XX:NewRatio=3（比例 3:1） |
| -XX:SurvivorRatio=<n> | Eden 区与 Survivor 区的比例（默认值 8，即 Eden:S0:S1=8:1:1） | -XX:SurvivorRatio=6（比例 6:1:1） |
| -XX:MetaspaceSize=<size> | 元空间（Metaspace）初始大小（取代 Java 8 之前的-XX:PermSize） | -XX:MetaspaceSize=256m |
| -XX:MaxMetaspaceSize=<size> | 元空间最大大小（默认无限制，但受物理内存限制） | -XX:MaxMetaspaceSize=512m |
| -XX:MaxDirectMemorySize=<size> | 直接内存（NIO 的 Direct Buffer）最大大小 | -XX:MaxDirectMemorySize=1g |

---

#### **二、垃圾回收（GC）相关参数通用参数** 

| 参数 | 说明 |
| --- | --- |
| -XX:+UseSerialGC | 使用 Serial（串行）垃圾回收器（单线程，适合客户端或小内存应用） |
| -XX:+UseParallelGC | 使用 Parallel Scavenge（并行）回收器（吞吐量优先） |
| -XX:+UseConcMarkSweepGC | 使用 CMS（并发标记清除）回收器（低停顿，已废弃，Java 14 后移除） |
| -XX:+UseG1GC | 使用 G1（Garbage-First）回收器（平衡吞吐与延迟，Java 9 后默认） |
| -XX:+UseZGC | 使用 ZGC（低延迟回收器，Java 11+ 实验性，Java 15+ 正式） |
| -XX:+UseShenandoahGC | 使用 Shenandoah 回收器（低延迟，需单独启用） |

 **GC 调优参数** 

| 参数 | 说明 |
| --- | --- |
| -XX:MaxGCPauseMillis=<ms> | 期望的最大 GC 停顿时间（G1/ZGC 等回收器参考此值调整策略） |
| -XX:G1HeapRegionSize=<size> | G1 回收器的 Region 大小（需为 2 的幂，范围 1MB~32MB） |
| -XX:InitiatingHeapOccupancyPercent=<n> | G1 触发并发标记周期的堆占用阈值（百分比） |
| -XX:ParallelGCThreads=<n> | 并行 GC 的线程数（默认与 CPU 核数相关） |
| -XX:ConcGCThreads=<n> | 并发 GC 的线程数（如 CMS、G1 的并发阶段） |

---

#### **三、性能优化参数**

| 参数 | 说明 |
| --- | --- |
| -XX:+TieredCompilation | 启用分层编译（C1 + C2 编译器结合，Java 8 默认开启） |
| -XX:CompileThreshold=<n> | 方法调用次数阈值，触发 JIT 编译（默认 1500） |
| -XX:+UseStringDeduplication | 开启字符串去重（G1 回收器特有，减少重复字符串内存占用） |
| -XX:+AggressiveOpts | 启用激进优化（JVM 内部实验性优化，不同版本效果不同） |
| -XX:ReservedCodeCacheSize=<size> | JIT 编译后的代码缓存区大小 |

---

#### **四、诊断与监控参数**

| 参数 | 说明 |
| --- | --- |
| -XX:+HeapDumpOnOutOfMemoryError | 内存溢出时自动生成堆转储文件（HEAP DUMP） |
| -XX:HeapDumpPath=<path> | 指定堆转储文件路径 |
| -XX:+PrintGCDetails | 打印详细的 GC 日志 |
| -Xloggc:<file> | 将 GC 日志输出到文件 |
| -XX:+PrintFlagsFinal | 打印所有 JVM 参数的最终值（用于确认参数是否生效） |
| -XX:+FlightRecorder | 启用 Java Flight Recorder（JFR，性能分析工具） |
| -XX:StartFlightRecording=<options> | 启动 JFR 记录（需指定参数） |

---

#### 五、参数使用示例

 **典型服务端启动参数** 

```bash
java -Xms4g -Xmx4g \
 -XX:+UseG1GC \
 -XX:MaxGCPauseMillis=200 \
 -XX:MetaspaceSize=256m \
 -XX:MaxMetaspaceSize=512m \
 -XX:+HeapDumpOnOutOfMemoryError \
 -XX:HeapDumpPath=/logs/dump.hprof \
 -Xloggc:/logs/gc.log \
 -jar app.jar
```

## 10. 说一下JDK的监控和 线上处理的一些case。
### 

### 一句话总结

 JDK监控常用工具包括JConsole、VisualVM、JMC等，用于实时查看内存、线程、GC状态。

 线上常见问题处理：内存泄漏通过heap dump分析对象引用链；频繁GC可调整-Xmx/-Xms或更换垃圾回收器；线程死锁用jstack生成线程快照定位；使用Arthas在线诊断方法执行耗时及热更新代码；借助JMX或Prometheus+Grafana实现指标可视化监控。

### 详细解析

 以下是 JDK 监控工具和线上常见问题的处理案例，结合实际场景说明排查思路和解决方法：

---

#### **一、JDK 监控工具概览**

| 工具 | 用途 | 关键命令/操作 |
| --- | --- | --- |
| jps | 查看 Java 进程的 PID 和主类名 | jps -l |
| jstat | 监控堆内存、类加载、GC 统计等 | jstat -gcutil <pid> 1000（每 1 秒输出 GC 统计） |
| jstack | 生成线程快照（Dump 线程栈），分析死锁、线程阻塞等 | jstack <pid> > thread.log |
| jmap | 生成堆内存快照（Heap Dump），分析内存泄漏 | jmap -dump:format=b,file=heap.hprof <pid> |
| jinfo | 查看或修改 JVM 参数 | jinfo -flags <pid> |
| jcmd | 多功能命令（生成堆转储、查看线程栈、触发 GC 等） | jcmd <pid> GC.run（触发 Full GC） |
| JConsole | 图形化监控堆内存、线程、类、MBean 等 | 通过jconsole启动 |
| VisualVM | 功能更强大的图形化工具（支持堆转储分析、线程分析、抽样器等） | 通过jvisualvm启动 |
| Java Flight Recorder (JFR) | 低开销的性能分析工具（记录 CPU、内存、IO 等事件） | jcmd <pid> JFR.start duration=60s filename=recording.jfr |

---

#### **二、线上常见问题处理案例Case 1：CPU 使用率飙升** 

 - **现象**：服务器 CPU 持续 100%，应用响应变慢。

 - **排查步骤**： **定位高 CPU 进程**： ```bash top -c # 查看进程 CPU 占用（找到 Java 进程 PID） ```

 - **定位高 CPU 线程**： ```bash top -H -p <pid> # 查看该进程下的线程 CPU 占用，记录线程 ID（十进制） ```

 - **转换线程 ID 为十六进制**： ```bash printf "%x\n" <线程ID> # 得到十六进制值（如 0x2a3b） ```

 - **分析线程栈**： ```bash jstack <pid> > thread.log grep -A 20 'nid=0x2a3b' thread.log # 查看该线程的栈信息 ```

 - **常见原因**： **死循环**：代码中存在无限循环（如while(true)未休眠或退出条件错误）。

 - **锁竞争**：线程频繁争抢锁（如synchronized块内执行耗时操作）。

 - **频繁 GC**：Full GC 导致 CPU 飙升（需结合 GC 日志分析）。

 - **解决**： 优化代码逻辑（如避免死循环、减少锁粒度）。

 - 使用异步处理或线程池控制并发。

---

 **Case 2：内存泄漏（OOM）** 

 - **现象**：频繁 Full GC，最终抛出OutOfMemoryError: Java heap space。

 - **排查步骤**： **查看 GC 情况**： ```bash jstat -gcutil <pid> 1000 # 观察 Old 代占用是否持续增长 ```

 - **生成堆转储文件**： ```bash jmap -dump:live,format=b,file=heap.hprof <pid> # 注意：live 参数会触发 Full GC ```

 - **使用 MAT 分析堆转储**： 打开heap.hprof，查找占用内存最大的对象。

 - 查看对象引用链，定位未释放的集合或缓存（如静态HashMap持续添加元素）。

 - **常见原因**： **静态集合未清理**：如全局缓存未设置过期策略。

 - **资源未关闭**：数据库连接、文件流未释放。

 - **第三方库内存泄漏**：如某些框架未正确释放资源。

 - **解决**： 修复代码，及时释放无用对象。

 - 使用弱引用（WeakReference）或限制缓存大小。

---

 **Case 3：线程死锁** 

 - **现象**：应用无响应，但 CPU 和内存使用正常。

 - **排查步骤**： **生成线程快照**： ```bash jstack <pid> > thread.log ```

 - **查找死锁标记**： ```bash grep -i 'deadlock' thread.log # 查看是否有 "Found one Java-level deadlock" ```

 - **分析死锁线程栈**： 查看线程持有的锁和等待的锁，找到循环等待的锁链。

 - **示例**： ```java // 线程 1 持有锁 A，等待锁 B synchronized (A) { synchronized (B) { ... } // 线程 1 在此阻塞 } // 线程 2 持有锁 B，等待锁 A synchronized (B) { synchronized (A) { ... } // 线程 2 在此阻塞 } ```

 - **解决**： 调整锁顺序，保证所有线程按相同顺序获取锁。

 - 使用超时锁（如ReentrantLock.tryLock()）。

---

 **Case 4：Metaspace 溢出** 

 - **现象**：抛出OutOfMemoryError: Metaspace。

 - **排查步骤**： **查看 Metaspace 使用情况**： ```bash jstat -gcutil <pid> 1000 # 关注 M（Metaspace）列 ```

 - **分析类加载情况**： ```bash jcmd <pid> VM.classloader_stats # 查看加载的类数量 ```

 - **生成堆转储（可选）**： ```bash jmap -dump:format=b,file=meta.hprof <pid> ```

 - **常见原因**： **动态生成类**：如大量使用 CGLib、反射生成代理类。

 - **重复加载类**：类加载器未释放（如 OSGi 环境或热部署工具）。

 - **解决**： 增大 Metaspace 大小（-XX:MaxMetaspaceSize=512m）。

 - 优化代码，避免重复生成类。

---

 **Case 5：频繁 Full GC** 

 - **现象**：GC 日志中频繁出现Full GC，应用停顿明显。

 - **排查步骤**： **查看 GC 原因**： ```bash -XX:+PrintGCDetails -XX:+PrintGCDateStamps # 在 GC 日志中查看触发原因 ```

 - **常见触发场景**： **晋升失败**：Young 代存活对象过多，Old 代空间不足。

 - **内存碎片**：CMS 回收器未开启内存压缩。

 - **优化方向**： 增大 Old 代空间（调整-Xmx或-XX:NewRatio）。

 - 更换为 G1 回收器（自动处理碎片）。

 - 启用 CMS 内存压缩（-XX:+UseCMSCompactAtFullCollection）。
