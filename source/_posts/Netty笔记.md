---
title: Netty笔记
date: 2021-07-09 12:07:09
tags: 学习
---

# Netty笔记

NIO三大组件，Channel，Buffer，Selector。

channel类似于stream，是读写数据的双向通道，可以从Channel将数据读入Buffer，可以将Buffer的数据写入Channel

常见的Channel有

* FileChannel 文件的数据传输通道。
* DatagramChannel UDP网络编程数据传输通道。
* SocketChannel TCP数据传输通道
* ServerSocketChannel TCP数据传输通道（只用作服务器）

buffer用来缓冲读写数据，如ByteBuffer

ByteBuffer是抽象类

* MappedByteBuffer
* DirectByteBuffer
* HeapByteBuffer

## Selector

阻塞IO的设计：

![image-20220422171848061](/typora-user-images/image-20220422171848061.png)

类比：如果有1000个人难道要雇佣1000个服务员吗？

缺点：

1.可能会造成大量线程处于休眠状态，只是等待输入或输出数据就绪。

2.需要为每个线程的调用栈分配内存。

3.即使 Java 虚拟机（JVM）在物理上可以支持非常大数量的线程，但是远在到达该极限之前，上下文切换所带来的开销就会带来麻烦，例如，在达到10000个连接的时候，但CPU只能处理一部分线程，其他线程就得等待，就得记录其他线程的当前状态，比如执行到哪几行代码，将来轮到这些线程运行了，又得把这些状态恢复，这就叫线程的上下文切换，成本比较高。

阻塞IP只适合连接数较少的操作。

线程池版设计：

![image-20220422173530808](/typora-user-images/image-20220422173530808.png)

只有当socket1断开连接的情况下，线程才能处理socket3，尽管socket1并没有读写，只是连接。也需要等待socket1断开的情况下处理socket3。

selcector的作用就是配合一个线程来管理多个channel，检测这些channel发生的读写事件，读写事件发生的时候，线程就会处理这些事件，这些channel工作在非阻塞模式下。不会让线程吊死在一个channel上。适合连接数特别多，但流量低的场景（low traffic）

![image-20220424103029414](/typora-user-images/image-20220424103029414.png)

优点：

1.使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换所带来开销；
2.当没有 I/O 操作需要处理的时候，线程也可以被用于其他任务。

## ByteBuffer

```java
        // 返回FileChannel 使用ByteBuffer暂存data.txt
        try (FileChannel channel = new FileInputStream("data.txt").getChannel()) {
            // 准备缓冲区,allocate划分一块内存作为缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(10);
            while (true) {
                // 从channel读取数据，向buffer里写
                int read = channel.read(buffer);
                if (read == -1) {// 说明没有内容了
                    break;
                }
                // print buffer
                buffer.flip();// 切换至读模式
                while (buffer.hasRemaining()) { // 是否还有剩余未读数据
                    byte b = buffer.get();
                    System.out.println((char) b);
                }
                // 切换为写模式，或者buffer.compact()
                buffer.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

### ByteBuffer 结构

* capacity 容量

* position 读写指针索引下表

* limit 限制

  ![image-20220424135726772](/typora-user-images/image-20220424135726772.png)

![image-20220424140224051](/typora-user-images/image-20220424140224051.png)

![image-20220424140252653](/typora-user-images/image-20220424140252653.png)

- 向 buffer 写入数据，例如调用 channel.read(buffer)

- 调用 flip() 切换至

  读模式

  - **flip会使得buffer中的limit变为position，position变为0**

- 从 buffer 读取数据，例如调用 buffer.get()

- 调用 clear() 或者compact()切换至

  写模式

  - 调用clear()方法时**position=0，limit变为capacity**
  - 调用compact()方法时，**会将缓冲区中的未读数据压缩到缓冲区前面**

- 重复以上步骤

## 粘包与半包

### 现象

网络上有多条数据发送给服务端，数据之间使用 \n 进行分隔
但由于某种原因这些数据在接收时，被进行了重新组合，例如原始数据有3条为

- Hello,world\n
- I’m YuanqingSese\n
- How are you?\n

变成了下面的两个 byteBuffer (粘包，半包)

- Hello,world\nI’m YuanqingSese\nHo
- w are you?\n

### 出现原因

**粘包**

发送方在发送数据时，并不是一条一条地发送数据，而是**将数据整合在一起**（效率高 ），当数据达到一定的数量后再一起发送。这就会导致多条信息被放在一个缓冲区中被一起发送出去

**半包**

接收方的缓冲区的大小是有限的，当接收方的缓冲区满了以后，就需要**将信息截断**，等缓冲区空了以后再继续放入数据。这就会发生一段完整的数据最后被截断的现象

### 处理方法

```java
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(32);
        // 模拟粘包+半包
        buffer.put("Hello,world\nI'm YuanqingSese\nHo".getBytes());
        // 调用split函数处理
        split(buffer);
        buffer.put("w are you?\n".getBytes());
        split(buffer);
    }

    private static void split(ByteBuffer buffer) {
        buffer.flip();
        for (int i = 0; i < buffer.limit(); i++) {
            // 找到完整的消息
            if (buffer.get(i) == '\n') {
                // 消息长度
                int length = i + 1 - buffer.position();
                // 存入新的ByteBuffer
                ByteBuffer target = ByteBuffer.allocate(length);
                // 从source读,向target写
                for (int j = 0; j < length; j++) {
                    target.put(buffer.get());
                }
                ByteBufferUtil.debugAll(target);
            }
        }
        buffer.compact();
    }
