# Java基础-牛客面经八股

> 来源：牛客网  |  共 23 题

## 1. 请你说说Java基本数据类型和引用类型
### 

### 一句话总结
 Java基本数据类型包括8种：byte、short、int、long（整型），float、double（浮点型），char（字符型），boolean（布尔型），直接存储数值于栈内存。引用类型（如类、数组、接口）存储对象内存地址，实际对象存储在堆内存，默认值为null。两者核心区别在于存储方式和内存分配机制。
 
### 详细解析
 
 Java的数据类型分为**基本数据类型（Primitive Data Types）**和**引用类型（Reference Types）**，两者在存储方式、行为特性和使用场景上有显著区别。 
 
---
 
#### **一、基本数据类型**
 
 Java有 **8种基本数据类型**，直接存储具体的数值，内存分配在栈（Stack）中，速度快但功能简单。 
 
| 类型 | 大小（字节） | 默认值 | 取值范围/描述 | 示例 |
| --- | --- | --- | --- | --- |
| byte | 1 | 0 | -128 ~ 127 | byte b = 100; |
| short | 2 | 0 | -32768 ~ 32767 | short s = 2000; |
| int | 4 | 0 | -2^31 ~ 2^31-1 | int i = 100000; |
| long | 8 | 0L | -2^63 ~ 2^63-1 | long l = 100L; |
| float | 4 | 0.0f | 单精度浮点数（6-7位有效数字） | float f = 3.14f; |
| double | 8 | 0.0d | 双精度浮点数（15位有效数字） | double d = 3.14; |
| char | 2 | '\u0000' | Unicode字符（0 ~ 65535） | char c = 'A'; |
| boolean | 1位 | false | true 或 false | boolean flag = true; |
 
 **特点**： 
 
 直接存储值，无需实例化。 
 
 作为类成员变量时会被赋予默认值，局部变量必须显式初始化。 
 
 传递时是**值传递**（复制一份值）。 
 
---
 
#### **二、引用类型**
 引用类型存储的是对象在堆（Heap）内存中的地址，而不是实际数据。所有非基本数据类型的对象均属于引用类型，包括： 类对象（如 String、自定义类） 接口 数组 枚举 **示例**： 
```java
String str = "Hello"; // String 是引用类型 
int[] arr = {1, 2, 3}; // 数组是引用类型 
Person p = new Person();// 自定义类的对象
```
 **特点**： 
 默认值为 null（未指向任何对象）。 
 
 传递时是**引用传递**（传递内存地址的副本，共享同一对象）。 
 
 内存分配在堆中，由垃圾回收器（GC）管理，可能引发内存泄漏或性能问题。 
 
 支持面向对象特性（如继承、多态）。 
 
---
 
#### **三、核心区别**
 
| 特性 | 基本数据类型 | 引用类型 |
| --- | --- | --- |
| 存储内容 | 直接存储值 | 存储对象的内存地址 |
| 内存位置 | 栈 | 栈存引用，对象在堆 |
| 默认值 | 有（如 0、false 等） | null |
| 赋值方式 | 直接复制值 | 复制引用（共享同一对象） |
| 性能 | 高效（无额外内存开销） | 较低（涉及堆分配和GC） |
| 用途 | 简单数据存储 | 复杂对象、面向对象编程 |

## 10. 请你说一下抽象类和接口的区别
### 

### 一句话总结
 抽象类用于继承关系，可包含部分方法实现和成员变量，单继承；接口定义行为规范，默认方法均为抽象（Java8后支持默认方法），允许多实现。抽象类强调"是什么"，接口强调"能做什么"。成员变量上抽象类可含普通变量，接口变量默认为常量。
 
### 详细解析
 
 在 Java 中，**接口（Interface）**和**抽象类（Abstract Class）**都是实现多态和代码复用的核心机制，但它们的设计目的和使用场景有显著差异。以下是它们的共同点和区别的详细对比： 
 
---
 
#### **一、核心区别**
 
| 特性 | 接口（Interface） | 抽象类（Abstract Class） |
| --- | --- | --- |
| 定义关键字 | interface | abstract class |
| 方法实现 | Java 8+ 支持default和static方法（有方法体） | 可以包含抽象方法和具体方法（有方法体） |
| 继承机制 | 支持多继承（一个类可实现多个接口） | 单继承（一个类只能继承一个抽象类） |
| 成员变量 | 只能是public static final（常量） | 可以是任意类型（实例变量、静态变量等） |
| 构造方法 | 无构造方法 | 可以有构造方法（用于初始化抽象类状态） |
| 访问修饰符 | 方法默认public（不可用private、protected） | 方法可以用public、protected等修饰符 |
| 代码复用能力 | 只能通过default方法复用代码（Java 8+） | 可以直接通过继承复用具体方法和属性 |
| 使用场景 | 定义跨继承体系的能力（如Comparable、Runnable） | 为同一继承体系中的子类提供通用实现（如模板方法） |
 
---
 
#### **二、代码示例1. 接口（Interface）** 
```java
public interface Animal {
 // 常量（默认 public static final）
 String TYPE = "Animal";

 // 抽象方法（默认 public abstract）
 void eat();

 // 默认方法（Java 8+）
 default void breathe() {
 System.out.println("呼吸氧气");
 }

 // 静态方法（Java 8+）
 static boolean isAnimal(Object obj) {
 return obj instanceof Animal;
 }
}
```
 **2. 抽象类（Abstract Class）** 
```java
public abstract class Bird {
 // 实例变量
 protected String name;

 // 构造方法
 public Bird(String name) {
 this.name = name;
 }

 // 抽象方法
 public abstract void fly();

 // 具体方法
 public void sing() {
 System.out.println(name + "在唱歌");
 }
}
```
 
---
 
#### **三、使用场景1. 优先使用接口的情况定义跨类的能力**：如Runnable（线程执行）、Serializable（序列化）。 **需要多继承**：一个类需要实现多个不相关的行为。 **定义轻量级契约**：不需要复用代码，只规范行为。 **解耦和扩展**：通过接口实现模块化设计（如策略模式）。 
 **示例**： 
 
```java
public class Sparrow extends Bird implements Flyable, Singable {
 // 实现接口的多个行为
}
```
 **2. 优先使用抽象类的情况复用代码模板**：多个子类有共同的方法或属性（如模板方法模式）。 **部分方法需要默认实现**：子类仅需实现差异部分。 **定义同一继承体系的共性**：如动物分类中的通用行为。 
 **示例**： 
 
```java
public abstract class AbstractList<E> {
 // 提供通用方法（如 addAll、isEmpty）
 public abstract void add(E element);

 public boolean isEmpty() {
 return size() == 0;
 }
}
```
 
---
 
#### **四、Java 8+ 对接口的增强**
 
 自 Java 8 起，接口可以通过default和static方法提供部分实现，但仍与抽象类有以下区别： 
 **default方法**：主要用于接口的向后兼容，避免破坏已有实现类。 **static方法**：提供工具方法，不依赖实例（如Comparator.comparing()）。 **接口仍不能有状态**：无法定义实例变量（只能有常量）。

## 12. 请你说一下final关键字
### 

### 一句话总结
 final关键字用于修饰类、方法或变量： **类**：不可被继承（如String类）； **方法**：不可被子类重写； **变量**：基本类型值不可变，引用类型地址不可变（对象内部状态可修改）。
 static final组合可定义常量。 
### 详细解析
 
 在 Java 中，final关键字用于表示“不可变”的语义，可以修饰变量、方法、类等不同对象，具体作用取决于其修饰的目标。以下是final关键字的详细说明： 
 
---
 
#### 一、修饰变量（常量）
 
 **基本类型变量**
 final修饰的基本类型变量一旦被赋值后，**值不可再修改**（常量）。 
 
```java
final int MAX_VALUE = 100;
// MAX_VALUE = 200; // 编译错误，无法重新赋值
```
 
 **引用类型变量**
 final修饰的引用变量**不能指向其他对象**，但对象内部的状态（属性）可能可以修改。 
 
```java
final StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // 允许修改对象内容
// sb = new StringBuilder(); // 编译错误，不能指向新对象
```
 
 **空白 final 变量**
 可以在声明时不初始化，但**必须在构造器或初始化块中赋值一次**。 
 
```java
class Example {
 final int x; // 空白 final
 public Example() {
 x = 10; // 必须在构造器中赋值
 }
}
```
 
---
 
#### 二、修饰方法
 
 **禁止子类重写**
 用final修饰的方法不能被子类覆盖（override），常用于保证方法逻辑不被篡改。 
 
```java
class Parent {
 public final void show() {
 System.out.println("Parent");
 }
}

class Child extends Parent {
 // public void show() { ... } // 编译错误，无法重写 final 方法
}
```
 
 **效率优化（历史原因）**
 早期 JVM 会对final方法进行内联优化（inline），但现代 JVM 已能自动优化，此用途不再重要。 
 
---
 
#### 三、修饰类
 
 **禁止继承**
 final类不能被其他类继承，常用于设计不可变类（如String、Integer）。 
 
```java
final class ImmutableClass {
 // 类内容
}

// class SubClass extends ImmutableClass { ... } // 编译错误
```
 
---
 
#### 四、其他场景
 
 **方法参数**
 方法参数被final修饰后，在方法内部**不能修改其值**（基本类型）或**引用**（引用类型）。 
 
```java
public void process(final int num) {
 // num = 10; // 编译错误
}
```
 
 **匿名内部类访问外部变量**
 匿名内部类访问的局部变量必须是final或等效 final（Java 8+ 隐式 final）。 
 
```java
void outerMethod() {
 final int localVar = 10;
 new Thread(() -> {
 System.out.println(localVar); // 必须为 final 或等效 final
 }).start();
}
```

## 13. 介绍下Java中的static关键字。静态方法能不能调用非静态成员?
### 

