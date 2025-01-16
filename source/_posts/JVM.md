---
title: JVM学习
date: 2023-11-09 12:07:09
tags: 学习
---
```java
        new java.lang.Thread(){
               @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("你是AKA");
                }
            }
        }.start();
        new java.lang.Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("大AKA");
                }
            }
        }).start();
```

```xm
    server {
        listen        51140;
        server_name   localhost;
        写跨域问题

        location / {
            root   /home/ruoyi/projects/xiangmu/dist;
            try_files $uri $uri/ /index.html;
            index  index.html index.htm;
        }
    }
```

守护进程执行

```shell
npm install forever -g #安装
forever start app.js #启动
forever stop app.js #关闭
forever start -l forever.log -o out.log -e err.log app.js #输出日志和错误
```

服务器地址：

```shell
蒙园青    华北2 E    172.17.231.180    i-2ze50hce9r0oo66kv3ui    47.93.150.59    http://47.93.150.59:51120/gitbook/    http://47.93.150.59:51110/swagger-ui.html#/

本机Swagger地址:
http://localhost:8080/swagger-ui.html#/
APP页面
http://47.93.150.59:51120/
PC页面
http://47.93.150.59:51130/
```

逻辑：根据请求接口数据，将每次接口返回的数组数据中，最后一项差值-倒数第二项差值的值，组成一个新的数组，新数组中，值=0，累加一次计数器，计数器=8，打鼾标志是，否则打鼾标志为否

新数组中如果值不等于0，第二个计数器+1，如果第二个计数器值大于0翻身次数+1

# JVM位置

1 Java在执行某一个方法会开辟内存作为栈帧，开始运行先开辟线程栈，运行main方法入栈作为栈帧，main方法内还有方法继续入栈作为栈帧，执行完毕出栈
2 在.class文件中执行javap 这是java自带的，javap-c进行反编译将.class字节码编译成我们能看懂的JVM运行的代码

3 局部变量名放this

iconst_1:将1压入操作数栈

istore_1:将1出栈，赋值给局部变量表里的值

iload_1:将局部变量表里的装载出来放到操作数栈

iadd：从操作数栈弹出做加法，并将结果从新压入操作数栈

bipush   10：往操作数栈放入10

如下图：程序计数器的值由字节码执行引擎修改 ，操作数栈是做加减乘除临时的内存空间，方法出口记录到执行完方法后返回到main方法的哪一步，局部变量表放的是对象在堆里面的内存地址。

new出的对象放在Eden园区，Eden园区放满后就会触发轻GC，存货的对象复制到from区，剩下的全部GC，经过GC没有被干掉分代年龄+1，分代年龄放在对象头中，分代年龄到15后到达老年代

GC的原理就是通过GCroots一般是方法引用什么的。一直往下走，找到的对象都标记为非垃圾对象，其余未标记的都是垃圾对象

什么对象会到老年代？单例模式的对象，数据库连接池 ，老年代放满后会进行fullGC，最后回收都回收不了OOM(内存溢出)

jvisualvm，JDK自带调优工具

arthas，阿里调优工具

我们要减轻fullGC,通过使用STW,来实现如果我们在GC的过程中线程程序还在继续往下走，刚刚GC完又有一堆垃圾，所以在GC的适合要暂停线程往下走，这叫STW

![image-20210316180400224](/typora-user-images/image-20210316180400224.png)

![image-20210316180602268](/typora-user-images/image-20210316180602268.png)

在执行math.compute方法时其实java中把这些方法都当作符号---》转变为在方法区的内存地址，找到compute得代码

![image-20210320163328788](/typora-user-images/image-20210320163328788.png)



-XX：UseBiasedLocking：是否打开偏向锁

-XX：BiasedLockingStartupDelay默认是四秒

AQS是一个JAVA线程同步的框架，是JDK中很多锁工具的核心实现框架。