```

### 强制写入

操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘，而是等到缓存满了以后将所有数据一次性的写入磁盘。可以调用 **force(true)** 方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘

## 两个Channel传输数据

### transferTo方法

使用transferTo方法可以快速、高效地将一个channel中的数据传输到另一个channel中，但**一次只能传输2G的内容**

```java
try (
    FileChannel from = new FileInputStream("data.txt").getChannel();
    FileChannel to = new FileOutputStream("to.txt").getChannel();
) {
    long size = from.size();
    // left遍历代表还剩余多少字节没有传输
    for (long left = size; left > 0; ) {
        // 效率高,底层会应用操作系统0拷贝优化, 最多传输2g数据
        left -= from.transferTo((size - left), left, to);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

- JD7引入Path和Paths
- Path 用来表示文件路径
- Paths 是工具类，用来获取 Path 实例

```java
Path source = Paths.get("1.txt"); // 相对路径 不带盘符 使用 user.dir 环境变量来定位 1.txt

Path source = Paths.get("d:\\1.txt"); // 绝对路径 代表了  d:\1.txt 反斜杠需要转义

Path source = Paths.get("d:/1.txt"); // 绝对路径 同样代表了  d:\1.txt

Path projects = Paths.get("d:\\data", "projects"); // 代表了  d:\data\projectsCopy
```

- `.` 代表了当前路径
- `..` 代表了上一级路径

例如目录结构如下

```shell
d:
	|- data
		|- projects
			|- a
			|- bCopy
```

代码

```java
Path path = Paths.get("d:\\data\\projects\\a\\..\\b");
System.out.println(path);
System.out.println(path.normalize()); // 正常化路径 会去除 . 以及 ..Copy
```

输出结果为

```shell
d:\data\projects\a\..\b
d:\data\projects\b
```

### 查找

检查文件是否存在

```java
Path path = Paths.get("helloword/data.txt");
System.out.println(Files.exists(path));
```

### 创建

创建**一级目录**

```java
Path path = Paths.get("helloword/d1");
Files.createDirectory(path);
```

- 如果目录已存在，会抛异常 FileAlreadyExistsException
- 不能一次创建多级目录，否则会抛异常 NoSuchFileException

创建**多级目录用**

```
Path path = Paths.get("helloword/d1/d2");
Files.createDirectories(path);
```

### 拷贝及移动

**拷贝文件**

```java
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/target.txt");

Files.copy(source, target);
```

- 如果文件已存在，会抛异常 FileAlreadyExistsException

如果希望用 source **覆盖**掉 target，需要用 StandardCopyOption 来控制

```java
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```

移动文件

```java
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/data.txt");

Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```

- **StandardCopyOption.ATOMIC_MOVE 保证文件移动的原子性**
- 多级目录拷贝操作：

```java
        String source = "/home/tsdl/Desktop/camera";
        String target = "/home/tsdl/Desktop/cameraAAA";
        try {
            Files.walk(Paths.get(source)).forEach(path -> {
                String targetName = path.toString().replace(source, target);
                Path pathInner = Paths.get(targetName);
                // 是目录就创建
                if (Files.isDirectory(path)) {
                    try {
                        Files.createDirectory(pathInner);
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                    // 是普通文件就copy
                } else if (Files.isRegularFile(path)) {
                    try {
                        Files.copy(path, pathInner);
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

### 删除

删除文件

```java
Path target = Paths.get("helloword/target.txt");

Files.delete(target);
```

- 如果文件不存在，会抛异常 NoSuchFileException

删除目录

```java
Path target = Paths.get("helloword/d1");

Files.delete(target);
```

- 如果**目录还有内容**，会抛异常 DirectoryNotEmptyException

### 遍历文件

可以**使用Files工具类中的walkFileTree(Path, FileVisitor)方法**，其中需要传入两个参数

- Path：文件起始路径

- FileVisitor：文件访问器，

  使用访问者模式

  - 接口的实现类

    SimpleFileVisitor

    有四个方法

    - preVisitDirectory：访问目录前的操作
    - visitFile：访问文件的操作
    - visitFileFailed：访问文件失败时的操作
    - postVisitDirectory：访问目录后的操作

```java
        AtomicInteger dirCount = new AtomicInteger();
        AtomicInteger fileCount = new AtomicInteger();
        try {
            Files.walkFileTree(Paths.get("/home/tsdl/Desktop/camera"), new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult preVisitDirectory(Path path, BasicFileAttributes basicFileAttributes) throws IOException {
                    System.out.println(path);
                    dirCount.incrementAndGet();
                    return super.preVisitDirectory(path, basicFileAttributes);
                }

                @Override
                public FileVisitResult visitFile(Path path, BasicFileAttributes basicFileAttributes) throws IOException {
                    System.out.println(path);
                    fileCount.incrementAndGet();
                    return super.visitFile(path, basicFileAttributes);
                }
            });
            System.out.println(dirCount);
            System.out.println(fileCount);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

## 网络编程

### 阻塞

- 阻塞模式下，相关方法都会导致线程暂停
  - ServerSocketChannel.accept 会在**没有连接建立时**让线程暂停
  - SocketChannel.read 会在**通道中没有数据可读时**让线程暂停
  - 阻塞的表现其实就是线程暂停了，暂停期间不会占用 cpu，但线程相当于闲置
- 单线程下，阻塞方法之间相互影响，几乎不能正常工作，需要多线程支持
- 但多线程下，有新的问题，体现在以下方面
  - 32 位 jvm 一个线程 320k，64 位 jvm 一个线程 1024k，如果连接数过多，必然导致 OOM，并且线程太多，反而会因为频繁上下文切换导致性能降低
  - 可以采用线程池技术来减少线程数和线程上下文切换，但治标不治本，如果有很多连接建立，但长时间 inactive，会阻塞线程池中所有线程，因此不适合长连接，只适合短连接

非阻塞模式下相关方法都不会让线程暂停

BIO服务端：

```java
        // 使用BIO理解阻塞模式,单线程处理,read的时候不能accept，accept的时候不能read。
        // 0.ByteBuffer
        ByteBuffer buffer = ByteBuffer.allocate(16);
        // 1.创建服务器
        try (ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            // 2.绑定监听端口
            serverSocketChannel.bind(new InetSocketAddress(8080));
            // 3.accept 建立连接集合
            List<SocketChannel> channels = new ArrayList<>();
            while (true) {
                System.out.println("connecting..." + serverSocketChannel + Thread.currentThread().getName());
                // 4.建立与客户端的连接,SocketChannel 用来与客户端通信，等待新的连接建立
                SocketChannel accept = serverSocketChannel.accept();
                System.out.println("connected..." + serverSocketChannel + Thread.currentThread().getName());
                channels.add(accept);
                // 5.接收客户端发送的数据
                for (SocketChannel channel : channels) {
                    System.out.println("before read..." + serverSocketChannel + Thread.currentThread().getName());
                    channel.read(buffer);// 阻塞方法，线程停止运行，没有数据就会干等
                    buffer.flip();
                    debugRead(buffer);
                    buffer.clear();
                    System.out.println("after read..." + serverSocketChannel + Thread.currentThread().getName());
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

NIO服务端：

```java
// 使用NIO理解阻塞模式,单线程处理。
// 0.ByteBuffer
ByteBuffer buffer = ByteBuffer.allocate(16);
// 1.创建服务器
try (ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
    serverSocketChannel.configureBlocking(false);// 切换非阻塞，影响accept方法
    // 2.绑定监听端口
    serverSocketChannel.bind(new InetSocketAddress(8080));
    // 3.accept 建立连接集合
    List<SocketChannel> channels = new ArrayList<>();
    // 缺点：这种NIO没有连接情况下会不断循环,没有数据可读的时候不断循环
    while (true) {
        // 4.建立与客户端的连接，SocketChannel 用来与客户端通信，非阻塞线程还会继续运行，如果没有连接建立，accept返回的是null
        SocketChannel accept = serverSocketChannel.accept();
        if (accept != null) {
            System.out.println("connected..." + serverSocketChannel + Thread.currentThread().getName());
            accept.configureBlocking(false);// 切换非阻塞，影响read方法
            channels.add(accept);
        }
        // 5.接收客户端发送的数据
        for (SocketChannel channel : channels) {
            int read = channel.read(buffer);// 非阻塞，线程任然会继续运行，如果没有读到数据read返回0
            if (read > 0) {
                buffer.flip();
                debugRead(buffer);
                buffer.clear();
                System.out.println("after read..." + serverSocketChannel + Thread.currentThread().getName());
            }
        }
    }
} catch (IOException e) {
    throw new RuntimeException(e);
}
```

客户端：

```java
        try (SocketChannel socketChannel = SocketChannel.open()) {
            socketChannel.connect(new InetSocketAddress("localhost", 8080));
            System.out.println("waiting..." + socketChannel.getLocalAddress());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

## Selector

### 多路复用

单线程可以配合 Selector 完成对多个 Channel 可读写事件的监控，这称之为多路复用

- **多路复用仅针对网络 IO**，普通文件 IO **无法**利用多路复用
- 如果不用 Selector 的非阻塞模式，线程大部分时间都在做无用功，而 Selector 能够保证
  - 有可连接事件时才去连接
  - 有可读事件才去读取
  - 有可写事件才去写入
    - 限于网络传输能力，Channel 未必时时可写，一旦 Channel 可写，会触发 Selector 的可写事件

### 使用Accept事件

要使用Selector实现多路复用，服务端代码如下改进

```java
        // 1.创建selector，管理多个channel
        try (Selector selector = Selector.open();
             ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            serverSocketChannel.configureBlocking(false);// 切换非阻塞，影响accept方法
            // 2.建立selector 和 channel的联系(注册)
            // SelectionKey，就是将来事件发生后，通过它可以知道事件和哪个channel事件
            SelectionKey selectionKey = serverSocketChannel.register(selector, 0, null);
            System.out.println(selectionKey);
            // key只关注accept事件
            selectionKey.interestOps(SelectionKey.OP_ACCEPT);
            serverSocketChannel.bind(new InetSocketAddress(8080));
            while (true) {
                // 3.select 方法，查看有没有事件发生，无就阻塞，有事件，线程才会恢复运行
                // 在事件未处理时，它不会阻塞，事件发生后，要么处理，要么取消不能置之不理
                selector.select();
                // 4.处理事件，selectedKeys 内部包含了所有发生的事件
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()){
                    SelectionKey key = iterator.next();
                    System.out.println(key);
                    ServerSocketChannel channel = (ServerSocketChannel) key.channel();
                    SocketChannel accept = channel.accept();
                    System.out.println(accept.getRemoteAddress());
//                    key.cancel();
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

**步骤解析**

- 获得选择器Selector

```java
Selector selector = Selector.open();
```

- 将通道设置为非阻塞模式，并注册到选择器中，并设置感兴趣的事件
  - channel 必须工作在非阻塞模式
  - FileChannel 没有非阻塞模式，因此不能配合 selector 一起使用
  - 绑定的事件类型可以有
    - connect - 客户端连接成功时触发
    - accept - 服务器端成功接受连接时触发
    - read - 数据可读入时触发，有因为接收能力弱，数据暂不能读入的情况
    - write - 数据可写出时触发，有因为发送能力弱，数据暂不能写出的情况

```java
// 通道必须设置为非阻塞模式
server.configureBlocking(false);
// 将通道注册到选择器中，并设置感兴趣的事件
server.register(selector, SelectionKey.OP_ACCEPT);
```

- 通过Selector监听事件，并获得就绪的通道个数，若没有通道就绪，线程会被阻塞

  - 阻塞直到绑定事件发生

    ```java
    int count = selector.select();
    ```

  - 阻塞直到绑定事件发生，**或是超时**（时间单位为 ms）

    ```java
    int count = selector.select(long timeout);
    ```

  - **不会阻塞**，也就是不管有没有事件，立刻返回，自己根据返回值检查是否有事件

    ```java
    int count = selector.selectNow();
    ```

- 获取就绪事件并**得到对应的通道**，然后进行处理

```java
// 获取所有事件
Set<SelectionKey> selectionKeys = selector.selectedKeys();
                
// 使用迭代器遍历事件
Iterator<SelectionKey> iterator = selectionKeys.iterator();

while (iterator.hasNext()) {
	SelectionKey key = iterator.next();
                    
	// 判断key的类型，此处为Accept类型
	if(key.isAcceptable()) {
        // 获得key对应的channel
        ServerSocketChannel channel = (ServerSocketChannel) key.channel();

        // 获取连接并处理，而且是必须处理，否则需要取消
        SocketChannel socketChannel = channel.accept();

        // 处理完毕后移除
        iterator.remove();
	}
}
```

**事件发生后能否不处理**

事件发生后，**要么处理，要么取消（cancel）**，不能什么都不做，**否则下次该事件仍会触发**，这是因为 nio 底层使用的是水平触发

### 使用Read事件

- 在Accept事件中，若有客户端与服务器端建立了连接，**需要将其对应的SocketChannel设置为非阻塞，并注册到选择器中**
- 添加Read事件，触发后进行读取操作

```java
        // 1.创建selector，管理多个channel
        try (Selector selector = Selector.open();
             ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            serverSocketChannel.configureBlocking(false);// 切换非阻塞，影响accept方法
            // 2.建立selector 和 channel的联系(注册)
            // SelectionKey，就是将来事件发生后，通过它可以知道事件和哪个channel事件
            SelectionKey sscKey = serverSocketChannel.register(selector, 0, null);
            System.out.println(sscKey);
            // key只关注accept事件
            sscKey.interestOps(SelectionKey.OP_ACCEPT);
            serverSocketChannel.bind(new InetSocketAddress(8080));
            while (true) {
                // 3.select 方法，查看有没有事件发生，无就阻塞，有事件，线程才会恢复运行
                // 在事件未处理时，它不会阻塞，事件发生后，要么处理，要么取消不能置之不理
                selector.select();
                // 4.处理事件，selectedKeys 内部包含了所有发生的事件
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();// accept,read
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    System.out.println(key);
                    // 5.区分事件类型
                    if (key.isAcceptable()) {
                        ServerSocketChannel channel = (ServerSocketChannel) key.channel();
                        SocketChannel accept = channel.accept();
                        accept.configureBlocking(false);
                        SelectionKey scKey = accept.register(selector, 0, null);
                        // 只关注读事件
                        scKey.interestOps(SelectionKey.OP_READ);
                        System.out.println(accept.getRemoteAddress());
                    }else if (key.isReadable()){// 如果是read事件
                        SocketChannel channel = (SocketChannel)key.channel();// 拿到触发事件的channel
                        ByteBuffer buffer = ByteBuffer.allocate(16);
                        channel.read(buffer);
                        buffer.flip();
                        ByteBufferUtil.debugRead(buffer);
                    }
//                    key.cancel();
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

**删除事件**

**当处理完一个事件后，一定要调用迭代器的remove方法移除对应事件，否则会出现错误**。原因如下

以我们上面的 **Read事件** 的代码为例

- 当调用了 sscKey.register(selector, SelectionKey.OP_ACCEPT)后，Selector中维护了一个集合，**用于存放SelectionKey以及其对应的通道**

  ```java
  // WindowsSelectorImpl 中的 SelectionKeyImpl数组
  private SelectionKeyImpl[] channelArray = new SelectionKeyImpl[8];
  ```

  ```java
  public class SelectionKeyImpl extends AbstractSelectionKey {
      // Key对应的通道
      final SelChImpl channel;
      ...
  }
  ```

![20210414192429](/typora-user-images/20210414192429.png)

* 当**选择器中的通道对应的事件发生后**，selecionKey会被放到发生事件集合中，这个集合只能添加不能删除，所有事件集合的**selecionKey不会自动移除**，所以需要我们在处理完一个事件后，通过迭代器手动移除其中的selecionKey。否则会导致已被处理过的事件再次被处理，就会引发错误。

![](/typora-user-images/20210414193143.png)

### 断开处理

当客户端与服务器之间的连接**断开时，会给服务器端发送一个读事件**，对异常断开和正常断开需要加以不同的方式进行处理

- **正常断开**

  - 正常断开时，服务器端的channel.read(buffer)方法的返回值为-1，**所以当结束到返回值为-1时，需要调用key的cancel方法取消此事件，并在取消后移除该事件**

    ```java
    int read = channel.read(buffer);
    // 断开连接时，客户端会向服务器发送一个写事件，此时read的返回值为-1
    if(read == -1) {
        // 取消该事件的处理
    	key.cancel();
        channel.close();
    } else {
        ...
    }
    // 取消或者处理，都需要移除key
    iterator.remove();
    ```

- **异常断开**

  - 异常断开时，会抛出IOException异常， 在try-catch的**catch块中捕获异常并调用key的cancel方法即可**

### 使用附件进行扩容

服务端：

```java
    // 最基本的扩容，超出字节缓冲区不会报错，会重新读剩下的，所以只会输出超出缓冲区的。应该要在超出缓冲区后进行扩容。
    private static void split(ByteBuffer buffer) {
        buffer.flip();
        for (int i = 0; i < buffer.limit(); i++) {
            // 找到完整的消息
            if (buffer.get(i) == '\n') {
                // 消息长度
                int length = i + 1 - buffer.position();
                // 存入新的ByteBuffer
                ByteBuffer target = ByteBuffer.allocate(length);
                // 从source读,向target写
                for (int j = 0; j < length; j++) {
                    target.put(buffer.get());
                }
                ByteBufferUtil.debugAll(target);
            }
        }
        buffer.compact();// 未拆分出完整消息0123456789abcdef position 16 limit 16
    }

    public static void main(String[] args) {
        // 1.创建selector，管理多个channel
        try (Selector selector = Selector.open();
             ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            serverSocketChannel.configureBlocking(false);// 切换非阻塞，影响accept方法
            // 2.建立selector 和 channel的联系(注册)
            // SelectionKey，就是将来事件发生后，通过它可以知道事件和哪个channel事件
            SelectionKey sscKey = serverSocketChannel.register(selector, 0, null);
            System.out.println(sscKey);
            // key只关注accept事件
            sscKey.interestOps(SelectionKey.OP_ACCEPT);
            serverSocketChannel.bind(new InetSocketAddress(8080));
            while (true) {
                // 3.select 方法，查看有没有事件发生，无就阻塞，有事件，线程才会恢复运行
                // 在事件未处理时，它不会阻塞，事件发生后，要么处理，要么取消不能置之不理
                selector.select();
                // 4.处理事件，selectedKeys 内部包含了所有发生的事件
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();// accept,read
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    iterator.remove();
                    System.out.println(key);
                    // 5.区分事件类型
                    if (key.isAcceptable()) {
                        ServerSocketChannel channel = (ServerSocketChannel) key.channel();
                        SocketChannel accept = channel.accept();
                        accept.configureBlocking(false);
                        ByteBuffer buffer = ByteBuffer.allocate(16);// attachment
                        // 将ByteBuffer作为附件关联到selectionKey
                        SelectionKey scKey = accept.register(selector, 0, buffer);
                        // 只关注读事件
                        scKey.interestOps(SelectionKey.OP_READ);
                        System.out.println(accept.getRemoteAddress());
                    } else if (key.isReadable()) {// 如果是read事件
                        try {
                            SocketChannel channel = (SocketChannel) key.channel();// 拿到触发事件的channel
                            //  获取selectionKey上关联的附件
                            ByteBuffer buffer = (ByteBuffer) key.attachment();
                            int read = channel.read(buffer);
                            // 正常断开
                            if (read == -1) {
                                key.cancel();
                            } else {
                                split(buffer);
                                // 发现position和limit一样说明一个都没消化掉
                                if (buffer.position() == buffer.limit()) {
                                    ByteBuffer newBuffer = ByteBuffer.allocate(buffer.capacity() * 2);
                                    buffer.flip();
                                    // 将旧的bytebuffer拷贝到新的里
                                    newBuffer.put(buffer);
                                    // 替换原有的SelectionKey的buffer使用attach关联
                                    key.attach(newBuffer);
                                }
                            }
                        } catch (IOException e) {
                            // 异常断开
                            key.cancel();
                            e.printStackTrace();
                        }
                    }
//                    key.cancel();
                }
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

客户端：

```java
        try (SocketChannel socketChannel = SocketChannel.open()) {
            socketChannel.connect(new InetSocketAddress("localhost", 8080));
            socketChannel.write(Charset.defaultCharset().encode("0123456789abcdef3333\n"));
            System.out.println("waiting..." + socketChannel.getLocalAddress());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

### ByteBuffer的大小分配

- 每个 channel 都需要记录可能被切分的消息，因为 **ByteBuffer 不能被多个 channel 共同使用**，因此需要为每个 channel 维护一个独立的 ByteBuffer
- ByteBuffer 不能太大，比如一个 ByteBuffer 1Mb 的话，要支持百万连接就要 1Tb 内存，因此需要设计大小可变的 ByteBuffer
- 分配思路可以参考
  - 一种思路是首先分配一个较小的 buffer，例如 4k，如果发现数据不够，再分配 8k 的 buffer，将 4k buffer 内容拷贝至 8k buffer，优点是消息连续容易处理，缺点是数据拷贝耗费性能
    - 参考实现 http://tutorials.jenkov.com/java-performance/resizable-array.html
  - 另一种思路是用多个数组组成 buffer，一个数组不够，把多出来的内容写入新的数组，与前面的区别是消息存储不连续解析复杂，优点是避免了拷贝引起的性能损耗

### 使用Write事件

服务器向客户端进行写操作。

服务端：

```java
    public static void main(String[] args) throws IOException {
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);
        Selector selector = Selector.open();
        ssc.register(selector, SelectionKey.OP_ACCEPT);
        ssc.bind(new InetSocketAddress(8080));
        while (true) {
            // 有事件发生继续向下运行
            selector.select();
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                if (key.isAcceptable()) {
                    // key.channel() == ssc.accept()
                    SocketChannel sc = ssc.accept();
                    sc.configureBlocking(false);
                    // 1.向客户端发送大量数据
                    StringBuilder sb = new StringBuilder();
                    for (int i = 0; i < 3000000; i++) {
                        sb.append("a");
                    }
                    ByteBuffer buffer = Charset.defaultCharset().encode(sb.toString());
                    while (buffer.hasRemaining()){
                        // 2.返回值代表实际写入的字节数
                        int write = sc.write(buffer);
                        System.out.println(write);
                    }
                }
            }
        }
    }
```

不要想着一次性从服务端将数据都写过去，可以对while修改未先写一次然后if判断是否有剩余数据。

客户端：

```java
    public static void main(String[] args) throws IOException {
        SocketChannel open = SocketChannel.open();
        open.connect(new InetSocketAddress("localhost",8080));
        // 3.接受数据
        int count = 0;
        while (true) {
            ByteBuffer buffer = ByteBuffer.allocate(1024 * 1024);
            count += open.read(buffer);
            System.out.println(count);
            buffer.clear();
        }
    }
```

使用可写事件处理一次写不完的情况：

```java
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);
        Selector selector = Selector.open();
        ssc.register(selector, SelectionKey.OP_ACCEPT);
        ssc.bind(new InetSocketAddress(8080));
        while (true) {
            // 有事件发生继续向下运行
            selector.select();
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                if (key.isAcceptable()) {
                    // key.channel() == ssc.accept()
                    SocketChannel sc = ssc.accept();
                    sc.configureBlocking(false);
                    SelectionKey scKey = sc.register(selector, 0, null);
                    scKey.interestOps(SelectionKey.OP_READ);
                    // 1.向客户端发送大量数据
                    StringBuilder sb = new StringBuilder();
                    for (int i = 0; i < 5000000; i++) {
                        sb.append("a");
                    }
                    ByteBuffer buffer = Charset.defaultCharset().encode(sb.toString());
                    // 2.返回值代表实际写入的字节数
                    int write = sc.write(buffer);
                    System.out.println(write);
                    // 3.判断是否有剩余内容
                    if (buffer.hasRemaining()) {
                        // 4.关注可写事件,
                        scKey.interestOps(scKey.interestOps() + SelectionKey.OP_WRITE);
//                        scKey.interestOps(scKey.interestOps() | SelectionKey.OP_WRITE);
                        // 5.把未写完的数据挂到scKey上
                        scKey.attach(buffer);
                    }
                } else if (key.isWritable()) {
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    SocketChannel sc = (SocketChannel) key.channel();
                    int write = sc.write(buffer);
                    System.out.println(write);
                    // 6.清理操作
                    if (!buffer.hasRemaining()) {
                        // 需要清除buffer
                        key.attach(null);
                        // 不需要关注可写事件
                        key.interestOps(key.interestOps() - SelectionKey.OP_WRITE);
                    }
                }
            }
        }
    }
```

### 多线程优化

充分利用多核CPU，分两组选择器

- 单线程配一个选择器（Boss），**专门处理 accept 事件**
- 创建 cpu 核心数的线程（Worker），**每个线程配一个选择器，轮流处理 read 事件**

#### 优化方式一：

会出现一种问题，如果select在前而register在后，那么上来就会直接被阻塞，先注册事件再执行select这样可以收到到注册的事件，否则即使收到了，未注册也无济于事。

```java
package netty.network;

import netty.bytebuffer.ByteBufferUtil;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.concurrent.ConcurrentLinkedQueue;

public class MultiThreadServer {
    public static void main(String[] args) throws IOException {
        Thread.currentThread().setName("Boss");
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);
        Selector boss = Selector.open();
        SelectionKey bossKey = ssc.register(boss,0,null);
        bossKey.interestOps(SelectionKey.OP_ACCEPT);
        ssc.bind(new InetSocketAddress(8080));
        // 1.创建固定数量的worker
        Worker worker = new Worker("worker-0");
        while (true){
            // 开始监听selector上的事件
            boss.select();
            Iterator<SelectionKey> iterator = boss.selectedKeys().iterator();
            while (iterator.hasNext()){
                SelectionKey key = iterator.next();
                iterator.remove();
                if (key.isAcceptable()){
                    // 读写交给worker来做
                    SocketChannel sc = ssc.accept();
                    sc.configureBlocking(false);
                    System.out.println("connected ....." + sc.getRemoteAddress());
                    // 2.关联Selector
                    System.out.println("before register ....." + sc.getRemoteAddress());
                    worker.register(sc); // 初始化 select ,启动worker-0
                    System.out.println("after register ....." + sc.getRemoteAddress());

                }
            }
        }
    }
    static class Worker implements Runnable{
        private Thread thread;
        private Selector selector;
        private String name;
        private boolean start = false;// 还未初始化
        private ConcurrentLinkedQueue<Runnable> queue;

        public Worker(String name) {
            this.name = name;
        }
        // 初始化线程和Selector
        public void register(SocketChannel sc) throws IOException {
            if (!start){
                thread = new Thread(this,name);
                selector = Selector.open();
                thread.start();
                queue = new ConcurrentLinkedQueue<>();
                start = true;
            }
            // 向队列里添加了任务，但任务并没有立刻执行，只是在boss线程进行添加这个任务
            queue.add(new Runnable() {
                @Override
                public void run() {
                    // selector 阻塞住了 只有register在前，selector再后就不会出现这个问题，再来一个新的客户端，肯定排在selector后面了，还是会阻塞住
                    try {
                        sc.register(selector,SelectionKey.OP_READ); // 改为 boss 添加这个任务
                    } catch (ClosedChannelException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
            selector.wakeup(); // 唤醒 selector 方法
        }

        @Override
        public void run() {
            while (true){
                try {
                    开始监听selector上的事件
                    selector.select();// worker-0 阻塞住了，wakeup
                    Runnable task = queue.poll();
                    if (task != null){
                        task.run(); // 执行的是 sc.register(selector,SelectionKey.OP_READ,null);
                    }
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()){
                        SelectionKey key = iterator.next();
                        iterator.remove();
                        if (key.isReadable()) {
                            SocketChannel channel = (SocketChannel) key.channel();
                            ByteBuffer buffer = ByteBuffer.allocate(16);
                            System.out.println("read ....." + channel.getRemoteAddress());
                            channel.read(buffer);
                            buffer.flip();
                            ByteBufferUtil.debugAll(buffer);
                        }
                    }
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```

#### 优化方式二：

或者使用wakeup这种方法处理，因为selector.wakeup(); 方法是一种发票机制，先执行wakeup方法的时候会发一张票，如果再执行到select方法的时候发现有票了，那我就不用阻塞了继续向下运行。先wakeup和先select效果是一样的。

```java
        // 接方法一代码做出改变，初始化线程和Selector
        public void register(SocketChannel sc) throws IOException {
            if (!start){
                thread = new Thread(this,name);
                selector = Selector.open();
                thread.start();
//                queue = new ConcurrentLinkedQueue<>();
                start = true;
            }
            selector.wakeup();// 唤醒select  方法 boss
            sc.register(selector,SelectionKey.OP_READ);
```

#### 多worker操作：

设置几个worker比较好呢，建议设置CPU核心数一致的woker或者动态设置可用核心数

```java
package netty.network;

import netty.bytebuffer.ByteBufferUtil;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.atomic.AtomicInteger;

public class MultiThreadServer {
    public static void main(String[] args) throws IOException {
        Thread.currentThread().setName("Boss");
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);
        Selector boss = Selector.open();
        SelectionKey bossKey = ssc.register(boss,0,null);
        bossKey.interestOps(SelectionKey.OP_ACCEPT);
        ssc.bind(new InetSocketAddress(8080));
        // 1.创建固定数量的worker
        Worker[] workers = new Worker[2];
        // 拿到CPU核心数，最好是手工指定一下，因为他是根据硬件核心数拿的。
//        Worker[] workers = new Worker[Runtime.getRuntime().availableProcessors()];
        for (int i = 0; i < workers.length;i++){
            workers[i] = new Worker("worker-" + i);
        }
        AtomicInteger index = new AtomicInteger();
        while (true){
            boss.select();
            Iterator<SelectionKey> iterator = boss.selectedKeys().iterator();
            while (iterator.hasNext()){
                SelectionKey key = iterator.next();
                iterator.remove();
                if (key.isAcceptable()){
                    // 读写交给worker来做
                    SocketChannel sc = ssc.accept();
                    sc.configureBlocking(false);
                    System.out.println("connected ....." + sc.getRemoteAddress());
                    // 2.关联Selector
                    System.out.println("before register ....." + sc.getRemoteAddress());
                    // round robin
                    workers[index.getAndIncrement() % workers.length].register(sc); // 初始化 select ,启动worker-0
                    System.out.println("after register ....." + sc.getRemoteAddress());

                }
            }
        }
    }
    static class Worker implements Runnable{
        private Thread thread;
        private Selector selector;
        private String name;
        private boolean start = false;// 还未初始化
        private ConcurrentLinkedQueue<Runnable> queue;

        public Worker(String name) {
            this.name = name;
        }
        // 初始化线程和Selector
        public void register(SocketChannel sc) throws IOException {
            if (!start){
                thread = new Thread(this,name);
                selector = Selector.open();
                thread.start();
                queue = new ConcurrentLinkedQueue<>();
                start = true;
            }
            // 向队列里添加了任务，但任务并没有立刻执行，只是在boss线程进行添加这个任务
            queue.add(new Runnable() {
                @Override
                public void run() {
                    // selector 阻塞住了 只有register在前，selector再后就不会出现这个问题，再来一个新的客户端，肯定排在selector后面了，还是会阻塞住
                    try {
                        sc.register(selector,SelectionKey.OP_READ); // 改为 boss 添加这个任务
                    } catch (ClosedChannelException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
            selector.wakeup(); // 唤醒 selector 方法
        }

        @Override
        public void run() {
            while (true){
                try {
                    // 监听selector上的事件
                    selector.select();// worker-0 阻塞住了，wakeup
                    Runnable task = queue.poll();
                    if (task != null){
                        task.run(); // 执行的是 sc.register(selector,SelectionKey.OP_READ,null);
                    }
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()){
                        SelectionKey key = iterator.next();
                        iterator.remove();
                        if (key.isReadable()) {
                            SocketChannel channel = (SocketChannel) key.channel();
                            ByteBuffer buffer = ByteBuffer.allocate(16);
                            System.out.println("read ....." + channel.getRemoteAddress() + Thread.currentThread());
                            channel.read(buffer);
                            buffer.flip();
                            ByteBufferUtil.debugAll(buffer);
                        }
                    }
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```

## Stream与Channel

- stream 不会自动缓冲数据，channel 会利用系统提供的发送缓冲区、接收缓冲区（更为底层）

- stream 仅支持阻塞 API，channel 同时支持阻塞、非阻塞 API，**网络 channel 可配合 selector 实现多路复用**

- 二者均为全双工，即读写可以同时进行

  - 虽然Stream是单向流动的，但是它也是全双工的

## IO模型

- 同步：线程自己去获取结果（一个线程）
  - 例如：线程调用一个方法后，需要等待方法返回结果
- 异步：线程自己不去获取结果，而是由其它线程返回结果（至少两个线程）
  - 例如：线程A调用一个方法后，继续向下运行，运行结果由线程B返回

当调用一次 channel.**read** 或 stream.**read** 后，会由用户态切换至操作系统内核态来完成真正数据读取，而读取又分为两个阶段，分别为：

- 等待数据阶段
- 复制数据阶段

![image-20220830211928009](/typora-user-images/image-20220830211928009.png)

IO模型主要分为以下几种：

### 阻塞IO：

![image-20220830212113223](/typora-user-images/image-20220830212113223.png)

- 用户线程进行read操作时，**需要等待操作系统执行实际的read操作**，此期间用户线程是被阻塞的，无法执行其他操作

### 非阻塞IO

![image-20220830215641725](/typora-user-images/image-20220830215641725.png)

- 用户线程在一个循环中一直调用read方法，若内核空间中还没有数据可读，立即返回
  - **只是在等待阶段非阻塞**
- 用户线程发现内核空间中有数据后，等待内核空间执行复制数据，待复制结束后返回结果，这个过程是被阻塞的。

### 多路复用

![image-20220830215701759](/typora-user-images/image-20220830215701759.png)

**Java中通过Selector实现多路复用**

- 当没有事件时，调用select方法会被阻塞住
- 一旦有一个或多个事件发生后，就会处理对应的事件，从而实现多路复用

**多路复用与阻塞IO的区别**

- 阻塞IO模式下，**若线程因accept事件被阻塞，发生read事件后，仍需等待accept事件执行完成后**，才能去处理read事件
- 多路复用模式下，一个事件发生后，若另一个事件处于阻塞状态，不会影响该事件的执行

### 异步IO

![image-20220830220725875](/typora-user-images/image-20220830220725875.png)

- 线程1调用方法后理解返回，**不会被阻塞也不需要立即获取结果**
- 当方法的运行结果出来以后，由线程2将结果**回调**给线程1
- 不存在异步阻塞这种情况，这是错的

## 零拷贝

**零拷贝指的是数据无需拷贝到 JVM 内存中**，同时具有以下三个优点

- 更少的用户态与内核态的切换
- 不利用 cpu 计算，减少 cpu 缓存伪共享
- 零拷贝适合小文件传输

### 传统 IO 问题

传统的 IO 将一个文件通过 socket 写出

```
File f = new File("helloword/data.txt");
RandomAccessFile file = new RandomAccessFile(file, "r");

byte[] buf = new byte[(int)f.length()];
file.read(buf);

Socket socket = ...;
socket.getOutputStream().write(buf);Copy
```

**内部工作流如下**

![image-20220831222326741](/typora-user-images/image-20220831222326741.png)

- Java 本身并不具备 IO 读写能力，因此 read 方法调用后，要从 Java 程序的**用户态切换至内核态**，去调用操作系统（Kernel）的读能力，将数据读入**内核缓冲区**。这期间用户线程阻塞，操作系统使用 DMA（Direct Memory Access）来实现文件读，其间也不会使用 CPU

> DMA 也可以理解为硬件单元，用来解放 cpu 完成文件 IO

- 从**内核态**切换回**用户态**，将数据从**内核缓冲区**读入**用户缓冲区**（即 byte[] buf），这期间 **CPU 会参与拷贝**，无法利用 DMA

- 调用 write 方法，这时将数据从**用户缓冲区**（byte[] buf）写入 **socket 缓冲区，CPU 会参与拷贝**

- 接下来要向网卡写数据，这项能力 Java 又不具备，因此又得从**用户态**切换至**内核态**，调用操作系统的写能力，使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 CPU

可以看到中间环节较多，java 的 IO 实际不是物理设备级别的读写，而是缓存的复制，底层的真正读写是操作系统来完成的

- 用户态与内核态的切换发生了 3 次，这个操作比较重量级
- 数据拷贝了共 4 次

### NIO 优化

通过 **DirectByteBuf**

- ByteBuffer.allocate(10)
  - 底层对应 HeapByteBuffer，使用的还是 Java 内存
- ByteBuffer.allocateDirect(10)
  - 底层对应DirectByteBuffer，**使用的是操作系统内存**(Java和操作系统都可以使用这片内存)

![image-20220831223736625](/typora-user-images/image-20220831223736625.png)

大部分步骤与优化前相同，唯有一点：**Java 可以使用 DirectByteBuffer 将堆外内存映射到 JVM 内存中来直接访问使用**

- 这块内存不受 JVM 垃圾回收的影响，因此内存地址固定，有助于 IO 读写
- Java 中的 DirectByteBuf 对象仅维护了此内存的虚引用，内存回收分成两步
  - DirectByteBuffer 对象被垃圾回收，将虚引用加入引用队列
    - 当引用的对象ByteBuffer被垃圾回收以后，虚引用对象Cleaner就会被放入引用队列中，然后调用Cleaner的clean方法来释放直接内存
    - DirectByteBuffer 的释放底层调用的是 Unsafe 的 freeMemory 方法
  - 通过专门线程访问引用队列，根据虚引用释放堆外内存
- **减少了一次数据拷贝，用户态与内核态的切换次数没有减少**

### 进一步优化

**以下两种方式都是零拷贝**，即无需将数据拷贝到用户缓冲区中（JVM内存中）

底层采用了 **linux 2.1** 后提供的 **sendFile** 方法，Java 中对应着两个 channel 调用 **transferTo/transferFrom** 方法拷贝数据

![image-20220831224536869](/typora-user-images/image-20220831224536869.png)

- Java 调用 transferTo 方法后，要从 Java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 CPU
- 数据从**内核缓冲区**传输到 **socket 缓冲区**，CPU 会参与拷贝
- 最后使用 DMA 将 **socket 缓冲区**的数据写入网卡，不会使用 CPU

这种方法下

- 只发生了1次用户态与内核态的切换
- 数据拷贝了 3 次

### 进一步优化

**linux 2.4** 对上述方法再次进行了优化

![image-20220831225349512](/typora-user-images/image-20220831225349512.png)

- Java 调用 transferTo 方法后，要从 Java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 CPU

- 只会将一些 offset 和 length 信息拷入 **socket 缓冲区**，几乎无消耗

- 使用 DMA 将 **内核缓冲区**的数据写入网卡，不会使用 CPU

整个过程仅发生了一次用户态与内核态的切换，数据拷贝了两次。所谓的【零拷贝】，并不是真正的无拷贝，而是在不会拷贝重复数据到JVM内存中，零拷贝的优点有：

* 更少的用户态与内核态的切换
* 不利用cpu计算，减少cpu缓存伪共享
* 零拷贝适合小文件传输

## AIO

AIO 用来解决数据复制阶段的阻塞问题

- 同步意味着，在进行读写操作时，线程需要等待结果，还是相当于闲置
- 异步意味着，在进行读写操作时，线程不必等待结果，而是将来由操作系统来通过回调方式由另外的线程来获得结果

> 异步模型需要底层操作系统（Kernel）提供支持
>
> - Windows 系统通过 IOCP **实现了真正的异步 IO**
> - Linux 系统异步 IO 在 2.6 版本引入，但其**底层实现还是用多路复用模拟了异步 IO，性能没有优势**

发现只有这种结果，原因是主线程结束掉了，其他线程也就结束了，以至于还没来得及执行completed中的方法就结束了，所以没有打印出来complete的log，加一行代码System.in.read()等待控制台输入。

```verilog
read begin...Thread[main,5,main]
read end...Thread[main,5,main]
```

```java
package netty.network;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousFileChannel;
import java.nio.channels.CompletionHandler;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;

import netty.bytebuffer.ByteBufferUtil;

public class AIOFileChannel {
    public static void main(String[] args) {
        try (AsynchronousFileChannel channel = AsynchronousFileChannel.open(Paths.get("data.txt"), StandardOpenOption.READ)){
            ByteBuffer buffer = ByteBuffer.allocate(16);
            // 参数1 ByteBuffer
            // 参数2 读取的起始位置
            // 参数3 附件,读满后的ByteBuffer
            // 参数4 回调对象
            System.out.println("read begin..."+ Thread.currentThread());
            channel.read(buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override  // read 成功
                public void completed(Integer result, ByteBuffer attachment) {
                    attachment.flip();
                    ByteBufferUtil.debugAll(attachment);
                    System.out.println("read completed..."+ Thread.currentThread());
                }

                @Override // read 异常
                public void failed(Throwable exc, ByteBuffer attachment) {
                    exc.printStackTrace();
                }
            });
            System.out.println("read end..."+ Thread.currentThread());
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            // 等待控制台输入
            System.in.read();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

# Netty

```java
Netty is an asynchronous event-driven network application framework
for rapid development of maintainable high performance protocol servers & clients.
```

Netty 是一个异步的、基于事件驱动的网络应用框架，用于快速开发可维护、高性能的网络服务器和客户端

**注意**：`netty的异步还是基于多路复用的，并没有实现真正意义上的异步IO`

## Netty的优势

如果使用传统NIO，其工作量大，bug 多

- 需要自己构建协议
- 解决 TCP 传输问题，如粘包、半包
- 因为bug的存在，epoll 空轮询导致 CPU 100%

Netty 对 API 进行增强，使之更易用，如

- FastThreadLocal => ThreadLocal
- ByteBuf => ByteBuffer

## HelloWorld

* HelloServer端代码

```java
package netty.netty;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringDecoder;

public class HelloServer {
    public static void main(String[] args) {
        // 1、启动器，负责装配netty组件，启动服务器
        new ServerBootstrap()
                // 2、创建 NioEventLoopGroup，可以简单理解为 线程池 + Selector
                .group(new NioEventLoopGroup())
                // 3、选择服务器的 ServerSocketChannel 实现
                .channel(NioServerSocketChannel.class)
                // 4、boss 负责处理链接 worker(child) 负责处理读写，该方法决定了 worker(child) 能执行哪些操作(handler)
                // 5、channel 代表和客户端进行数据读写的通道 Initializer 初始化 ChannelInitializer 处理器（仅执行一次）
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    // 它的作用是待客户端SocketChannel建立连接后，执行initChannel以便添加更多的处理器
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        // 6、SocketChannel的处理器，使用StringDecoder解码，ByteBuf=>String
                        nioSocketChannel.pipeline().addLast(new StringDecoder());
                        // 7、SocketChannel的业务处理，使用上一个处理器的处理结果
                        nioSocketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                System.out.println(msg);
                            }
                        });
                    }
                    // 8、ServerSocketChannel绑定8080端口
                }).bind(8080);
    }
}

