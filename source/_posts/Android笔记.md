---
title: Android笔记
date: 2024-01-01 00:07:09
tags: 学习
---

# Android笔记

| grep:

| 的意思，不是并行计算，而是 将 前者命令的输出，作为 | 后面命令的输入信息。
比如 ps -ef | grep java 不是并行执行的意思，而是 先执行 ps -ef，将输出的内容 xxx 传递给后者，相当于 grep java xxx

响应什么事件，就该为true

XML实现on Click public方法，传参数View

全局变量必须m开头

新建对象，findViewById绑定ID Set如下

```java
    View.OnClickListener clickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {

        }
    };
```

两种intent方法

隐藏的intent需要传递action，通过广播监听action

Intent 传递数据

```
intent.putExtra("username", username.getText());
```

接收其他Activity通过控件传递的数据

```java
usernameText.setText(getIntent().getCharSequenceExtra("username"));
```

在碎片里调用活动中的方法

```java
MainActivity activity = (MainActivity) getActivity();
```

设置颜色时在项目中一般这么用

```java
colorView.setBackgroundResource(R.color.green);
```

1.listView基本使用分三步，1创建一个listView通过findViewByid绑定listView控件2创建adapter引入布局3将adapter，set进listView。

2.使用convertView重用池，不为空的时候重用，checkbox需要监听后根据position赋值避免重用

3.使用viewHolder避免重新创建对象，减少内存占用

4.RecycleView通过linearLayoutManager设置上下或者左右滑动

```shell
refs/for/[brach] 需要经过code review之后才可以提交，而refs/heads/[beanch]不需要code review。
```

ANR：

不要把耗时处理放在UI主线程。

死锁导致ANR，检查主线程是否需要这个锁。

## MVP

MVC中View和Controller操作在Activity和Fragment中实现，而MVP将MVC中的Controller变为Presenter。View与Model隔离，Presenter负责完成View层与Model的交互，Controller层响应用户输入进行逻辑处理。

![MVVM](/typora-user-images/image-20250109214842503.png)

## MVVM

 在MVP的基础上实现数据双向绑定

Activity和Bean：

```java
    ActivityMainBinding binding;
    Account yest;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding =
                DataBindingUtil.setContentView(this, R.layout.activity_main);
        yest = new Account("YEST", 100);
        binding.setAccount(yest);
        binding.setActivity(this);
    }

    public void onclick(View view) {
        int level = yest.getLevel();
        yest.setLevel(level+1);
    }
```

```JAVA
package com.thundersoft.mvx.data;

import androidx.databinding.BaseObservable;
import androidx.databinding.Bindable;

import com.thundersoft.mvx.BR;

public class Account extends BaseObservable {
    private String name;
    private int Level;

    public Account(String name, int level) {
        this.name = name;
        Level = level;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Bindable
    public int getLevel() {
        return Level;
    }

    public void setLevel(int level) {
        Level = level;
        notifyPropertyChanged(BR.level);
    }
}

```

xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

        <data>
                <variable
                    name="account"
                    type="com.thundersoft.mvx.data.Account" />
                <variable
                    name="activity"
                    type="com.thundersoft.mvx.data.MainActivity" />
        </data>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            tools:context=".mvp.MainActivity">

                <EditText
                    android:id="@+id/mEtInput"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content" />

                <Button
                    android:id="@+id/mBtnGet"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:onClick="@{activity.onclick}"
                    android:text="获取账号信息" />

                <TextView
                    android:id="@+id/mTvInfo"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@{account.name+'|'+account.level}" />
        </LinearLayout>
