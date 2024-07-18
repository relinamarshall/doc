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

Java内存模型是Java规范的一部分，主要定义了在多线程环境中如何通过内存进行交互和通信，并发访问共享变量时的规范。JMM解决了以下几个关键问题：

1. **内存可见性**：一个线程对变量的写操作何时对另一个线程可见。
2. **指令重排序**：允许编译器和处理器在不改变程序正确性前提下对指令进行重排序，以优化性能。
3. **同步机制**：提供了一套规则来管理多线程之间的同步操作，如锁（`synchronized`）、volatile变量等。

简单说JMM就是屏蔽了各种硬件和操作系统的访问差异，保证Java程序在各种平台下对内存的访问都能保证效果一致的机制规范。

JMM还抽象出**主存储器(Main Memory)**和**工作存储器(Working Memory)**两种：

* **主存储器**是实例对象所在的区域，所有实例都存在于主存储器内，主存储器是所有线程共享的
* **工作存储器**是线程所拥有的作业区，每个线程都有其专用的工作存储器；工作存储器存有主存储器中必要部分的拷贝，称为工作拷贝(Working Copy)

所以线程无法直接对主内存进行操作，线程A想要和线程B通信，只能通过主存进行。

总结：Java语言中用于描述多线程并发访问共享变量时的规范。它定义了线程如何与**主内存**和**工作内存**进行交互，以及对**共享变量**的访问和操作应该遵循的规则。在此之前，主流程序语言（如 C/C++等）直接使用物理硬件和操作系统的内存模型，因此，会由于不同平台上内存模型的差异，可能导致程序在一套平台上并发完全正常，而在另外一套平台上并发访问却经常出错，这导致在某些场景下必须针对不同的平台来编写不同的代码。 **JMM 屏蔽了不同处理器内存模型的差异，它在不同的处理器平台之上为 Java 程序员呈现了一个一致的内存模型。**通过JMM的规范，Java程序员可以利用各种同步机制（如synchronized、volatile等）来控制线程之间的互动和数据共享，从而编写正确且高效的多线程程序。

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

**数据依赖性**：如果两个操作访问同一个变量，且这两个操作中有一个为写，此时这两个操作存在数据依赖性；分为以下列三种类型，下面三种情况，只要重排两个操作执行顺序，程序的执行结果就会发生改变；所以编译器和处理器不会改变单线程或单处理器环境下存在数据依赖性操作的执行顺序；在多处理器或多线程之间的数据依赖性不被编译器和处理器考虑。

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

### **volatile原理和实现机制**

观察加volatile关键字和没有加入volatile关键字时所生产的汇编代码发现，加volatile关键字时，会多出一个lock前缀指令；lock前缀指令相当于一个内存屏障(也称内存栅栏)，内存屏障会提供3个功能

* 他确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障后面；即在执行到内存屏障这句指令时，在它之前的操作已经全部完成
* 会强制将对缓存的修改操作立即写入主存
* 如果是写操作，它会导致其他CPU中对应的缓存行无效

## as-if-serial语义

不管怎么重排序(编译器和处理器都是为了提高并行度)，(单线程)程序的执行结果不能改变；编译器和处理器必须遵守as-if-serial语义

## happens-before

### happens-before原则

happens-before原则是JMM中的一个重要部分，用于定义操作之间的顺序关系。具体来说，如果一个操作happens-before另一个操作，那么第一个操作的结果对第二个操作是可见的，并且第一个操作的执行顺序在第二个操作之前。

### 与JMM的关系

1. **内存可见性保障**：happens-before关系确保了操作之间的可见性。如果一个操作happens-before另一个操作，那么第一个操作的结果对第二个操作是可见的。这样可以避免可见性问题。
2. **指令重排序限制**：编译器和处理器可以对指令进行重排序以优化性能，但这种重排序不能违反happens-before规则。happens-before关系限制了重排序的范围，以确保程序的正确性。
3. **同步机制实现**：JMM通过happens-before原则来定义同步机制的行为。例如，`synchronized`块和`volatile`变量都依赖于happens-before原则来确保线程之间的操作顺序和内存可见性。

### happens-before规则

JMM定义了一系列具体的happens-before(之前发生)规则，包括但不限于：

1. **程序顺序规则**：在一个线程内，按照程序代码顺序，前面的操作happens-before后面的操作。
2. **监视器锁规则**：一个解锁操作happens-before于后续对同一个锁的加锁操作。
3. **volatile变量规则**：对一个volatile变量的写操作happens-before于后续对这个volatile变量的读操作。
4. **线程启动规则**：对Thread对象的start()方法的调用happens-before于该线程开始的所有操作。
5. **线程终止规则**：线程中的所有操作happens-before于对线程的终止检测，比如通过Thread.join()方法结束、Thread.isAlive()返回检测到等。
6. **中断规则**：对线程interrupt()方法的调用happens-before于被中断线程的代码检测到中断事件的发生。
7. **对象终结规则**：一个对象的构造函数执行完成happens-before于它的finalize()方法的开始。
8. **传递性**：如果A happens-before B，且B happens-before C，那么A happens-before C。

{% code overflow="wrap" lineNumbers="true" %}
```java
public class HappensBeforeExample {
    private int a = 0;
    private volatile boolean flag = false;

    public void writer() {
        a = 1;                // 写普通变量
        flag = true;          // 写volatile变量
    }

    public void reader() {
        if (flag) {           // 读volatile变量
            int i = a;        // 读普通变量
            System.out.println(i); // 保证i的值为1
        }
    }

    public static void main(String[] args) {
        HappensBeforeExample example = new HappensBeforeExample();
        Thread t1 = new Thread(example::writer);
        Thread t2 = new Thread(example::reader);
        t1.start();
        t2.start();
    }
}

```
{% endcode %}