```

* HelloClient端代码

```java
package netty.netty;

import java.net.InetSocketAddress;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;

public class HelloClient {

    public static void main(String[] args) throws InterruptedException {
        new Bootstrap()
                .group(new NioEventLoopGroup())
                // 选择客户 Socket 实现类，NioSocketChannel 表示基于 NIO 的客户端实现
                .channel(NioSocketChannel.class)
                // ChannelInitializer 处理器（仅执行一次）
                // 它的作用是待客户端SocketChannel建立连接后，执行initChannel以便添加更多的处理器
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel channel) throws Exception {
                        // 消息会经过通道 handler 处理，这里是将 String => ByteBuf 编码发出
                        channel.pipeline().addLast(new StringEncoder());
                    }
                })
                // 指定要连接的服务器和端口
                .connect(new InetSocketAddress("localhost", 8080))
                // Netty 中很多方法都是异步的，如 connect
                // 这时需要使用 sync 方法等待 connect 建立连接完毕
                .sync()
                // 获取 channel 对象，它即为通道抽象，可以进行数据读写操作
                .channel()
                // 写入消息并清空缓冲区
                .writeAndFlush("hello world");
    }
}

```

* 流程分析（左Client端右服务器端）：

![image-20220904164624414](/typora-user-images/image-20220904164624414.png)

## 组件解释：

- channel 可以理解为数据的通道

- msg 理解为流动的数据，最开始输入是 ByteBuf，但经过 pipeline 中的各个 handler 加工，会变成其它类型对象，最后输出又变成 ByteBuf

- handler 可以理解为数据的处理工序

  - 工序有多道，合在一起就是 pipeline（传递途径），pipeline 负责发布事件（读、读取完成…）传播给每个 handler， handler 对自己感兴趣的事件进行处理（重写了相应事件处理方法）

    - pipeline 中有多个 handler，处理时会依次调用其中的 handler

  - handler 分 Inbound 和 Outbound 两类

    - Inbound 入站
  - Outbound 出站
  
- eventLoop 可以理解为处理数据的工人

  - eventLoop 可以管理多个 channel 的 io 操作，并且一旦 eventLoop 负责了某个 channel，就**会将其与channel进行绑定**，以后该 channel 中的 io 操作都由该 eventLoop 负责
  - eventLoop 既可以执行 io 操作，**也可以进行任务处理**，每个 eventLoop 有自己的任务队列，队列里可以堆放多个 channel 的待处理任务，任务分为普通任务、定时任务
  - eventLoop 按照 pipeline 顺序，依次按照 handler 的规划（代码）处理数据，可以为每个 handler 指定不同的 eventLoop

## 组件

### EventLoop

**事件循环对象** EventLoop

EventLoop 本质是一个**单线程执行器**（同时**维护了一个 Selector**），里面有 run 方法处理一个或多个 Channel 上源源不断的 io 事件

它的继承关系如下

- 继承自 j.u.c.ScheduledExecutorService 因此包含了线程池中所有的方法
- 继承自 netty 自己的 OrderedEventExecutor
  - 提供了 boolean inEventLoop(Thread thread) 方法判断一个线程是否属于此 EventLoop
  - 提供了 EventLoopGroup parent() 方法来看看自己属于哪个 EventLoopGroup

**事件循环组** EventLoopGroup

EventLoopGroup 是一组 EventLoop，Channel 一般会调用 EventLoopGroup 的 register 方法来绑定其中一个 EventLoop，后续这个 Channel 上的 io 事件都由此 EventLoop 来处理（保证了 io 事件处理时的线程安全）

- 继承自 netty 自己的 EventExecutorGroup
  - 实现了 Iterable 接口提供遍历 EventLoop 的能力
  - 另有 next 方法获取集合中下一个 EventLoop

### 处理普通与定时任务

```java
package netty.netty;