### 一句话总结
 static关键字用于修饰类级别的成员，独立于实例存在。静态方法不能直接调用非静态成员（变量/方法），因为非静态成员依赖对象实例，而静态方法在类加载时即可调用，此时实例可能不存在。需通过对象实例间接访问非静态成员。
 
### 详细解析
 
 在 Java 中，static关键字用于修饰类的成员（变量、方法、代码块、嵌套类），表示这些成员属于类本身，而非类的某个实例。以下是关键点说明： 
 
---
 
#### **static 关键字的作用静态变量（类变量）** 
 所有实例共享同一份静态变量，内存中只有一份拷贝。 通过类名.变量名直接访问，无需创建对象。 
```java
class MyClass {
 static int count = 0; // 静态变量
}
```
 
 **静态方法** 
 属于类的方法，可通过类名.方法名()调用。 常用于工具类（如Math.sqrt()）或不需要实例状态的操作。 
```java
class MathUtils {
 static int add(int a, int b) { return a + b; }
}
```
 
 **静态代码块** 
 在类加载时执行一次，用于初始化静态变量。 
```java
class MyClass {
 static { System.out.println("类已加载"); }
}
```
 
 **静态嵌套类** 
 静态内部类不依赖外部类的实例。 
```java
class Outer {
 static class Inner {} // 静态嵌套类
}
```
 
---
 
#### **静态方法能否调用非静态成员？直接调用：不能。间接调用：可以（通过对象实例）。间接调用的方式：**若在静态方法中创建实例或传入实例参数，则可通过实例访问非静态成员： 
```java
class Example {
 int nonStaticVar = 10;

 static void staticMethod() {
 Example obj = new Example(); // 创建实例
 System.out.println(obj.nonStaticVar); // 合法
 }
}
```

## 11. String、StringBuffer、Stringbuilder有什么区别？
### 

### 一句话总结
 String不可变，每次修改生成新对象；StringBuffer和StringBuilder可变。StringBuffer线程安全但性能较低，StringBuilder非线程安全但效率更高。单线程用StringBuilder，多线程用StringBuffer，少量操作用String。
 
### 详细解析
 
 在 Java 中，**String**、**StringBuffer** 和 **StringBuilder** 是处理字符串的核心类，它们的核心区别在于**可变性**、**线程安全性**和**性能**。以下是详细对比及使用场景建议： 
 
---
 
#### **一、核心对比表**
 
| 特性 | String | StringBuffer | StringBuilder |
| --- | --- | --- | --- |
| 可变性 | 不可变（Immutable） | 可变（Mutable） | 可变（Mutable） |
| 线程安全 | 线程安全（天然不可变） | 线程安全（方法用synchronized修饰） | 非线程安全 |
| 性能 | 低（频繁修改产生大量对象） | 中（同步开销） | 高（无同步开销） |
| 使用场景 | 常量字符串、少量拼接 | 多线程环境下的字符串操作 | 单线程环境下的字符串操作 |
| 内存效率 | 低（频繁操作时） | 较高 | 最高 |
 
---
 
#### **二、详细解析1.String（不可变字符串）特点**： 
 字符串内容一旦创建**不可修改**，任何修改操作（如concat、substring）都会生成新对象。 线程安全（因为不可变）。 
 **示例**： 
 
```java
String s1 = "Hello";
String s2 = s1.concat(" World"); // 生成新对象
System.out.println(s1); // Hello（原对象未变）
System.out.println(s2); // Hello World
```
 
 **适用场景**： 
 字符串常量（如配置信息）。 少量字符串拼接（如String result = "A" + "B";，JVM 会优化为StringBuilder）。 
---
 **2.StringBuffer（可变字符串，线程安全）特点**： 
 内部通过char[]动态扩容，直接修改原对象。 所有方法用synchronized修饰，保证线程安全，但性能较低。 
 **示例**： 
 
```java
StringBuffer buffer = new StringBuffer("Hello");
buffer.append(" World"); // 直接修改原对象
System.out.println(buffer); // Hello World
```
 
 **适用场景**： 
 **多线程环境**下的字符串操作（如并发日志拼接）。 需要线程安全的字符串修改。 
---
 **3.StringBuilder（可变字符串，非线程安全）特点**： 
 与StringBuffer功能相同，但**无同步开销**，性能更高。 **非线程安全**，适用于单线程环境。 
 **示例**： 
 
```java
StringBuilder builder = new StringBuilder("Hello");
builder.append(" World"); // 直接修改原对象
System.out.println(builder); // Hello World
```
 
 **适用场景**： 
 **单线程环境**下的高频字符串操作（如循环中拼接 SQL、JSON）。 优先选择的字符串修改类（除非需要线程安全）。 
---
 
#### **三、性能对比1. 高频字符串拼接测试** 
```java
// 测试代码
public class PerformanceTest {
 public static void main(String[] args) {
 int loopCount = 100000;

 // String 测试
 long start1 = System.currentTimeMillis();
 String s = "";
 for (int i = 0; i < loopCount; i++) {
 s += i;
 }
 long end1 = System.currentTimeMillis();

 // StringBuffer 测试
 long start2 = System.currentTimeMillis();
 StringBuffer buffer = new StringBuffer();
 for (int i = 0; i < loopCount; i++) {
 buffer.append(i);
 }
 long end2 = System.currentTimeMillis();

 // StringBuilder 测试
 long start3 = System.currentTimeMillis();
 StringBuilder builder = new StringBuilder();
 for (int i = 0; i < loopCount; i++) {
 builder.append(i);
 }
 long end3 = System.currentTimeMillis();

 System.out.println("String 耗时: " + (end1 - start1) + "ms");
 System.out.println("StringBuffer 耗时: " + (end2 - start2) + "ms");
 System.out.println("StringBuilder 耗时: " + (end3 - start3) + "ms");
 }
}
```
 **2. 结果分析String**：耗时最长（频繁创建新对象）。 **StringBuffer**：耗时中等（同步开销）。 **StringBuilder**：耗时最短（无同步开销）。

## 4. 请你说说==与equals()的区别
### 

### 一句话总结
 ==比较对象内存地址是否相同，适用于基本数据类型和对象引用比较。
 equals()用于对象内容比较，默认实现等同于==，但常被重写（如String类）。
 String等包装类重写equals()后比较实际值而非地址。
 基本类型只能用==比较数值，对象类型需根据需求选择比较方式。
 正确使用时应确保equals()方法被目标类正确实现。
 
### 详细解析
 
 在 Java 中，**==** 和 **equals()** 是两种不同的比较操作，它们的核心区别在于**比较的维度和适用场景**。以下是详细对比和解释： 
 
---
 
#### **一、核心区别总结**
 
| 特性 | == | equals() |
| --- | --- | --- |
| 比较内容 | 基本类型比较值，引用类型比较内存地址 | 默认比较内存地址，但可重写为对象内容比较 |
| 操作符/方法 | 操作符 | Object类方法 |
| 默认行为 | 不可修改 | Object类默认用==，需重写实现逻辑相等 |
| 适用场景 | 基本类型值比较、判断引用是否指向同一对象 | 对象内容逻辑相等比较 |
 
---
 
#### **二、==的用法**
 1. **基本类型比较** 
 直接比较**值**是否相等： 
 
```java
int a = 10;
int b = 10;
System.out.println(a == b); // true（值相等）

char c1 = 'A';
char c2 = 'B';
System.out.println(c1 == c2); // false（值不等）
```
 2. **引用类型比较** 
 比较两个引用是否指向**同一个对象**（内存地址是否相同）： 
 
```java
String s1 = new String("Java");
String s2 = new String("Java");
String s3 = s1;

System.out.println(s1 == s2); // false（不同对象）
System.out.println(s1 == s3); // true（同一对象）
```
 
---
 
#### **三、equals()的用法**
 1. **默认行为** 
 Object类中的equals()方法默认使用==比较内存地址： 
 
```java
public boolean equals(Object obj) {
 return (this == obj);
}
```
 2. **重写后的行为** 
 通过重写equals()，实现**对象内容逻辑相等**的比较。
 **示例**：String类重写了equals()，比较字符串内容： 
 
```java
String s1 = new String("Java");
String s2 = new String("Java");
System.out.println(s1.equals(s2)); // true（内容相同）
```
 3. **正确重写规则一致性**：若a.equals(b)为true，则b.equals(a)必须为true。 **传递性**：若a.equals(b)和b.equals(c)为true，则a.equals(c)必须为true。 **非空性**：a.equals(null)必须返回false。 **与hashCode()同步**：若两个对象equals()为true，它们的hashCode()必须相同。 
 **重写示例**： 
 
```java
public class Person {
 private String name;
 private int age;

 @Override
 public boolean equals(Object o) {
 if (this == o) return true; // 同一对象直接返回 true
 if (o == null || getClass() != o.getClass()) return false; // 类型不同返回 false
 Person person = (Person) o; // 类型转换
 return age == person.age && Objects.equals(name, person.name); // 比较属性
 }

 @Override
 public int hashCode() {
 return Objects.hash(name, age); // 生成哈希码
 }
}
```
 
---
 
#### **四、总结**
 
| 场景 | 使用== | 使用equals() |
| --- | --- | --- |
| 基本类型比较 | ✅（比较值） | ❌（不能用于基本类型） |
| 引用类型是否为同一对象 | ✅（判断内存地址） | ❌（除非未重写equals()） |
| 对象内容是否逻辑相等 | ❌ | ✅（需正确重写equals()） |
 
 **核心原则**： 
 **基本类型**（int,char等）用==比较值。 **引用类型**： 若需判断是否为同一对象 →==。 若需判断内容是否相等 → 重写equals()，并确保同步重写hashCode()。 **包装类**和**字符串**优先使用equals()比较内容。

## 5. 请你说说hashCode()和equals()的区别,为什么重写equals()就要重写hashCode()
### 

### 一句话总结
 hashCode()用于计算对象的哈希值，主要用于哈希表快速定位；equals()用于判断对象内容是否相等。两者用途不同，但需遵循一致性原则：若两个对象equals()相等，则hashCode()必须相同。若仅重写equals()会导致在哈希集合（如HashMap）中出现逻辑矛盾（如相同对象存在不同哈希桶），破坏哈希表正常运作。因此必须同时重写以保证哈希结构正确性。
 
