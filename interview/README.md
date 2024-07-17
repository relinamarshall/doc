---
description: Java Memory Model（JMM）
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# Java内存模型

## 什么是Java内存模型

JMM本身只是一个抽象的概念，并不真实存在，它描述的是一种规则或规范；通过这组规范，定义了程序中对各种变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。需要每个JVM的实现都要遵守这样的规范；有了JMM规范的保障后，并发程序运行在不同虚拟机上时，得到的程序结果才是安全可靠可信赖的，如果没有JMM内存模型来规范，那经过不同JVM翻译之后，就可能出现，运行结果不相同或不正确。

简单说JMM就是屏蔽了各种硬件和操作系统的访问差异，保证Java程序在各种平台下对内存的访问都能保证效果一致的机制规范。

JMM还抽象出主存储器(Main Memory)和工作存储器(Working Memory)两种：

* 主存储器是实例对象所在的区域，所有实例都存在于主存储器内，主存储器是所有线程共享的
* 工作存储器是线程所拥有的作业区，每个线程都有其专用的工作存储器；工作存储器存有主存储器中必要部分的拷贝，称为工作拷贝(Working Copy)

所以线程无法直接对主内存进行操作，线程A想要和线程B通信，只能通过主存进行。

## 三大特性

JMM有三大特性：原子性、可见性、有序性

### 原子性

JMM保证了对共享变量的读取和写入可以被视为原子操作

为支持JMM，Java定义了8种原子操作，用来控制主存和工作内存之间的交互

* read读取：作用于主内存，将共享变量从主内存传送到线程的工作内存中
* load载入：作用于工作内存，把read读取的值放到工作内存中的副本变量中
* store存储：作用于工作内存，把工作内存中的变量传送到主内存中
* write写入：作用于主内存，把从工作内存中store传送过来的值写到主内存变量中
* use使用：作用于工作内存，把工作内存的值传递给执行引擎，当虚拟机遇到需要使用这个变量的指令时，就会执行这个动作
* assign赋值：作用于工作内存，把执行引擎获取到的值赋值给工作线程中的变量，当虚拟机遇到给变量赋值的指令时，就执行此操作
* lock锁定：作用于主内存，把变量标记为现场独占状态
* unlock解锁：作用于主内存，它将释放独占状态

### 可见性

多个线程访问共享变量时，一个线程如果修改变量值，在刷新到主内存之前，其他线程不一定能立即看到这个修改。

在JVM中，栈负责运行（主要是方法），堆中负责存储（比如new的对象），JVM运行程序的实体是线程，每个线程创建时，JVM都会为其创建一个工作内存，工作内存是每个现场私有数据区域；而Java内存模型中规定，所有变量都存储在主内存中，主内存是共享内存区域，所有线程都可以访问；但线程对变量的操作（读写）必须在自己工作内存中进行，首先要将变量从主内存拷贝到自己的工作内存空间，然后对变量操作，操作完成后，再将变量回写到主内存；由于不能直接操作主内存的变量，各个线程工作内存中存储着主内存变量副本，因此不同线程无法直接访问对方工作内存，线程间通信必须通过主内存完成。

**同步的规定**

* 线程解锁前，必须把共享变量的值刷新回主内存
* 线程加锁前，必须将主内存的最新值读取到自己的工作内存
* 加锁解锁是同一把锁

**可见性问题（缓存一致性问题）**:指在未加同步锁的多线程环境下，同时修改共享变量，导致结果与预期不符的问题。