import java.util.concurrent.TimeUnit;

import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;

public class TestEventLoop {
    public static void main(String[] args) {
        // 1创建事件循环组
        EventLoopGroup group = new NioEventLoopGroup(2); // io事件，普通任务，定时任务
//        EventLoopGroup defaultEventLoopGroup = new DefaultEventLoopGroup(); // 普通任务，定时任务
        // 2.获取下一个事件循环对线
        System.out.println(group.next());
        System.out.println(group.next());
        System.out.println(group.next());
        // 3.执行普通任务
        group.next().submit(new Runnable() {
            @Override
            public void run() {
                System.out.println("OK==" + Thread.currentThread());
            }
        });
        System.out.println("Main==" + Thread.currentThread());
        // 4.执行定时任务
        group.next().scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println("OK" + Thread.currentThread());
            }
        },1,1, TimeUnit.SECONDS);
        // 优雅地关闭
        group.shutdownGracefully();
    }
}

```

**关闭 EventLoopGroup**

优雅关闭 `shutdownGracefully` 方法。该方法会首先切换 `EventLoopGroup` 到关闭状态从而拒绝新的任务的加入，然后在任务队列的任务都处理完成后，停止线程的运行。从而确保整体应用是在正常有序的状态下退出的

### 处理IO任务

#### 细分1服务器代码

```java
package netty.netty;