### 详细解析
 
 在 Java 中，重写equals()时必须重写hashCode()方法，是因为 **Java 对象契约（Object Contract）** 规定了这两个方法的行为必须一致。如果违反这一规则，会导致对象在使用哈希表（如HashMap、HashSet）时出现逻辑错误，破坏数据结构的正确性。以下是详细解释： 
 
---
 
#### **一、Java 对象契约的核心规则**
 
 根据 Java 官方规范，若两个对象通过equals()方法判定为相等，则它们的hashCode()**必须返回相同的值**。反之则不一定成立（哈希值相同的对象不一定equals为true）。
 **规则公式**： 
 
```cpp
a.equals(b) == true --> a.hashCode() == b.hashCode()
```
 
---
 
#### **二、为什么必须同时重写？**
 1. **哈希表依赖hashCode()** 
 哈希表（如HashMap、HashSet）的工作机制基于哈希码（hashCode）来定位存储位置（桶）。 
 **存储时**：根据hashCode计算对象的存储位置。 **查找时**：先通过hashCode快速缩小范围，再用equals()精确比较。 
 **示例**：
 若两个对象equals为true，但hashCode不同，它们会被存储到不同的桶中，导致哈希表无法正确工作： 
 
```java
Person p1 = new Person("Alice", 25);
Person p2 = new Person("Alice", 25);

// 假设只重写 equals()，未重写 hashCode()
System.out.println(p1.equals(p2)); // true（内容相同）
System.out.println(p1.hashCode() == p2.hashCode()); // false（默认 hashCode 不同）

// 存入 HashMap
Map<Person, String> map = new HashMap<>();
map.put(p1, "Alice");
map.get(p2); // 返回 null（因 p2 的 hashCode 不同，无法找到正确桶）
```
 2. **违反契约的后果哈希集合行为异常**：HashSet可能包含重复元素，HashMap可能无法正确检索值。 **程序逻辑错误**：依赖哈希表的功能（如缓存、索引）会出现不可预知的错误。 
---
 
#### **三、如何正确重写hashCode()？**
 1. **实现示例** 
 使用Objects.hash()工具类简化实现： 
 
```java
public class Person {
 private String name;
 private int age;

 @Override
 public boolean equals(Object o) {
 if (this == o) return true;
 if (o == null || getClass() != o.getClass()) return false;
 Person person = (Person) o;
 return age == person.age && Objects.equals(name, person.name);
 }

 @Override
 public int hashCode() {
 return Objects.hash(name, age); // 基于 name 和 age 生成哈希码
 }
}
```
 2. **IDE 自动生成** 
 现在很多的IDE（如 IntelliJ、Eclipse）支持自动生成equals()和hashCode()，确保两者一致性。 
 
---
 
#### **四、常见问题**
 1. **如果只重写equals()不重写hashCode()会怎样？** 违反对象契约，导致哈希集合无法正确工作。 **示例**：两个相等的对象被存入HashSet，会被误判为不同对象，导致重复存储。 2. **如何验证hashCode()的正确性？** 使用单元测试框架（如 JUnit）验证equals()和hashCode()的一致性： 
```java
@Test
void testEqualsAndHashCode() {
 Person p1 = new Person("Alice", 25);
 Person p2 = new Person("Alice", 25);
 assertTrue(p1.equals(p2));
 assertEquals(p1.hashCode(), p2.hashCode());
}
```
 3. **是否所有对象都需要重写hashCode()？** 仅当对象会被用于哈希表（如HashMap的键）时，才需要重写。 若对象仅用于普通集合（如ArrayList），可不重写（但建议始终同步重写）。 
---
 
####

## 3. 介绍一下包装类的自动拆箱与自动装箱。
### 

### 一句话总结
 自动装箱是基本数据类型自动转换为对应的包装类对象（如int→Integer），自动拆箱则是包装类对象转为基本类型（如Integer→int）。该机制由编译器在编译阶段实现，简化代码书写，例如集合存储基本类型时会自动装箱。需注意频繁拆装箱可能影响性能，且拆箱时若对象为null会抛出空指针异常。
 
### 详细解析
 
 Java的**自动装箱（Autoboxing）**和**自动拆箱（Unboxing）**是编译器提供的语法糖，用于简化基本类型与对应包装类型之间的转换。它们使得代码更简洁，但需注意潜在的性能问题和陷阱。 
 
---
 
#### **一、核心概念自动装箱（Autoboxing）**
 将**基本类型**自动转换为对应的**包装类对象**。 
 
```java
Integer num = 100; // 基本类型int → 包装类型Integer
// 等价于：Integer num = Integer.valueOf(100);
```
 
 **自动拆箱（Unboxing）**
 将**包装类对象**自动转换为对应的**基本类型**。 
 
```java
int n = num; // 包装类型Integer → 基本类型int
// 等价于：int n = num.intValue();
```
 
#### **二、实现原理**
 1. 自动装箱：调用valueOf() 
 编译器将装箱操作替换为调用包装类的静态方法valueOf()： 
 
```java
Integer num = 100; 
// 编译后：Integer num = Integer.valueOf(100);
```
 
 **缓存优化**：部分包装类（如Integer、Long）对特定范围值（如-128~127）缓存对象，避免重复创建。 
 
```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true（复用缓存对象）

Integer c = 200;
Integer d = 200;
System.out.println(c == d); // false（新建对象）
```
 2. 自动拆箱：调用xxxValue() 
 编译器将拆箱操作替换为调用实例方法xxxValue()： 
 
```java
int n = num; 
// 编译后：int n = num.intValue();
```
 
---
 
#### **三、注意事项与陷阱**
 1. **性能问题频繁装箱**：大量循环中自动装箱会生成临时对象，增加垃圾回收（GC）压力。 
```java
// 低效写法（每次循环都装箱）
Long sum = 0L;
for (int i = 0; i < 10000; i++) {
 sum += i; // sum自动拆箱为long，计算后自动装箱为Long
}
```
 2. **空指针异常（NullPointerException）** 
 包装类型为null时，自动拆箱会抛出异常： 
 
```java
Integer num = null;
int n = num; // 运行时抛出 NullPointerException
```
 3. **比较操作** ==比较包装类型时，实际比较的是对象地址（而非值）： 
```java
Integer a = 200;
Integer b = 200;
System.out.println(a == b); // false（超出缓存范围）
System.out.println(a.equals(b)); // true（比较值）
```

## 6. 请你说说Java中重载和重写的区别。
### 

### 一句话总结
 重载（Overload）是在同一类中方法名相同但参数列表不同（类型、数量、顺序），与返回类型和访问修饰符无关，属于编译时多态。重写（Override）是子类覆盖父类方法，要求方法名、参数、返回类型相同，访问权限不能更严格且异常范围不能扩大，属于运行时多态。
 
### 详细解析
 
 在 Java 中，**重载（Overloading）**和**重写（Overriding）**是两种不同的多态实现方式，它们的核心区别在于方法的作用范围、参数规则、绑定时机及设计目的。以下是详细对比： 
 
#### **一、核心对比表**
 
| 特性 | 重载（Overloading） | 重写（Overriding） |
| --- | --- | --- |
| 作用范围 | 同一类中（或父子类之间） | 父子类之间（子类重写父类方法） |
| 方法签名 | 方法名相同，参数列表不同（类型/顺序/数量） | 方法名、参数列表、返回类型必须相同 |
| 返回类型 | 可以不同 | 必须相同（或为父类返回类型的子类） |
| 访问权限 | 可以不同（无限制） | 子类方法不能比父类更严格（如父类public，子类不能是private） |
| 异常处理 | 可以抛出任何异常 | 不能抛出比父类更宽泛的检查异常（或不抛出） |
| 绑定时机 | 编译时多态（静态绑定） | 运行时多态（动态绑定） |
| 目的 | 提供同一功能的多种实现方式 | 子类定制父类行为，实现多态 |
 
#### **二、重载（Overloading）**
 1. **定义** 
 在同一类中定义多个**同名方法**，但**参数列表不同**（类型、数量、顺序不同）。 
 2. **规则参数必须不同**：仅返回类型不同不足以构成重载（会编译报错）。 **与访问修饰符无关**：可以是public、private等任意修饰符。 **可抛出不同异常**：无限制。 3. **示例** 
```java
public class Calculator {
 // 重载方法1：两个int参数
 public int add(int a, int b) {
 return a + b;
 }

 // 重载方法2：三个int参数
 public int add(int a, int b, int c) {
 return a + b + c;
 }

 // 重载方法3：两个double参数
 public double add(double a, double b) {
 return a + b;
 }
}
```
 
#### **三、重写（Overriding）**
 1. **定义** 
 子类重新定义父类的方法，**方法签名必须完全相同**，用于实现多态。 
 2. **规则方法签名一致**：方法名、参数列表、返回类型（或子类）必须相同。 **访问权限不能更严格**：例如父类是public，子类不能是protected。 **异常限制**：子类方法抛出的检查异常不能比父类更宽泛。 **使用@Override注解**：显式声明重写，编译器会检查是否符合规则。 3. **示例** 
```java
class Animal {
 public void makeSound() {
 System.out.println("动物叫声");
 }
}

class Dog extends Animal {
 @Override
 public void makeSound() { // 重写父类方法
 System.out.println("汪汪汪");
 }
}
```

## 7. 介绍一下Java的泛型。
### 

### 一句话总结
 Java泛型是一种类型参数化机制，允许在类、接口和方法中定义类型占位符（如<T>）。它通过编译时类型检查确保数据安全，避免了运行时的强制类型转换和ClassCastException。泛型提高了代码重用性，支持集合框架的类型安全操作，并通过类型擦除机制实现向后兼容。
 