{% tabs %}
{% tab title="代码复现" %}
{% code overflow="wrap" lineNumbers="true" %}
```java
public class Demo {
    private static int num;
    public static void main(String[] args) throws InterruptedException {
        Thread[] threads = new Thread[100];
        CountDownLatch latch = new CountDownLatch(threads.length);
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    num++;
                }
                latch.countDown();
            });
        }
        Arrays.stream(threads).forEach(Thread::start);
        latch.await();
        System.out.println("预期值:" + threads.length * 10000 + ",实际值:" + num);
        // 预期值:1000000,实际值:189067
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="同步锁" %}
{% code overflow="wrap" lineNumbers="true" %}
```java
public class Demo {
    private static int num;
    public static void main(String[] args) throws InterruptedException {
        Thread[] threads = new Thread[100];
        CountDownLatch latch = new CountDownLatch(threads.length);
        ReentrantLock lock = new ReentrantLock();
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    lock.lock();
                    num++;
                    lock.unlock();
                }
                latch.countDown();
            });
        }
        Arrays.stream(threads).forEach(Thread::start);
        latch.await();
        System.out.println("预期值:" + threads.length * 10000 + ",实际值:" + num);
        // 预期值:1000000,实际值:1000000
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

### 有序性

在本(单)线程内执行顺序按照代码的先后顺序来执行，所有的操作都是有序的，线程内似表现为串行；但在多线程内，所有的操作都是无序的。

**重排序**：处理器为提高程序运行效率，提高并行效率，可能会对代码进行优化，编译器认为重排序后程序的执行效率更优，这样一来代码执行顺序就未必是编写代码时候的顺序，在多线程情况下就可能会出错；但它也需要满足以下两个条件

* 在的单线程环境下不能改变程序运行的结果
* 存在数据依赖关系的不允许重排序

**数据依赖性**：如果两个操作访问同一个变量，且这两个操作中有一个为写，此时这两个操作存在数据依赖性；分为以下列三种类型，下面三种情况，只要重排两个操作执行顺序，程序的执行结果就会发生改变；所以编译器和处理器不会改变单线程或单处理器环境下存在数据依赖性操作的执行顺序；在多处理器或多线程之前的数据依赖性不被编译器和处理器考虑。

<table><thead><tr><th>名称</th><th width="156">代码示例</th><th>说明</th></tr></thead><tbody><tr><td>写后读</td><td>a = 1;b = a;</td><td>写一个变量之后，再读这个变量</td></tr><tr><td>写后写</td><td>a = 1;a = 2;</td><td>写一个变量之后，再写这个变量</td></tr><tr><td>读后写</td><td>a = b;b = 1;</td><td>读一个变量之后，再写这个变量</td></tr></tbody></table>

**有序性问题（指令重排序）**:指在多线程环境下，由于执行语句重排序后，重排序代码块没有执行完，就切换到其他线程，导致计算结果与预期不符的问题；这就是编译器的编译优化给并发编程带来的有序性问题。

{% tabs %}
{% tab title="代码复现" %}
{% code overflow="wrap" lineNumbers="true" fullWidth="false" %}
```java
public class Demo {
    private static int a, b, x, y;
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1_0000_0000; i++) {
            a = 0;
            b = 0;
            x = 0;
            y = 0;
            CountDownLatch latch = new CountDownLatch(2);
            Thread t1 = new Thread(() -> {
                a = 1;
                x = b;
                latch.countDown();
            });
            Thread t2 = new Thread(() -> {
                b = 1;
                y = a;
                latch.countDown();
            });
            t1.start();
            t2.start();
            latch.await();
            if (x == 0 && y == 0) {
                System.err.println("第" + i + "次出现(x=0,y=0)");
                break;
            }
            // 第144654次出现(x=0,y=0)
        }
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="禁止指令重排" %}
{% code overflow="wrap" lineNumbers="true" %}
```java
public class Demo {
    private static volatile int a, b, x, y;
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 1_0000_0000; i++) {
            a = 0;
            b = 0;
            x = 0;
            y = 0;
            CountDownLatch latch = new CountDownLatch(2);
            Thread t1 = new Thread(() -> {
                a = 1;
                x = b;
                latch.countDown();
            });
            Thread t2 = new Thread(() -> {
                b = 1;
                y = a;
                latch.countDown();
            });
            t1.start();
            t2.start();
            latch.await();
            if (x == 0 && y == 0) {
                System.err.println("第" + i + "次出现(x=0,y=0)");
                break;
            }
        }
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}