import java.nio.charset.Charset;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

public class EventLoopServer {
    public static void main(String[] args) {
        new ServerBootstrap()
                // Boss 和 worker
                // Boss只负责accept事件 worker 负责 socketChannel 上的读写事件
            	// 第一个NioEventLoopGroup不写参数也OK因为NioServerSocketChannel只有一个所以默认是1
                .group(new NioEventLoopGroup(),new NioEventLoopGroup(2))
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf =  (ByteBuf)msg;
                                System.out.println(buf.toString(Charset.defaultCharset())  + Thread.currentThread());
                            }
                        });
                    }
                })
                .bind(8080);
    }
}

```

#### 客户端代码

```java
package netty.netty;

import java.net.InetSocketAddress;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;

public class EventLoopClient {

    public static void main(String[] args) throws InterruptedException {
        Channel localhost = new Bootstrap()
                .group(new NioEventLoopGroup())
                // 选择客户 Socket 实现类，NioSocketChannel 表示基于 NIO 的客户端实现
                .channel(NioSocketChannel.class)
                // ChannelInitializer 处理器（仅执行一次）
                // 它的作用是待客户端SocketChannel建立连接后，执行initChannel以便添加更多的处理器
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel channel) throws Exception {
                        // 消息会经过通道 handler 处理，这里是将 String => ByteBuf 编码发出
                        channel.pipeline().addLast(new StringEncoder());
                    }
                })
                // 指定要连接的服务器和端口
                .connect(new InetSocketAddress("localhost", 8080))
                // Netty 中很多方法都是异步的，如 connect
                // 这时需要使用 sync 方法等待 connect 建立连接完毕
                .sync()
                // 获取 channel 对象，它即为通道抽象，可以进行数据读写操作
                .channel();
        System.out.println(localhost);
        // 用于断点调试
        System.out.println();

    }
}