### 详细解析
 
 Java 的**泛型（Generics）**是一种在编译时提供类型安全性的机制，允许开发者定义类、接口或方法时使用**类型参数**，从而增强代码复用性、可读性和安全性。以下是泛型的核心概念和应用： 
 
---
 
#### **一、为什么需要泛型？类型安全**：避免运行时类型转换错误（如ClassCastException）。 **代码复用**：编写通用逻辑，支持多种数据类型。 **消除强制类型转换**：减少冗余的类型转换代码。 **增强可读性**：明确代码操作的数据类型。 
 **示例**：未使用泛型的集合可能引发类型错误： 
 
```java
List list = new ArrayList();
list.add("hello");
list.add(100); // 编译通过，但运行时可能出错
String str = (String) list.get(1); // 抛出 ClassCastException
```
 
 使用泛型后： 
 
```java
List<String> list = new ArrayList<>();
list.add("hello");
// list.add(100); // 编译时报错
String str = list.get(0); // 无需强制转换
```
 
#### **二、泛型的核心机制**
 1. **泛型类（Generic Class）** 
 通过类型参数T定义通用类： 
 
```java
public class Box<T> {
 private T content;

 public void setContent(T content) {
 this.content = content;
 }

 public T getContent() {
 return content;
 }
}

// 使用示例
Box<String> stringBox = new Box<>();
stringBox.setContent("Java");
String value = stringBox.getContent(); // 无需类型转换
```
 2. **泛型方法（Generic Method）** 
 在方法中定义类型参数： 
 
```java
public <T> void printArray(T[] array) {
 for (T element : array) {
 System.out.print(element + " ");
 }
}

// 调用示例
Integer[] intArray = {1, 2, 3};
printArray(intArray); // 自动推断类型为 Integer
```
 3. **泛型接口（Generic Interface）** 
 定义通用接口： 
 
```java
public interface Comparator<T> {
 int compare(T o1, T o2);
}

// 实现示例
public class StringComparator implements Comparator<String> {
 @Override
 public int compare(String s1, String s2) {
 return s1.length() - s2.length();
 }
}
```
 
#### **三、类型通配符（Wildcards）**
 
 用于增强泛型的灵活性，支持未知类型或范围限制。 
 
| 通配符类型 | 说明 | 示例 |
| --- | --- | --- |
| <?> | 未知类型（任意类型） | List<?> list = new ArrayList<>(); |
| <? extends T> | 上界通配符（接受T及其子类） | List<? extends Number> numbers; |
| <? super T> | 下界通配符（接受T及其父类） | List<? super Integer> integers; |
 
 **示例**： 
 
```java
// 上界通配符：读取数据
public static double sum(List<? extends Number> list) {
 double sum = 0;
 for (Number num : list) {
 sum += num.doubleValue();
 }
 return sum;
}

// 下界通配符：写入数据
public static void addNumbers(List<? super Integer> list) {
 list.add(10);
 list.add(20);
}
```

## 14. 请说说你对反射的了解。
### 

### 一句话总结
 Java反射是程序在运行时检查、修改自身结构和行为的能力。通过获取类型信息可动态创建对象、调用方法或访问属性，常用于框架开发与序列化等场景。
 
### 详细解析
 
 Java反射（Reflection）是Java语言提供的一种**动态机制**，允许程序在运行时获取类的信息、操作类的属性和方法，甚至调用私有成员。这种能力使得Java代码可以突破编译时的限制，实现高度灵活的动态编程。 
 
---
 
#### **一、反射的核心概念核心类Class<T>**：表示一个类或接口，是反射的入口。 **Constructor<T>**：表示类的构造方法。 **Method**：表示类的方法。 **Field**：表示类的成员变量。 
---
 
#### **二、反射的使用步骤1. 获取Class对象** 
 反射的起点是获取类的Class对象，常见方式： 
 
```java
// 方式1：通过类名.class
Class<?> clazz1 = String.class;

// 方式2：通过对象.getClass()
String str = "Hello";
Class<?> clazz2 = str.getClass();

// 方式3：通过Class.forName("全限定类名")
Class<?> clazz3 = Class.forName("java.lang.String");
```
 **2. 创建对象实例** 
 通过Constructor创建实例，可调用私有构造方法： 
 
```java
Class<?> clazz = User.class;
// 获取无参构造方法
Constructor<?> constructor = clazz.getDeclaredConstructor();
constructor.setAccessible(true); // 解除私有限制
Object user = constructor.newInstance();

// 获取带参构造方法
Constructor<?> paramConstructor = clazz.getDeclaredConstructor(String.class, int.class);
Object user2 = paramConstructor.newInstance("Alice", 25);
```
 **3. 调用方法** 
 动态调用方法（包括私有方法）： 
 
```java
Method method = clazz.getDeclaredMethod("setName", String.class);
method.setAccessible(true); // 解除私有限制
method.invoke(user, "Bob"); // 调用方法
```
 **4. 访问字段** 
 读写字段值（包括私有字段）： 
 
```java
Field field = clazz.getDeclaredField("age");
field.setAccessible(true);
field.set(user, 30); // 设置值
int age = (int) field.get(user); // 获取值
```
 
---
 
#### **三、反射的应用场景举例框架开发Spring**：通过反射创建Bean、依赖注入。 **Hibernate/MyBatis**：动态映射数据库字段到Java对象。 **JUnit**：通过反射执行测试方法。 
 **动态代理**
 JDK动态代理（Proxy）基于反射生成代理对象，实现AOP。 
 
 **注解处理**
 结合反射解析自定义注解（如@RequestMapping）。 
 
 **泛型擦除后操作**
 通过反射获取泛型实际类型（如List<String>中的String）。

## 20. 你知道哪些线程安全的集合？举例你是怎么使用的？
### 

### 一句话总结
 常见线程安全的集合包括ConcurrentHashMap、CopyOnWriteArrayList和BlockingQueue系列。例如在多线程统计时使用ConcurrentHashMap的compute方法保证原子计数，用CopyOnWriteArrayList维护监听器列表避免遍历时加锁，通过LinkedBlockingQueue实现生产者-消费者任务队列。Java并发包中的集合通过分段锁或写时复制机制实现高效线程安全。
 
### 详细解析
 
 **线程安全集合**是指在多线程环境下能够保证数据一致性和操作正确性的集合类。它们通过内部同步机制（如锁、CAS操作等）确保多个线程同时访问或修改集合时不会引发竞态条件、数据损坏等问题。以下是一些常见的线程安全集合及其示例： 
 
---
 
#### **1. Java中的线程安全集合示例(1)ConcurrentHashMap（推荐使用）特点**：键值对存储，支持高并发读写。 **实现原理**：分段锁（JDK7）或CAS+节点锁（JDK8+），不同段或节点可并行操作。 **适用场景**：高并发缓存、计数器等。 **示例**： 
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("key", 1); // 线程安全的写操作
int value = map.get("key"); // 线程安全的读操作
```
 **(2)CopyOnWriteArrayList特点**：读操作无锁，写操作复制新数组。 **实现原理**：写操作时复制整个底层数组，保证读操作始终访问旧数组。 **适用场景**：读多写少的场景（如监听器列表）。 **示例**： 
```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("item"); // 写操作触发数组复制
String item = list.get(0); // 读操作无需加锁
```
 **(3)BlockingQueue接口的实现类常见实现**：ArrayBlockingQueue、LinkedBlockingQueue。 **特点**：支持阻塞式插入（put）和移除（take）。 **适用场景**：生产者-消费者模型。 **示例**： 
```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
queue.put(100); // 队列满时阻塞
int num = queue.take(); // 队列空时阻塞
```
 **(4)ConcurrentLinkedQueue特点**：非阻塞线程安全队列，基于CAS实现。 **适用场景**：高并发环境下的任务队列。 **示例**： 
```java
ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
queue.offer("task1"); // 非阻塞添加
String task = queue.poll(); // 非阻塞取出
```
 **(5)Collections.synchronizedXXX()（传统方式）示例**：通过工具类包装非线程安全集合。 **缺点**：使用全局锁，性能较差。 
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
```
 
---
 
#### **3. 线程安全集合 vs 非线程安全集合**
 
| 集合类型 | 线程安全实现 | 典型非线程安全集合 |
| --- | --- | --- |
| Map | ConcurrentHashMap | HashMap |
| List | CopyOnWriteArrayList | ArrayList |
| Queue | BlockingQueue实现类 | LinkedList |
| Set | ConcurrentSkipListSet | HashSet |

## 19. 请你说说HashMap底层原理和扩容机制。
### 

### 一句话总结
 HashMap底层采用数组+链表/红黑树结构，通过哈希算法确定元素存储位置。默认初始容量16，负载因子0.75，当元素数量超过（容量×负载因子）时触发扩容。扩容时创建双倍容量新数组，通过高位运算重新计算节点位置（JDK8优化为无需重新hash），原数据通过尾插法迁移到新数组。链表长度超过8且数组长度≥64时会转为红黑树，提升查询效率。
 
### 详细解析
 
#### **一、底层数据结构：数组 + 链表（JDK7） → 数组 + 链表/红黑树（JDK8）**
 
 HashMap 的底层是一个 **哈希表（Hash Table）**，本质是一个动态扩容的数组（称为table），数组的每个元素是一个 **链表**（或 **红黑树**，JDK8 及以后），用于存储哈希冲突的键值对。 
 1. 核心结构术语 **桶（Bucket）**：数组的每个槽位（table[i]）称为一个桶，用于存放哈希值相同的键值对。 **哈希冲突（Hash Collision）**：不同键通过哈希函数计算出相同的桶下标，导致多个键值对需要存放在同一个桶中。 **链表（Entry 或 Node 节点）**：JDK7 中桶内元素以链表形式存储（节点类型为Entry<K,V>）；JDK8 中改为Node<K,V>，当链表长度超过阈值时转换为红黑树。 **红黑树（TreeNode）**：JDK8 引入，当链表长度 ≥8 且数组长度 ≥64 时，链表转换为红黑树（节点类型为TreeNode<K,V>），以将查找时间复杂度从 O(n) 优化到 O(logn)。 