AQS维护了一个信号量和state和一个线程组成的双向链表队列。其中这个线程队列就是用来给线程排队的而state就像一个红绿灯，用来控制线程排队或者放行的。在不同场景下，有不同的意义

在可重入锁这个场景下，state表示加锁的次数。0表示无锁，每加一次锁，state就加1。释放锁state就减1。

JVM其实是一个软件，处于操作系统之上，操作系统之下是硬件

![image-20200614185151839](/typora-user-images/image-20200614185151839.png)

类加载器：加载Class文件 Student student=new Student(); 实例在栈里，具体的人在堆里，（student名字在栈里，有内存地址，而具体的数据在堆里，通过内存地址去找）类是一个模板而对象是具体的

类装载器负责加载 .class 文件,class 文件在文件开头有特定的文件标示(cafe babe),将.class文件加载到内存中，将其放在运行时数据区的**方法区**（放类的描述信息）内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构， ClassLoader 只负责 class 文件的加载,至于它是否可以运行,则由 Execution Engine 决定。ClassLoader有多个

例如：![image-20200615151428025](/typora-user-images/image-20200615151428025.png)

所以垃圾一般都在堆里存在

![image-20200615151830442](/typora-user-images/image-20200615151830442.png)

类加载器有好几种，

1.虚拟机自带的加载器

2.启动类（根）加载器

3.扩展类加载器

4.应用程序加载器

![image-20200615152614587](/typora-user-images/image-20200615152614587.png)

双亲委派机制保证安全

![image-20200615153849226](/typora-user-images/image-20200615153849226.png)

运行时会想上找先找的 APP（应用程序加载器）---->EXC（扩展类加载器）---->根加载器（最终执行）

1.类加载器收到类加载请求

2.将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器

3.启动类加载器检查是否能够加载这个类，能加载就结束，使用当前的加载器，否再，抛出异常，通知子类加载器进行加载

4.重复步骤3

Class Not Found

null java调用不到~ Java=C++--:去掉繁琐的东西，如指针，内存管理交给JVM

## 什么是双亲委派机制

当某个类加载器需要加载某个`.class`文件时，它首先把这个任务委托给他的上级类加载器，递归这个操作，如果上级的类加载器没有加载，自己才会去加载这个类。

### 类加载器的类别

#### BootstrapClassLoader（启动类加载器）

`c++`编写，加载`java`核心库 `java.*`,构造`ExtClassLoader`和`AppClassLoader`。由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作

#### ExtClassLoader （标准扩展类加载器）

`java`编写，加载扩展库，如`classpath`中的`jre` ，`javax.*`或者
 `java.ext.dir` 指定位置中的类，开发者可以直接使用标准扩展类加载器。

#### AppClassLoader（系统类加载器）

```
java`编写，加载程序所在的目录，如`user.dir`所在的位置的`class
```

#### CustomClassLoader（用户自定义类加载器）

`java`编写,用户自定义的类加载器,可加载指定路径的`class`文件

#### 源码分析



