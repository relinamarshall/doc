---
description: 出处：https://space.bilibili.com/439507793
---

# 经验总结

## 1、分布式

### 1.1、Nginx负载均衡算法

**轮询（Round Robin）**：这是默认的负载均衡算法。Nginx按顺序将每个新的请求分发给后端服务器，依次循环。这意味着每个后端服务器都会接收到大致相等数量的请求，适用于负载均衡下服务器性能相似的情况。

示例配置：

```nginx
upstream backend {
    server server1.example.com;
    server server2.example.com;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

**加权轮询（Weighted Round Robin）**：这个算法允许为每个后端服务器分配一个权重值，Nginx按权重分发请求。具有更高权重的服务器会接收到更多的请求，适用于服务器性能不均匀的情况。

示例配置：

```nginx
upstream backend {
    server server1.example.com weight=3;
    server server2.example.com weight=1;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

**IP哈希（IP Hash）**：Nginx使用客户端的IP地址来计算一个哈希值，并将请求分发给特定的后端服务器。这对于确保特定客户端的请求一直路由到同一个后端服务器很有用，例如在会话保持的情况下。

示例配置：

```nginx
upstream backend {
    ip_hash;
    server server1.example.com;
    server server2.example.com;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

**最小连接数（Least Connections）**：Nginx会将请求发送到当前连接数最少的后端服务器。这有助于避免某些服务器上的负载过大，但需要Nginx跟踪每个连接的状态。

示例配置：

```nginx
http {
    upstream backend {
        least_conn;  # 使用最少连接算法
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }
    server {
    listen 80;
    server_name mywebsite.com;

    location / {
        proxy_pass http://backend;
    }
}
```

### 1.2、HTTP会话（HTTP session）

是一种在Web开发中常用的概念，用于跟踪用户与Web应用程序之间的交互状态。HTTP会话允许Web应用程序在多个HTTP请求之间保持用户的身份和状态信息，以便提供个性化的用户体验和持久性的数据存储。

以下是HTTP会话的主要特征和工作原理：

1. **状态保持**：HTTP是一种无状态协议，这意味着每个HTTP请求都是独立的，服务器不会自动记住之前的请求或用户信息。为了在多个请求之间保持状态，需要使用HTTP会话来存储和检索用户的状态信息。
2. **会话标识符**：在HTTP会话中，服务器通常会为每个用户分配一个唯一的会话标识符（Session ID），通常以cookie或URL参数的形式发送给客户端浏览器。这个会话标识符用于识别用户的会话，从而关联之前和之后的请求。
3. **服务器端存储**：Web服务器通常会将用户的会话数据存储在服务器端，可以是内存、数据库或其他持久性存储介质。这些数据包括用户的身份信息、登录状态、购物车内容、用户首选项等。
4. **会话生命周期**：HTTP会话有一个生命周期，通常由用户的登录和注销来定义。会话可以在用户登录后创建，并在用户注销、超时或关闭浏览器时终止。会话超时时间通常由应用程序配置决定。
5. **用户身份验证**：HTTP会话通常与用户身份验证相关联，以确保用户已经登录并且具有访问权限。一旦用户进行身份验证，他们的会话可以包括与其身份相关的信息，以便在后续请求中进行授权检查。
6. **会话管理**：Web应用程序通常使用编程语言和框架提供的会话管理工具来创建、管理和终止会话。这包括创建新会话、存储和检索会话数据以及清除会话数据等操作。

HTTP会话是Web应用程序中实现用户认证、个性化体验和状态跟踪的关键机制之一。通过使用HTTP会话，Web应用程序可以跟踪用户在站点上的行为，并根据其活动提供不同的内容和功能。但是，需要小心处理会话数据，以确保用户的隐私和安全。

### 1.3、B/S 架构

B:浏览器，S:网页服务器，传输协议：HTTP，内容协议：HTML

<figure><img src="../.gitbook/assets/BS架构" alt=""><figcaption><p>B/S架构</p></figcaption></figure>

Web Server 举例： Tomcat 、 Jetty

### 1.4、分布式下的文件存储？

问题：上传文件到A服务器，那么负载均衡之后，如果不是Hash路由。那么请求到B服务器怎么办？

解决方案： 多服务器共享文件的方案：需要单独的文件存储集群。比如阿里云的OSS（可以简单理解成云盘）。

### 1.5、主从同步机制的缺点

主从同步是很多服务都有的概念。比如MySQL, Redis。

问题是：理论上小时间窗口可能从库读不出数据。

解决方案是：极端一致性的业务可以读主库，也就是写主库读主库。其他的读从库。

### 1.6、全链路日志

全链路日志（End-to-End Logs）在分布式系统和微服务架构中具有重要意义，它可以提供以下多个方面的价值：

* **故障排查和问题定位**：全链路日志允许开发人员和运维团队跟踪整个请求或事务的执行路径。当系统中发生故障或异常时，可以使用全链路日志追踪问题的根本原因，找到导致问题的具体组件或服务，快速定位和解决问题。
* **性能监测和调优**：通过分析全链路日志，可以了解请求的执行时间、延迟和吞吐量等性能指标。这有助于发现性能瓶颈、优化关键路径和提高系统的响应速度。
* **事务追踪**：全链路日志记录了一个请求或事务在系统中的每个步骤，包括调用其他服务、访问数据库等。这对于跟踪事务的处理流程和了解每个步骤的状态非常有用。
* **监控和警报**：全链路日志可以用于设置监控和警报规则。当系统中的某些条件达到或超过阈值时，可以触发警报，通知管理员或自动化系统采取措施。
* **安全审计**：全链路日志记录了用户的操作和访问历史，可以用于安全审计和合规性监督。它有助于检测潜在的安全问题和不正常的行为。
* **容量规划**：通过全链路日志，可以分析系统的负载情况和资源使用情况，帮助规划和预测系统的容量需求。
* **改进用户体验**：全链路日志可以提供对用户行为和体验的洞察，帮助优化用户界面和用户交互，提供更好的用户体验。
* **版本追踪**：对于分布式系统的不同版本或部署，全链路日志可以用于追踪不同版本之间的行为和性能差异。

总之，全链路日志是一种强大的工具，可以帮助组织更好地了解其分布式系统的运行情况，从而更快速地响应问题、改进性能、提高安全性，并提供更好的用户体验。在大规模和复杂的分布式环境中，全链路日志对于保持系统的健壮性和可维护性至关重要。

## 2、Java基础

### **2.1**、**面试题finallize、finally、final区别。**

**final：**

1.  **在变量上声明：** 通过在变量前面添加 `final` 关键字，可以声明一个不可变的变量，即一旦被赋值，其值不能再次修改。

    final int x = 10; // 不可变的整数变量 final String name = "John"; // 不可变的字符串变量
2.  **在方法上声明：** 通过在方法定义前面添加 `final` 关键字，可以声明一个方法为不可重写的方法，子类不能覆盖该方法。

    ```java
     public final void myMethod() {
         // 不可被子类重写的方法
     }
    ```
3.  **在类上声明：** 通过在类定义前面添加 `final` 关键字，可以声明一个类为不可继承的类，即该类不能被其他类继承。

    ```java
     final class MyClass {
         // 不可被继承的类
     }
    ```
4.  **在实例变量上声明：** 在类中的实例变量前面添加 `final` 关键字可以声明实例变量为不可变的，即一旦初始化后不能再次修改。

    ```java
     public class MyClass {
         final int age; // 不可变的实例变量
         
         public MyClass(int age) {
             this.age = age;
         }
     }
    ```
5.  **方法参数上声明：** 在方法的参数前面添加 `final` 关键字可以表示该参数为不可变的，即在方法内部不能对参数进行重新赋值。

    ```java
     public void printName(final String name) {
         // name 参数在方法内部是不可变的
         System.out.println("Name: " + name);
     }
    ```

**finally：**

```java
 try {
        // 一些可能抛出异常的代码
 } catch (Exception e) {
         // 异常处理代码
 } finally {
         // 无论是否发生异常，都会执行的代码
         // 通常用于资源释放等操作
 }
```

**finalize：**

`finalize` 是一个方法，用于在Java对象被垃圾回收前执行一些清理操作。这个方法在 Java 9 中已经被标记为废弃，不推荐使用，因为它的行为不稳定且容易导致资源泄漏。

## 3、缓存

### **3.1**、**Redis-LRU（Least Recently Used）**

是一种常见的缓存替换策略，用于管理缓存中的数据。LRU策略基于以下原则：当需要替换缓存中的数据时，选择最近最少被使用的数据进行替换。换句话说，LRU算法会优先保留最近被访问的数据，而淘汰那些很久没有被访问的数据。

LRU算法通常通过一个数据结构来跟踪和维护缓存中数据的使用顺序。最常见的实现方式是使用双向链表（Doubly Linked List）和哈希表（Hash Table）的组合。具体操作如下：

1. 当某个数据被访问（读取或写入）时，它会被移到链表的头部，表示它是最近被使用的数据。
2. 当需要替换缓存中的数据时，选择链表尾部的数据进行替换，因为尾部的数据是最近最少被使用的。
3. 如果某个数据再次被访问，它会被移到链表头部，更新其位置为最近被使用。

这种方式确保了最近被访问的数据总是位于链表的前部，而最久未被访问的数据位于链表的后部，从而实现了LRU的替换策略。

LRU算法的优点是它相对容易实现，并且在某些场景下表现良好，尤其是当访问模式具有局部性时（即最近被访问的数据很可能会在短时间内再次被访问）。然而，它也有一些缺点，例如它对内存的要求较高，需要额外的数据结构来维护使用顺序，并且在某些情况下可能无法有效处理变化频繁的访问模式。

因此，根据具体应用场景和性能需求，可能需要考虑其他缓存替换策略，如LFU（Least Frequently Used）或基于权重的策略。

<figure><img src="../.gitbook/assets/LRU" alt=""><figcaption><p>LRU</p></figcaption></figure>

### **3.2**、**Redis多核心CPU上部署策略**

你可以采取一些策略来充分利用多核心的性能优势，提高Redis的性能和并发处理能力。以下是一些关键考虑因素和策略：

1. **Redis线程模型**：Redis是单线程的，意味着它在任何给定时刻只能处理一个请求。然而，Redis的单线程模型通常不会成为性能瓶颈，因为其内部实现非常高效。多核心的CPU可以通过并行处理多个客户端请求来提高性能。
2. **多实例部署**：如果你的服务器有多个CPU核心，并且你希望充分利用这些核心，可以考虑在同一台服务器上运行多个独立的Redis实例。每个Redis实例将占用一个CPU核心，并且可以独立运行，提供不同的服务。
3. **操作系统优化**：在多核心CPU上运行Redis时，确保操作系统也进行了适当的优化。例如，Linux内核通常会自动将进程和线程调度到可用的CPU核心上，但你可以配置CPU亲和性（CPU Affinity）来更精确地控制进程和CPU核心的关系。

总之，在多核心CPU上部署Redis需要综合考虑硬件、操作系统、Redis配置和应用程序架构等多个因素。通过合理的架构和配置，你可以实现更好的性能和扩展性，充分利用多核心CPU的性能潜力。同时，监控和调整Redis的性能参数也是保持系统高效运行的关键。

问题是：单线程的Redis是没办法发挥多核心CPU的威力。如果32核心只用一个核心，那么31个核心是浪费的。

解决方案：可以按核心数部署多个Redis在一个服务器上。

## 4、数据库

### 4.1、数据库优化的硬件方案

SSD硬盘比机械硬盘性能更高，性能的差距取决于多个因素，包括SSD和HDD的型号、技术规格以及具体使用情况。一般来说，以下是SSD与机械硬盘性能方面的一些主要差距：

1. **读写速度**：SSD的读写速度通常比机械硬盘快数倍甚至更多。典型的SSD可以达到每秒几百MB到几千MB的读写速度，而机械硬盘通常在每秒几十到几百MB之间。
2. **随机访问速度**：SSD在随机访问速度方面明显优于机械硬盘。SSD可以几乎立即访问存储块，而机械硬盘需要等待机械臂移动到正确的磁道上，因此其随机访问速度较慢。
3. **响应时间**：由于没有机械部件，SSD的响应时间非常低，通常在毫秒级别。机械硬盘的响应时间通常在毫秒或更高。
4. **耐久性和寿命**：SSD通常比机械硬盘更耐用，因为它们没有机械运动部件。SSD的使用寿命通常比机械硬盘长。
5. **能效**：SSD通常更节能，因为不需要机械运动，并且在闲置时可以进入低功耗状态。

总体而言，SSD在性能方面远远优于机械硬盘，这使得它们成为许多应用场景的首选，包括操作系统安装、应用程序加载、数据存储和服务器性能提升。然而，机械硬盘仍然用于存储大容量数据，因为它们在成本效益方面具有优势。因此，具体的性能差距会受到硬盘型号和用途的影响。

问题是：怎么快速提升数据库性能？

解决方案：有一种硬件解决方案是机械硬盘换SSD硬盘。前提是现在SSD硬盘特别便宜。以前是因为太贵了换不起。现在太便宜了。随便换。

<figure><img src="../.gitbook/assets/机械硬盘" alt=""><figcaption><p>机械硬盘</p></figcaption></figure>

<figure><img src="../.gitbook/assets/磁道扇区" alt=""><figcaption><p>磁道扇区</p></figcaption></figure>

#### 4.2、分库分表

问题：美团外卖怎么分？

解决方案：可以按城市分库。

问题：微博数据怎么分？

解决方案的一部分：历史数据可以按年封存。

## 5、linux操作系统

### 5.1、Linux系统的文件索引inode

在Linux文件系统中，inode（索引节点）是一个用于存储文件和目录元数据的数据结构。每个文件和目录都有一个关联的inode，它包含了文件的各种属性和数据块的指针。inode在文件系统中起着关键的作用，它存储了文件的元信息，包括但不限于以下内容：

1. **文件类型**：inode标识文件的类型，例如普通文件、目录、符号链接、设备文件等。
2. **文件权限**：文件的所有者、所属组和其他用户的访问权限。
3. **文件大小**：文件的大小以字节为单位。
4. **时间戳**：文件的三种时间戳，包括修改时间、访问时间和更改时间。
5. **链接计数**：与inode关联的文件和目录的硬链接数量。
6. **数据块指针**：指向文件数据的指针。这些指针指向文件数据的实际存储位置，这些位置可以位于不同的数据块中。

inode的使用使得文件系统可以高效地管理文件和目录，因为它存储了文件的元信息，而文件的实际数据则分布在数据块中。这允许文件系统快速查找文件的属性和数据块位置。

值得注意的是，每个文件系统都有一个有限数量的inode，这意味着在创建文件和目录时，inode会逐渐耗尽。如果inode用尽，文件系统将无法创建更多文件或目录，即使磁盘上仍有空闲空间。因此，inode的数量通常是在文件系统创建时固定的，并且通常不可更改。

你可以使用命令如`ls -i`来查看文件的inode号，或者使用`stat`命令来获取更详细的inode信息。

问题是：inode满了，文件系统将无法创建更多文件或目录，即使磁盘上仍有空闲空间。

解决方案是：给inode分配更多的硬盘空间。

## 6、网络通信

长连接和短连接是网络通信中两种不同的连接管理方式，用于在客户端和服务器之间传输数据。它们在如何处理连接的建立、使用和终止等方面有不同的特点和优劣势。

### 6.1、短连接（Short Connection）

1. **连接建立和终止频繁**：每次客户端需要与服务器通信时都会建立一个新的连接，通信结束后立即关闭连接。这意味着在数据传输之前需要进行连接建立握手，传输后需要进行连接终止握手，这些握手会占用额外的时间和资源。
2. **资源消耗低**：由于连接是短暂的，所以在服务器和客户端上的连接资源占用相对较低。这对于服务器来说是一种好处，因为它可以释放连接资源以响应其他客户端的请求。
3. **适用于瞬时通信**：短连接适用于那些只需要一次或少数几次通信的场景，例如HTTP/1.0协议的默认行为，每个HTTP请求都建立一个新连接。

### 6.2、长连接（Persistent Connection）

1. **连接复用**：长连接允许客户端和服务器在一次连接建立后多次通信，而不是每次通信都建立一个新连接。这减少了握手的开销，因为只有在连接建立时才需要进行握手，后续通信都共享相同的连接。
2. **资源占用较高**：由于长连接会保持打开状态，所以它会占用服务器和客户端上的连接资源。这可能会导致资源泄漏或过度占用的问题，需要谨慎管理。
3. **适用于频繁通信**：长连接适用于需要频繁通信的场景，例如HTTP/1.1协议的默认行为，其中默认启用了长连接以减少每个HTTP请求的延迟。

总的来说，选择短连接还是长连接取决于特定的应用程序和使用情况：

* 短连接适用于临时性通信，其中客户端和服务器之间的交互不频繁。
* 长连接适用于需要频繁通信的场景，其中客户端和服务器之间需要维护状态或传输多个请求和响应。

在实际应用程序中，通常需要权衡连接建立的开销、资源占用和通信频率，以确定使用哪种连接类型以满足性能和可伸缩性需求。

## 7、**某**大厂JVM配置

```properties
# 指定了Java日志配置文件的位置。这个文件定义了应用程序的日志记录规则。
-Djava.util.logging.config.file=/data1/weibo/conf/logging.properties
# 指定了Java日志管理器的实现类。在这里，使用了Apache Tomcat的日志管理器。
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
# 设置TLS/SSL协议中临时Diffie-Hellman密钥交换的位数为2048位。
-Djdk.tls.ephemeralDHKeySize=2048
# 指定了用于处理URL协议的Java包。这里使用了Apache Tomcat的org.apache.catalina.webresources包来处理Web资源。
-Djava.protocol.handler.pkgs=org.apache.catalina.webresources
# 设置Apache Tomcat的安全监听器的UMASK值。
-Dorg.apache.catalina.security.SecurityListener.UMASK=0022
# 指定了JVM用于名称解析的服务提供者。在这里，首选使用DNS和dnsjava库，次选使用默认提供者。
-Dsun.net.spi.nameservice.provider.1=dns,dnsjava
-Dsun.net.spi.nameservice.provider.2=default
# 禁用偏向锁。并发高的时候偏向锁会带来很大的消耗，因此可以禁用它。
-XX:-UseBiasedLocking
# 设置JVM的最大堆内存为8GB。
-Xmx8g
# 设置JVM的初始堆内存为8GB。
-Xms8g
# 设置JVM的年轻代堆内存为4GB。
-Xmn4g
# 设置每个线程的堆栈大小为1MB。
-Xss1m
```

偏向锁（Biased Locking）是Java虚拟机（JVM）中的一项优化技术，用于提高多线程并发性能。它是一种轻量级锁，旨在降低线程争用的开销，特别是对于短暂的同步块。

以下是偏向锁的主要特点和原理：

1. **适用场景：** 偏向锁适用于单线程访问同步块的情况。在实际应用中，许多同步块只会被一个线程频繁地访问，而其他线程很少或几乎不访问。偏向锁正是为了这种情况而设计的。
2. **锁升级过程：** 当一个线程第一次进入同步块时，JVM会尝试将对象头标记为偏向锁，并将线程ID存储在对象头中。之后，当同一个线程再次访问同步块时，JVM会检查对象头，如果发现是偏向锁并且线程ID匹配，就可以直接进入临界区，无需竞争锁。
3. **竞争情况下：** 当有其他线程尝试访问同步块时，偏向锁会失效，转为常规锁，即自旋锁或重量级锁，这时会涉及到竞争和线程切换。
4. **优势：** 偏向锁在无竞争的情况下几乎没有性能开销，因为它避免了自旋等待和线程切换的成本。它提高了单线程执行同步块的效率。
5. **缺点：** 偏向锁只在一定条件下有效，如果多个线程频繁地竞争同一个锁，那么偏向锁的开销可能超过了其性能优势。此时，JVM会自动撤销偏向锁，将其恢复为无锁状态。

偏向锁通常在Java 6及以后的版本中默认开启，但可以通过JVM参数来关闭或手动配置。在多数情况下，偏向锁对于那些只有一个线程访问的同步块来说，是一种有益的优化。但在高度竞争的情况下，它可能不如其他锁（如自旋锁或重量级锁）有效。因此，在使用偏向锁时，需要根据实际应用场景进行评估和测试。

## 8、Spring AOP 和cglib

Spring AOP 和 CGLIB 是两个在 Spring 框架中用于实现面向切面编程 (AOP) 的关键组件，它们之间有一些重要的区别：

1. **AOP 和 CGLIB 的基本区别：**
   * **Spring AOP：**
     * **基于代理模式：** Spring AOP 主要基于 Java 的代理模式来实现切面编程。它可以使用 JDK 动态代理或 CGLIB 来创建代理对象，具体取决于目标对象是否实现了接口。
     * **支持接口代理：** 如果目标对象实现了接口，Spring AOP 将使用 JDK 动态代理来生成代理对象。
     * **不支持非接口代理：** 如果目标对象没有实现接口，Spring AOP 将使用 CGLIB 来生成代理对象。
   * **CGLIB：**
     * **基于字节码生成：** CGLIB 是一个字节码生成库，它通过生成目标类的子类来创建代理对象。这意味着它可以代理那些没有实现接口的类。
     * **支持非接口代理：** CGLIB 可以用于代理任何类，包括那些没有实现接口的类。
     * **性能优势：** 由于 CGLIB 基于字节码生成，通常在性能上比 JDK 动态代理更快。
2. **性能和用途的区别：**
   * **Spring AOP：**
     * Spring AOP 更适合用于处理那些实现了接口的目标对象，并且它主要用于处理方法级别的切面逻辑，如日志记录、性能监视等。由于使用 JDK 动态代理，它在运行时会创建一个代理对象，以便在方法调用前后添加切面逻辑。
   * **CGLIB：**
     * CGLIB 更适合用于代理那些没有实现接口的类，它可以代理类级别的切面逻辑，包括对非公开方法和构造函数的访问。由于它直接生成字节码，因此通常比 Spring AOP 更快，但也更复杂。
3. **配置和使用的区别：**
   * **Spring AOP：**
     * 在 Spring 中，您可以通过声明式的方式配置切面和通知，通常使用 XML 或基于注解的配置来定义切面和通知。Spring AOP 提供了 `@AspectJ` 注解，用于声明方面（aspects）和通知（advices）。
     * Spring AOP 的配置相对简单，因为它隐藏了底层代理的细节，使得使用者更容易理解和配置。
   * **CGLIB：**
     * CGLIB 的使用通常需要更多的编程，因为您需要直接操作字节码生成。这可能会导致配置更复杂，需要更多的代码。

总之，Spring AOP 和 CGLIB 在实现 AOP 方面有不同的优势和适用场景。Spring AOP 更适用于基于接口的代理和简单的声明式配置，而 CGLIB 更适用于非接口代理和对类级别的切面逻辑的需求。您可以根据您的项目需求选择适当的方法来实现切面编程。

## 9、工具类

### 9.1、Googel Guava

Guava 是 Google 开发的一个 Java 标准库扩展，它提供了许多实用的工具类和方法，用于简化 Java 编程中的常见任务。以下是一些 Guava 的示例用法：

使用 Guava 的集合工具类：

```java
import com.google.common.collect.Lists;

public class GuavaCollectionsExample {
    public static void main(String[] args) {
        // 创建一个不可变的列表
        ImmutableList<String> immutableList = ImmutableList.of("apple", "banana", "cherry");

        // 使用 Lists 工具类创建可变列表
        List<String> mutableList = Lists.newArrayList("apple", "banana", "cherry");
        
        // 使用 Guava 进行集合操作
        List<String> filteredList = Lists.newArrayList(Iterables.filter(mutableList, s -> s.length() > 5));
        
        System.out.println("Immutable List: " + immutableList);
        System.out.println("Mutable List: " + mutableList);
        System.out.println("Filtered List: " + filteredList);
    }
}
```

使用 Guava 的缓存：

```java
import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

public class GuavaCacheExample {
    public static void main(String[] args) {
        // 创建一个缓存，设置最大大小和过期时间
        Cache<String, Integer> cache = CacheBuilder.newBuilder()
                .maximumSize(100)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .build();

        // 向缓存中放入数据
        cache.put("one", 1);
        cache.put("two", 2);

        // 从缓存中获取数据
        Integer value = cache.getIfPresent("one");

        System.out.println("Value from cache: " + value);
    }
}
```

使用 Guava 的字符串工具类：

```java
import com.google.common.base.Splitter;

public class GuavaStringExample {
    public static void main(String[] args) {
        String input = "apple,banana,cherry";
        
        // 使用 Guava 的 Splitter 拆分字符串
        Iterable<String> parts = Splitter.on(',').split(input);
        
        for (String part : parts) {
            System.out.println("Part: " + part);
        }
    }
}
```

### 9.2、Apache Commons

Apache Commons 是一个开源的 Java 类库项目，它提供了许多实用的工具类和方法，用于简化 Java 编程中的常见任务。其中，Apache Commons Lang 和 Apache Commons IO 是两个广泛使用的模块，下面是一些使用这两个模块的示例

字符串处理：

```java
import org.apache.commons.lang3.StringUtils;

public class ApacheStringUtilsExample {
    public static void main(String[] args) {
        String input = "Hello, World!";
        
        // 检查字符串是否为空
        boolean isEmpty = StringUtils.isEmpty(input);
        
        // 判断字符串是否包含某个子字符串
        boolean contains = StringUtils.contains(input, "World");
        
        // 反转字符串
        String reversed = StringUtils.reverse(input);
        
        System.out.println("Is empty: " + isEmpty);
        System.out.println("Contains 'World': " + contains);
        System.out.println("Reversed: " + reversed);
    }
}
```

数组操作：

```java
import org.apache.commons.lang3.ArrayUtils;

public class ApacheArrayUtilsExample {
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5};
        
        // 合并两个数组
        int[] merged = ArrayUtils.addAll(numbers, new int[]{6, 7, 8});
        
        // 检查数组是否为空
        boolean isEmpty = ArrayUtils.isEmpty(numbers);
        
        System.out.println("Merged array: " + Arrays.toString(merged));
        System.out.println("Is empty: " + isEmpty);
    }
}
```