#### **二、核心机制：哈希计算、冲突解决、扩容**
 1. 哈希计算与桶定位 
 HashMap 通过以下步骤确定键值对的存储位置： 
 
 **步骤 1：计算键的哈希值**
 键的哈希值通过key.hashCode()方法获取，但 HashMap 会对其进行二次哈希（hash()方法），目的是 **减少哈希冲突**。
 JDK7 的hash()方法通过多次位运算（如异或、右移）扩散哈希值；
 JDK8 简化为：(h = key.hashCode()) ^ (h >>> 16)（高 16 位与低 16 位异或），让高位参与低位计算，减少低位冲突。 
 
 **步骤 2：确定桶下标** 
 桶下标通过(n - 1) & hash计算（n是数组长度，必须是 2 的幂次）。该计算等价于hash % n，但位运算更高效。数组长度为 2 的幂次是为了保证(n-1) & hash的结果均匀分布。若n非 2 的幂次，&运算会导致某些桶永远无法被访问。 

 比如n=10，那么n-1=9，二进制为1001。(n-1) & hash 等价于取 hash 的二进制与 1001 的按位与，结果的二进制只能是以下四种可能（因为 1001 只有第 0 位和第 3 位是 1）：0000（0）、0001（1）、1000（8）、1001（9）。因此，无论 hash 是什么值，最终的桶下标只能是 0、1、8、9 这四个值。 2. 哈希冲突解决：链地址法 
 当不同键的哈希值映射到同一个桶时，HashMap 使用 **链地址法** 解决冲突：将冲突的键值对以链表形式挂在同一个桶下。
 JDK8 之前链表的插入方式是 **头插法**（新节点插入链表头部），JDK8 改为 **尾插法**（新节点插入链表尾部），避免扩容时的死循环问题（下文详述）。 
 3. 扩容机制：动态调整数组大小 
 HashMap 通过 **扩容（resize）** 保持负载因子（Load Factor）在合理范围，避免哈希冲突过多导致性能下降。 
 
 **触发条件**：当元素数量（size）超过容量（capacity） × 负载因子（loadFactor）时触发扩容。
 默认容量为 16，负载因子为 0.75（空间与时间的权衡：负载因子过小会导致频繁扩容；过大则哈希冲突概率增加）。 
 
 **扩容过程**： 
 **创建新数组**：容量翻倍（newCap = oldCap << 1），新数组长度仍为 2 的幂次。 **迁移元素**：将旧数组中的所有键值对重新分配到新数组的桶中（JDK7 需重新计算哈希值，JDK8 优化为通过hash & oldCap判断是否需要移动）。 
 **JDK8 扩容优化**： 
 由于数组长度是 2 的幂次，旧数组的桶下标为i，新数组长度为2×oldCap，新桶下标只能是i或i + oldCap（通过hash & oldCap是否为 0 判断）。因此，无需重新计算哈希值，只需判断hash的最高位（相对于旧容量的位置）是否为 0，即可确定新位置，大幅提升扩容效率，可以看下图容量从16扩充为32的resize示意图加深理解。 
 
#### **三、JDK7 与 JDK8 的核心差异**
 
| 特性 | JDK7 实现 | JDK8 实现 |
| --- | --- | --- |
| 底层结构 | 数组 + 链表（Entry 节点） | 数组 + 链表/红黑树（Node/TreeNode 节点） |
| 冲突解决 | 链表（长度无限制，查找 O(n)） | 链表长度 ≥8 且数组长度 ≥64 时转为红黑树（查找 O(logn)） |
| 插入方式 | 头插法（新节点插入链表头部） | 尾插法（新节点插入链表尾部） |
| 扩容时哈希计算 | 重新计算哈希值（hash()方法） | 通过hash & oldCap判断是否需要移动（无需重新计算哈希） |
| 死循环问题 | 多线程扩容时可能导致链表成环（死循环） | 尾插法避免了死循环，但仍存在数据覆盖问题（线程不安全本质未变） |
 
#### **四、线程不安全的原因**
 
 HashMap 是 **线程不安全** 的，多线程环境下可能导致以下问题： 
 **数据覆盖（最常见）**：多个线程同时执行put操作时，若哈希冲突导致两个线程同时修改同一个桶的链表，可能出现后写入的数据覆盖先写入的数据。 **扩容死循环（JDK7）**：JDK7 采用头插法扩容，多线程迁移元素时可能导致链表成环（如线程 A 和 B 同时迁移同一链表，指针互相引用），后续查询时陷入死循环。 **数据丢失**：多线程扩容时，可能丢失部分键值对（如两个线程同时计算出相同的新桶位置，导致其中一个线程的数据被覆盖）。

## 23. ConcurrentHashMap 线程安全的具体实现方式。
### 

### 一句话总结
 ConcurrentHashMap通过分段锁+CAS机制实现线程安全：1.JDK7采用Segment分段锁，不同段可并发操作；2.JDK8改为Node数组+链表/红黑树，使用synchronized锁单个桶节点；3.volatile保证变量可见性；4.CAS实现无锁化原子操作；5.扩容时多线程协同迁移数据。
 
### 详细解析
 
#### 一、JDK 1.7 与 JDK 1.8 的核心差异
 
| 维度 | JDK 1.7 | JDK 1.8 |
| --- | --- | --- |
| 底层结构 | 数组 + 链表 + 分段锁（Segment） | 数组 + 链表/红黑树 + CAS + synchronized |
| 锁粒度 | 段级锁（每个 Segment 独立加锁） | 节点级锁（仅锁定链表头或红黑树根节点） |
| 并发性能 | 中等（支持多段并发） | 高（细粒度锁 + 无锁读） |
| 扩容机制 | 全量扩容（单线程完成） | 并发扩容（多线程协作） |
 
---
 
#### **二、JDK 1.7 的实现原理（可略过）分段锁（Segment）机制**
 1.1 数据结构： 
 
 • 顶层是Segment[]数组（默认 16 段），每个Segment继承自ReentrantLock。 
 
 • 每个Segment内部维护一个HashEntry[]数组（类似 HashMap 的链表结构）。 
 
 1.2 线程安全 
 
 • 写操作时，仅锁定当前操作的Segment（通过ReentrantLock实现）。 
 
 • 读操作无锁（HashEntry的value和next字段用volatile修饰，保证可见性）。 
 
 1.3 缺点： 
 
 • 扩容时需全量重建所有Segment，性能较差。 
 • 默认 16 段可能浪费内存（若实际并发度低）。 
 ** 2. 核心方法（以put为例）** 
```java
// 伪代码：加锁后操作 HashEntry 链表
void put(K key, V value) {
 int hash = hash(key);
 Segment segment = segmentFor(hash); // 根据 hash 定位 Segment
 segment.lock(); // 锁定当前段
 try {
 HashEntry<K,V>[] tab = segment.table;
 int index = hash & (tab.length - 1);
 HashEntry<K,V> e = tab[index];
 while (e != null) {
 if (e.hash == hash && e.key.equals(key)) {
 e.value = value; // 覆盖旧值
 return;
 }
 e = e.next;
 }
 tab[index] = new HashEntry<>(hash, key, value, e); // 插入新节点
 } finally {
 segment.unlock(); // 释放锁
 }
}
```
 
---
 
#### **三、JDK 1.8 的实现原理无锁化设计 + 细粒度锁**
 1.1 数据结构： 
 
 • 数组Node<K,V>[]存储链表或红黑树头节点。 
 
 • 链表长度 > 8 且数组长度 > 64 时，链表转换为红黑树（提升查询效率）。 
 
 1.2 线程安全： 
 
 • CAS 操作：插入节点时，通过compareAndSet保证原子性。 
 
 • synchronized 锁：仅在链表/红黑树头节点加锁，减少锁竞争。 
 
 1.3 优点： 
 
 • 读操作完全无锁（volatile保证可见性）。 
 • 并发扩容时，多线程协作迁移数据。 
 ** 2.核心方法（以putVal为例）** 
```java
// 伪代码：CAS + synchronized 实现线程安全
final V putVal(K key, V value) {
 if (table == null) {
 if (casTabAt(table, 0, null, initTable())) // CAS 初始化数组
 return null;
 }
 int hash = spread(key.hashCode());
 Node<K,V>[] tab = table;
 int n = tab.length, i = (n - 1) & hash;
 Node<K,V> f = tabAt(tab, i);
 if (f == null) {
 if (casTabAt(tab, i, null, new Node<>(hash, key, value))) // CAS 插入空节点
 return null;
 } else {
 synchronized (f) { // 锁定头节点
 // 遍历链表/红黑树，更新或插入节点
 // 判断是否需要转换为红黑树
 }
 }
 return null;
}
```
 
 ** 3. CAS 操作的关键作用**
 • 初始化数组：initTable()通过 CAS 确保单线程初始化。 
 
 • 插入节点：casTabAt()尝试无锁插入，避免加锁开销。 
 
 • 扩容标记：transferIndex的 CAS 操作控制多线程协作扩容。 
 
#### 四、与其他线程安全 Map 的对比
 
| 实现类 | 线程安全机制 | 读性能 | 写性能 | 适用场景 |
| --- | --- | --- | --- | --- |
| ConcurrentHashMap | CAS + 细粒度锁 | 极高 | 高 | 高并发读写 |
| Hashtable | 全局锁（synchronized） | 低 | 低 | 低并发场景（已淘汰） |
| Collections.synchronizedMap | 方法级同步锁 | 中等 | 中等 | 简单同步需求 |
| CopyOnWrite系列 | 写时复制 | 极高 | 极低 | 读多写少（如配置缓存） |

####

## 18. 请你说说ArrayList和LinkedList的区别。
### 

