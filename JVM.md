## JVM探究

- 对JVM的理解，java8和以前的变化
- 什么是OOM，什么是栈溢出StackOverFlowError?
- JVM的常用调优参数
- 内存快照如何抓取，如何分析Dump文件
- JVM中，类加载器的认识



1. ### JVM的位置

2. ### JVM的体系结构

3. ### 类加载器

   加载Class文件

   ![image-20201017123542223](assets/image-20201017123542223.png)

   ```
   
   ```

   

   1. 虚拟机自带的加载器

   2. 启动类（根）加载器

   3. 扩展类加载器

   4. 应用程序加载器

      

4. ### 双亲委派机制

   APP--->EXE-->BOOT(最终执行)

   ```
   为了安全，避免程序员去修改包下的类
   ```

   (1) 类加载器收到类加载的请求

   (2) 将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器

   (3) 启动加载器检查是否能够加载当前这个类，能加载就结束，使用当前的加载器，否则抛出异常，通知子加载器进行加载

   (4) 重复步骤3

   Class Not Found

5. ### 沙箱安全机制

   组成的基本组件

   - 字节码校验   确保java类文件遵循Java语言规范，并不是所有的类文件都会经过字节码校验，比如核心类。
   - 类装载器       防止恶意代码去干涉善意的代码，守护被信任的类库边界，将代码归入保护域，确定代码可以进行哪些操作。

6. ### Native

   带了native关键字的，说明java的作用范围达不到了，要调用底层c语言的库

7. ### PC寄存器

   每个线程都有哟个程序计数器，是线程私有的，就是哟个指针，指向方法区中的方法字节码(用来存储指向对象的一条指令的地址，也即将要执行的指令代码)，在执行引擎读取下一条指令，是哟个非常小的内存空间，几乎可以忽略不计。

8. ### 方法区

   被所有线程共享，所有的字段和方法字节码，特殊的方法，构造函数。所有定义方法的信息都保存到这个区域，属于共享区间

   静态变量、常量、类信息(构造方法、接口定义)、运行时的常量池存在在方法区中，但是实例的

   栈变量存在堆内存中，和方法区无关。

   **static、final、Class、常量池**

   

9. ### 栈

   栈：栈内存，主管程序的运行，生命周期和线程同步；线程结束后，栈内存就释放了。**不存在垃圾回收问题**

   栈：8大基本类型+对象引用+实例的方法。

10. ### 堆

    heap, 一个jvm只有一个堆内存，堆内存的大小是可以调节的。

    类加载器读取了类文件后，一般会把类，方法，常量，变量。保存我们所有引用类型的真实对象。

    gc垃圾回收主要是在伊甸园区和养老区

    内存满了OMM 堆内存不够

    

11. ### 新生区、年老区

12. ### 永久区

13. ### 堆内存调优

14. ### GC

    - 常用算法

15. ### JMM

16. 总结

    

![image-20201017113442429](assets/image-20201017113442429.png)

![image-20201020202230586](assets/image-20201020202230586.png)

1、类加载子系统：通过这个对编译好的class进行加载

堆：放对象

程序计数器：程序走到哪了。

### 字节码

- java源码经过javac编译生成的二进制(文件)，成为字节码
- JVM通过字节码保证平台无关特性
- Java并不是唯一生成class文件的语言
- Class是结构紧凑的二进制流，其格式固定，要求严谨。

#### 字节码要素

组成结构：

1. 魔数

   魔数：区分文件类型的依据。保存在前4个字节

   Java字节码：0xCAFEBABE

2. 文件版本

   字节码版本

   5-6字节是次要版本

   7-8字节是主要版本，Java 8=52.0

   javap 将字节码进行反编译

   字节码版本高于JVM版本，会产生UnsupportedClassVersionError

3. 常量池

   字节码中的保存的基础数据

   ![image-20201020210816702](assets/image-20201020210816702.png)

4. 访问标志

   当前类的访问的修饰符

5. 类/父类索引与接口索引集合

   ![image-20201021083047196](assets/image-20201021083047196.png)

6. 字段表集合

7. 方法表集合

   methods：<init>构造方法

8. 属性表

   辅助信息



字节码指令：

- 字节码指令是包含在字节码的指令
- 字节码指令将源码编译时有编译器生成保存在Method中
- 字节码与平台无关，运行时JVM读取后翻译各个平台底层指令
- 总数不超过256个

![image-20201021085739835](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201021085739835.png)

执行8号索引位的方法。#8是去找java/lang/StringBuilder.append 方法





字符指令的分类

![image-20201021090308891](assets/image-20201021090308891.png)

## 类加载子系统

类加载子系统负责从文件或者网络加载Class字节流