```

一个线程可以管理多个channel。不断启动client可以发现如果启动4个client，线程1管理1、3client，线程2管理2、4client

当有的**任务需要较长的时间处理时，可以使用非NioEventLoopGroup**，避免同一个NioEventLoop中的其他Channel在较长的时间内都无法得到处理

#### 细分2服务器代码

```java
package netty.netty;

import java.nio.charset.Charset;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.DefaultEventLoopGroup;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

public class EventLoopServer {
    public static void main(String[] args) {
        // 细分2,Handler如果执行时间长可以创建一个独立的EventLoopGroup
        EventLoopGroup group = new DefaultEventLoopGroup();
        new ServerBootstrap()
                // Boss 和 worker
                // 细分1,Boss只负责accept事件 worker 负责读写 socketChannel 上的读写
                .group(new NioEventLoopGroup(), new NioEventLoopGroup(2))
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast("Handler1", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf = (ByteBuf) msg;
                                System.out.println(buf.toString(Charset.defaultCharset()) + Thread.currentThread());
                                ctx.fireChannelRead(msg);// 让消息传给下一个Handler
                            }
                        }).addLast(group, "Handler2", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf = (ByteBuf) msg;
                                System.out.println(buf.toString(Charset.defaultCharset()) + Thread.currentThread());
                            }
                        });
                    }
                })
                .bind(8080);
    }
}