### 一句话总结
 ArrayList基于动态数组实现，随机访问快（O(1)），但增删元素需移动数据（O(n)）；LinkedList基于双向链表实现，增删元素快（O(1)），但随机访问需遍历（O(n)）。ArrayList内存连续但可能预留空间，LinkedList每个节点含前后指针更占内存。线程均不安全，适用场景取决于读写操作比例。
 
### 详细解析
 
 一、底层数据结构 
 
| 维度 | ArrayList | LinkedList |
| --- | --- | --- |
| 实现原理 | 基于动态数组（连续内存） | 基于双向链表（非连续内存） |
| 内存布局 | 元素连续存储，通过索引直接访问 | 每个节点存储元素及前后节点的引用 |
| 扩容机制 | 容量不足时扩容（默认1.5倍） | 无扩容操作，直接新增节点 |
 
---
 
 二、性能对比 
 
| 操作类型 | ArrayList | LinkedList |
| --- | --- | --- |
| 随机访问 | O(1)（直接通过索引计算内存地址） | O(n)（需从头/尾遍历链表） |
| 插入/删除 | - 尾部操作：O(1) - 中间/头部：O(n) | - 尾部/头部：O(1) - 中间：O(n)（需遍历定位） |
| 内存占用 | 较低（仅存储元素） | 较高（每个节点额外存储前后指针） |
| 缓存友好性 | 高（连续内存提升CPU缓存命中率） | 低（节点分散导致缓存未命中） |
 
---
 
 三、核心方法差异 
 
| 功能 | ArrayList | LinkedList |
| --- | --- | --- |
| 快速访问 | get(int index)、set(int index, E element) | 需遍历实现，无原生优化方法 |
| 头尾操作 | 无原生方法 | addFirst(E e)、addLast(E e)、removeFirst()、removeLast() |
| 队列操作 | 不支持 | 实现Deque接口，支持队列/栈操作 |
| 批量操作 | addAll(int index, Collection<? extends E> c) | 需遍历插入，效率较低 |
 
---
 
 四、适用场景 
 
| 场景特征 | 推荐选择 | 原因 |
| --- | --- | --- |
| 频繁随机访问（如按索引查询） | ArrayList | 直接内存寻址，时间复杂度O(1) |
| 头尾频繁插入/删除（如队列） | LinkedList | 头尾操作O(1)，避免ArrayList的元素迁移开销 |
| 内存敏感型应用 | ArrayList | 节省指针存储空间（LinkedList每个节点多24字节） |
| 中间位置高频增删 | LinkedList | 避免ArrayList的O(n)元素移动 |
| 实现栈/双端队列 | LinkedList | 天然支持push、pop、peek等操作 |

## 2. 介绍一下基本类型和包装类型的区别？
### 

### 一句话总结

 基本类型（如int、char）直接存储数据值，占用固定内存，默认值不为null；包装类型（如Integer、Character）是对象，存储在堆中，默认值为null。包装类型支持泛型、提供方法（如转换、比较），且可通过自动装箱/拆箱与基本类型互换，但性能开销更大。两者适用场景不同，基本类型高效，包装类型用于对象化需求。

### 详细解析

 Java 中的**基本类型（Primitive Types）**和**包装类型（Wrapper Classes）**是两种不同的数据类型，主要区别体现在内存管理、默认值、功能特性以及使用场景上。以下是它们的核心差异：

#### **一、本质区别**

| 特性 | 基本类型 | 包装类型 |
| --- | --- | --- |
| 类型 | 原始数据类型（如 int、char） | 对象类型（如 Integer、Character） |
| 存储方式 | 直接存储值（在栈内存） | 存储对象引用（堆内存中的对象） |
| 默认值 | 有默认值（如 0、false） | 默认值为 null |
| 内存占用 | 内存占用固定且小（高效） | 内存占用较大（对象头等额外开销） |
| 比较方式 | == 直接比较值是否相等 | == 比较对象地址，equals() 比较值 |
| 是否可为 null | 不可为 null | 可为 null |

#### **二、核心差异详解**

 1. **内存与性能** 

 - **基本类型**：直接存储值，无需堆内存分配，效率高。 

```java
int a = 100; // 直接存储在栈中
```

 **包装类型**：是对象，需要堆内存分配，依赖垃圾回收（GC），性能略低。

```java
Integer b = 100; // 存储在堆中，栈中存储引用
```

 2. **自动装箱与拆箱（Autoboxing & Unboxing）** 

 Java 支持基本类型与包装类型的自动转换，但可能引发性能问题：

 - **自动装箱**：基本类型 → 包装类型（如 int → Integer）。 

```java
Integer num = 42; // 等价于 Integer.valueOf(42)
```

 **自动拆箱**：包装类型 → 基本类型（如 Integer → int）。

```java
int n = num; // 等价于 num.intValue() 
```

 3. **缓存机制（以 Integer 为例）** 

 包装类型对部分值做了缓存优化（如 -128 ~ 127），直接复用对象：

```java
Integer a = 127; 
Integer b = 127; 
System.out.println(a == b); // true（缓存复用对象） 
Integer c = 200; 
Integer d = 200; 
System.out.println(c == d); // false（超出缓存范围，新建对象）
```

4. 空值风险 

 包装类型可为 null，使用时需避免空指针异常（NullPointerException）：

```java
Integer num = null; 
int n = num; // 运行时抛出 NullPointerException（拆箱时）
```

#### **三、使用场景**

| 场景 | 推荐类型 | 原因 |
| --- | --- | --- |
| 高频计算（如循环） | 基本类型 | 性能高，无对象开销 |
| 泛型或集合（如 List） | 包装类型 | 泛型不支持基本类型（如 List<Integer>） |
| 数据库映射（可为空） | 包装类型 | 字段可能为 null |
| 对象化操作（如反射） | 包装类型 | 需要对象特性（如方法调用） |

## 8. Java的Object类有哪些方法？
### 

### 一句话总结

 Java的Object类包含以下方法：

getClass(), hashCode(), equals(), toString(), clone(),

finalize(), notify(), notifyAll(), wait()（含三个重载版本）。

### 详细解析

 在 Java 中，**Object类是所有类的根父类**，所有对象（包括数组）都隐式继承自Object类。它定义了 11 个方法（不同 JDK 版本可能有差异）。以下是这些方法的作用及使用场景：

---

#### **一、核心方法列表**

| 方法 | 作用 | 常见场景 |
| --- | --- | --- |
| toString() | 返回对象的字符串表示形式 | 调试、日志输出 |
| equals(Object obj) | 判断两个对象是否“逻辑相等” | 自定义对象内容比较 |
| hashCode() | 返回对象的哈希码（用于哈希表存储） | 集合类（如HashMap、HashSet） |
| getClass() | 返回对象的运行时类（Class对象） | 反射、类型检查 |
| clone() | 创建并返回对象的副本（浅拷贝） | 对象复制 |
| finalize() | 对象被垃圾回收前调用（已废弃） | 资源清理（不推荐使用） |
| wait()、wait(long timeout)、wait(long timeout, int nanos) | 让当前线程进入等待状态（需在同步块中使用） | 线程间通信（生产者-消费者模型） |
| notify()、notifyAll() | 唤醒等待该对象锁的线程（需在同步块中使用） | 线程间通信 |
| registerNatives() | 本地方法，用于注册本地方法实现（由 JVM 内部使用） | 很少用，无需关注 |

---

#### **二、一些重要方法**

 - **equals()与hashCode()必须同步重写**：否则在使用哈希表时会导致逻辑错误。

 - **clone()是浅拷贝**：若对象包含引用类型字段，需手动实现深拷贝。

 - **线程方法需在同步块中使用**：wait()、notify()必须在synchronized块中调用。

## 9. 说一下Java中的深拷贝、浅拷贝、引用拷贝。
### 

### 一句话总结

 - **引用拷贝**：复制对象引用地址，新旧变量指向同一对象。

 - **浅拷贝**：创建新对象，复制基本类型值，引用类型仍指向原对象。

 - **深拷贝**：完全复制对象及关联的所有子对象，新旧对象完全独立。

 - **实现方式**：浅拷贝常用clone()方法（需重写），深拷贝需递归复制或序列化实现。

 - **核心区别**：深拷贝隔离数据修改，浅拷贝和引用拷贝存在数据关联性。

### 详细解析

 在 Java 中，**深拷贝（Deep Copy）**、**浅拷贝（Shallow Copy）** 和 **引用拷贝（Reference Copy）** 是对象复制的三种不同方式，它们的核心区别在于对对象及其内部数据的复制深度。以下是它们的详细对比和实现方式：

---

#### **一、核心概念对比**

| 类型 | 定义 | 特点 | 示例 |
| --- | --- | --- | --- |
| 引用拷贝 | 两个变量指向同一个堆内存对象（本质是引用的赋值） | 修改任意变量会影响另一个变量 | Person p1 = new Person();
Person p2 = p1; |
| 浅拷贝 | 创建一个新对象，并复制原对象的字段值（基本类型复制值，引用类型复制地址） | 新对象与原对象共享内部引用类型字段的实例 | Person p2 = p1.clone();
（未深拷贝内部的Address对象） |
| 深拷贝 | 创建一个新对象，并递归复制所有内部引用对象的字段值 | 新对象与原对象完全独立，不共享任何引用类型字段的实例 | Person p2 = p1.deepClone();
（内部Address对象也被复制） |

---