列加载子系统会读取字节码中的信息，运行时存储到 JVM内存

如何Class要被类加载子系统加载，都要符合 JVM字节码规范

加载阶段

读取字节码二进制流解析字节码二进制流的静态数据转换为运行时JVM方法区数据生成类的java.lang.Class对象,放入堆中,作为方法区的访问入口在加载类过程中，必然会触发父类加载。

![image-20201021091751831](assets/image-20201021091751831.png)

linking阶段：

文件格式校验（文件格式，版本号）

元数据匹配，语义分析（是否有父类，是否继承了final修饰的类..）

字节码验证，数据流与控制流分析(类型转换是否有效，无法执行到return)

符号引用（通过全限定名是否能找到对应的类  ClassNotFoundException)

(通过全限定名是否找到对应的方法  NoSuchMethodError)

![image-20201021093019055](assets/image-20201021093019055.png)

类加载器种类

![image-20201021095505085](assets/image-20201021095505085.png)

**启动类加载器**使用C语言开发

用于加载java核心类

- 运行时核心库
- ${sun.boot.class.path}路径下的jar

基于沙箱机制，只加载java、javax、sun包开头的类

**扩展类加载器**

上级加载器为启动类加载器

加载${JAVA_HOME}/jre/lib/ext扩展目录也会被扩展加载器加载

**应用程序类加载器**

应用程序加载器是默认的类加载器，绝大多数类都是由它加载

上级加载器为扩展类加载器

负责加载classpath下的应用程序(三方)类库

```
 		ClassLoader appClassLoader = 		BabytunSeckillApplicationTests.class.getClassLoader();
        System.out.println(appClassLoader);

        //扩展类加载器
        ClassLoader extClassLoader = SunEC.class.getClassLoader();
        System.out.println(extClassLoader);

        //启动类加载器，因为启动类加载器使用C语言编写，没有被JVM管理，所以启动类加载器返回null
        ClassLoader bootstrapClassLoader = Object.class.getClassLoader();
        System.out.println(bootstrapClassLoader);
```

Class实例在JVM是全局唯一的吗？

同一个类加载器对class加载是全局唯一的，不是同一个类加载器对class加载不是全局唯一的。

### 双亲委派模型

加载类时加载器逐级将加载任务向上委派至引导类加载器.然后逐级向下尝试加载,直至加载完成。

双亲委派模型
加载类时加载器逐级将加载任务向上委派至
引导类加载器.然后逐级向下尝试加载,直至
加载完成。
向上
 双亲委派优点:

- 双亲委派机制保护了类不会被重复加载
- 加载机制提供沙箱机制,禁止用户污染java开头核心包

## JVM虚拟机模型

### 程序计数器

程序计数器是记录着当前线程所即将执行的字节码指令行号

每个线程都拥有自己的计数器

执行java方法时，程序计数器时有值的

执行native本地方法时，计数器值为空

占有内存非常少

### 虚拟机栈

保存方法的调用过程

说明了程序运行中的瞬时状态

栈是线程私有的，生命周期与线程相同

每次方法的调用，都会产生对应栈帧



java启动参数   -Xss 数值[k|m|g]

栈分配的内存决定了栈的最大深度



**特点**

线程是私有的

不会被垃圾回收

栈的生命周期和线程相同

栈的深度有限制



两种空间设置：

固定长度：达到上限，StackOverflowError

动态扩展：可用内存不足，OutOfMemoryError

**栈帧**

局部变量表-局部变量

操作数栈-将符号引用转为直接引用

动态链接-将符号引用转为直接引用

返回地址-存放调用方法的程序计数器值



**局部变量表**的特点：

线程私有,不允许跨线程访问,随方法调用创建,方法退出销毁

编译期间长度已确定,局部变量元数据存储在字节码中

局部变量表是栈帧最主要的存储空间,决定了栈的深度

### 操作数栈

字节码指令在执行过程中的中间计算过程存储在操作数栈中。

### 堆

- 堆Heap是 JVM最核心的内存区域，存放运行时实例化的对象实例
- 堆在 JVM启动时就被创建，空间也被分配，是 JVM内存的主要占用区域
- 堆是线程共享，堆内存在物理上可用分散的，在逻辑上是联系的
  - 堆中包含线程私有的缓冲区(TLAB)用于提高 JVM的并发处理效率

#### 引用类型与堆的关系

- 引用类本质是指针，指向存放在堆中的对象实例地址
- 堆是垃圾回收(GC)的重点区域，方法结束后堆对象不会被立即清除，在GC时才会被清理

#### 堆结构和空间

![image-20201021124307594](assets/image-20201021124307594.png)