```

启动四个客户端发送数据

```java
nioEventLoopGroup-4-1 hello1
defaultEventLoopGroup-2-1 hello1
nioEventLoopGroup-4-2 hello2
defaultEventLoopGroup-2-2 hello2
nioEventLoopGroup-4-1 hello3
defaultEventLoopGroup-2-3 hello3
nioEventLoopGroup-4-2 hello4
defaultEventLoopGroup-2-4 hello4
```

关系如下：
![image-20220911174307196](/typora-user-images/image-20220911174307196.png)

ctx.fireChannelRead(msg);// 让消息传给下一个Handler 源码如下：

```java
    static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
        final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
        EventExecutor executor = next.executor();// 返回下一个Handler的EventLoop
        // 当前Handler中的线程，是否和EventLoop是同一个线程
        if (executor.inEventLoop()) {
            // 是则直接调用
            next.invokeChannelRead(m);
        } else {
            //  不是，将要执行的代码作为任务提交给下一个事件循环处理（换人）
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    next.invokeChannelRead(m);
                }
            });
        }
    }
```

- 如果两个 handler 绑定的是**同一个EventLoopGroup（线程）**，那么就直接调用
- 否则，把要调用的代码封装为一个任务对象，由下一个 handler 的 EventLoopGroup（线程） 来调用

## Channel

Channel 的常用方法

- close() 可以用来关闭Channel
- closeFuture() 用来处理 Channel 的关闭
  - sync 方法作用是同步等待 Channel 关闭
  - 而 addListener 方法是异步等待 Channel 关闭
- pipeline() 方法用于添加处理器
- write() 方法将数据写入
  - 因为缓冲机制，数据被写入到 Channel 中以后，不会立即被发送
  - **只有当缓冲满了或者调用了flush()方法后**，才会将数据通过 Channel 发送出去
- writeAndFlush() 方法将数据写入并**立即发送（刷出）**

### ChannelFuture

#### 连接问题

#### 拆分客户端代码

```java
package netty.netty;