```java
protected Class<?> loadClass(String name, boolean resolve)
            throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先检查这个classsh是否已经加载过了
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // c==null表示没有加载，如果有父类的加载器则让父类加载器加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        //如果父类的加载器为空 则说明递归到bootStrapClassloader了
                        //bootStrapClassloader比较特殊无法通过get获取
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {}
                if (c == null) {
                    //如果bootstrapClassLoader 仍然没有加载过，则递归回来，尝试自己去加载class
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

### 委派机制的流程图

![img](/typora-user-images/image-20250110001146166.png)

## 双亲委派机制的作用

1、防止重复加载同一个`.class`。通过委托去向上面问一问，加载过了，就不用再加载一遍。保证数据安全。
 2、保证核心`.class`不能被篡改。通过委托方式，不会去篡改核心`.class`，即使篡改也不会去加载，即使加载也不会是同一个`.class`对象了。不同的加载器加载同一个`.class`也不是同一个`Class`对象。这样保证了`Class`执行安全。

## 沙箱安全机制

沙箱是一个限制程序运行的环境，沙箱机制就是将java代码限定在虚拟机（JVM）特定的运行范围中，并且严格限制代码对本地资源的访问，通过这样的措施来保证对代码的有效隔离，防止对本地系统造成破坏。沙箱主要限制系统资源的访问，不同的沙箱对资源的限制也可以不一样

组成沙箱的基本条件:

字节码校验器：确保java文件遵循java语言规范

类装载器：防止恶意代码去干涉善意代码：双亲委派机制，守护被信任的类库边界，将代码归入保护域，确定了代码可以进行那些操作，从内层JVM自带类加载器开始加载，外层恶意同名类得不到加载而无法使用，由于杨哥通过包来区分了访问域。外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效

存取控制器(access controller) :存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定可以由用户指定。

安全管理器(security manager) :是核心API和操作系统之间的主要接口。实现权限控制,比存取控制器优先级高

安全软件包(security package) : java.security 下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性

## Native

凡是带了native关键字的方法，说明java的作用范围达不到了，会去调用底层C语言的库！会进入本地方法栈，调用本地方法接口 JNI

JNI作用：扩展Java的使用，融合不同的编程语言为java锁用！最初：C，C++

Java诞生的时候 C，C++横行，想要立足，必须要又调用C，C++的程序

它在内存区域中专门开辟了一块标记区域：Native Method Stack，登记native 方法

在最终执行的时候，加载本地方法库中的方法通过JNI    例如java程序驱动打印机

调用其他接口：Socket..Web Service~..http~

## PC寄存器

程序计数器：Program Counter Register 

​    每个线程都有一一个程序计数器，是线程私有的，就是一个指针，指向方法区中的方法字节码(用来存储指向一条指令的地址， 也即将要执行的指令代码)，在执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不计，也就是为什么线程有123456....不乱

## 方法区

Method Area 方法区

​    方法区是被所有线程共享，所有字段和方法字节码，以及一些特殊方法，如构造函数,接口代码也在此定义，简单说，所有定义的方法的信息都保存在该区域，**此区域属于共享区间**; 

​    **静态变量，常量，类信息（构造方法，接口定义），运行时的常量池存在方法区中，但是实例变量存在堆内存中，和方法区无关**

​    方法区：static final Class 常量池



![image-20200616171420524](/typora-user-images/image-20200616171420524.png)

## 栈

程序=数据结构+算法

栈：数据结构，先进后出，后进先出：桶

队列：先进先出（FIFO: First input First Output）

为什么main()方法先执行最后结束？因为main()方法总是第一个压栈执行，只要等到栈中只要一个main方法，才能出栈

![image-20200616184617940](/typora-user-images/image-20200616184617940.png)

递归为什么会造成栈溢出？因为递归会一直压栈，不出栈，栈就会越来越多，抛出 StackOverflowError异常

栈：栈内存，主管程序运行，生命周期和线程同步；线程结束，栈内存也就释放了，对于栈来说，**不存在垃圾回收问题**

一旦线程结束，栈就Over

栈：八大基本类型+对象的引用+实例的方法

栈运行原理：栈帧 

![image-20200616185655919](/typora-user-images/image-20200616185655919.png)

 栈+堆+方法区：交互关系

new的过程：

![image-20200616192745399](/typora-user-images/image-20200616192745399.png)

## 三种JVM

* Sun公司 HotSpot（学的最多）
* BEA公司 JRockit
* IBM公司j9vm

# 堆

Heap，一个JVM只有一个堆内存，栈是线程级的，堆内存的大小是可以调节的

类加载器读取了类文件后，一般会把什么东西放到堆中呢？类，方法，常量，变量，保存我们所有引用类型的真实对象

堆内存中还要细分为三个区域：

* 新生区（伊甸园区）Young/New
  * 类，诞生和成长的地方，甚至死亡；
  * 伊甸园，所有的对象都是在伊甸园区new出来的！
* 养老区 old
* 永久区 Perm

![image-20200617163507267](/typora-user-images/image-20200617163507267.png)

GC垃圾回收，主要是在伊甸园区和养老区，幸存区是用来过度的

假设内存满了，OOM，堆内存不够！    java.lang.OutOfMemoryError: Java heap space

在JDK8以后，永久存储区改了个名字（元空间）

![image-20200617165111528](/typora-user-images/image-20200617165111528.png)

真理：经过研究，百分之99的对象都是临时对象！

永久区

这个区常驻内存的，用来存放一些JDK自身携带的JDK对象，Interface元数据，存储的是Java运行时的一些环境或类信息，这个区域不存在垃圾回收！关闭JVM就会释放这个区域的内存

什么操作会导致永久区崩掉，一个启动类加载了大量的第三方jar包。tomcat部署了太多的应用，或者大量动态生成的反射类。不断地被加载直到内存满，就会出现OOM

jdk1.6之前：永久代，常量池是在方法区

jdk1.7：永久代，但是慢慢退化了`去永久代`，常量池在堆中

jdk1.8：无永久代，常量池在元空间

 ![image-20200617205523133](/typora-user-images/image-20200617205523133.png)

元空间逻辑上存在，物理上不存在，物理上堆只有年轻代（Eden，幸存0，1）和老年代

在一个项目中，突然出现了OOM故障，那么该如何排除，研究为什么出错

* 能够看到代码第几行出错：内存快照分析工具，Jprofiler
* Debug，一行行分析

MAT,Jprofiler 作用：

* 分析Dump内存文件，快速定位内存泄漏
* 获得堆中的数据
* 获得大的对象
  
  

果遇到OOM问题首先要先把堆内存调大，如果轻GC清理不过来，那么重GC清理直到年轻代，老年代，元空间都满了，就会报OOM。

命令：-Xms1m -Xmx8m -XX:+PrintGCDetails// 打印GC垃圾回收西信息 -XX:+HeapDumpOnOutOfMemoryError// oom dump

Xms1m 设置初始化内存分配大小64/1

Xmx8m 设置最大分配内存，默认4/1



GC：垃圾回收机制

![image-20200619005410368](/typora-user-images/image-20200619005410368.png)

JVM 在进行GC时，并不是对这三个区域统一回收。大部分时候，回收都是新生代

* 新生代
* 幸存区（from ，to）不为0，1它既可以from变成to也可以to变成from
* 老年区

GC 两种类型：轻GC（普通的GC），重GC（全局GC）

GC题目：

* JVM的内存模型和分区，每个区放什么
* 堆里的分区有哪些？Eden，from，to，老年区，说说他们的特点！
* GC的算法有那些？标记清楚法，标记整理，复制算法，引用技术器，怎么用？
* 轻GC和重GC分别在什么时候发生？

引用计数法：

给每个对象分配一个计数器，看引用了多少次，清除没引用的

![image-20200619010724166](/typora-user-images/image-20200619010724166.png)

复制算法：假设to 和 from 都是有对象是活的的我为了保证一个是空的，把其中一个区复制到另外的那个区，把from的对象复制到to，不停的交换保证to永远是空的

幸存1和幸存0谁空谁是to，1.每次GC对象都会将活的Eden对象移到幸存区中，一旦Eden区被GC后，就会是空的

![image-20200619234226192](/typora-user-images/image-20200619234226192.png)

![image-20200619234705678](/typora-user-images/image-20200619234705678.png)

* 好处：没有内存碎片
* 坏处：浪费了内存空间：多了一半空间永远是空to。假设对象%100存活（极端情况）使用复制算法很low

复制算法使用最佳场景：对象存活度较低的时候；在新生区GC，也在新生区使用复制算法。

标记清楚法：扫描对象对活着的进行标记，清楚没有标记的对象

![image-20200619235410991](/typora-user-images/image-20200619235410991.png)

* 优点：不需要额外的空间！（复制算法的缺点干掉！）
* 缺点：两次扫描严重浪费时间，会产生内存碎片。

标记压缩：

再优化，压缩：再次扫描，向一端移动存活的对象，防止内存碎片产生，但是多了一个移动成本

![image-20200619235904444](/typora-user-images/image-20200619235904444.png)

标记清除压缩：

* **先区标记清除几次，在进行标记压缩**

## 总结：

内存效率：复制算法>标记清除算法>标记压缩算法（时间复杂度）

内存整齐度：复制算法=标记压缩算法>标记清除算法

内存利用率：标记压缩算法=标记清除算法>复制算法

没有最好的算法，只有最合适的----->GC：分代收集算法



年轻代：

* 存活率低
* 复制算法！

老年代：

* 区域大：存活率高
* 标记清除（内存碎片不是太多）+标记压缩混合 实现

# JMM:Java Memory Model

* Java内存模型
* 作用：缓存一致性协议，用于定义数据读写的规则（遵守，找到这个规则 ）。
* JMM定义了线程工作内存和主内存之间的抽象关系：线程之间的共享变量存储在主内存中，每个栈线程都有一个私有的本地内存（Local Memory）

![image-20200624143900875](/typora-user-images/image-20200624143900875.png)

解决共享对象可见性这个问题：volatile 保证线程工作内存修改立马刷新到主内存中

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![img](/typora-user-images/image-20250110001355358.png)



read：把一个变量的值从主内存传输到工作内存中

load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中

use：把工作内存中一个变量的值传递给执行引擎

assign：把一个从执行引擎接收到的值赋给工作内存的变量

store：把工作内存的一个变量的值传送到主内存中

write：在 store 之后执行，把 store 得到的值放入主内存的变量中



Mybatis生命周期

![image-20200715225430136](/typora-user-images/image-20200715225430136.png)

可以通过再Mapper.xml文件中去配置cache达到优化数据库的作用，更新操作频繁不用配置，读操作频繁可以配置

**Mybatis缓存原理**

![image-20200715225534246](/typora-user-images/image-20200715225534246.png)



**spring**

什么是IOC？

原先我们要通过new去创建对象，现在我们把创建对象这一操作交给spring中的ioc容器去管理，我们想用的时候直接取出来就可以

控制：传统方式是通过程序本身new去创建对象的，使用spring后，对象是由spring来创建的

反转：现在程序不创建对象了，而变成被动的接收对象

通过xml的ByName方式去自动装配，那么ByType则是查找和自己对象属性类型相同的bean

![image-20200716005951990](/typora-user-images/image-20200716005951990.png)

![image-20200716001031617](/typora-user-images/image-20200716001031617.png)

总结：

ByName的时候，需要保证所有的bean的id唯一，并且这个bean需要和自动注入的属性的set方法的值一致

ByType的时候，需要保证所有的bean的class唯一，并且这个bean需要和自动注入的属性的类型一致



在把bean交给IOC容器管理的时候，有两个参数一个是id一个是class

我们自动装配的时候，ByName就去Person里找名字和bean的id相同的，class就找和class相同的



自动装配的两个注解

@Autowired

@Resource

把类交给spring容器管理，注册bean

@Component

注入值用@Value



集合

![image-20200702011138093](/typora-user-images/image-20200702011138093.png)



使用`ReadWriteLock`可以解决这个问题，它保证：

- 只允许一个线程写入（其他线程既不能写入也不能读取）；
- 没有写入时，多个线程允许同时读（提高性能）。
  
  
  
  

# 设计模式

GOF -23 23种设计模式

设计模式的本质是面向对象设计原则的实际运用，是对类的封装性，继承性和多态性以及类的关联关系和组合关系的充分理解

![image-20200624151218689](/typora-user-images/image-20200624151218689.png)

面向对象七大原则（OOP）

![image-20200624152109183](/typora-user-images/image-20200624152109183.png)