- 新生代：用来存放新生的对象，该区域对象会被频繁GC
- 老年代：新生带保存的稳定对象放入老年代，老年代不会频繁执行GC
- 元空间：内存的永久保存区域，主要存放Class元数据，几乎不会GC。



设置堆空间大小的参数

-Xms  用来设置堆空间(年轻代+老年代)的初始内存大小

-Xmx  用来设置对空间(年轻代+老年代)的最大内存大小

默认堆空间大小

初始内存大小：物理电脑内存大小/64

最大												 /4



占比

年轻代固定占用1/3，老年代固定占用2/3

查看设置的参数： -XX：+PrintGCDetails

#### 对象分配的一般过程

绝大多数创建的对象会存入新生代的Eden伊甸园区

Eden与S0(幸存者0区)/S1(幸存者1区)的内存分配比例是8：1：1

#### GC



![](assets/image-20201021134218814.png)

![image-20201021124324534](assets/image-20201021124324534.png)

### 方法区

方法区是线程共享区域，是物理分散存储而逻辑为整体的内存区域

方法区保存的内容：

- 类加载器信息/类信息(包含字段/方法)/常量/即时编译器编译后的代码缓存/静态变量

方法去是一个概念

1.7 hotspot方法区的实现为 永久代

1.8后的方法去实现陈伟 元空间

### 根搜索算法

> 对象创建的几种方式

1、用new语句创建对象

2、利用反射

- 调用Java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法

3、调用对象的clone()方法

4、运用反序列化手段

- 调用java.io.ObjectInputStream对象的readObject()方法

根搜索算法，也叫"可达性分析算法"，从GCRoot触发，有引用的对象都是"不可回收的"，其他可标记后再回收，是 JVM默认的算法。

### 垃圾回收算法

#### Mark-Sweep 标记清除算法

- 简单粗暴算法
- 内存不连续，碎片化，不在使用

#### Coping 复制(交换)算法

- 复制(交换)-清除

- 速度快

- 无碎片

- 浪费空间

- 年轻代Minor GC使用

  因为年轻代占用的空间比较小

#### Mark-Compact 标记压缩算法

- 标记-压缩-整理-清除
- 效率比较低
- 无碎片
- 老年代Full GC使用

### GC垃圾收集器

#### 分代收集器（年轻代）

##### serial收集器

- 单线程GC
- 年轻代收集器
- 效率差
- 适合单核CPU
- (几十mb)小内存场景
- 已淘汰

##### ParNew收集器

- STW短时有限，GC多
- 年轻代收集器
- 多线程GC，效率不错
- 适合多核CPU
- n个G内存场景
- 适合用户交互场景
- 与CMS配合使用

##### Parallel Scavenge 收集器

- 吞吐率有限
- 年轻代收集器
- STW时间场，GC次数少
- 多线程GC,效率号
- 适合多核CPU
- 适合后台计算任务

#### 分代收集器（老年代）

##### serialOld收集器

- serial老年代版本
- 1.8老年代收集器默认
- 老年代收集器
- 单线程GC
- 效率差
- 设单核CPU
- 作为CMS备用

##### Parallel Old收集器

- PS老年代版本
- 关注吞吐量
- STW时间长，GC次数少
- 多线程GC，效率不错
- 适合多核CPU
- 适合后台计算任务

##### Concurrent Mark Sweep(CMS)

- 用户线程与GC并行执行

- 超短STW,适合几十G内存

- 会产生浮动垃圾

- 会产生标记失败

- 降低用户线程CPU效率

- 标记-清除产生大量碎片

- SerialOld进行全局整理(标记-压缩)

- JDK14被放弃

- 初始标记:只标记GCRoot第一层关联

- 浮动垃圾是指:运行中未及时发现的垃圾对象,会进行重新标记

- 标记失败:被认定的垃圾再次获得引用,在重新标记阶段修正

#### 不分代的收集器

##### G1收集器

- 多线程并发标记，并发回收
- 逻辑分代，物理分区
- 极短GC时间，单词STW默认最多200ms
- 支持上百G内存GC

## JVM调优

- 最大堆核最小堆大小
- GC收集器
- 新生代(年轻代)大小

当对象没有任何引用的时候就代表对象无用了。

类加载机制

字节码执行机制

JVM内存模型

GC垃圾回收

JVM性能监控与故障处理

JVM调优

### JVM选项规则

- Java  -version 标准选项，任何版本 JVM/任何平台都可以使用
- java  -Xms10m   非标准选项，部分版本识别
- java  -XX:+PrintGCDetail  不稳定参数，不同 JVM有差异，随时可能会被移除

Ps: +代表开启/-代表关闭

![image-20201021152603247](assets/image-20201021152603247.png)

### JVM监控

![image-20201021153209610](assets/image-20201021153209610.png)