import java.net.InetSocketAddress;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;

public class ChannelClient {

    public static void main(String[] args) throws InterruptedException {
        ChannelFuture channelFuture = new Bootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel channel) throws Exception {
                        channel.pipeline().addLast(new StringEncoder());
                    }
                })
                // 该方法为异步非阻塞方法，主线程调用后不会被阻塞，真正去执行连接操作的是NIO线程
                // NIO线程：NioEventLoop 中的线程
                .connect(new InetSocketAddress("localhost", 8080));
        // 该方法用于等待连接真正建立
        channelFuture.sync();
        Channel channel = channelFuture.channel();
        channel.writeAndFlush("Hello world");
    }
}
```

如果我们去掉`channelFuture.sync()`方法，会服务器无法收到`hello world`

这是因为建立连接(connect)的过程是**异步非阻塞**的，若不通过`sync()`方法阻塞主线程，等待连接真正建立，这时通过 channelFuture.channel() **拿到的 Channel 对象，并不是真正与服务器建立好连接的 Channel**，也就没法将信息正确的传输给服务端

所以需要通过`channelFuture.sync()`方法，阻塞主线程，**同步处理结果**，等待连接真正建立好以后，再去获得 Channel 传递数据。使用该方法，获取 Channel 和发送数据的线程**都是主线程**

下面还有一种方法，用于**异步**获取建立连接后的 Channel 和发送数据，使得执行这些操作的线程是 NIO 线程（去执行connect操作的线程）

```java
package netty.netty;

import java.net.InetSocketAddress;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.string.StringEncoder;

public class ChannelClient {

    public static void main(String[] args) throws InterruptedException {
        // 2.带有这种future，Promise 的类型都是和异步方法配套使用的，用来正确处理结果
        ChannelFuture channelFuture = new Bootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel channel) throws Exception {
                        channel.pipeline().addLast(new StringEncoder());
                    }
                })
                // 该方法为异步非阻塞方法，主线程调用后不会被阻塞，真正去执行连接操作的是NIO线程
                // NIO线程：NioEventLoop 中的线程
                .connect(new InetSocketAddress("localhost", 8080));
        // 该方法用于等待连接真正建立，会被阻塞住
        // 方法1使用sync方法同步处理结果
//        channelFuture.sync();// 阻塞当前线程，直到NIO线程连接建立完毕
//        Channel channel = channelFuture.channel();
//        channel.writeAndFlush("AKA");
//        System.out.println(channel);
        // 使用addListener（回调对象）方法异步处理结果
        channelFuture.addListener(new ChannelFutureListener() {
            @Override
            // 在NIO线程连接建立好之后，会调用operationComplete
            public void operationComplete(ChannelFuture future) throws Exception {
                Channel channel = future.channel();
                System.out.println(channel + "===" + Thread.currentThread());
                channel.writeAndFlush("AKA");
            }
        });
    }
}

```