#### **二、具体实现与示例1. 引用拷贝（Reference Copy）** 

 - **本质**：多个引用指向同一对象，没有创建新对象。

 - **代码示例**： ```java Person p1 = new Person("Alice", new Address("北京")); Person p2 = p1; // 引用拷贝 p2.setName("Bob"); System.out.println(p1.getName()); // 输出 "Bob"（共享同一对象） ```

 **2. 浅拷贝（Shallow Copy）** 

 - **实现方式**：实现Cloneable接口，重写clone()方法。

 - **示例**： ```java class Person implements Cloneable { private String name; private Address address; @Override public Person clone() throws CloneNotSupportedException { return (Person) super.clone(); // 默认浅拷贝（address 字段共享） } } // 测试 Person p1 = new Person("Alice", new Address("北京")); Person p2 = p1.clone(); p2.getAddress().setCity("上海"); System.out.println(p1.getAddress().getCity()); // 输出 "上海"（共享 address 对象） ```

 **3. 深拷贝（Deep Copy）** 

 - **实现方式**：递归复制所有引用类型字段。

 - **实现示例**： ```java class Address implements Cloneable { private String city; @Override public Address clone() throws CloneNotSupportedException { return (Address) super.clone(); } } class Person implements Cloneable { private String name; private Address address; @Override public Person clone() throws CloneNotSupportedException { Person cloned = (Person) super.clone(); cloned.address = this.address.clone(); // 深拷贝 address 对象 return cloned; } } // 测试 Person p1 = new Person("Alice", new Address("北京")); Person p2 = p1.clone(); p2.getAddress().setCity("上海"); System.out.println(p1.getAddress().getCity()); // 输出 "北京"（address 对象独立） ```

---

#### 三、关键区别总结

| 特性 | 引用拷贝 | 浅拷贝 | 深拷贝 |
| --- | --- | --- | --- |
| 对象独立性 | 完全共享同一对象 | 对象独立，但内部引用共享 | 对象及内部引用均独立 |
| 内存开销 | 无额外开销 | 仅复制外层对象 | 递归复制所有引用对象 |
| 实现复杂度 | 直接赋值即可 | 需实现Cloneable接口 | 需递归处理所有引用类型字段 |
| 典型应用场景 | 临时引用共享 | 对象结构简单（无嵌套可变引用） | 对象结构复杂（需完全独立） |

## 15. JDK动态代理是什么？和cglib区别？
### 

### 一句话总结

 JDK动态代理是Java基于接口实现的代理技术，通过反射机制在运行时生成代理类。与CGLib的区别：1. JDK需实现接口，CGLib通过继承直接代理普通类；2. JDK无需第三方库，CGLib需引入ASM包；3. CGLib无法代理final类/方法，JDK无法代理无接口类；4. CGLib生成代理类性能通常更高。

### 详细解析

 JDK动态代理和CGLIB动态代理是Java中两种主流的动态代理技术，它们在实现机制、适用场景和性能表现上有显著差异。以下是两者的核心区别及原理分析：

---

#### **一、JDK动态代理定义与原理** 

 JDK动态代理是基于**接口**的代理技术，通过反射机制在运行时动态生成实现目标接口的代理类。其核心依赖于java.lang.reflect.Proxy类和InvocationHandler接口：

 - **代理对象生成**：通过Proxy.newProxyInstance()方法生成代理对象，该对象会实现目标类所有接口。

 - **方法调用**：代理对象的方法调用会被转发到InvocationHandler.invoke()方法中，开发者可在此处添加增强逻辑（如日志、事务等）。

 **特点** 

 - **依赖接口**：目标类必须实现至少一个接口，否则无法生成代理。

 - **性能**：早期版本因反射调用性能较低，但现代JDK（如JDK8+）通过反射优化显著提升了效率，尤其在单次或少量调用时性能优于CGLIB。

 - **无额外依赖**：基于JDK原生API，无需引入第三方库。

---

#### **二、CGLIB动态代理定义与原理** 

 CGLIB（Code Generation Library）是基于**继承**的代理技术，通过操作字节码生成目标类的子类来代理未实现接口的类。其核心使用Enhancer类和MethodInterceptor接口：

 - **代理对象生成**：通过Enhancer创建目标类的子类，覆盖父类方法。

 - **方法调用**：通过MethodInterceptor.intercept()方法拦截父类方法调用，实现增强逻辑。

 **特点** 

 - **不依赖接口**：可直接代理普通类，适用于无接口的场景。

 - **性能**：生成的代理类直接调用目标方法（非反射），在大量调用时性能更优，但生成代理对象的耗时较长。

 - **限制**：无法代理final类或final方法（因继承机制）。

 - **依赖第三方库**：需引入CGLIB库。

---

#### **三、核心区别对比**

| 对比维度 | JDK动态代理 | CGLIB动态代理 |
| --- | --- | --- |
| 代理对象 | 必须基于接口 | 可代理普通类（无需接口） |
| 实现机制 | 反射生成接口的匿名实现类 | 字节码技术生成目标类的子类 |
| 性能 | 单次调用快（JDK8+优化后） | 大量调用快（直接操作字节码） |
| 依赖 | 无需第三方库 | 需CGLIB依赖 |
| 限制 | 无法代理无接口的类 | 无法代理final类或方法 |
| Spring框架默认选择 | Spring默认优先使用（若目标类有接口） | Spring在目标类无接口时使用，开发者也可强制使用CGLIB（通过配置proxy-target-class=true）。 |

## 16. 什么是序列化?什么是反序列化?
### 

### 一句话总结

 序列化是将对象转换为可存储或传输的格式（如字节流、JSON），便于保存或网络传输。反序列化是逆向过程，将序列化后的数据恢复为原始对象结构，实现数据的重构和使用。两者常用于数据持久化、缓存或跨平台通信场景。

### 详细解析

#### 

 **一、核心区别** 

| 维度 | 序列化 | 反序列化 |
| --- | --- | --- |
| 方向性 | 对象 → 字节流/文本 | 字节流/文本 → 对象 |
| 功能 | 数据持久化或传输 | 数据恢复与对象重建 |
| 依赖关系 | 需定义序列化格式（如JSON Schema） | 需与序列化格式匹配 |

 **二、技术实现与协议** 

 - Java序列化 • 实现Serializable接口，使用ObjectOutputStream/ObjectInputStream。 • 缺点：生成二进制流不可读，跨语言兼容性差。

 - 文本协议 • JSON/XML：可读性强，适合Web API（如RESTful接口）。 • YAML：结构化配置文件常用（如Spring Boot的application.yml）。

 - 二进制协议 • Protocol Buffers（Protobuf）：Google开发，高效紧凑，适合RPC通信。 • MessagePack：混合文本与二进制，性能优于JSON。

 - 框架支持 • Jackson/Gson：Java中JSON序列化库。 • Python Pickle：Python原生序列化工具（注意安全风险）。

---

 **三、应用场景** 

 - 持久化存储 • 数据库存储对象（如Hibernate将实体类序列化为BLOB）。 • 文件缓存（如Redis存储序列化后的Java对象）。

 - 网络通信 • RPC框架（如gRPC使用Protobuf传输数据）。 • 分布式系统（如微服务间传递DTO对象）。

 - 会话管理 • Web服务器（如Tomcat通过序列化Session实现集群会话共享）。

 - 跨语言交互 • Java与Python通过JSON交换数据。

## 17. 常见序列化协议有哪些？
### 

### 一句话总结

 常见序列化协议包括JSON（轻量级文本格式）、XML（可扩展标记语言）、Protocol Buffers（高效二进制协议）、MessagePack（紧凑二进制格式）、Apache Avro（支持动态模式）。其他如Thrift、BSON等也广泛应用。

### 详细解析

#### 一、文本协议（可读性强，适合调试）

 - JSON（JavaScript Object Notation） • 特点：轻量级、键值对结构、人类可读、跨语言支持广泛。 • 优点：易于调试，广泛用于Web API（如RESTful接口）。 • 缺点：序列化后体积较大，性能较低（解析速度慢于二进制协议）。 • 适用场景：配置文件、Web服务交互、实时数据传输（如Ajax请求）。

 - XML（eXtensible Markup Language） • 特点：标记语言、支持复杂数据结构、自描述性强。 • 优点：结构化清晰，支持Schema定义（如XSD）。 • 缺点：冗余字符多（如标签），解析速度慢，体积大。 • 适用场景：企业级配置（如Spring XML）、文档存储。

---

 **二、二进制协议（高性能，体积小）** 

 - Protocol Buffers（Protobuf） • 特点：由Google开发，基于.proto文件定义Schema，生成代码。 • 优点：体积小（比JSON/XML节省30%-70%空间）、解析速度快（100倍于XML）。 • 缺点：需预定义Schema，灵活性较低。 • 适用场景：高性能RPC（如微服务通信）、跨语言数据存储。

 - Thrift • 特点：由Facebook开发，支持IDL定义数据结构，支持多种传输协议。 • 优点：跨语言兼容（Java/Python/C++等）、支持压缩、高效网络传输。 • 缺点：调试困难（二进制不可读），生态不如Protobuf成熟。 • 适用场景：分布式系统RPC、需要灵活数据结构的场景。

 - MessagePack • 特点：二进制格式，类似JSON但更紧凑。 • 优点：体积小（比JSON少30%-50%）、解析速度快。 • 缺点：无内置类型描述，需额外Schema管理。 • 适用场景：实时通信（如游戏数据传输）、移动端数据存储。

---

 **三、语言专用协议** 

 - Java原生序列化（JDK Serialization） • 特点：实现Serializable接口，自动生成序列化ID。 • 优点：无需额外依赖，直接支持Java对象。 • 缺点：性能差、跨语言不兼容、存在安全风险（反序列化漏洞）。 • 适用场景：Java内部持久化、临时数据缓存。

 - Kryo • 特点：Java高性能序列化框架，基于ASM字节码生成。 • 优点：序列化速度极快，体积小。 • 缺点：需注册类，跨语言支持有限。 • 适用场景：Java高性能RPC、缓存序列化。

 - Hessian • 特点：自描述二进制协议，支持动态类型。 • 优点：跨语言兼容（Java/Python/C#等）、无需预定义Schema。 • 缺点：调试复杂，字段顺序敏感。 • 适用场景：Java与异构系统交互、快速原型开发。

---

 **四、其他协议** 

 - BSON（Binary JSON） • 特点：二进制格式的JSON扩展，支持嵌套结构。 • 优点：与JSON兼容、支持复杂查询（如MongoDB）。 • 缺点：体积略大于JSON，解析速度一般。 • 适用场景：MongoDB数据存储、需要JSON兼容的场景。

 - FlatBuffers • 特点：Google开发，零拷贝解析（直接访问内存）。 • 优点：极致性能（无需解析直接读取）、内存占用极低。 • 缺点：Schema定义复杂，生态较小众。 • 适用场景：游戏开发、高频数据读取场景。