</layout>
```

## Binder

进程的内存大小是有限制的，使用如下命令查看进程间内存大小限制。超过会报错om，如下情况可单独起一个进程

adb shell

getprop dalvik.vm.heapsize

* 占用内存的模块；
* 需要独立通信进程保持长连接的稳定性；
* webView很容易内存泄露，webView不用的话直接将进程给Kill掉；
* 不稳定的功能放入独立进程；

Binder可以为每个APP分配UID同时支持实名和匿名，去ServiceManager注册的是实名，没去的是匿名，系统服务是实名，个人服务是匿名。

进程间通信和线程间通信的内存机制不同，进程间通信是不共享内存的，线程间通信是共享内存的。

AIDL中Interface方法通过DESCRIPTOR判断如果是同一进程，那么就返回Stub对象本身(obj.queryLocalInterface(DESCRIPTOR))，否则如果是跨进程则返回Stub的代理内部类Proxy。

![进程通信](/typora-user-images/进程通信.png)

![binder](/typora-user-images/binder.png)



## Android启动模式

standard：不断启动新的Activity实例，在OneActivity通过StartActivity启动OneActivity，返回栈中会有两个OneActivity，按下两次back才会回到home界面。

singleTop：如果当前返回栈的栈顶为当前实例不会在继续创建，在OneActivity通过StartActivity启动OneActivity，按下一次back直接回到home界面。

singleTask：创建实例时，如果当前返回栈中存在当前实例，那么不会在继续创建，并且在这个实例之上的实例会统统出栈。

singleInstance：指定这种启动方式的Activity， 在创建实例时，自己拥有一个返回栈。

## Android消息机制

​        通常handler会被认为更新UI，因为我们不能再子线程中访问UI控件，否则就会触发程序异常，这个时候通过Handler就可以将更新UI的操作切换到主线程中执行。

​        MessageQueue内部存储结构并不是真正的队列，而是采用单链表的数据结构来存储消息列表。Looper的意思是循环。由于MessageQueue只是一个消息的存储单元，它不能去处理消息，而Looper就填补了这个功能，Looper会以无限循环的形式去查找是否有新消息，如果有的话就处理消息，否则就一直等待着。Looper中还有一个特殊的概念，那就是ThreadLocal，ThreadLocal并不是线程，它的作用是可以在每个线程中存储数据。Handler创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handler内部如何获取到当前线程的Looper呢？这就要使用ThreadLocal了，ThreadLocal可以在不同的线程中互不干扰地存储并提供数据，通过ThreadLocal可以轻松获取每个线程的Looper。要注意，线程默认是没有Looper的，如果需要使用Handler就必须为线程创建Looper。我们经常提到的主线程，也就是UI线程，它是ActivityThread，ActivityThread被创建时就会初始化Looper，这也是在主线程中默认可以使用Handler的原因。

​        Handler的主要作用是将一个任务切换到某个指定的线程中去执行。Android为什么要提供这个功能呢？这是因为Android规定访问UI只能在主线程中进行，如果在子线程中访问UI，那么程序就会抛出异常。基于此必须要在主线程中访问UI，但Android又不建议在主线程中进行耗时操作，否则会导致程序无法响应即ANR，假如要从服务端拉取信息并显示在UI上，这个时候必须在子线程中进行拉取工作，拉取完毕后又不能在子线程中直接访问UI，如果没有Handler，那么我们的确没有办法将访问UI的工作切换到主线程中，系统之所以提供Handler，主要原因就是为了解决在子线程中无法访问UI的矛盾。

​        这里再延伸一点，系统为什么不允许在子线程中访问UI呢？这是因为Android的UI控件不是线程安全的，如果在多线程中并发访问可能会导致UI控件处于不可预期的状态，那为什么系统不对UI控件的访问加上锁机制呢？缺点有两个：首先加上锁机制会让UI访问的逻辑变得复杂；其次锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行。鉴于这两个缺点，最简单且高效的方法就是采用单线程模型来处理UI操作，对于开发者来说也不是很麻烦，只是需要通过Handler切换一下UI访问的执行线程即可。

​        Handler的使用方法这里就不做介绍了，这里描述一下Handler的工作原理。Handler创建时会采用当前线程的Looper来构建内部的消息循环系统，如果当前线程没有Looper，那么就会报错，如下所示。

```shell
E/AndroidRuntime(27568): FATAL EXCEPTION: Thread-43484
E/AndroidRuntime(27568): java.lang.RuntimeException: Can't create handler  inside thread that has not called Looper.prepare()
E/AndroidRuntime(27568): at android.os.Handler.<init>(Handler.java:121)
E/AndroidRuntime(27568): at com.ryg.chapter_10.TestActivity$3.run(TestActivity.java:57)
```

只需要为当前线程创建Looper即可，或者在一个有Looper的线程中创建Handler也行

Handler的post方法将一个Runnable投递到Handler内部的Looper中去处理，也可以通过Handler的send方法发送一个消息，这个消息同样会在Looper中去处理。post方法也是通过send方法来完成的。当Handler的send方法被调用时，它会调用MessageQueue的enqueueMessage方法将这个消息放入消息队列中，然后Looper发现有新消息到来时，就会处理这个消息，最终消息中的Runnable或者Handler的handleMessage方法就会被调用。注意Looper是运行在创建Handler所在的线程中的，这样一来Handler中的业务逻辑就被切换到创建Handler所在的线程中去执行了

![image-20211020173428649](/typora-user-images/image-20211020173428649.png)

### ThreadLocal工作原理

当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal。比如对于Handler来说，它需要获取当前线程的Looper，很显然Looper的作用域就是线程并且不同线程具有不同的Looper，这个时候通过ThreadLocal就可以轻松实现Looper在线程中的存取。可以使用ThreadLocal当做监视器贯穿整个线程的执行过程，如果不使用ThreadLocal，可以有以下两种方法

* 将监视器通过参数的形式在函数调用栈中进行传递。

当函数调用栈很深的时候，通过函数参数来传递监听器对象这几乎是不可接受的，这会让程序的设计看起来很糟糕。

* 将监视器作为静态遍历供线程访问。

这种方法可接受但不是可扩充性的，比如同时有两个线程在执行，那么就需要提供两个静态的监听器对象，如果有10个线程在并发执行呢？提供10个静态的监听器对象？



具体示例：

定义一个ThreadLocal对象

```java
private ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<Boolean>();
```

然后分别在主线程、子线程1和子线程2中设置和访问它的值，代码如下所示。

```java
mBooleanThreadLocal.set(true);
    Log.d(TAG,"[Thread#main]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
    new Thread("Thread#1") {
        @Override
        public void run() {
                mBooleanThreadLocal.set(false);
                Log.d(TAG,"[Thread#1]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
        };
    }.start();
```

```java
 new Thread("Thread#2") {
        @Override
        public void run() {
                Log.d(TAG,"[Thread#2]mBooleanThreadLocal=" + mBooleanThreadLocal.get());
        };
    }.start();
```

在上面的代码中，在主线程中设置mBooleanThreadLocal的值为true，在子线程1中设置mBooleanThreadLocal的值为false，在子线程2中不设置mBooleanThreadLocal的值。然后分别在3个线程中通过get方法获取mBooleanThreadLocal的值，根据前面对ThreadLocal的描述，这个时候，主线程中应该是true，子线程1中应该是false，而子线程2中由于没有设置值，所以应该是null。安装并运行程序，日志如下所示。

```shell
D/TestActivity(8676): [Thread#main]mBooleanThreadLocal=true
D/TestActivity(8676): [Thread#1]mBooleanThreadLocal=false
D/TestActivity(8676): [Thread#2]mBooleanThreadLocal=null
```

ThreadLocal之所以有这么奇妙的效果，是因为不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后再从数组中根据当前ThreadLocal的索引去查找出对应的value值。很显然，不同线程中的数组是不同的，这就是为什么通过ThreadLocal可以在不同的线程中维护一套数据的副本并且彼此互不干扰。

ThreadLocal是一个泛型类，它的定义为public class ThreadLocal<T>，只要弄清楚ThreadLocal的get和set方法就可以明白他的工作原理

JDK1.8前：

* set方法中，首先会通过values方法来获取当前线程中的ThreadLocal数据，如何获取呢？其实获取的方式也是很简单的，在Thread类的内部有一个成员专门用于存储线程的ThreadLocal的数据：ThreadLocal.Values localValues，因此获取当前线程的ThreadLocal数据就变得异常简单了。如果localValues的值为null，那么就需要对其进行初始化，初始化后再将ThreadLocal的值进行存储。下面看一下ThreadLocal的值到底是如何在localValues中进行存储的。在localValues内部有一个数组：private Object[] table，ThreadLocal的值就存在在这个table数组中。ThreadLocal的值在table数组中的存储位置总是为ThreadLocal的reference字段所标识的对象的下一个位置，比如ThreadLocal的reference对象在table数组中的索引为index，那么ThreadLocal的值在table数组中的索引就是index+1。最终ThreadLocal的值将会被存储在table数组中：table[index + 1] = value。

* 调用get方法它同样是取出当前线程的local-Values对象，如果这个对象为null那么就返回初始值，初始值由ThreadLocal的initialValue方法来描述
  
  

* 那么我们可以知道在使用ThreadLocal时他是将对象存储到数组的table[index]中，将实际的值存到table[index+1]中。

### 消息队列的工作原理

消息队列在Android中指的是MessageQueue，MessageQueue主要包含两个操作：插入和读取。读取操作本身会伴随着删除操作，插入和读取对应的方法分别为enqueueMessage和next，其中enqueueMessage的作用是往消息队列中插入一条消息，而next的作用是从消息队列中取出一条消息并将其从消息队列中移除。尽管MessageQueue叫消息队列，但是它的内部实现并不是用的队列，实际上它是通过一个单链表的数据结构来维护消息列表，单链表在插入和删除上比较有优势。

next方法是一个无限循环的方法，如果消息队列中没有消息，那么next方法会一直阻塞在这里。当有新消息到来时，next方法会返回这条消息并将其从单链表中移除。

### Looper的工作原理

Looper在Android的消息机制中扮演着消息循环的角色，具体来说就是它会不停地从MessageQueue中查看是否有新消息，如果有新消息就会立刻处理，否则就一直阻塞在那里。首先看一下它的构造方法，在构造方法中它会创建一个MessageQueue即消息队列，然后将当前线程的对象保存起来，如下所示。

```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

Handler的工作需要Looper，没有Looper的线程就会报错，那么如何为一个线程创建Looper呢？其实很简单，通过Looper.prepare()即可为当前线程创建一个Looper，接着通过Looper.loop()来开启消息循环，如下所示。

```java
ew Thread("Thread#2") {
        @Override
        public void run() {
                Looper.prepare();
                Handler handler = new Handler();
                Looper.loop();
        };
    }.start();
```

Looper除了prepare方法外，还提供了prepareMainLooper方法，这个方法主要是给主线程也就是ActivityThread创建Looper使用的，其本质也是通过prepare方法来实现的。由于主线程的Looper比较特殊，所以Looper提供了一个getMainLooper方法，通过它可以在任何地方获取到主线程的Looper。Looper也是可以退出的，Looper提供了quit和quitSafely来退出一个Looper，二者的区别是：quit会直接退出Looper，而quitSafely只是设定一个退出标记，然后把消息队列中的已有消息处理完毕后才安全地退出。Looper退出后，通过Handler发送的消息会失败，这个时候Handler的send方法会返回false。在子线程中，如果手动为其创建了Looper，那么在所有的事情完成以后应该调用quit方法来终止消息循环，否则这个子线程就会一直处于等待的状态，而如果退出Looper以后，这个线程就会立刻终止，因此建议不需要的时候终止Looper。

Looper最重要的方法是loop方法，调用loop后，调用loop后消息循环系统才会真正的起作用。

Looper的loop方法的工作过程也比较好理解，loop方法是一个死循环，唯一跳出循环的方式是MessageQueue的next方法返回了null。

```java
    while(true) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
```

当Looper的quit方法被调用时，Looper就会调用MessageQueue的quit或者quitSafely方法来通知消息队列退出，当消息队列被标记为退出状态时，它的next方法就会返回null。

```java
    public void quit() {
        this.mQueue.quit(false);
    }

    public void quitSafely() {
        this.mQueue.quit(true);
    }
```

也就是说，Looper必须退出，否则loop方法就会无限循环下去。loop方法会调用MessageQueue的next方法来获取新消息，而next是一个阻塞操作，当没有消息时，next方法会一直阻塞在那里，这也导致loop方法一直阻塞在那里。如果MessageQueue的next方法返回了新消息，Looper就会处理这条消息：msg.target.dispatchMessage(msg)，这里的msg.target是发送这条消息的Handler对象，这样Handler发送的消息最终又交给它的dispatchMessage方法来处理了。但是这里不同的是，Handler的dispatchMessage方法是在创建Handler时所使用的Looper中执行的，这样就成功地将代码逻辑切换到指定的线程中去执行了。

### Handler的工作原理

Handler的工作主要包含消息的发送和接收过程，发送的post一系列方法，最终是通过send的一系列方法来实现的。

Handler发送消息的过程仅仅是向消息队列中插入了一条消息，MessageQueue的next方法就会返回这条消息给Looper，Looper收到消息后就开始处理了，最终消息由Looper交由Handler处理，即Handler的dispatchMessage方法会被调用，这时Handler就进入了处理消息的阶段。

dispatchMessage的实现如下所示。

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (this.mCallback != null && this.mCallback.handleMessage(msg)) {
            return;
        }
        this.handleMessage(msg);
    }
}
```

首先，检查Message的callback是否为null，不为null就通过handleCallback来处理消息。Message的callback是一个Runnable对象，实际上就是Handler的post方法所传递的Runnable参数。

handleCallback逻辑很简单如下所示：

```java
private static void handleCallback(Message message) {
    message.callback.run();
}
```

其次，检查mCallback是否为null，不为null就调用mCallback的handleMessage方法来处理消息。Callback是个接口，它的定义如下：

```java
public interface Callback {
    boolean handleMessage(Message var1);
}
```

通过Callback可以采用如下方式来创建Handler对象：Handler handler = new Handler(callback)。那么Callback的意义是什么呢？源码里面的注释已经做了说明：可以用来创建一个Handler的实例但并不需要派生Handler的子类。在日常开发中，创建Handler最常见的方式就是派生一个Handler的子类并重写其handleMessage方法来处理具体的消息，而Callback给我们提供了另外一种使用Handler的方式，当我们不想派生子类时，就可以通过Callback来实现。

```java
mHandler = new Handler(new Handler.Callback() {
    @Override
    public boolean handleMessage(Message message) {
        return false;
    }
});
```

![handler流程图](/typora-user-images/handler流程图.png)

handler也可以使用特定的Looper来构造

Handler的无参构造方法会检查当前线程没有Looper的话抛出"Can't create handler inside thread that has not called Looper.prepare()"的运行时异常。

### 主线程消息循环

Android的主线程就是ActivityThread，主线程的入口方法为main，在main方法中系统会通过Looper.prepareMainLooper()来创建主线程的Looper以及MessageQueue，并通过Looper.loop()来开启主线程的消息循环，这个过程如下所示。

```java
    public static void main(String[] args) {
        SamplingProfilerIntegration.start();
        ...
        Process.setArgV0("<pre-initialized>");
        Looper.prepareMainLooper();
        ActivityThread thread = new ActivityThread();
        thread.attach(false);
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

主线程的消息循环开始了以后，ActivityThread还需要一个Handler来和消息队列进行交互，这个Handler就是ActivityThread.H，它内部定义了一组消息类型，主要包含了四大组件的启动和停止等过程。

ActivityThread通过ApplicationThread和AMS进行进程间通信，AMS以进程间通信的方方式完成ActivityThread的请求后会回调ApplicationThread中的Binder方法，然后ApplicationThread会向H发送消息，H收到消息后会将ApplicationThread中的逻辑切换到ActivityThread中去执行，即切换到主线程中去执行，这个过程就是主线程的消息循环模型。

### Android线程和线程池

线程分为主线程和子线程，主线程主要处理和界面相关的事情，而子线程则往往用于执行耗时操作。由于Android的特性，如果在主线程中执行耗时操作那么就会导致程序无法及时地响应，因此耗时操作必须放在子线程中去执行。

HandlerThread是一种具有消息循环的线程，在它的内部可以使用Handler。IntentService是一个服务，系统对其进行了封装使其可以更方便地执行后台任务，IntentService内部采用HandlerThread来执行任务，当任务执行完毕后IntentService会自动退出。从任务执行的角度来看，IntentService的作用很像一个后台线程，但是IntentService是一种服务，它不容易被系统杀死从而可以尽量保证任务的执行，而如果是一个后台线程，由于这个时候进程中没有活动的四大组件，那么这个进程的优先级就会非常低，会很容易被系统杀死，这就是IntentService的优点。

在操作系统中，线程是操作系统调度的最小单元，同时线程又是一种受限的系统资源，即线程不可能无限制地产生，并且线程的创建和销毁都会有相应的开销。当系统中存在大量的线程时，系统会通过时间片轮转的方式调度每个线程，因此线程不可能做到绝对的并行，除非线程数量小于等于CPU的核心数，一般来说这是不可能的。试想一下，如果在一个进程中频繁地创建和销毁线程，这显然不是高效的做法。正确的做法是采用线程池，一个线程池中会缓存一定数量的线程，通过线程池就可以避免因为频繁创建和销毁线程所带来的系统开销。

* HandlerThread

HandlerThread继承了Thread，它是一种可以使用Handler的Thread，它的实现也很简单，就是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环，这样在实际的使用中就允许在HandlerThread中创建Handler了。run方法如下：

```java
public void run() {
    this.mTid = Process.myTid();
    Looper.prepare();
    synchronized(this) {
        this.mLooper = Looper.myLooper();
        this.notifyAll();
    }

    Process.setThreadPriority(this.mPriority);
    this.onLooperPrepared();
    Looper.loop();
    this.mTid = -1;
}
```

从HandlerThread的实现来看，它和普通的Thread有显著的不同之处。普通Thread主要用于在run方法中执行一个耗时任务，而HandlerThread在内部创建了消息队列，外界需要通过Handler的消息方式来通知HandlerThread执行一个具体的任务。HandlerThread是一个很有用的类，它在Android中的一个具体的使用场景是IntentService，由于HandlerThread的run方法是一个无限循环，因此当明确不需要再使用HandlerThread时，可以通过它的quit或者quitSafely方法来终止线程的执行，这是一个良好的编程习惯。

* IntentService

IntentService是一种特殊的Service，它继承了Service并且它是一个抽象类，因此必须创建它的子类才能使用IntentService。IntentService可用于执行后台耗时的任务，当任务执行后它会自动停止，同时由于IntentService是服务的原因，这导致它的优先级比单纯的线程要高很多，所以IntentService比较适合执行一些高优先级的后台任务，因为它优先级高不容易被系统杀死。在实现上，IntentService封装了HandlerThread和Handler，这一点可以从它的onCreate方法中看出来，如下所示。

```java
public void onCreate() {
    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + this.mName + "]");
    thread.start();
    this.mServiceLooper = thread.getLooper();
    this.mServiceHandler = new IntentService.ServiceHandler(this.mServiceLooper);
}
```

当IntentService被第一次启动时，它的onCreate方法会被调用，onCreate方法会创建一个HandlerThread，然后使用它的Looper来构造一个Handler对象mServiceHandler，这样通过mServiceHandler发送的消息最终都会在HandlerThread中执行，从这个角度来看，IntentService也可以用于执行后台任务。每次启动IntentService，它的onStartCommand方法就会调用一次，IntentService在onStartCommand中处理每个后台任务的Intent。下面看一下onStartCommand方法是如何处理外界的Intent的，onStartCommand调用了onStart，onStart方法的实现如下:

```java
public void onStart(Intent intent, int startId) {
    Message msg = this.mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    this.mServiceHandler.sendMessage(msg);
}
```

# 填坑记

多线程中使用递归出现问题

```java
colorView.setBackgroundColor(0xffff0000);
```

此方法使用ARGB，0X             FF            FF              FF           00

​                                               透明度         R               G             B

在调用tabLayout.setupWithViewPager(viewPager);会出现没有Tab的情况

因为setupWithViewPager(viewPager)会调用removeAllTabs()，移除全部的Tab

之前向TabLayout添加的Tab项调用这个方法后都被移除了，它会重新创建新的Tab项，tab显示的文字是从适配器getPageTitle()获取的 。上面我们写适配器的时候没有重写适配器的getPageTitle()方法 ，所以效果就是有Tab项但Tab项上没有文字了，解决办法就是重写适配器的getPageTitle()方法

解决办法：

1.在Adapter中重写方法重新给TableLayout赋值。

```java
public CharSequence getPageTitle(int position) {
    return MainActivity.mTitles[position];
}
```

2.可以执行完setupWithViewPager（）后，再添加标题。

```java
       tabLayout.setupWithViewPager(allArtPager);
       tabLayout.getTabAt(0).setText("全部");
       tabLayout.getTabAt(1).setText("Ping²it");
       tabLayout.getTabAt(2).setText("同款");
```

出现异常java.lang.NullPointerException: Attempt to invoke virtual method 'void android.widget.TextView.setText(java.lang.CharSequence)' on a null object reference

查看 当前inflate的layout是否含有frag_text。

```java
        View view = inflater.inflate(R.layout.text_color, container, false);
        TextView textView = view.findViewById(R.id.frag_text);
```

要先如果在收到广播消息的onReceive的方法中去为控件赋值，是无法赋值的，即使赋值也是在DetailActivity.class启动前赋值，跳转后无法接收。

要先跳转到DetailActivity在通过Bundle把对象取出进行赋值。

```java
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //实例化IntentFilter对象
        IntentFilter filter = new IntentFilter();
        filter.addAction(Common.INFO);
        DetailActivity.class
        //注册广播接收
        DynamicReceiver dynamicReceiver = new DynamicReceiver();
        registerReceiver(dynamicReceiver, filter);

    }

    class DynamicReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(getApplication(), "接收广播：" + intent.getStringExtra(Common.INFO), Toast.LENGTH_SHORT).show();
            intent.setClass(MainActivity.this, DetailActivity.class);
            startActivity(intent);
        }
    }
```

```java
// 这个simple_list_item_1是SDK自有的布局，如果在构造Adapter的时候传自己定义布局可能会报
// ArrayAdapter requires the resource ID to be a TextView
arrayAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, contactsList);
```

天坑问题：使用AIDL配置Service，一定要核对包名也就是package！如果包名不对会连接不到服务，如果.不出来（也就是android:name=需要写全部路径）证明包名不对，服务肯定无法连接。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.aidl.service">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Aidl">
        <service
            android:name=".PersonService"
            android:enabled="true"
            android:exported="true">
        </service>
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

开发相册删除时在相对布局中，RecyclerView的Item布局，layout_width和layout_height设置为warp_content,蒙版ImageView控件需要设置为定值

```xml
      <com.hryt.design.base.HHRelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
          android:layout_width="warp_content"
          android:layout_height="warp_content"
          android:orientation="vertical">

      <com.hryt.design.base.HHImageView
          android:id="@+id/image_mask"
          android:layout_width="@dimen/item_album_list_image_width"  定值
          android:layout_height="@dimen/item_album_list_image_height"  定值
          android:background="@color/image_mask_color"
          android:visibility="gone" />
```

如果出现蒙版后挡住checkbox,可以调节layoutxml的位置，让蒙版控件在checkbox之前

```xml
      <com.hryt.design.base.HHImageView
          android:id="@+id/image_mask"
          android:layout_width="@dimen/item_album_list_image_width"
          android:layout_height="@dimen/item_album_list_image_height"
          android:background="@color/image_mask_color"
          android:visibility="gone" />

      <com.hryt.design.checkbox.v2.HHCheckBox
          android:id="@+id/item_select_chk"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:layout_alignRight="@+id/imageView"
          android:layout_alignBottom="@id/imageView"
          android:singleLine="true" />
```

项目中，由于同时选中收藏相册，和普通相册，而收藏相册里的图片是基于普通相册的，会进行两次回调，用于通知用户是否删除成功，那么如何解决

```java
    private void deleteSuccessResetView(List<DeleteResult> deleteResults) {
        // 判断如果选中的相册中包含删除并且数量为两个以上。
        ArrayList<AlbumListBean> selectAlbumItems = findSelectAlbumItems();
        boolean isCollect = false;
        if (selectAlbumItems.size() > 1) {
            for (AlbumListBean selectAlbumItem : selectAlbumItems) {
                if (selectAlbumItem.getType() == DeviceItem.DEVICE_TYPE_COLLECT) {
                    isCollect = true;
                    break;
                }
            }
        }
        // 因为如果选中收藏相册，和普通相册那么一定会回调两次。
        mCallBackDeleteCount++;
        LogUtil.debug("mDeleteSuccessSize: " + mDeleteSuccessSize + " | mDeleteFailSize: " + mDeleteFailSize
                + " | mDeleteSize =" + mDeleteSize);
        for (int i = 0; i < deleteResults.size(); i++) {
            if (deleteResults.get(i).getStatus() == 0) {
                mDeleteSuccessSize = mDeleteSuccessSize + 1;
            } else if (deleteResults.get(i).getStatus() == 1) {
                mDeleteFailSize = mDeleteFailSize + 1;
            }
        }
        LogUtil.debug("mDeleteSuccessSize: " + mDeleteSuccessSize + " | mDeleteFailSize: " + mDeleteFailSize
                + " | mDeleteSize =" + mDeleteSize);
        if (mDeleteSuccessSize + mDeleteFailSize >= mDeleteSize) {
            if (!mIsPagePause) {
                if (isCollect) {
                    // 回调第一次将第一次删除相册的结果数量赋值，有可能删除的是收藏，也有可能是普通。不管是什么类型先累加起来。
                    mDeleteSuccessAll += mDeleteSuccessSize;
                    // 如果是第二次那么进行提示。
                    if (mCallBackDeleteCount == 2) {
                        HHToast.makeText(getContext(), mDeleteSuccessAll + getResources().getString(
                                R.string.delete_success_toast_prompt)).show();
                    }
                    // 如果选中的相册不满足，选中收藏并且选中数量大于一
                } else {
                    HHToast.makeText(getContext(), mDeleteSuccessSize + getResources().getString(
                            R.string.delete_success_toast_prompt)).show();
                }
            }
            mAlbumListAdapter.setSelecting(false);
            mNavibar.setBackVisible(View.VISIBLE);
            mNavibar.setNavigationRightLVisible(View.INVISIBLE);
            mNavibar.setNavigationRightRIcon(getResources()
                    .getDrawable(R.drawable.img_choose_selector));
            mDeleteSize = 0;
            mDeleteSuccessSize = 0;
            mDeleteFailSize = 0;
        }
    }
```

RecyclerAdapter天坑，有mData对象，但.size值为0的时候不加载RecyclerAdapter

```java
public AlbumListAdapter(Context context, List<AlbumListBean> mData) {
    this.mContext = context;
    this.mData = mData;
}
```

Application调用多次onCreate方法会导致bindService时出现类型转换异常的错误需要获取当前app进程，让service在app进程中进行bind

```java
    public boolean isAppProcess() {
        String processName = getRunningProcessName();
        return processName != null && processName.equalsIgnoreCase(this.getPackageName());
    }

    public String getRunningProcessName() {
        int processId = android.os.Process.myPid();
        ActivityManager manager = (ActivityManager) getApplicationContext().getSystemService(Context.ACTIVITY_SERVICE);
        for (ActivityManager.RunningAppProcessInfo processInfo : manager.getRunningAppProcesses()) {
            try {
                if (processInfo.pid == processId) {
                    return processInfo.processName;
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }
```

在使用AlarmManager

取消alarm使用AlarmManager.cancel()函数，传入参数是个PendingIntent实例。

该函数会将所有跟这个PendingIntent相同的Alarm全部取消，怎么判断两者是否相同，android使用的是intent.filterEquals()，具体就是判断两个PendingIntent的action、data、type、class和category是否完全相同。即便使用intent.putExtras(bundle)方法，两者的bundle不一样，也可以进行取消。

* 开发过程中，使用AlarmManager时遇到了一个问题至今网上没有明确原因：
  构造intent，设置定时广播闹钟，在时间到达后，广播接收到的bundle为空

```java
    private void setMessageEndTrigger(MessageRecommend requireMessage) {
        Intent startMessage = new Intent(MESSAGE_START_ACTION);
        Bundle bundle = new Bundle();
        bundle.putSerializable(MESSAGE, requireMessage);
        startMessage.putExtras(bundle);
        PendingIntent intent = PendingIntent.getBroadcast(mContext,
                MESSAGE_WHAT, startMessage, PendingIntent.FLAG_CANCEL_CURRENT);
        if (requireMessage == null) {
            LogUtil.debug(TAG + "TimingMessage end is cancel");
            mAlarmManager.cancel(intent);
            return;
        }
        long endTime = TimeParseUtil.parseLong(requireMessage.getEndTime());
        mAlarmManager.setExact(AlarmManager.RTC_WAKEUP, endTime, intent);
    }
```

```java
    private void checkStartMessage(Intent intent) {
        // rec是空的
        MessageRecommend rec = (MessageRecommend) intent.getExtras().get(MESSAGE);
        if (!mAreaManager.checkArea(rec.getProvinceName(), rec.getCityName())) {
            LogUtil.warning(TAG + "the city is change");
            return;
        }
        if (!mAccountManager.getAccountId().equals(rec.getAccount())) {
            LogUtil.warning(TAG + "the Account is change");
            return;
        }
        sendOperatorMessage(rec);
    }
```

* 第一种解决：将bundle使用putExtra方法设置Key和Value，放弃putExtras方法直接put进bundle

```java
    private void setMessageStartTrigger(MessageRecommend requireMessage) {
        Intent startMessage = new Intent(MESSAGE_START_ACTION);
        Bundle bundle = new Bundle();
        bundle.putSerializable(MESSAGE, requireMessage);
        startMessage.putExtra(MESSAGE, bundle);
        PendingIntent intent = PendingIntent.getBroadcast(mContext,
                MESSAGE_WHAT, startMessage, PendingIntent.FLAG_CANCEL_CURRENT);
        if (requireMessage == null) {
            LogUtil.debug(TAG + "TimingMessage start is cancel");
            mAlarmManager.cancel(intent);
            return;
        }
        long startTime = TimeParseUtil.parseLong(requireMessage.getStartTime());
        LogUtil.debug(TAG + "TimingMessage start time" + startTime);
        mAlarmManager.setExact(AlarmManager.RTC_WAKEUP, startTime, intent);
    }
```

* 广播接收时使用先去调用getBundleExtra方法获取bundle，在通过bundle的getSerializable方法获取对象

```java
    private void checkStartMessage(Intent intent) {
        MessageRecommend rec = (MessageRecommend) intent.
                getBundleExtra(MESSAGE).getSerializable(MESSAGE);
        if (rec == null) {
            LogUtil.debug(TAG + "start time Message is null ");
            return;
        }
        LogUtil.debug(TAG + "start time Message: " + rec.toString());
//        if (!mAreaManager.checkArea(rec.getProvinceName(), rec.getCityName())) {
//            LogUtil.warning(TAG + "the city is change");
//            return;
//        }
        if (!mAccountManager.getAccountId().equals(rec.getAccount())) {
            LogUtil.warning(TAG + "the Account is change");
            return;
        }
        sendOperatorMessage(rec);
    }
```

* 第二种解决：
  可以把传递的参数都转成String传，对象也通过Json转成字符串来传，如果既传对象又传String，那么String依旧穿不过来。

## android:sharedUserId=”android.uid.system”

通过SharedUserId,拥有同一个User id的多个APK可以配置成运行在同一个进程中。

那么把程序的UID配成android.uid.system，也就是要让程序运行在系统进程中，这样就有权限来修改系统时间了。

android:sharedUserId不只可以把apk放到系统进程中，也可以配置多个APK运行在一个进程中，这样可以共享数据。

若无法使用系统权限则有如下两种解决方法：

方法一：

* 在应用程序的AndroidManifest.xml中的manifest节点中加入android:sharedUserId="android.uid.system"这个属性。

* 修改Android.mk文件，加入LOCAL_CERTIFICATE := platform这一行

* 使用mm命令来编译，生成的apk就有修改系统时间的权限了。

方法二：

* 加入android:sharedUserId="android.uid.system"这个属性。

* 使用IDE编译出未加签名的apk文件，但是这个apk文件是不能用的。

* 使用目标系统的platform密钥来重新给apk文件签名。首先找到密钥文件，在我的Android源码目录中的位置是"build/target/product/security"，下面的platform.pk8和platform.x509.pem两个文件。然后用Android提供的Signapk工具来签名，signapk的源代码是在"build/tools/signapk"下，用法为"signapk platform.x509.pem platform.pk8 input.apk output.apk"，文件名最好使用绝对路径防止找不到，也可以修改源代码直接使用。

但项目中加上这条配置可能会导致无法运行：

原因是将加上这条配置的APP进行编译，导致编译后的应用签名和系统签名不一致。

程序想要运行在系统进程中还要有目标系统的platform key，就是上面的方法二提到的platform.pk8和platform.x509.pem两个文件。用这两个key签名后apk才真正可以放入系统进程中。第一个方法中加入LOCAL_CERTIFICATE := platform其实就是用这两个key来签名。

# 项目经验总结

* mediaPlayer中的setLooping(true) 可以在播放音乐中调用，也可以通过Service接口如果监听成功调用，传递当前音乐播放状态的方法中全局赋值通过mediaPlayer.setLooping(true)
* 在音乐播放中，使用SeekBar控件开启线程通过Handler向Activity更新进度条，切记一首歌曲播放完成后停掉线程
* 在使用switch中切记要写break；否则执行完case1还会执行case2
* 焦点问题，第一次点击如果没有触发监听，考虑是不是其他监听控件操作设置了焦点
* 第一次点击切换歌曲出现再次播放同样歌曲的情况，考虑在MusicService中的切换歌曲方法的position++ 或position-- 变为--position，++position先赋值再操作
* Notification中多次点击前台服务，android的back键需要点击多次。考虑每次点击前台服务都会入栈很多次，back出栈很多次，使用Activity的启动模式SingleTop，在mainfeast.xml设置任意Activity该Activity总是会在栈顶

```xml
    <activity
        android:name=".Activity.PlayingMusicActivity"
        android:launchMode="singleTop"/>
```

* PendingIntent nIntent = PendingIntent.getBroadcast(context,requestcode, pIntent, PendingIntent.FLAG_UPDATE_CURRENT);
  主要用来推送服务
  第一个参数，上下文；
  第二个参数，需求码，来标记不同的意图
  第三个参数，意图，包含着要传递的消息体
  第四个参数，有四种，常用的两种：FLAG_CANCEL_CURRENT和FLAG_UPDATE_CURRENT
  FLAG_CANCEL_CURRENT，有个cancel，说明如果使用同一个requestcode的两条pendingintent，只有最新发送的有效
  FLAG_UPDATE_CURRENT，如果requestcode一样，之前的消息体会被最新的覆盖掉
* Message.obtain()方式相比于直接new对象更高效，通过obtain方法获取Message对象使得Message到了重复的利用，减少了每次获取Message时去申请空间的时间。同时，这样也不会永无止境的去创建新对象，减小了Jvm垃圾回收的压力，提高了效率
* 如果将context作为参数传入handler会造成内存泄露问题，因为有可能这个context已经要onDestory了但handler还持有这个实例

问题1：

Can't create handler inside thread that has not called Looper.prepare()

解决：

```java
 Looper.prepare()；
```

问题2：

getMainLooper()' on a null object reference

解决：

```java
Looper.getMainLooper()
```

```java
import static android.view.MotionEvent.ACTION_CANCEL;// 划出屏幕之外
import static android.view.MotionEvent.ACTION_DOWN;// 屏幕摁下
import static android.view.MotionEvent.ACTION_MOVE;// 屏幕移动
import static android.view.MotionEvent.ACTION_UP;// 屏幕抬起
```

## Android9以后使用Notification必须要设置channelID

```java
NotificationChannel channel = new NotificationChannel("001", "default", NotificationManager.IMPORTANCE_DEFAULT);
channel.enableLights(true);
channel.setLightColor(Color.GREEN);
channel.setShowBadge(true);
channel.setLockscreenVisibility(Notification.VISIBILITY_SECRET);

// 获取NotificationManager实例
mNotificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.createNotificationChannel(channel);
// 开启前台服务
        if (mMediaPlayer.isPlaying()) {
            startForeground(NOTIFICATION_ID, mNotification);
        }
```

// 使用前台服务还要加上权限

```xml
<!-- android 9.0上使用前台服务，需要添加权限 -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```

## 项目如何使用回调

“只有我们才知道做些什么，但是我们并不清楚什么时候去做这些，只有其它模块才知道，因此我们必须把我们知道的封装成回调函数告诉其它模块”，异步回调最本质上就是事件驱动编程。

回调函数（callback）是什么？ - 码农的荒岛求生的回答 - 知乎 https://www.zhihu.com/question/19801131/answer/1641403537

拿音乐举例子：先写一个回调接口

```java
public interface MusicCallback {

    void getMusic(Music music);

}
```

在Service中的Binder类写一个注册回调，和接触回调的方法参数为定义的接口，并定义一个回调接口泛型的集合

```java
private List<MusicCallback> mMusicCallbackList = new ArrayList<>();
public void registerMusicCallback(MusicCallback musicCallback) {

    mMusicCallbackList.add(musicCallback);

}
public void unregisterMusicCallback(MusicCallback musicCallback) {

    mMusicCallbackList.remove(musicCallback);

}
```

在每次播放音乐时调用

```java
public void playMusic(String path) {
    try {
        fileName = path;
        mMediaPlayer.reset();
        mMediaPlayer.setVolume(0.8f, 0.8f);
        mMediaPlayer.setDataSource(path);
        mMediaPlayer.prepare();
        mMediaPlayer.start();
        updateSeekBar(path);
        updateNotification();
        for (MusicCallback musicCallback : mMusicCallbackList) {
            // 每次有音乐播放时，将该音乐传递给Service
            musicCallback.getMusic(mMusic);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

```java
// 通过服务对象注册接口，参数是定义的Callback
mMusicServiceStub.registerMusicCallback(new MusicCallback() {
    @Override
    public void getMusic(Music music) {
        // 参数即为当前正在播放的音乐，创建消息体通过handler更新UI
        Message message = new Message();
        message.what = 1;
        handler.sendMessage(message);
    }
});
```

```java
public static Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        if (msg.what == 1) {
            // 更新UI
            update(music);
        }
    }
```

### AIDL使用RemoteCallBackList

client：

* 新建IPersonListener.aidl文件

```java
package com.aidl.client;

import com.aidl.client.Person;

interface IPersonListener {

void onDemandReceiver(in Person msg);

}
```

* ManagerAIDL:新增两个回调方法，参数均为上述aidl文件

```java
void registerListener(IPersonListener listener);

void unregisterListener(IPersonListener listener);
```

* 在Activity监听并注册。

```java
private ServiceConnection mServiceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mPersonManager = PersonManager.Stub.asInterface(service);
        IPersonListener.Stub listener = new IPersonListener.Stub() {
            @Override
            public void onDemandReceiver(Person msg) throws RemoteException {
                // 方法运行在Binder线程池中，是非ui线程，需要使用runOnUiThread，在UI主线程中更新UI。
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        tvPerson.append(msg.toString());
                        Log.d("TAG", "onServiceConnected: " + msg.toString());
                    }
                });
            }
        };
        try {
            mPersonManager.registerListener(listener);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
```

service：

* ManagerStub:

```java
private RemoteCallbackList<IPersonListener> demandList = new RemoteCallbackList<>();
@Override
public void registerListener(IPersonListener listener) {
    Log.d("TAG", "registerListener: ");
    demandList.register(listener);
    thread.start();
}

@Override
public void unregisterListener(IPersonListener listener) {
    demandList.unregister(listener);
    System.out.println("回调关闭");
}

Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        if (demandList != null) {
            while (true) {
                int nums = demandList.beginBroadcast();
                for (int i = 0; i < nums; i++) {
                    Person person = new Person();
                    person.setPrice(i);
                    person.setName("SESE");
                    Log.d("TAG", "run: ");
                    try {
                        demandList.getBroadcastItem(i).onDemandReceiver(person);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
                demandList.finishBroadcast();
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
});
```

### 死亡代理

​    Binder运行在服务端进程，如果服务端进程由于某种原因被异常终止，这个时候我们到服务端的Binder连接断裂（称之为Binder死亡），会导致我们的远程调用失败。更为关键的是，如果我们不知道Binder连接已经断裂，那么客户端的功能就会收到影响。为了解决这个问题，Binder中提供了两个配对的方法linkToDeath和unlinkToDeath，通过linkToDeath我们可以为Binder设置一个死亡代理，当Binder死亡时，我们会收到通知，这个时候我们就可以重新发起连接请求从而恢复连接。

​    为Binder设置死亡代理的方法：首先声明一个DeathRecipient对象。DeathRecipient是一个接口，其内部只有一个方法binderDied，我们需要实现这个方法，当Binder死亡的时候，系统就会回调binderDied方法，然后我们就可以移除之前绑定的binder代理并重新绑定远程服务：

```java
    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
        @Override
        // Binder死亡的时候，系统回调binderDied方法
        public void binderDied() {
            LogUtil.debug("BtMusicService  binderDied");
            if (mBtMusicManager != null) {
                // Clear status
                // 解除死亡代理
                mBtMusicManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
                mBtMusicManager = null;
            }
            // Re-bind service
            LogUtil.debug("Re-bind, when app exception exit.");
            // 这里重新绑定远程Service
            bindService();
        }
    };
            // 在ServiceConnection的onServiceConnected实现方法中重新获取死亡代理
            // 重新获取binder实例
            mBtMusicManager = IBtMusicManager.Stub.asInterface(iBinder);
            // 重新设置死亡代理
            iBinder.linkToDeath(mDeathRecipient, 0);
```

* ubantu不能同时开启两个虚拟机，执行命令即可，sudo rmmod kvm_intel kvm

## makefile

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
# 给自己编译的模块名起一个名字
LOCAL_PACKAGE_NAME := BtPhoneService
LOCAL_PROGUARD_FLAG_FILES := proguard.flags
LOCAL_PROGUARD_ENABLED := disabled
# androidAPP都会写的一句话用来参与编译相关内容
LOCAL_MODULE_TAGS := optional
# 给APK一个签名
LOCAL_CERTIFICATE := platform
#LOCAL_PROPRIETARY_MODULE := true
LOCAL_MODULE_OWNER := ts
#LOCAL_JACK_ENABLED := disabled
# 用来制定哪些文件参与编译，必须是src下的 := +=在原来基础之上追加一些内容，call可以理解为查找的意思
LOCAL_SRC_FILES := $(call all-java-files-under, java)
# 用来编译目录下res下的文件
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/res

# 模块不想让你看，你直接用.jar文件就可以了
LOCAL_STATIC_JAVA_LIBRARIES := \
    android-support-v7-recyclerview \
    android-support-v7-appcompat \
    android-support-v4 \
    btphonelib.static

# 编译外部模块，从外部加载进来，动态加载，不用将该jar包放到我的目录下
# 使用mm编译的话是不会找到这个模块的，使用mma编译的话就会先编译ts_framework在编译自身模块
LOCAL_JAVA_LIBRARIES := \
    ts-framework

include vendor/ts/proprietary/common/Features/general-package-app-module.mk

# 预编译相关的内容，为编译其他内容所准备
include $(CLEAR_VARS)
# 预编译格式:=模块名:具体内容，jar包位置
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := ts_btphone:libs/ts_btphone_sdk.jar \
include $(BUILD_MULTI_PREBUILT)
使用adb push xxx.apk /vender/app/xx目录=模块名
```

## 使用接口解耦

项目中我们通常会发现在一个类中干了好多的事情,比如在service中不仅进行了业务逻辑，还进行了协议发送，如何解耦使得service只进行service操作，逻辑在一个类里操作，而协议在一个类里操作

```java
public interface Excuter {

    boolean excutable(String funcId);

    ResultBean excute(String funcId, String params);
    // 回调注册
    void start();
    // 回调销毁
    void end();
}
```

创建SystemSetting实现Excuter接口只进行业务逻辑操作

```java
public class SystemSetting implements Excuter {
    RemoteSettingManager mRemoteSettingManager;
    GwRequestManager gwRequestManager;
    Context mContext;

    // 3在service中获取当前实例，并将当前实例作为参数调用协议manager中的add方法将实现Exuter的实例，add进Exuter的list里
    public SystemSetting(Context context) {
        mRemoteSettingManager = RemoteSettingManager.getInstance(context);
        gwRequestManager = GwRequestManager.getInstance();
        mContext = context;
    }

    @Override
    public boolean excutable(String funcId) {
        // 这个funcID是否存在
    }

    @Override
    public ResultBean excute(String funcId, String params) {
        ResultBean resultBean = new ResultBean();
        switch (funcId) {
            case REQ_MUTE_TYPE_SET:
                resultBean = setMuteStatus(params);
                break;
            case REQ_MUTE_TYPE_GET:
                resultBean = getMuteStatus();
                break;
            case REQ_MEDIA_VOLUME_SET:
                resultBean = setMediaVolume(params);
                break;
            case REQ_MEDIA_VOLUME_GET:
                resultBean = getMediaVolume();
                break;
            case REQ_EQUALIZER_SET:
                resultBean = setEqualizerMode(params);
                break;
            case REQ_EQUALIZER_GET:
                resultBean = getEqualizerMode();
                break;
            case REQ_CUS_EQUALIZER_SET:
                resultBean = setCusEqualizer(params);
                break;
            case REQ_CUS_EQUALIZER_GET:
                resultBean = getCusEqualizer();
                break;
            default:
                LogUtil.warning("FUNCTION_ID" + funcId);
                break;
        }
        resultBean.setFuncId(funcId);
        return resultBean;
    }

    // mISoundCallback是系统音乐设置各种操作的回调
    @Override
    public void start() {
        mRemoteSettingManager.registSoundCallback(mISoundCallback);
    }

    @Override
    public void end() {
        mRemoteSettingManager.unregistSoundCallback(mISoundCallback);
    }
```

创建Manager只进行协议发送操作

```java
public class GwRequestManager {
    private static final String TAG = "GwSettingServices";

    private static volatile GwRequestManager sInstance = null;
    // 1在service中获取当前实例，并调用init进行初始化
    public static GwRequestManager getInstance() {
        if (sInstance == null) {
            synchronized (SoundManager.class) {
                if (sInstance == null) {
                    sInstance = new GwRequestManager();
                }
            }
        }
        return sInstance;
    }
    // 2调用init进行初始化
    public void init(Context context) {
        receiveRequest(context);
    }

    private void receiveRequest(Context context) {
        OpenGatewayManager.getInstance(context)
                .registService(SERVICE_NAME, NODE_NAME, new ServiceCallback() {
                    @Override
                    public void onRequest(String requestId, String requestBody) throws RemoteException {
                        super.onRequest(requestId, requestBody);
                        RequestBean requestBean = ParseUtil.fromJson(requestBody, RequestBean.class);
                        LogUtil.debug(TAG + "\n" + "requestBean:" + requestBean + "requestBody:" + requestBody);
                        String funcId = requestBean.getFuncId();
                        remoteControl(funcId, requestBean.getParams(), requestId);
                    }

                    @Override
                    public void onSubscribe(String eventName, String subscribeBody) throws RemoteException {
                        super.onSubscribe(eventName, subscribeBody);
                        Log.d(TAG, "onSubscribe: " + eventName);
                    }
                });
    }

    private Set<Excuter> excuters = Collections.synchronizedSet(new HashSet<>());

    public void addExucter(Excuter excuter) {
        excuters.add(excuter);
        excuter.start();
    }

    public void removeExucter(Excuter excuter) {
        excuter.end();
        excuters.remove(excuter);
    }


    private void remoteControl(String funcId, String params, String requestId) {
        LogUtil.debug(TAG + "\n" + "funcId:" + funcId + "params:" + params + "requestId:" + requestId);
        ResultBean resultBean = new ResultBean();
        resultBean.setFuncId(funcId);
        for (Excuter excuter : excuters) {
            // 4遍历全局变量，判断当前协议的funcId是否属于规定funcId
            if (excuter.excutable(funcId)) {
                // 5进行逻辑操作
                resultBean = excuter.excute(funcId, params);
                break;
            }
        }
        sendResponse(resultBean, requestId);
    }

    private void sendResponse(ResultBean result, String requestId ) {
        String data = ParseUtil.toJson(new ResponseBean(result.getCode(),
                result.getMsg(), result.getFuncId(), result.getResult()));
        // mContext未定义若使用全局context，必须保证该context非activity避免内存泄露
        OpenGatewayManager.getInstance(mContext).response(requestId, data);
        LogUtil.debug(TAG + "" + data);
    }

    public void sendNotify(String funcId, String data) {
        OpenGatewayManager.getInstance(mContext).notify(EVT_MUTE_STATUS, data);
    }
```

service的onCreate方法操作

```java
        GwRequestManager.getInstance().init(getApplicationContext());
        SystemSetting systemSetting = new SystemSetting(getApplicationContext());
        GwRequestManager.getInstance().addExucter(systemSetting);
```

## OKHTTP的构建实例拦截器问题

```java
public OkHttpClient getOkClient() {
    HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor(new HttpLoggingInterceptor.Logger() {
        @Override
        // 记录每次请求的log
        public void log(String message) {
            LogUtil.debug("message =" + message);
        }
    });

    loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.HEADERS);
    return new OkHttpClient.Builder()
            // 返回的okhttp client 请求有可能被重定向时使用addNetworkInterceptor,可使拦截器记录多次，而Application Interceptors只会记录一次
            .addNetworkInterceptor(loggingInterceptor)
            .build();
}
```

## 使用java解压缩

```java
    private void unZip(File srcFile, String destDirPath) throws RuntimeException {
        // 判断源文件是否存在
        if (!srcFile.exists()) {
            return;
        }
        // 开始解压
        ZipFile zipFile = null;
        try {
            zipFile = new ZipFile(srcFile);
            Enumeration<?> entries = zipFile.entries();
            while (entries.hasMoreElements()) {
                ZipEntry entry = (ZipEntry) entries.nextElement();
                LogUtil.debug(TAG + "unzip" + entry.getName());
                // 如果是文件夹，就创建个文件夹
                if (entry.isDirectory()) {
                    String dirPath = destDirPath + "/" + entry.getName();
                    new File(dirPath).mkdir();
                } else {
                    // 如果是文件，就先创建一个文件，然后用io流把内容copy过去
                    File targetFile = new File(destDirPath + "/" + entry.getName());
                    targetFile.createNewFile();
                    // 将压缩文件内容写入到这个文件中
                    InputStream is = zipFile.getInputStream(entry);
                    FileOutputStream fos = new FileOutputStream(targetFile);
                    int len;
                    byte[] buf = new byte[BUFFER_SIZE];
                    while ((len = is.read(buf)) != -1) {
                        fos.write(buf, 0, len);
                    }
                    // 关流顺序，先打开的后关闭
                    fos.close();
                    is.close();
                }
            }
        } catch (Exception e) {
            throw new RuntimeException("unzip error from ZipUtils", e);
        } finally {
            if (zipFile != null) {
                try {
                    zipFile.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

## 在使用AIDL创建实体时，如何使用自定义类型，和自定义类型集合

例子：

提示，集合中的类型也必须实现Parcelable

```java
    private NaviPoiBase poi;
    private List<NaviPoiBase> childPois;
    private List<Position> poiPolygonBounds;
    public NaviPoi() {
    }
    public NaviPoi(Parcel in) {
        poi = in.readParcelable(NaviPoiBase.class.getClassLoader());
        childPois = in.createTypedArrayList(NaviPoiBase.CREATOR);
        poiPolygonBounds = in.createTypedArrayList(Position.CREATOR);
    }
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeParcelable(poi, flags);
        dest.writeTypedList(childPois);
        dest.writeTypedList(poiPolygonBounds);
    }
```

## 项目中如何在启动时开启adb

项目启动时会加载init.rc

```c
# Always start adbd on userdebug and eng builds
on property:ro.debuggable=1
    write /sys/class/android_usb/android0/enable 1
    # start adbd

# Restart adbd so it can run as root
on property:service.adb.root=1
    write /sys/class/android_usb/android0/enable 0
    # restart adbd
    write /sys/class/android_usb/android0/enable 1
```

可以看到start adbd 和 restart adbd 都被注释掉了，所以启动时没有adb设备

取消注释即可。

## android10以上无法在设备上创建文件夹

```xml
在<application

</application>
中加上这句，配置存储权限
android:requestLegacyExternalStorage="true"
```

**使用Gradle无法下载**

在工程Gradle配置google()和镜像

```xml
buildscript {
    repositories {
        google()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/google' }
        maven { url 'http://maven.aliyun.com/nexus/content/repositories/gradle-plugin' }
        maven{
            url 'http://maven.google.com'
            name 'Google'
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.2'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        maven{url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        maven{
            url 'http://maven.google.com'
            name 'Google'
        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

## 其他项目集成aar包出现无法访问方法的情况

* 可以查看这个集成的aar包是否已经成为系统的动态jar包，所有项目尽管集成aar包，也需要首先访问系统的同名aar包。

解决方法：将自己的包替换系统引用工程的aar包，编译后即可找到方法

## 想使用Get方法在URL上发送JSON

可以使用URLEncoder.encode进行转义

```java
                NLog.i(TAG, "Method: " + data.getMethod());
                String params = "Method=" + data.getMethod() + "&RequestId=" + data.getRequestId() + "&EventName=" + data.getEventName() + "&ChannelId=" + data.getChannelId() + "&Body={}";
                uri = "http://" + host + ":" + port + path + "?";
                try {
                    uri += URLEncoder.encode(params, "UTF8");
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
                NLog.i(TAG, "uri: " + uri);
```

## 使用Glide将String图片地址显示到控件中

```java
Glide.with(mAlbumImageView).asDrawable().load(info.getpCoverArtURI()).into(mAlbumImageView);
```

## 同一个进程获取单例后进行注册回调发现问题

比如我在写信号回调两个类同属一个进程获取实例后，调用registerListener进行注册发现这是单纯的赋值listener，那么这两个类不管谁注册第二个总会覆盖掉，所以导致第一个类的回调无法收到，改成将listener，add进list即可。

```java
    public boolean registerListener(onNotifyListener listener) {
        this.mOnNotifyListener = listener;
        return true;
    }
```

## 遇到无法调用在framework中新加的方法，该怎么操作

因为framework在android中作为动态.jar包的存在，想调用framework中已经存在的方法或者是新加的方法需要在java包下，新建一个android包用于调用

例如如下格式：

![image-20220331104928251](/typora-user-images/image-20220331104928251.png)

这个类里的内容如下：

只需要写需要调用方法的方法签名（不用写方法体，写方法名和参数），让编译器通过编译，运行时android系统会自动引用framework中的方法。

```java
package android.location;

import android.os.Bundle;

public class LocationManager {
    public void sendLocationChange(Bundle bundle) {

    }
    public void requestLocationUpdates(String provider, long minTime, float minDistance,
                                       LocationListener listener) {

    }
}
```

## 简单的获取定位demo

```java
LocationManager locationManager;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(com.hht.microservice.R.layout.activity_main);
    startService(new Intent(MainActivity.this, MicroService.class));
    //        startService(new Intent(this, OpenGateWayService.class));
    getSystemLocation();

}

private void getSystemLocation() {
    if (locationManager == null) {
        locationManager = (LocationManager) this.getSystemService(Context.LOCATION_SERVICE);
        if (ActivityCompat.checkSelfPermission(this, Manifest
                                               .permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && ActivityCompat
            .checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            return;
        }
        // 可使用GPS或者network获取位置，GPS比较准确，而network不准确。
        locationManager.requestLocationUpdates("gps", 5000, 0, locationListener);
    }
}

LocationListener locationListener = new LocationListener() {
    @Override
    public void onLocationChanged(Location location) {
        double latitude = location.getLatitude();
        double longitude = location.getLongitude();
        Log.d("TAG", "onLocationChanged lat: " + latitude + "lon: " + longitude);
    }

    @Override
    public void onStatusChanged(String provider, int status, Bundle extras) {

    }

    @Override
    public void onProviderEnabled(String provider) {

    }

    @Override
    public void onProviderDisabled(String provider) {

    }
};
// 主屏模拟定位
mHandler = new Handler(Looper.getMainLooper()) {
    @Override
    public void handleMessage(Message msg) {
        final String eventName = "LocationChange";
        Map<String, String> result = new HashMap<>();
        putMethod(result, eventName);
        result.put("Data", System.currentTimeMillis() / 1000000.0 + "," + System.currentTimeMillis() / 1000000.0);
        result.put("Timestamp", String.valueOf(System.currentTimeMillis()));
        EventDispatcher.getInstance().publishEvent(eventName,
                                                   new TextWebSocketFrame(JSON.toJSONString(result)));
        Log.d(TAG, "-------------------> onLocationChanged: " + eventName);
        Log.d(TAG, "------------> notify: " + JSON.toJSONString(result));
        mHandler.sendEmptyMessageDelayed(0, 5000);
    }
};
mHandler.sendEmptyMessage(0);
```

## Gerrit报错

* 当一笔代码已经入库，在带这这笔gerrit去编的时候就会报如下错误。

```sh
在cherry-pick的时候出现以下错误提示，是对同一提交重复做cherry-pick引起的。
# On branch***
# You are currentlycherry-picking.
#   (all conflicts fixed: run "gitcommit")
#
nothing to commit,working directory clean
The previouscherry-pick is now empty, possibly due to conflict resolution.
If you wish to commitit anyway, use:
    git commit --allow-empty
Otherwise, please use'git reset'

分析：没有冲突输出，提示如果要提交，需要做空提交，说明这次cherry-pick的内容可能已经在这个分支上提交过了。
验证：1、查看这个哈希值所修改的文件  2、查看某个文件的修改log     3、有相同的提交注释
```

* 如果赋值OK的情况下通过序列化后发现参数名不一致考虑是不是开启了混淆

* 如下这种错为gerrit服务器没有这种路径，尝试更改本地.gradle输出APK文件名

```sh
[exec]: cp /home/iovadmin/.space_jenkins/workspace/FsemGestureService/FsemGestureService/app/build/outputs/apk/release/FsemGestureService.apk .
cp: cannot stat '/home/iovadmin/.space_jenkins/workspace/FsemGestureService/FsemGestureService/app/build/outputs/apk/release/FsemGestureService.apk': No such file or directory
```

```java
        applicationVariants.all { variant ->
            variant.outputs.all { output ->
                def outputFile = output.outputFile
                def fileName
                if (outputFile != null && outputFile.name.endsWith('.apk')) {
                    if (variant.buildType.name == 'release') {
                    -   fileName = "GestureService.apk"
                    +   fileName = "FsemGestureService.apk"
                    } else if (variant.buildType.name == 'debug') {
                    -   fileName = "GestureService_Debug.apk"
                    +   fileName = "FsemGestureService_Debug.apk"
                    }
                    outputFileName = fileName
                }
            }
        }
```

## 微服务绑定问题

若有两个微服务，A和B，一个项目同时用到了两个微服务。一些功能是在A发送，另一些是在B，如果A连接失败了B连接未失败，但连接A功能的初始化是在回调连接B里去做的，这样会导致B连接未失败就永远无法初始化需要A传输的功能。

分开进行判断连接成功的回调。

## repo sync更新git仓库报错：已拒绝，会破坏现有的标签

碰到好几次这种情况，这个问题的原因是，上一次拉代码将一个远程的tag拉到了本地，随后远程的tag被更新了，这就导致远程的tag和我们本地的tag有冲突。

我的解决方式是，先删除本地的这个tag，以远程的为准。假设报错的tag是zqb_all_tag

即先

```bash
repo forall -c git tag -d zqb_all_tag
```

再重新

```bash
repo sync
```

## 规范的HandlerThread写法

```java
private Handler mOnFailedRetryHandler;
private final HandlerThread mHandlerThread = new HandlerThread(TAG);

    @Override
    public void onCreate() {
        NLog.i(TAG, "onCreate");
        ServiceManager.addService("com.hht.microservice", microStub);
        mHandlerThread.start();
        mOnFailedRetryHandler = new Handler(mHandlerThread.getLooper());
        mClient = new HttpClient(mOnFailedRetryHandler);
        mClient.start("192.168.4.4", 9999,  );
    }
    @Override
    public void onDestroy() {
        NLog.i(TAG, "onDestroy");
        super.onDestroy();
        mHandlerThread.quitSafely();
        if (mOnFailedRetryHandler != null) {
            mOnFailedRetryHandler.removeCallbacksAndMessages(null);
            mOnFailedRetryHandler = null;
        }
    }
```

## 使用WindowManager

WindowManager相关参数参考：https://blog.csdn.net/wjr1949/article/details/71054975

```java

    // 黑天白夜切换容易导致重复addView或者更新不存在的Layout，应该每次init的时候都remove掉Layout
    @SuppressLint("ClickableViewAccessibility")
    private void initChildLockManager() {
        removeChildLockWindow();
        mChildLockWindow = (WindowManager) this.getSystemService(WINDOW_SERVICE);
        LayoutInflater inflater = LayoutInflater.from(this);
        mChildLockLayout = (RelativeLayout) inflater.inflate(R.layout.child_lock_layout, null);
        if (mChildLockStatus) {
            NLog.d(TAG, "child lock is open");
            addChildLockView();
        }
    }
    // 已经addview的解决方法。
    private void addChildLockView() {
        removeChildLockWindow();
        mChildLockWindow.addView(mChildLockLayout, getLayoutParams());
    }

    private void removeChildLockWindow() {
        if (mChildLockLayout == null) {
            return;
        }
        if (mChildLockLayout.getParent() != null) {
            mChildLockWindow.removeViewImmediate(mChildLockLayout);
        }
    }

    public void onChildLockModeChanged(boolean mode) {
        this.mChildLockStatus = mode;
        if (mode) {
            mMaskManager.closeHiphiMask();
            addChildLockView();
        } else {
            if (mChildLockWindow != null) {
                removeChildLockWindow();
            }
        }
    }

    private WindowManager.LayoutParams getLayoutParams() {
        WindowManager.LayoutParams lp = new WindowManager.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT,
                WindowManager.LayoutParams.TYPE_APPLICATION,
                WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                        | WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL,
                PixelFormat.TRANSLUCENT);
        lp.setTitle("ChildLock");
        lp.windowAnimations = 0;
        lp.gravity = Gravity.CENTER;
        return lp;
    }

```

使用Glide加载图片

```java
Glide.with(mAlbumImageView).asDrawable()// 转为Drawable
    .placeholder(mAlbumImageView.getDrawable())// 切换图片的瞬间用于显示上一张图片，多次更新数据的时候避免闪烁问题，占位属性内部可以设置默认图片比如placeholder(R.mipmap.ic_launcher)
    .load(info.getpCoverArtURI())// 这里你可以指定哪个图片应该被加载，同上它会是一个   字符串的形式表示一个网络图片的 URL
    .into(mAlbumImageView);// 你的图片会显示到对应的 ImageView 中。
```

```java
// touch事件中：
    view.getParent().requestDisallowInterceptTouchEvent(true);  // 告诉父view不要拦截此事件
    view.getParent().requestDisallowInterceptTouchEvent(false);  // 告诉父view可以拦截此事件
```

用FragmentTransaction来控制fragment的hide和show时，那么这个方法就会被调用。每当你对某个Fragment使用hide或者是show的时候，那么这个Fragment就会自动调用这个方法。

```java
@Override
public void onHiddenChanged(boolean hidden) {
    super.onHiddenChanged(hidden);
    NLog.i(TAG, "onHiddenChanged hidden:" + hidden);

    if (!hidden) {//重新显示到最前端中
        sendNotify(Constants.ACTION_APP_LOAD_DATA, Constants.INVALID_VALUE);
    }
}
```

Gson序列化List

```java
List<RequestSecondScreenActivity> list = gson.fromJson(data, new TypeToken<ArrayList<RequestSecondScreenActivity>>() {}.getType());
```

TextView增加一个小图标方法

```java
                mShortcutPlayText.setCompoundDrawablesWithIntrinsicBounds(ResourcesCompat.getDrawable(getResources(), Utils.getResId(R.drawable.ui_ic_btn_shortcut_play_selector), null), null, null, null);
```

## Android Studio报错：the minSdk version should not be declared in the android manifest file 解决办法

删除AndroidManifest.xml中的

```xml
    <uses-sdk
        android:minSdkVersion="24"
        android:targetSdkVersion="27" />
```

BuildConfig.dex冲突：

```sh
Caused by: com.android.tools.r8.utils.b: Error: /home/tsdl/Fsem/fsemlauncher/app/build/intermediates/project_dex_archive/release/out/com/hht/launcher/BuildConfig.dex, Type com.hht.launcher.BuildConfig is defined multiple times: /home/tsdl/Fsem/fsemlauncher/app/build/intermediates/project_dex_archive/release/out/com/hht/launcher/BuildConfig.dex, /home/tsdl/Fsem/fsemlauncher/MultiMediaLabrary/build/.transforms/fd72623b18dd208da5b9995f8a9f639a/classes/classes.dex
```

解决方法：AndroidManifest.xml package名称冲突，改个名后面加个1

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.hht.launcher1">
</manifest>
```

## Android 在push APK的时候发现总是起不来

* 调查是否有已存在的apk安装，uninstall一下

* 调查安装app目录下同级目录是否存在apk，或上一级目录存在apk
  
# Android Studio中XML文件属性不提示的解决办法！

点击菜单file-project structure，将各个module的compile sdk version版本降低至29。

# repo init 报错

```shell
Traceback (most recent call last):
  File "/home/tsdl/.repo/repo/main.py", line 513, in <module>
    _Main(sys.argv[1:])
  File "/home/tsdl/.repo/repo/main.py", line 478, in _Main
    _CheckWrapperVersion(opt.wrapper_version, opt.wrapper_path)
  File "/home/tsdl/.repo/repo/main.py", line 209, in _CheckWrapperVersion
    exp = Wrapper().VERSION
  File "/home/tsdl/.repo/repo/wrapper.py", line 29, in Wrapper
    _wrapper_module = imp.load_source('wrapper', WrapperPath())
IOError: [Errno 2] No such file or directory
```

看一看自己的上层目录是否含有.repo文件，有则给他删了

Andorid查找差异命令
```shell
pm dump com.seres.settings | grep version
pm dump com.seres.publicadapter | grep version
pm path com.seres.publicadapter
```

将该包引入系统使用implementation

已经引入系统的包使用compileOnly编译通过即可

JAVA无论如何代码都会走到finally{}代码块中

# Android系统个人总结

    请求Activity或Service启动首先会请求AMS，而AMS判断该进程是否存在，如果不存在的话就走AMS的startProcessLocked方法最后需要与Zygote创建socket连接，而Zygote会runSelectLoop无限循环等待是否有Socket的连接请求，如果有则拿到参数去fork一个进程，该进程会通过反射去拿到对应的ActivityThread的main方法，并且该main方法实在异常抛出的时候去执行的，异常是由ZygoteInit的main方法去捕获的，这种抛出异常的方式会清除所有的设置过程需要的堆栈帧，在ActivitThread的main方法中，还会创建主线程的H类，用于主线程内部的消息循环，执行完之后还需要调用binder的ProcessState::startThreadPool函数来启动线程池，还需要调用IPCThreadState::self()->joinThreadPool将当前线程注册到binder驱动程序中，这些都是在Zygote里完成的，而进程又是从Zygote孵化而来所以可以使用binder进行通信，而NativeService是init进程拉起来的需要手动执行binder线程池，并且手动注册binder驱动。

    在桌面上点击的所有APP都是Launcher请求AMS去启动该应用程序的。AMS会首先检查调用者权限，根据Intent的flag决定启动模式，最后通过binder的方式往ApplicationThread发，而ApplicationThread继承IApplicationThread.Stub，才会收到AMS的回调，ApplicationThread作为ActivityThread的内部类，因为ApplicationThread是一个Binder通信接口他在binder的线程池中不在主线程，所以要通过H的Handler消息到主线程，在主线程中启动Activity，创建Activity的上下文，通过类加载器去启动Activity，最后执行Activity的onCreate()方法。

    Window分为三种窗口应用程序窗口（1-99），子窗口（1000-1999），系统窗口（2000-2999），会根据其中的type值显示窗口次序

窗口添加过程主要是，配置视图属性，还有Flag，最后通过WindowManager的addViews（）而方法的实现在WindowManagerImpl，而WindowManagerImpl最后回到WindowManagerGlobal中，WindowManagerGlobal中维护了三个列表，view列表，布局参数列表，ViewRootImpl列表，通过addToDisplay进行Binder通信，addToDisplay调用了WMS的addWindow（），将自身作为参数（Session），每个应用程序对应一个Session，WMS通过ArrayList保存这些Session，WMS会为这个添加的窗口分配Surface，并确定显示次序，负责界面的是Surface，WMS会把管理的Surface交给SurfaceFlinger处理，SurfaceFlinger会将这些Sufrace混合并绘制到屏幕上。

WMS的创建会直接调用WMS的main方法

### 为什么不去强转呢？子类去转父类啊

在一些开发中我们发现如果我们拿到了子类，但子类缺没有父类的方法，我们想调用父类特有的方法该怎么办，**为什么不去强转呢？子类去转父类啊**

### Android使用观察者模式进行解耦

编写观察者模式接口

```java
package com.seres.adapter;

import java.util.Observable;

public class ObservableData<T> extends Observable {
    private T data;

    public ObservableData(T initData) {
        data = initData;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
        setChanged();
        notifyObservers(data);
    }
}

```

## Kotlin

懒加载时会遇到什么问题

看一下这张图

![image-20241227150120431](/typora-user-images/image-20241227150120431.png)

这个时候调用getKey会lateinit property key has not been initialized

因为RwKeyFun的构造函数要等待BaseKeyFuncation完成构造函数走完，那么怎么可以等待初始化完成后在调用呢

可以对setKey进行重写，在重写中去调用getKey的相关方法

## SOMEIP ITEH-ONE

在使用SOMEIP工具进行模拟的时候需要把模拟端的网络适配器修改为对应的IP，如果你要模拟服务器你就修改为服务器的IP如果你要模拟客户端就修改成客户端的IP