---

 **五、选型建议** 

| 需求场景 | 推荐协议 |
| --- | --- |
| 跨语言、高性能RPC | Protobuf、Thrift |
| 配置文件、Web API | JSON、YAML |
| 大数据存储、动态模式 | Avro、Parquet（列式存储） |
| Java内部序列化 | Kryo、Protostuff |
| 移动端/实时通信 | MessagePack、FlatBuffers |
| 调试友好、快速开发 | JSON、Hessian |

## 22. 什么是 BlockingQueue？实现类有哪些？
### 

### 一句话总结

 BlockingQueue是Java中支持线程安全阻塞操作的队列接口，用于协调生产者和消费者线程。常见实现包括：ArrayBlockingQueue（固定容量数组）、LinkedBlockingQueue（可选容量链表）、SynchronousQueue（直接传递队列）、PriorityBlockingQueue（优先级排序）和DelayQueue（延迟元素处理）。

### 详细解析

 **BlockingQueue** 是 Java 并发包（java.util.concurrent）中定义的一个**线程安全队列接口**，专为多线程环境设计。其核心特点是**支持阻塞操作**：当队列为空时，消费者线程尝试获取元素会被阻塞，直到队列中有数据；当队列已满时，生产者线程尝试添加元素也会被阻塞，直到队列有空闲空间。这种特性使其成为实现**生产者-消费者模型**的理想工具。

---

#### **一、BlockingQueue 的核心特性**

 - **线程安全**：所有实现类保证多线程并发操作的安全性。

 - **阻塞操作**： **插入阻塞**：队列满时，put(e)方法会阻塞生产者线程。

 - **移除阻塞**：队列空时，take()方法会阻塞消费者线程。

 - **超时操作**：offer(e, timeout, unit)和poll(timeout, unit)支持设定阻塞时间。

 - **容量限制**：分为**有界队列**（固定容量）和**无界队列**（理论无限容量）。

---

#### **二、BlockingQueue 的实现类**

 以下是 Java 中常见的BlockingQueue实现类及其特点：

| 实现类 | 数据结构 | 容量 | 特点与适用场景 |
| --- | --- | --- | --- |
| ArrayBlockingQueue | 数组 | 有界 | 基于数组的固定容量队列，公平锁可选，吞吐量较低但内存紧凑。适合固定资源池场景。 |
| LinkedBlockingQueue | 链表 | 可选有界/无界 | 默认无界（Integer.MAX_VALUE），高并发下吞吐量更高，但内存占用较大。适合任务队列。 |
| PriorityBlockingQueue | 堆（数组） | 无界 | 元素按优先级排序（需实现Comparable或提供Comparator）。适合任务调度系统。 |
| SynchronousQueue | 无存储 | 容量为 0 | 不存储元素，直接传递任务给消费者线程。适合线程间直接交换数据的场景（如线程池）。 |
| DelayQueue | 优先级堆 | 无界 | 元素需实现Delayed接口，按延迟时间出队。适合定时任务调度（如缓存过期）。 |
| LinkedTransferQueue | 链表 | 无界 | 结合了阻塞队列和同步队列的特性，支持transfer()直接传递数据给消费者。 |

---

#### **三、关键实现类详解1. ArrayBlockingQueue** 

 - **特点**：基于数组的有界队列，通过ReentrantLock实现线程安全，支持公平锁。

 - **示例**： ```java BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10); // 容量为10 queue.put(1); // 队列满时阻塞 int num = queue.take(); // 队列空时阻塞 ```

 **2. LinkedBlockingQueue** 

 - **特点**：基于链表的队列，默认无界（可指定容量），生产者和消费者使用独立锁，高并发下吞吐量更高。

 - **示例**： ```java BlockingQueue<String> queue = new LinkedBlockingQueue<>(100); // 容量100 queue.offer("data", 5, TimeUnit.SECONDS); // 超时5秒插入 ```

 **3. SynchronousQueue** 

 - **特点**：不存储元素，生产者插入操作必须等待消费者移除操作，反之亦然。常用于线程池（如Executors.newCachedThreadPool）。

 - **示例**： ```java BlockingQueue<Object> syncQueue = new SynchronousQueue<>(); // 生产者线程 new Thread(() -> { syncQueue.put("task"); // 阻塞直到消费者取走 }).start(); // 消费者线程 new Thread(() -> { Object task = syncQueue.take(); // 取走任务 }).start(); ```

 **4. DelayQueue** 

 - **特点**：元素需实现Delayed接口，按延迟时间排序。只有延迟期满的元素才能被取出。

 - **示例**（缓存过期）： ```java class DelayedItem implements Delayed { private String data; private long expireTime; // 过期时间戳 public DelayedItem(String data, long delayMs) { this.data = data; this.expireTime = System.currentTimeMillis() + delayMs; } @Override public long getDelay(TimeUnit unit) { return unit.convert(expireTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS); } @Override public int compareTo(Delayed o) { return Long.compare(this.expireTime, ((DelayedItem) o).expireTime); } } DelayQueue<DelayedItem> delayQueue = new DelayQueue<>(); delayQueue.put(new DelayedItem("A", 5000)); // 5秒后过期 DelayedItem item = delayQueue.take(); // 5秒后才能取出 ```

---

#### **四、适用场景总结**

 - **资源池管理**：如数据库连接池（ArrayBlockingQueue）。

 - **高吞吐任务队列**：如日志处理、消息中间件（LinkedBlockingQueue）。

 - **实时任务调度**：如延迟任务、定时器（DelayQueue）。

 - **直接线程间通信**：如线程池任务传递（SynchronousQueue）。

####

## 21. 讲一下PriorityQueue。
### 

### 一句话总结

 PriorityQueue是Java中基于优先级堆实现的无界队列，元素按自然顺序或自定义Comparator排序，队首元素总是优先级最高（最小或最大）。默认使用小顶堆结构，插入和删除操作时间复杂度为O(log n)。非线程安全，常用方法包括offer()、poll()、peek()。

### 详细解析

PriorityQueue是 Java 集合框架中基于优先堆（Priority Heap）实现的队列，其核心特点是元素按优先级排序，而非传统队列的先进先出（FIFO）。它允许用户根据自然顺序（元素实现Comparable接口）或自定义比较器（Comparator）定义优先级，每次从队列中取出的是优先级最高的元素（堆顶）。 

#### **一、核心特点**

 - **非线程安全** PriorityQueue是**非线程安全**的集合类，多线程环境下若多个线程同时修改队列（如插入、删除），可能导致数据不一致或异常。如需线程安全的优先队列，需使用PriorityBlockingQueue（java.util.concurrent包中提供的线程安全版本）。

 - **基于堆的有序性** 内部通过**二叉堆（通常是小顶堆）**实现，堆顶是优先级最高的元素（最小或最大，取决于排序规则）。插入或删除元素时，会通过堆的“上浮”（siftUp）或“下沉”（siftDown）操作自动调整堆结构，确保堆顶始终是当前优先级最高的元素。

 - **无界性（默认）** PriorityQueue是“无界队列”（底层数组会自动扩容），但可以通过构造函数指定初始容量（默认初始容量为 11）。

 - **不允许null元素** 所有元素必须非空，插入null会抛出NullPointerException。

#### **二、实现原理：二叉堆**

 PriorityQueue内部使用**动态数组**存储元素，数组按“完全二叉树”的结构组织，满足堆的性质：

 - 对于索引为i的节点： 父节点索引：(i-1)/2（向下取整）

 - 左子节点索引：2i + 1

 - 右子节点索引：2i + 2

 堆的调整操作

 - **插入（offer/add）**：新元素添加到数组末尾，然后通过siftUp（上浮）操作与父节点比较，若优先级更高（如小顶堆中更小），则交换位置，直到满足堆的性质。

 - **删除（poll）**：移除堆顶元素（数组第一个元素），将数组末尾元素移到堆顶，然后通过siftDown（下沉）操作与子节点比较，若优先级更低（如小顶堆中更大），则与较小的子节点交换位置，直到满足堆的性质。

#### **三、使用场景**

 PriorityQueue适用于需要**按优先级处理元素**的场景，例如：

 - **任务调度**：如线程池中的任务队列，优先级高的任务先执行。

 - **Top K 问题**：维护一个固定大小的小顶堆，快速获取最大的 K 个元素（如“查找数组中前 10 大的数”）。

 - **合并有序序列**：多路归并排序中，用优先队列选择当前最小的元素合并。

#### **四、**示例代码

```java
import java.util.PriorityQueue;
import java.util.Comparator;

public class PriorityQueueDemo {
 public static void main(String[] args) {
 // 1. 自然排序（元素需实现 Comparable，如 Integer）
 PriorityQueue<Integer> minHeap = new PriorityQueue<>();
 minHeap.offer(3);
 minHeap.offer(1);
 minHeap.offer(2);
 System.out.println("小顶堆（自然排序）: " + minHeap); // 输出 [1, 3, 2]（堆结构，非数组顺序）
 System.out.println("取出堆顶: " + minHeap.poll()); // 1
 System.out.println("取出堆顶: " + minHeap.poll()); // 2
 System.out.println("取出堆顶: " + minHeap.poll()); // 3

 // 2. 自定义比较器（大顶堆）
 PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
 maxHeap.add(3);
 maxHeap.add(1);
 maxHeap.add(2);
 System.out.println("大顶堆（自定义排序）: " + maxHeap); // 输出 [3, 1, 2]
 System.out.println("取出堆顶: " + maxHeap.poll()); // 3
 System.out.println("取出堆顶: " + maxHeap.poll()); // 2
 System.out.println("取出堆顶: " + maxHeap.poll()); // 1
 }
}
```
