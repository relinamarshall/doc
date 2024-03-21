---
description: Du，disk usage简称是用于估算文件或目录的磁盘使用情况 ，参数的不同组合，可以更快的提高工作效率
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Nginx

Nginx简介

## 1、Nginx介绍

**官网** [**http://nginx.org/**](http://nginx.org/)

* Nginx (engine x) 是一个高性能的**HTTP**和**反向代理**web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，公开版本1.19.6发布于2020年12月15日
* Nginx是一款轻量级的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好
* Nginx 是高性能的 HTTP 和反向代理的服务器，处理高并发能力是十分强大的，能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数。

**Nginx（engine x）**一个具有高性能的【HTTP】和【反向代理】的【WEB服务器】，同时也是一个【POP3/SMTP/IMAP代理服务器】，是由伊戈尔·赛索耶夫(俄罗斯人)使用C语言编写的，Nginx的第一个版本是2004年10月4号发布的0.1.0版本。另外值得一提的是伊戈尔·赛索耶夫将Nginx的源码进行了开源，这也为Nginx的发展提供了良好的保障。

**名词解释**

* WEB服务器：WEB服务器也叫网页服务器，英文名叫Web Server，主要功能是为用户提供网上信息浏览服务。
* HTTP：HTTP是超文本传输协议的缩写，是用于从WEB服务器传输超文本到本地浏览器的传输协议，也是互联网上应用最为广泛的一种网络协议。HTTP是一个客户端和服务器端请求和应答的标准，客户端是终端用户，服务端是网站，通过使用Web浏览器、网络爬虫或者其他工具，客户端发起一个到服务器上指定端口的HTTP请求。
* POP3/SMTP/IMAP：
  * POP3(Post Offic Protocol 3)邮局协议的第三个版本
  * SMTP(Simple Mail Transfer Protocol)简单邮件传输协议
  * IMAP(Internet Mail Access Protocol)交互式邮件存取协议
  * 通过上述名词的解释，可以了解到Nginx也可以作为电子邮件代理服务器
* 正向代理： 正向代理是客户端与正向代理客户端在同一局域网，客户端发出请求，正向代理 替代客户端向服务器发出请求。服务器不知道谁是真正的客户端，正向代理隐藏了真实的请求客户端。
* 反向代理： 服务器与反向代理在同一个局域网，客服端发出请求，反向代理接收请求 ，反向代理服务器会把我们的请求分转发到真实提供服务的各台服务器Nginx就是性能非常好的反向代理服务器，用来做负载均衡。

<figure><img src="../.gitbook/assets/image (379).png" alt=""><figcaption></figcaption></figure>

## 2、Nginx优点

**(1)速度更快、并发更高**

单次请求或者高并发请求的环境下，Nginx都会比其他Web服务器响应的速度更快。一方面在正常情况下，单次请求会得到更快的响应，另一方面，在高峰期(如有数以万计的并发请求)，Nginx比其他Web服务器更快的响应请求。Nginx之所以有这么高的并发处理能力和这么好的性能原因在于Nginx采用了多进程和I/O多路复用(epoll)的底层实现。

**(2)配置简单，扩展性强**

Nginx的设计极具扩展性，它本身就是由很多模块组成，这些模块的使用可以通过配置文件的配置来添加。这些模块有官方提供的也有第三方提供的模块，如果需要完全可以开发服务自己业务特性的定制模块。

**(3)高可靠性**

Nginx采用的是多进程模式运行，其中有一个master主进程和N多个worker进程，worker进程的数量我们可以手动设置，每个worker进程之间都是相互独立提供服务，并且master主进程可以在某一个worker进程出错时，快速去"拉起"新的worker进程提供服务。

**(4)热部署**

现在互联网项目都要求以7\*24小时进行服务的提供，针对于这一要求，Nginx也提供了热部署功能，即可以在Nginx不停止的情况下，对Nginx进行文件升级、更新配置和更换日志文件等功能。

**(5)成本低、BSD许可证**

BSD是一个开源的许可证，世界上的开源许可证有很多，现在比较流行的有六种分别是GPL、BSD、MIT、Mozilla、Apache、LGPL。这六种的区别是什么，我们可以通过下面一张图来解释下：

<figure><img src="../.gitbook/assets/image (378).png" alt=""><figcaption></figcaption></figure>

Nginx本身是开源的，我们不仅可以免费的将Nginx应用在商业领域，而且还可以在项目中直接修改Nginx的源码来定制自己的特殊要求。这些点也都是Nginx为什么能吸引无数开发者继续为Nginx来贡献自己的智慧和青春。

## 3、Nginx功能

Nginx提供的基本功能服务从大体上归纳为**基本HTTP服务**、**高级HTTP服务**和**邮件服务**等三大类。

**基本HTTP服务**：Nginx可以提供基本HTTP服务，可以作为HTTP代理服务器和反向代理服务器，支持通过缓存加速访问，可以完成简单的负载均衡和容错，支持包过滤功能，支持SSL等。

* 处理静态文件、处理索引文件以及支持自动索引；
* 提供反向代理服务器，并可以使用缓存加上反向代理，同时完成负载均衡和容错；
* 提供对FastCGI、memcached等服务的缓存机制，，同时完成负载均衡和容错；
* 使用Nginx的模块化特性提供过滤器功能。Nginx基本过滤器包括gzip压缩、ranges支持、chunked响应、XSLT、SSI以及图像缩放等。其中针对包含多个SSI的页面，经由FastCGI或反向代理，SSI过滤器可以并行处理。
* 支持HTTP下的安全套接层安全协议SSL.
* 支持基于加权和依赖的优先权的HTTP/2

**高级HTTP服务**

* 支持基于名字和IP的虚拟主机设置
* 支持HTTP/1.0中的KEEP-Alive模式和管线(PipeLined)模型连接
* 自定义访问日志格式、带缓存的日志写操作以及快速日志轮转。
* 提供3xx\~5xx错误代码重定向功能
* 支持重写（Rewrite)模块扩展
* 支持重新加载配置以及在线升级时无需中断正在处理的请求
* 支持网络监控
* 支持FLV和MP4流媒体传输

**邮件服务**

Nginx提供邮件代理服务也是其基本开发需求之一，主要包含以下特性：

* 支持IMPA/POP3代理服务功能
* 支持内部SMTP代理服务功能

**Nginx常用的功能模块**

```
 静态资源部署
 Rewrite地址重写
     正则表达式
 反向代理
 负载均衡
     轮询、加权轮询、ip_hash、url_hash、fair
 Web缓存
 环境部署
     高可用的环境
 用户认证模块
 ...
```

**Nginx的核心组成**

```
nginx二进制可执行文件
nginx.conf配置文件
error.log错误的日志记录
access.log访问日志记录
```

## Nginx使用

## 1、安装

**源码包安装**

```sh
 第一步：进入官网下载
 http://nginx.org/en/download.html
 ​
 第二步：安装依赖
 yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
 ​
 第三步：解压
 tar -zxvf nginx-版本.tar.gz
 ​
 第四步：进入nginx文件夹
 cd nginx-版本
 ​
 第五步：进行配置
 ./configure --prefix=/usr/local/nginx
 ​
 第六步：编译&&安装
 make && make install
 ​
 第七步：启动
 nginx start
```

**Yum源安装**

```sh
 第一步：添加yum源文件
 vim /etc/yum.repos.d/nginx.repo
 ​
 [nginx-stable]
 name=nginx stable repo
 baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
 gpgcheck=1
 enabled=1
 gpgkey=https://nginx.org/keys/nginx_signing.key
 module_hotfixes=true
 ​
 [nginx-mainline]
 name=nginx mainline repo
 baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
 gpgcheck=1
 enabled=0
 gpgkey=https://nginx.org/keys/nginx_signing.key
 module_hotfixes=true
 ​
 第二步：查看是否安装成功
 yum list | grep nginx
 ​
 第三步：使用yum进行安装
 yum install -y nginx
 ​
 第四步：查看nginx的安装位置
 whereis nginx
 ​
 第五步：查看版本
 nginx -V
```

**注**： 在 windows 系统中访问 linux 中 nginx，默认不能访问的，因为防火墙问题

```sh
 关闭防火墙 或者 开放访问的端口号，80 443 端口 
 ​
 查看开放的端口号
 firewall-cmd --list-all
 ​
 设置开放的端口号
 firewall-cmd --add-service=http –permanent
 firewall-cmd --add-port=80/tcp --permanent
 ​
 重启防火墙
 firewall-cmd –reload
```

**Nginx常用文件目录**

{% code overflow="wrap" %}
```
 ├── client_body_temp                 # POST 大文件暂存目录
 ├── conf                             # Nginx所有配置文件的目录
 │   ├── nginx.conf                   #这是Nginx默认的主配置文件，日常使用和修改的文件
 │   ├── nginx.conf.default
 ├── html                             # Nginx默认站点目录
 │   ├── 50x.html                     # 错误页面优雅替代显示文件，如出现502错误时会调用此页面
 │   └── index.html                   # 默认的首页文件
 ├── logs                             # Nginx日志目录
 │   ├── access.log                   # 访问日志文件
 │   ├── error.log                    # 错误日志文件
 │   └── nginx.pid                    # pid文件,Nginx进程启动后,会把进程的ID号写到此文件
 ├── proxy_temp                       # 临时目录
 ├── sbin                             # Nginx 可执行文件目录
 │   └── nginx                        # Nginx 二进制可执行程序
```
{% endcode %}

## 2、命令

**Nginx的命令行控制**

```sh
 进入 nginx 目录中
 cd /usr/local/nginx/sbin
 查看 nginx 版本号
 ./nginx -v
 启动 nginx
 ./nginx
 停止 nginx
 ./nginx -s stop
 重新加载 nginx
 ./nginx -s reload
 ​
 -?和-h:显示帮助信息
 -v:打印版本号信息并退出
 -V:打印版本号信息和配置信息并退出
 -t:测试nginx的配置文件语法是否正确并退出
 -T:测试nginx的配置文件语法是否正确并列出用到的配置文件信息然后退出
 -q:在配置测试期间禁止显示非错误消息
 -s:signal信号，后面可以跟 ：
     stop[快速关闭，类似于TERM/INT信号的作用]
     quit[优雅的关闭，类似于QUIT信号的作用] 
     reopen[重新打开日志文件类似于USR1信号的作用] 
     reload[类似于HUP信号的作用]
 -p:prefix，指定Nginx的prefix路径，(默认为: /usr/local/nginx/)
 -c:filename,指定Nginx的配置文件路径,(默认为: conf/nginx.conf)
 -g:用来补充Nginx配置文件，向Nginx服务指定启动时应用全局的配置
```

**Nginx服务的信号控制**

管理员通过给master进程发送信号就可以来控制Nginx,这个时候我们需要有两个前提条件，一个是要操作的master进程，一个是信号

要想操作Nginx的master进程，就需要获取到master进程的进程号ID。获取方式介绍两种

{% code overflow="wrap" %}
```sh
 通过 ps -ef | grep nginx
 ​
 nginx的`./configure`的配置参数的时候，有一个参数是`--pid-path=PATH`默认是`/usr/local/nginx/logs/nginx.pid`,所以可以通过查看该文件来获取nginx的master进程ID
```
{% endcode %}

| 信号       | 作用                                |
| -------- | --------------------------------- |
| TERM/INT | 立即关闭整个服务                          |
| QUIT     | "优雅"地关闭整个服务                       |
| HUP      | 重读配置文件并使用服务对新配置项生效                |
| USR1     | 重新打开日志文件，可以用来进行日志切割               |
| USR2     | 平滑升级到最新版的nginx                    |
| WINCH    | 所有子进程不在接收处理新连接，相当于给work进程发送QUIT指令 |

{% code overflow="wrap" %}
```sh
 调用命令为 kill -signal PID
 signal:即为信号；PID即为获取到的master线程ID
 ​
 发送TERM/INT信号给master进程，会将Nginx服务立即关闭。
 kill -TERM PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
 kill -INT PID / kill -INT `cat /usr/local/nginx/logs/nginx.pid`
 ​
 发送QUIT信号给master进程，master进程会控制所有的work进程不再接收新的请求，等所有请求处理完后，在把进程都关闭掉
 kill -QUIT PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
 ​
 发送HUP信号给master进程，master进程会把控制旧的work进程不再接收新的请求，等处理完请求后将旧的work进程关闭掉，然后根据nginx的配置文件重新启动新的work进程
 kill -HUP PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
 ​
 发送USR1信号给master进程，告诉Nginx重新开启日志文件
 kill -USR1 PID / kill -TERM `cat /usr/local/nginx/logs/nginx.pid`
 ​
 发送USR2信号给master进程，告诉master进程要平滑升级，这个时候，会重新开启对应的master进程和work进程，整个系统中将会有两个master进程，并且新的master进程的PID会被记录在`/usr/local/nginx/logs/nginx.pid`而之前的旧的master进程PID会被记录在`/usr/local/nginx/logs/nginx.pid.oldbin`文件中，接着再次发送QUIT信号给旧的master进程，让其处理完请求后再进行关闭
 kill -USR2 PID / kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
 ​
 发送WINCH信号给master进程,让master进程控制不让所有的work进程在接收新的请求了，请求处理完后关闭work进程。注意master进程不会被关闭掉
 kill -WINCH PID /kill -WINCH`cat /usr/local/nginx/logs/nginx.pid`
```
{% endcode %}

**Nginx服务操作的问题**

{% code overflow="wrap" %}
```sh
 经过前面的操作，我们会发现，如果想要启动、关闭或重新加载nginx配置文件，都需要先进入到nginx的安装目录的sbin目录
 ​
 # 方式一：Ln软连接
 ln -s /usr/local/nginx/sbin/* /usr/local/sbin/  #给命令做软连接，以便PATH能找到
 ​
 # 方式二：Nginx配置成系统服务
 在/usr/lib/systemd/system目录下添加nginx.service,内容如下
 vim /usr/lib/systemd/system/nginx.service
 ​
 [Unit]
 Description=nginx web service
 Documentation=http://nginx.org/en/docs/
 After=network.target
 ​
 [Service]
 Type=forking
 PIDFile=/usr/local/nginx/logs/nginx.pid
 ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
 ExecStart=/usr/local/nginx/sbin/nginx
 ExecReload=/usr/local/nginx/sbin/nginx -s reload
 ExecStop=/usr/local/nginx/sbin/nginx -s stop
 PrivateTmp=true
 ​
 [Install]
 WantedBy=default.target
 ​
 添加完成后如果权限有问题需要进行权限设置
 chmod 755 /usr/lib/systemd/system/nginx.service
 ​
 使用系统命令来操作Nginx服务
 启动: systemctl start nginx
 停止: systemctl stop nginx
 重启: systemctl restart nginx
 重新加载配置文件: systemctl reload nginx
 查看nginx状态: systemctl status nginx
 开机启动: systemctl enable nginx
 ​
 # 方式三：Nginx命令配置到系统环境
 修改/etc/profile文件
 vim /etc/profile
 在最后一行添加
 export PATH=$PATH:/usr/local/nginx/sbin
 ​
 使之立即生效
 source /etc/profile
 ​
 执行nginx命令
 nginx -V
```
{% endcode %}

## 3、版本升级和新增模块

如果想对Nginx的版本进行更新，或者要应用一些新的模块，最简单的做法就是停止当前的Nginx服务，然后开启新的Nginx服务。但是这样会导致在一段时间内，用户是无法访问服务器。

为了解决这个问题，我们就需要用到Nginx服务器提供的平滑升级功能。这个也是Nginx的一大特点，使用这种方式，就可以使Nginx在7\*24小时不间断的提供服务了

需求：Nginx的版本最开始使用的是Nginx-1.14.2,由于服务升级，需要将Nginx的版本升级到Nginx-1.16.1,要求Nginx不能中断提供服务。

**环境准备**

```sh
 先准备两个版本的Nginx分别是 1.14.2和1.16.1
 ​
 使用Nginx源码安装的方式将1.14.2版本安装成功并正确访问
 进入安装目录
 ./configure
 make && make install
 ​
 将Nginx1.16.1进行参数配置和编译，不需要进行安装。
 进入安装目录
 ./configure
 make
```

**使用Nginx服务信号进行升级**

```sh
 将1.14.2版本的sbin目录下的nginx进行备份
 cd /usr/local/nginx/sbin
 mv nginx nginxold
 ​
 将Nginx1.16.1安装目录编译后的objs目录下的nginx文件，拷贝到原来/usr/local/nginx/sbin目录下
 cd ~/nginx/core/nginx-1.16.1/objs
 cp nginx /usr/local/nginx/sbin
 ​
 发送信号USR2给Nginx的1.14.2版本对应的master进程
 ​
 发送信号QUIT给Nginx的1.14.2版本对应的master进程
 kill -QUIT more /usr/local/logs/nginx.pid.oldbin
```

**使用Nginx安装目录的make命令完成升级**

```sh
 将1.14.2版本的sbin目录下的nginx进行备份
 cd /usr/local/nginx/sbin
 mv nginx nginxold
 ​
 将Nginx1.16.1安装目录编译后的objs目录下的nginx文件，拷贝到原来/usr/local/nginx/sbin目录下
 cd ~/nginx/core/nginx-1.16.1/objs
 cp nginx /usr/local/nginx/sbin
 ​
 进入到安装目录，执行make upgrade
 ​
 查看是否更新成功
 ./nginx -v
```

在整个过程中，其实Nginx是一直对外提供服务的。

## 4、核心配置

Nginx的核心配置文件默认是放在 **/usr/local/nginx/conf/nginx.conf**

将其中的注释部分【Nginx的配置文件中可以使用**#**来注释】删除掉后内容

```nginx
 worker_processes  1;
 ​
 events {
     worker_connections  1024;
 }
 ​
 http {
     include       mime.types;
     default_type  application/octet-stream;
     sendfile        on;
     keepalive_timeout  65;
 ​
     server {
         listen       80;
         server_name  localhost;
         location / {
             root   html;
             index  index.html index.htm;
         }
         error_page   500 502 503 504  /50x.html;
         location = /50x.html {
             root   html;
         }
     }
 }
```

```nginx
 #全局块,主要设置Nginx服务器整体运行的配置指令
 指令名 指令值; 
 ​
 #events块,主要设置,Nginx服务器与用户的网络连接,这一部分对Nginx服务器的性能影响较大
 events {     
     指令名 指令值;
 }
 ​
 #http块,是Nginx服务器配置中的重要部分，代理、缓存、日志记录、第三方模块配置...             
 http {
     指令名 指令值;
     
     #server块，是Nginx配置和虚拟主机相关的内容
     server {
         指令名 指令值;
         
         #location块，基于Nginx服务器接收请求字符串与location后面的值进行匹配，对特定请求进行处理
         location / { 
             指令名 指令值;
         }
     }
     ...
 }
 ​
 简单小结下:
 nginx.conf配置文件中默认有三大块：全局块、events块、http块
 http块中可以配置多个server块，每个server块又可以配置多个location块
```

### 4.1、全局块

**user**：用于配置运行Nginx服务器的worker进程的用户和用户组

| 语法  | user user \[group] |
| --- | ------------------ |
| 默认值 | nobody             |

该属性也可以在编译的时候指定，语法如下`./configure --user=user --group=group`

如果两个地方都进行了设置，最终生效的是配置文件中的配置

该指令的使用步骤:

```nginx
设置一个用户信息"www"
user www;
```

<figure><img src="../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```sh
创建一个用户
useradd www

修改user属性
user www

创建/root/html/index.html页面，添加如下内容
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
	<h1>Welcome to nginx!</h1>
</body>
</html>

修改nginx.conf
location / {
	root   /root/html;
	index  index.html index.htm;
}

测试启动访问
	页面会报403拒绝访问的错误
	
分析原因
	因为当前用户没有访问/root/html目录的权限
	
将文件创建到 /home/www/html/index.html,修改配置
location / {
	root   /home/www/html;
	index  index.html index.htm;
}

再次测试启动访问
	正常访问
	
综上所述，使用user指令可以指定启动运行工作进程的用户及用户组，这样对于系统的权限访问控制的更加精细，也更加安全
```
{% endcode %}

**master\_process**：用来指定是否开启工作进程。

| 语法  | master\_process on/off; |
| --- | ----------------------- |
| 默认值 | master\_process on;     |

**worker\_processes**：用于配置Nginx生成工作进程的数量，这个是Nginx服务器实现并发处理服务的关键所在。

理论上来说workder process的值越大，可以支持的并发处理量也越多，但事实上这个值的设定是需要受到来自服务器自身的限制，**建议将该值和服务器CPU的内核数保存一致**。

| 语法  | worker\_processes num/auto; |
| --- | --------------------------- |
| 默认值 | 1                           |

如果将worker\_processes设置成2，则会看到如下内容:

<figure><img src="../.gitbook/assets/image (381).png" alt=""><figcaption></figcaption></figure>

**daemon**：设定Nginx是否以守护进程的方式启动。

守护式进程是linux后台执行的一种服务进程，特点是独立于控制终端，不会随着终端关闭而停止。

| 语法  | daemon on/off; |
| --- | -------------- |
| 默认值 | daemon on;     |

**pid**：用来配置Nginx当前master进程的进程号ID存储的文件路径。

该属性可以通过`./configure --pid-path=PATH`来指定

| 语法  | pid file;                           |
| --- | ----------------------------------- |
| 默认值 | 默认为:/usr/local/nginx/logs/nginx.pid |

**error\_log**：用来配置Nginx的错误日志存放路径

该属性可以通过**./configure --error-log-path=PATH**来指定

| 语法  | error\_log file \[日志级别];         |
| --- | -------------------------------- |
| 默认值 | error\_log logs/error.log error; |
| 位置  | 全局块、http、server、location         |

其中日志级别的值有：**debug|info|notice|warn|error|crit|alert|emerg**，翻译过来为试**|信息|通知|警告|错误|临界|警报|紧急**，这块建议大家设置的时候不要设置成**info**以下的等级，因为会带来大量的**磁盘I/O消耗**，**影响Nginx的性能**。

**include**:用来引入其他配置文件，使Nginx的配置更加灵活

| 语法  | include file; |
| --- | ------------- |
| 默认值 | 无             |
| 位置  | any           |

### 4.2、Events块

**accept\_mutex**：用来设置Nginx网络连接序列化

| 语法  | accept\_mutex on/off; |
| --- | --------------------- |
| 默认值 | accept\_mutex on;     |

这个配置主要可以用来解决常说的"**惊群**"问题。大致意思是在某一个时刻，客户端发来一个请求连接，Nginx后台是以多进程的工作模式，也就是说有多个worker进程会被同时唤醒，但是最终只会有一个进程可以获取到连接，如果每次唤醒的进程数目太多，就会影响Nginx的整体性能。如果将上述值设置为**on(开启状态)**，将会对多个Nginx进程接收连接进行序列号，一个个来唤醒接收，就防止了多个进程对连接的争抢。

<figure><img src="../.gitbook/assets/image (382).png" alt=""><figcaption></figcaption></figure>

**multi\_accept**：用来设置是否允许同时接收多个网络连接

| 语法  | multi\_accept on/off; |
| --- | --------------------- |
| 默认值 | multi\_accept off;    |

如果multi\_accept被禁止了，nginx一个工作进程只能同时接受一个新的连接。否则，一个工作进程可以同时接受所有的新连接

**worker\_connections**：用来配置单个worker进程最大的连接数

| 语法  | worker\_connections number; |
| --- | --------------------------- |
| 默认值 | worker\_commections 1024;   |

这里的连接数不仅仅包括和前端用户建立的连接数，而是包括所有可能的连接数。另外，number值不能大于操作系统支持打开的最大文件句柄数量。

**use：**用来设置Nginx服务器选择哪种事件驱动来处理网络消息。

| 语法  | use method; |
| --- | ----------- |
| 默认值 | 根据操作系统定     |
| 位置  | events      |

注意：此处所选择事件处理模型是**Nginx优化部分**的一个重要内容，method的可选值有**select/poll/epoll/kqueue**等，之前在准备centos环境的时候，我们强调过要使用**linux内核在2.6以上**，就是为了能**使用epoll函数来优化Nginx**。

另外这些值的选择，也可以在编译的时候使用**--with-select\_module、--without-select\_module、--with-poll\_module、--without-poll\_module**来设置是否需要将对应的事件驱动模块编译到Nginx的内核。

**events指令配置实例**

```nginx
 打开Nginx的配置文件 nginx.conf,修改events配置
 events{
     accept_mutex on;
     multi_accept on;
     worker_commections 1024;
     use epoll;
 }
 ​
 启动测试
 ./nginx -t
 ./nginx -s reload
```

### 4.3、Http块

**定义MIME-Type**

浏览器中可以显示的内容有HTML、XML、GIF等种类繁多的文件、媒体等资源，浏览器为了区分这些资源，就需要使用MIME Type。所以说MIME Type是网络资源的媒体类型。Nginx作为web服务器，也需要能够识别前端请求的资源类型。

在Nginx的配置文件中，默认有两行配置

```nginx
 include mime.types;
 default_type application/octet-stream;
```

**default\_type**：用来配置Nginx响应前端请求默认的MIME类型

| 语法  | default\_type mime-type;  |
| --- | ------------------------- |
| 默认值 | default\_type text/plain； |
| 位置  | http、server、location      |

在**default\_type**之前还有一句**include mime.types,include**之前我们已经介绍过，相当于把**mime.types**文件中MIMT类型与相关类型文件的文件后缀名的对应关系加入到当前的配置文件中。

举例来说明：有些时候请求某些接口的时候需要返回指定的文本字符串或者json字符串，如果逻辑非常简单或者干脆是固定的字符串，那么可以使用nginx快速实现，这样就不用编写程序响应请求了，可以减少服务器资源占用并且响应性能非常快。

```nginx
 实现
 location /get_text {
     #这里也可以设置成text/plain
     default_type text/html;
     return 200 "This is nginx's text";
 }
 location /get_json{
     default_type application/json;
     return 200 '{"name":"TOM","age":18}';
 }
```

**自定义服务日志**

Nginx中日志的类型分**access.log**、**error.log**。

**access.log**：用来记录用户所有的**访问请求**。

**error.log**：记录nginx本身运行时的**错误信息**，不会记录用户的访问请求。

Nginx服务器支持对服务日志的格式、大小、输出等进行设置，需要使用到两个指令，分别是access\_log和log\_format指令。

**access\_log**：用来设置用户访问日志的相关属性。

| 语法  | access\_log path\[format\[buffer=size]] |
| --- | --------------------------------------- |
| 默认值 | access\_log logs/access.log main;       |
| 位置  | http、server、location                    |

**log\_format**：用来指定日志的输出格式。

| 语法  | log\_format name \[escape=default/json/none] string....; |
| --- | -------------------------------------------------------- |
| 默认值 | log\_format combined "...";                              |
| 位置  | http                                                     |

**其他配置指令**

**sendfile**：用来设置Nginx服务器是否使用sendfile()传输文件，该属性可以大大提高Nginx处理静态资源的性能

| 语法  | sendfile on/off；     |
| --- | -------------------- |
| 默认值 | sendfile off;        |
| 位置  | http、server、location |

**keepalive\_timeout**：用来设置长连接的超时时间。

{% code overflow="wrap" lineNumbers="true" %}
```
为什么要使用keepalive？
HTTP是一种无状态协议，客户端向服务端发送一个TCP请求，服务端响应完毕后断开连接。
如何客户端向服务端发送多个请求，每个请求都需要重新创建一次连接，效率相对来说比较差，使用keepalive模式，可以告诉服务器端在处理完一个请求后保持这个TCP连接的打开状态，若接收到来自这个客户端的其他请求，服务端就会利用这个未被关闭的连接，而不需要重新创建一个新连接，提升效率，但是这个连接也不能一直保持，这样的话，连接如果过多，也会是服务端的性能下降，这个时候就需要进行设置其的超时时间。
```
{% endcode %}

| 语法  | keepalive\_timeout time; |
| --- | ------------------------ |
| 默认值 | keepalive\_timeout 65s;  |
| 位置  | http、server、location     |

**keepalive\_requests**:用来设置一个keep-alive连接使用的次数。

| 语法  | keepalive\_requests number; |
| --- | --------------------------- |
| 默认值 | keepalive\_requests 100;    |
| 位置  | http、server、location        |

### 4.4、Server块

上网去搜索访问资源对于我们来说并不陌生，通过浏览器发送一个HTTP请求实现从客户端发送请求到服务器端获取所需要内容后并把内容回显展示在页面的一个过程。这个时候，我们所请 求的内容就分为两种类型，一类是静态资源、一类是动态资源。 静态资源即指在服务器端真实存在并且能直接拿来展示的一些文件，比如常见的html页面、css文件、js文件、图 片、视频等资源； 动态资源即指在服务器端真实存在但是要想获取需要经过一定的业务逻辑处理，根据不同的条件展示在页面不同这 一部分内容，比如说报表数据展示、根据当前登录用户展示相关具体数据等资源；

Nginx处理静态资源的内容，我们需要考虑下面这几个问题：

```
 静态资源的配置指令
 静态资源的配置优化
 静态资源的压缩配置指令
 静态资源的缓存处理
 静态资源的访问控制，包括跨域问题和防盗链问题
```

**listen指令**

**listen**：用来配置监听端口。

| 语法  | listen address\[:port] \[default\_server]...; listen port \[default\_server]...; |
| --- | -------------------------------------------------------------------------------- |
| 默认值 | listen \*:80 / \*:8000                                                           |
| 位置  | server                                                                           |

listen的设置比较灵活，通过几个例子来把常用的设置方式熟悉下：

```nginx
 listen 127.0.0.1:8000;
 #不加端口，默认监听80端口;
 listen 127.0.0.1 
 listen 8000
 listen *:8000
 listen localhost:8000
```

**default\_server**：属性是标识符，用来将此虚拟主机设置成默认主机。所谓的默认主机指的是如果没有匹配到对应的address:port，则会默认执行的。**如果不指定默认使用的是第一个server**。

```nginx
 server{
     listen 8080;
     server_name 127.0.0.1;
     location /{
         root html;
         index index.html;
     }
 }
 server{
     # 监听同一个端口，server_name如果没有匹配到 则跳转到 default_server标识的server中
     listen 8080 default_server;
     server_name localhost;
     default_type text/plain;
     return 444 'This is a error request';
 }
```

**server\_name指令**

**server\_name**：用来设置虚拟主机服务名称。

127.0.0.1 、 localhost 、book.zhouwen.top 、zhouwen.top

| 语法  | server\_name name ...; name可以提供多个中间用空格分隔 |
| --- | ---------------------------------------- |
| 默认值 | server\_name "";                         |
| 位置  | server                                   |

关于server\_name的配置方式有三种，分别是：

```
 精确匹配
 通配符匹配
 正则表达式匹配
```

**配置方式一**：精确匹配

```nginx
 server {
     listen 80;
     server_name www.itcast.cn www.itheima.cn;
     ...
 }
```

补充小知识点:

{% code overflow="wrap" %}
```
 hosts是一个没有扩展名的系统文件，可以用记事本等工具打开，其作用就是将一些常用的网址域名与其对应的IP地址建立一个关联“数据库”，当用户在浏览器中输入一个需要登录的网址时，系统会首先自动从hosts文件中寻找对应的IP地址，一旦找到，系统会立即打开对应网页，如果没有找到，则系统会再将网址提交DNS域名解析服务器进行IP地址的解析。
 ​
 windows:C:\Windows\System32\drivers\etc
 centos：/etc/hosts
 ​
 因为域名是要收取一定的费用，所以我们可以使用修改hosts文件来制作一些虚拟域名来使用。需要修改 **/etc/hosts**文件来添加
 ​
 vim /etc/hosts
 127.0.0.1 www.itcast.cn
 127.0.0.1 www.itheima.cn
```
{% endcode %}

**配置方式二**：使用通配符配置

server\_name中支持通配符"**\***",但需要注意的是通配符**不能出现在域名的中间**，只能出现在**首段或尾段**，如：

```nginx
 server {
     listen 80;
     server_name  *.itcast.cn    www.itheima.*;
     # www.itcast.cn abc.itcast.cn www.itheima.cn www.itheima.com
     ...
 }
 ​
 下面的配置就会报错
 server {
     listen 80;
     server_name  www.*.cn www.itheima.c*
     ...
 }
```

**配置三**：使用正则表达式配置

server\_name中可以使用正则表达式，并且使用**\~**作为正则表达式字符串的开始标记。

常见的正则表达式

| 代码     | 说明                                           |
| ------ | -------------------------------------------- |
| ^      | 匹配搜索字符串开始位置                                  |
| $      | 匹配搜索字符串结束位置                                  |
| .      | 匹配除换行符\n之外的任何单个字符                            |
| \\     | 转义字符，将下一个字符标记为特殊字符                           |
| \[xyz] | 字符集，与任意一个指定字符匹配                              |
| \[a-z] | 字符范围，匹配指定范围内的任何字符                            |
| \w     | 与以下任意字符匹配 A-Z a-z 0-9 和下划线,等效于\[A-Za-z0-9\_] |
| \d     | 数字字符匹配，等效于\[0-9]                             |
| {n}    | 正好匹配n次                                       |
| {n,}   | 至少匹配n次                                       |
| {n,m}  | 匹配至少n次至多m次                                   |
| \*     | 零次或多次，等效于{0,}                                |
| +      | 一次或多次，等效于{1,}                                |
| ?      | 零次或一次，等效于{0,1}                               |

配置如下：

```sh
 server{
         listen 80;
         server_name ~^www\.(\w+)\.com$;
         default_type text/plain;
         return 200 $1  $2 ..;
 }
 注意 ~后面不能加空格，括号可以取值
```

**匹配执行顺序**

由于server\_name指令支持通配符和正则表达式，因此在包含多个虚拟主机的配置文件中，可能会出现一个名称被多个虚拟主机的server\_name匹配成功，当遇到这种情况，当前的请求交给谁来处理呢？

```nginx
 server{
     listen 80;
     server_name ~^www\.\w+\.com$;
     default_type text/plain;
     return 200 'regex_success';
 }
 ​
 server{
     listen 80;
     server_name www.itheima.*;
     default_type text/plain;
     return 200 'wildcard_after_success';
 }
 ​
 server{
     listen 80;
     server_name *.itheima.com;
     default_type text/plain;
     return 200 'wildcard_before_success';
 }
 ​
 server{
     listen 80;
     server_name www.itheima.com;
     default_type text/plain;
     return 200 'exact_success';
 }
 ​
 server{
     listen 80 default_server;
     server_name _;
     default_type text/plain;
     return 444 'default_server not found server';
 }
 ​
```

结论：

```
 exact_success
 wildcard_before_success
 wildcard_after_success
 regex_success
 default_server not found server!!
```

```
 No1:准确匹配server_name
 No2:通配符在开始时匹配server_name成功
 No3:通配符在结束时匹配server_name成功
 No4:正则表达式匹配server_name成功
 No5:被默认的default_server处理，如果没有指定默认找第一个server
```

### 4.5、Location块

**location**：用来设置请求的URI

| 语法  | location \[ = / \~ / \~\* / ^\~ / @ ] uri{...} |
| --- | ---------------------------------------------- |
| 默认值 | —                                              |
| 位置  | server,location                                |

**uri**变量是待匹配的请求字符串，可以不包含正则表达式，也可以包含正则表达式，那么nginx服务器在搜索匹配location的时候，是先**使用不包含正则表达式进行匹配**，找到一个匹配度最高的一个，然后在通过包含正则表达式的进行匹配，如果能匹配到直接访问，匹配不到，就使用刚才匹配度最高的那个location来处理请求。

属性介绍:

```nginx
 不带符号，要求必须以指定模式开始
 server {
     listen 80;
     server_name 127.0.0.1;
     location /abc{
         default_type text/plain;
         return 200 "access success";
     }
 }
 ​
 以下访问都是正确的
 http://192.168.200.133/abc
 http://192.168.200.133/abc?p1=TOM
 http://192.168.200.133/abc/
 http://192.168.200.133/abcdef
```

**=** : 用于**不包含正则表达式的uri前**，必须与指定的模式精确匹配

```nginx
 server {
     listen 80;
     server_name 127.0.0.1;
     location =/abc{
         default_type text/plain;
         return 200 "access success";
     }
 }
 ​
 可以匹配到
 http://192.168.200.133/abc
 http://192.168.200.133/abc?p1=TOM
 ​
 匹配不到
 http://192.168.200.133/abc/
 http://192.168.200.133/abcdef
```

**\~**：用于表示**当前uri中包含了正则表达式**，并且**区分大小写**

**\~\***：用于表示**当前uri中包含了正则表达式**，并且**不区分大小写**

换句话说，如果uri包含了正则表达式，需要用上述两个符合来标识

```nginx
 server {
     listen 80;
     server_name 127.0.0.1;
     location ~^/abc\w${
         default_type text/plain;
         return 200 "access success";
     }
 }
 ​
 server {
     listen 80;
     server_name 127.0.0.1;
     location ~*^/abc\w${
         default_type text/plain;
         return 200 "access success";
     }
 }
```

**^\~**：用于**不包含正则表达式的uri前**，功能和不加符号的一致，唯一不同的是，如果**模式匹配**，那么就**停止搜索其他模式**了。

```nginx
 server {
     listen 80;
     server_name 127.0.0.1;
     location ^~/abc{
         default_type text/plain;
         return 200 "access success";
     }
 }
```

**设置请求资源的目录root / aliass**

**root**：设置请求的根目录

| 语法  | root path;           |
| --- | -------------------- |
| 默认值 | root html;           |
| 位置  | http、server、location |

**path**：为Nginx服务器接收到请求以后查找资源的根目录路径。

**alias**：用来更改location的URI

| 语法  | alias path; |
| --- | ----------- |
| 默认值 | —           |
| 位置  | location    |

path为修改后的根路径。

**以上两个指令都可以来指定访问资源的路径**，那么这**两者之间的区别是什么**?

```
# Root
在/usr/local/nginx/html目录下创建一个 images目录,并在目录下放入一张图片mv.png图片
location /images {
	root /usr/local/nginx/html;
}

访问图片的路径为:
http://192.168.200.133/images/mv.png

# Alias
如果把root改为alias
再次访问上述地址，页面会出现404的错误，查看错误日志会发现是因为地址不对
root的处理结果是: root路径+location路径
/usr/local/nginx/html/images/mv.png
alias的处理结果是:使用alias路径替换location路径
/usr/local/nginx/html/images

需要在alias后面路径改为
location /images {
	alias /usr/local/nginx/html/images;
}

alias是一个目录别名的定义，root则是最上层目录的定义
如果location路径是以/结尾,则alias也必须是以/结尾，root没有要求

将上述配置修改为
location /images/ {
	alias /usr/local/nginx/html/images;
}
访问就会出问题，查看错误日志还是路径不对，所以需要把alias后面加上 / 

# 小结
root的处理结果是: root路径+location路径
alias的处理结果是:使用alias路径替换location路径
alias是一个目录别名的定义，root则是最上层目录的含义。
如果location路径是以/结尾,则alias也必须是以/结尾，root没有要求
```

## 5、常用配置

```nginx
# 将所有http请求重定向到https
rewrite ^(/.*)$ https://$host$1 permanent;
if ($server_port !~ 443){
	rewrite ^(/.*)$ https://$host$1 permanent;
}

# 从定向 如果访问域名不是www.a.cn，强制从定向到http://www.a.cn
if ( $host != 'www.a.cn' ) {
	rewrite ^(.*)$ https://www.a.cn$1 permanent;
}

# https配置
server {
    listen      443 ssl http2;
    server_name book.zhouwen.top;
	index index.php index.html index.htm default.php default.htm default.html;
root /www/wwwroot/book.zhouwen.top;

    ssl_certificate   /etc/ssl/jkdev.cn/cert.pem;
    ssl_certificate_key  /etc/ssl/jkdev.cn/key.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
}
ssl_certificate：证书公钥文件
ssl_certificate_key：证书私钥文件
ssl_session_timeout：缓存有效期
ssl_protocols：安全链接可选的加密协议
ssl_ciphers：加密算法
ssl_prefer_server_ciphers：使用服务器端的首选算法
```

### 5.1、静态资源优化配置

Nginx对静态资源如何进行优化配置。这里从三个属性配置进行优化：

```nginx
 sendfile on;
 tcp_nopush on;
 tcp_nodeplay on;
```

**sendﬁle**：用来开启高效的文件传输模式。

| 语法  | sendﬁle on /oﬀ;         |
| --- | ----------------------- |
| 默认值 | sendﬁle oﬀ;             |
| 位置  | http、server、location... |

请求静态资源的过程：客户端通过网络接口向服务端发送请求，操作系统将这些客户端的请求传递给服务器端应用程序，服务器端应用程序会处理这些请求，请求处理完成以后，操作系统还需要将处理得到的结果通过网络适配器传递回去。

如：

```nginx
 server {
     listen 80;
     server_name localhost；
     location / {
         root html;
         index index.html;
     }
 }
 在html目录下有一个welcome.html页面，访问地址
 http://192.168.139.6/welcome.html
```

<figure><img src="../.gitbook/assets/image (383).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (384).png" alt=""><figcaption></figcaption></figure>

**tcp\_nopush**：该指令必须在**sendfile**打开的状态下才会生效，主要是用来提升网络包的**传输'效率'**

| 语法  | tcp\_nopush on/off;  |
| --- | -------------------- |
| 默认值 | tcp\_nopush oﬀ;      |
| 位置  | http、server、location |

**tcp\_nodelay**：该指令必须在**keep-alive**连接开启的情况下才生效，来提高网络包**传输的'实时性'**

| 语法  | tcp\_nodelay on/off; |
| --- | -------------------- |
| 默认值 | tcp\_nodelay on;     |
| 位置  | http、server、location |

<figure><img src="../.gitbook/assets/image (385).png" alt=""><figcaption></figcaption></figure>

经过刚才的分析，"tcp\_nopush"和”tcp\_nodelay“**看起来是**"**互斥**的"，那么为什么要将这两个值都打开，这个大家需要知道的是在linux2.5.9以后的版本中两者是可以兼容的，三个指令都开启的好处是，sendfile可以开启高效的文件传输模式，tcp\_nopush开启可以确保在发送到客户端之前数据包已经充分“填满”， 这大大减少了网络开销，并加快了文件发送的速度。 然后，当它到达最后一个可能因为没有“填满”而暂停的数据包时，Nginx会忽略tcp\_nopush参数， 然后，tcp\_nodelay强制套接字发送数据。由此可知，TCP\_NOPUSH可以与TCP\_NODELAY一起设置，它比单独配置TCP\_NODELAY具有更强的性能。所以我们可以使用如下配置来优化Nginx静态资源的处理

```nginx
 sendfile on;
 tcp_nopush on;
 tcp_nodelay on;
```

### 5.2、静态资源压缩配置

经过上述内容的优化，我们再次思考一个问题，假如在满足上述优化的前提下，我们传送一个1M的数据和一个10M的数据那个效率高?，答案显而易见，传输内容小，速度就会快。那么问题又来了，同样的内容，如果把大小降下来，我们脑袋里面要蹦出一个词就是"压缩"，接下来，我们来学习Nginx的静态资源压缩模块。

在Nginx的配置文件中可以通过配置**gzip**来对静态资源进行压缩，相关的指令可以配置在http块、server块和location块中，

```sh
 # Nginx可以通过以下模块进行解析和处理。
 ngx_http_gzip_module模块
 ngx_http_gzip_static_module模块
 ngx_http_gunzip_module模块
 ​
 Gzip各模块支持的配置指令
 Gzip压缩功能的配置
 Gzip和sendfile的冲突解决
 浏览器不支持Gzip的解决方案
```

#### **5.2.1、Gzip模块配置指令**

接下来所学习的指令都来自ngx\_http\_gzip\_module模块，该模块会在nginx安装的时候内置到nginx的安装环境中，也就是说我们可以直接使用这些指令

**gzip指令**：该指令用于开启或者关闭gzip功能

| 语法  | gzip on/off;            |
| --- | ----------------------- |
| 默认值 | gzip off;               |
| 位置  | http、server、location... |

注意只有该指令为打开状态，下面的指令才有效果

```nginx
 http{
    gzip on;
 }
```

**gzip\_types指令**：该指令可以根据响应页的MIME类型选择性地开启Gzip压缩功能

| 语法  | gzip\_types mime-type ...; |
| --- | -------------------------- |
| 默认值 | gzip\_types text/html;     |
| 位置  | http、server、location       |

所选择的值可以从mime.types文件中进行查找，也可以使用"\*"代表所有。

```nginx
 http{
     gzip_types application/javascript;
 }
```

**gzip\_comp\_level指令**：该指令用于设置Gzip压缩程度，级别从1-9,1表示要是程度最低，效率最高，9刚好相反，压缩程度最高，但是效率最低最费时间。

| 语法  | gzip\_comp\_level level; |
| --- | ------------------------ |
| 默认值 | gzip\_comp\_level 1;     |
| 位置  | http、server、location     |

```nginx
 http{
     gzip_comp_level 6;
 }
```

**gzip\_vary指令**：该指令用于设置使用Gzip进行压缩发送是否携带“Vary:Accept-Encoding”头域的响应头部。主要是告诉接收方，所发送的数据经过了Gzip压缩处理

| 语法  | gzip\_vary on/off;   |
| --- | -------------------- |
| 默认值 | gzip\_vary off;      |
| 位置  | http、server、location |

<figure><img src="../.gitbook/assets/image (386).png" alt=""><figcaption></figcaption></figure>

**gzip\_buffers指令**：该指令用于处理请求压缩的缓冲区数量和大小。

| 语法  | gzip\_buffers number size;   |
| --- | ---------------------------- |
| 默认值 | gzip\_buffers 32 4k / 16 8k; |
| 位置  | http、server、location         |

其中**number**:指定Nginx服务器向系统申请缓存空间个数，**size**指的是每个缓存空间的大小。主要实现的是申请number个每个大小为size的内存空间。这个值的设定一般会和服务器的操作系统有关，所以建议此项不设置，使用默认值即可。

```nginx
 # 缓存空间大小
 gzip_buffers 4 16K;  
```

**gzip\_disable指令**：针对不同种类客户端发起的请求，可以选择性地开启和关闭Gzip功能。

| 语法  | gzip\_disable regex ...; |
| --- | ------------------------ |
| 默认值 | —                        |
| 位置  | http、server、location     |

**regex**：根据客户端的浏览器标志(user-agent)来设置，支持使用正则表达式。指定的浏览器标志不使用Gzip.该指令一般是用来排除一些明显不支持Gzip的浏览器。

```nginx
 # 例如
 gzip_disable "MSIE [1-6]\.";
```

**gzip\_http\_version指令**：针对不同的HTTP协议版本，可以选择性地开启和关闭Gzip功能。

| 语法  | gzip\_http\_version 1.0\|1.1; |
| --- | ----------------------------- |
| 默认值 | gzip\_http\_version 1.1;      |
| 位置  | http、server、location          |

该指令是指定使用Gzip的HTTP最低版本，该指令一般采用默认值即可。

**gzip\_min\_length指令**：该指令针对传输数据的大小，可以选择性地开启和关闭Gzip功能

| 语法  | gzip\_min\_length length; |
| --- | ------------------------- |
| 默认值 | gzip\_min\_length 20;     |
| 位置  | http、server、location      |

```
 nignx计量大小的单位：bytes[字节] / kb[千字节] / M[兆]
 例如: 1024 / 10k|K / 10m|M
```

Gzip压缩功能对大数据的压缩效果明显，但是如果要压缩的数据比较小的化，可能出现越压缩数据量越大的情况，因此我们需要根据响应内容的大小来决定是否使用Gzip功能，响应页面的大小可以通过头信息中的**Content-Length**来获取。但是如何使用了Chunk编码动态压缩，该指令将被忽略。建议设置为**1K**或以上。

**gzip\_proxied指令**：该指令设置是否对服务端返回的结果进行Gzip压缩。

| 语法  | gzip\_proxied off / expired / no-cache / no-store / private / no\_last\_modified / no\_etag / auth / any; |
| --- | --------------------------------------------------------------------------------------------------------- |
| 默认值 | gzip\_proxied off;                                                                                        |
| 位置  | http、server、location                                                                                      |

```nginx
 off              - 关闭Nginx服务器对后台服务器返回结果的Gzip压缩
 expired          - 启用压缩，如果header头中包含 "Expires" 头信息
 no-cache         - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
 no-store         - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
 private          - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
 no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
 no_etag          - 启用压缩 ,如果header头中不包含 "ETag" 头信息
 auth             - 启用压缩 , 如果header头中包含 "Authorization" 头信息
 any              - 无条件启用压缩
```

#### 5.2.2、Gzip压缩功能的实例配置

```nginx
 gzip on;               #开启gzip功能
 gzip_types *;          #压缩源文件类型,根据具体的访问资源类型设定
 gzip_comp_level 6;     #gzip压缩级别
 gzip_min_length 1024;  #进行压缩响应页面的最小长度,content-length
 gzip_buffers 4 16K;    #缓存空间大小
 gzip_http_version 1.1; #指定压缩响应所需要的最低HTTP请求版本
 gzip_vary  on;         #往头信息中添加压缩标识
 gzip_disable "MSIE [1-6]\."; #对IE6以下的版本都不进行压缩
 gzip_proxied  off；  #nginx作为反向代理压缩服务端返回数据的条件
```

这些配置在很多地方可能都会用到，所以我们可以将这些内容抽取到一个配置文件中，然后通过include指令把配置文件再次加载到nginx.conf配置文件中，方法使用。

```nginx
 # nginx_gzip.conf
 gzip on;
 gzip_types *;
 gzip_comp_level 6;
 gzip_min_length 1024;
 gzip_buffers 4 16K;
 gzip_http_version 1.1;
 gzip_vary  on;
 gzip_disable "MSIE [1-6]\.";
 gzip_proxied  off;
 ​
 # nginx.conf
 include nginx_gzip.conf
```

#### 5.2.3、Gzip和sendfile共存问题

前面在讲解sendfile的时候，提到过，开启sendfile以后，在读取磁盘上的静态资源文件的时候，可以减少拷贝的次数，可以不经过用户进程将静态文件通过网络设备发送出去，但是Gzip要想对资源压缩，是需要经过用户进程进行操作的。所以如何解决两个设置的共存问题。

可以使用**ngx\_http\_gzip\_static\_module**模块的**gzip\_static**指令来解决。

**gzip\_static指令**：检查与访问资源同名的.gz文件时，response中以gzip相关的header返回.gz文件的内容。

| 语法  | **gzip\_static** on / off / always; |
| --- | ----------------------------------- |
| 默认值 | gzip\_static off;                   |
| 位置  | http、server、location                |

添加上述命令后，会报一个错误，`unknown directive "gzip_static"`主要的原因是Nginx默认是没有添加ngx\_http\_gzip\_static\_module模块。如何来添加?

```sh
 # 添加模块到Nginx的实现步骤
 (1)查询当前Nginx的配置参数
 nginx -V
 ​
 (2)将nginx安装目录下sbin目录中的nginx二进制文件进行更名
 cd /usr/local/nginx/sbin
 mv nginx nginxold
 ​
 (3)进入Nginx的安装目录
 cd /root/nginx/core/nginx-版本
 ​
 (4)执行make clean清空之前编译的内容
 make clean
 ​
 (5)使用configure来配置参数
 ./configure --with-http_gzip_static_module
 ​
 (6)使用make命令进行编译
 make
 ​
 (7)将objs目录下的nginx二进制执行文件移动到nginx安装目录下的sbin目录中
 mv objs/nginx /usr/local/nginx/sbin
 ​
 (8)执行更新命令
 make upgrade
```

**gzip\_static测试使用**

直接访问 [http://192.168.139.6/jquery.js](http://192.168.139.6/jquery.js)

<figure><img src="../.gitbook/assets/image (387).png" alt=""><figcaption></figcaption></figure>

使用gzip命令进行压缩

```
 cd /usr/local/nginx/html
 gzip jquery.js
 ​
```

再次访问 [http://192.168.139.6/jquery.js](http://192.168.139.6/jquery.js)

<figure><img src="../.gitbook/assets/image (388).png" alt=""><figcaption></figcaption></figure>

### 5.3、静态资源缓存处理

{% code overflow="wrap" %}
```
 # 什么是缓存
 缓存（cache），原始意义是指访问速度比一般随机存取存储器（RAM）快的一种高速存储器，通常它不像系统主存那样使用DRAM技术，而使用昂贵但较快速的SRAM技术。缓存的设置是所有现代计算机系统发挥高性能的重要因素之一。
 ​
 # 什么是web缓存
 Web缓存是指一个Web资源（如html页面，图片，js，数据等）存在于Web服务器和客户端（浏览器）之间的副本。缓存会根据进来的请求保存输出内容的副本；当下一个请求来到的时候，如果是相同的URL，缓存会根据缓存机制决定是直接使用副本响应访问请求，还是向源服务器再次发送请求。比较常见的就是浏览器会缓存访问过网站的网页，当再次访问这个URL地址的时候，如果网页没有更新，就不会再次下载网页，而是直接使用本地缓存的网页。只有当网站明确标识资源已经更新，浏览器才会再次下载网页。
 ​
 # web缓存的种类
 客户端缓存
     浏览器缓存
 服务端缓存
     Nginx / Redis / Memcached等
 ​
 # 浏览器缓存
 是为了节约网络的资源加速浏览，浏览器在用户磁盘上对最近请求过的文档进行存储，当访问者再次请求这个页面时，浏览器就可以从本地磁盘显示文档，这样就可以加速页面的阅览。
 ​
 # 为什么要用浏览器缓存
     成本最低的一种缓存实现
     减少网络带宽消耗
     降低服务器压力
     减少网络延迟，加快页面打开速度
```
{% endcode %}

**浏览器缓存的执行流程**

HTTP协议中和页面缓存相关的字段，我们先来认识下：

| header        | 说明                      |
| ------------- | ----------------------- |
| Expires       | 缓存过期的日期和时间              |
| Cache-Control | 设置和缓存相关的配置信息            |
| Last-Modified | 请求资源最后修改时间              |
| ETag          | 请求变量的实体标签的当前值，比如文件的MD5值 |

{% code overflow="wrap" %}
```
 （1）用户首次通过浏览器发送请求到服务端获取数据，客户端是没有对应的缓存，所以需要发送request请求来获取数据；
 （2）服务端接收到请求后，获取服务端的数据及服务端缓存的允许后，返回200的成功状态码并且在响应头上附上对应资源以及缓存信息；
 （3）当用户再次访问相同资源的时候，客户端会在浏览器的缓存目录中查找是否存在响应的缓存文件
 （4）如果没有找到对应的缓存文件，则走(2)步
 （5）如果有缓存文件，接下来对缓存文件是否过期进行判断，过期的判断标准是(Expires),
 （6）如果没有过期，则直接从本地缓存中返回数据进行展示
 （7）如果Expires过期，接下来需要判断缓存文件是否发生过变化
 （8）判断的标准有两个，一个是ETag(Entity Tag),一个是Last-Modified
 （9）判断结果是未发生变化，则服务端返回304，直接从缓存文件中获取数据
 （10）如果判断是发生了变化，重新从服务端获取数据，并根据缓存协商(服务端所设置的是否需要进行缓存数据的设置)来进行数据缓存。
```
{% endcode %}

**浏览器缓存相关指令**

**expires指令**：该指令用来控制页面缓存的作用。可以通过该指令控制HTTP应答中的“Expires"和”Cache-Control"

| 语法  | expires \[modified] time expires epoch / max / off; |
| --- | --------------------------------------------------- |
| 默认值 | expires off;                                        |
| 位置  | http、server、location                                |

* **time**：可以整数也可以是负数，指定过期时间，**如果是负数，Cache-Control则为no-cache**，**如果为整数或0，则Cache-Control的值为max-age=time;**
* **epoch**：指定Expires的值为'1 January,1970,00:00:01 GMT'(1970-01-01 00:00:00)，Cache-Control的值no-cache
* **max**：指定Expires的值为'31 December2037 23:59:59GMT' (2037-12-31 23:59:59) ，Cache-Control的值为10年
* **off**：默认不缓存

**add\_header指令**：指令是用来添加指定的**响应头和响应值**。

| 语法  | add\_header name value \[always]; |
| --- | --------------------------------- |
| 默认值 | —                                 |
| 位置  | http、server、location...           |

Cache-Control作为响应头信息，可以设置如下值：

```
 # 缓存响应指令：
 Cache-control: must-revalidate   # 可缓存但必须再向源服务器进行确认
 Cache-control: no-cache          # 缓存前必须确认其有效性
 Cache-control: no-store          # 不缓存请求或响应的任何内容
 Cache-control: no-transform      # 代理不可更改媒体类型
 Cache-control: public            # 可向任意方提供响应的缓存
 Cache-control: private           # 仅向特定用户返回响应
 Cache-control: proxy-revalidate  # 要求中间缓存服务器对缓存的响应有效性再进行确认
 Cache-Control: max-age=<seconds> # 响应最大Age值
 Cache-control: s-maxage=<seconds># 公共缓存服务器响应的最大Age值
```

### 5.4、跨域问题解决

```
 # 这块内容，我们主要从以下方面进行解决：
 什么情况下会出现跨域问题?
 实例演示跨域问题
 具体的解决方案是什么?
```

**同源策略**

浏览器的同源策略：是一种**约定**，是浏览器**最核心也是最基本的安全功能**，如果**浏览器少了同源策略**，则浏览器的**正常功能可能都会受到影响**。

同源：**协议**、**域名(IP)**、**端口相同即为同源**

```
 http://192.168.200.131/user/1
 https://192.168.200.131/user/1
 不
 ​
 http://192.168.200.131/user/1
 http://192.168.200.132/user/1
 不
 ​
 http://192.168.200.131/user/1
 http://192.168.200.131:8080/user/1
 不
 ​
 http://www.nginx.com/user/1
 http://www.nginx.org/user/1
 不
 ​
 http://www.nginx.org:80/user/1
 http://www.nginx.org/user/1
 满足
```

**跨域问题**

{% code overflow="wrap" %}
```
 # 简单描述下:
 有两台服务器分别为A,B,如果从服务器A的页面发送异步请求到服务器B获取数据，如果服务器A和服务器B不满足同源策略，则就会出现跨域问题。
```
{% endcode %}

**跨域问题的案例演示**

出现跨域问题会有什么效果?,接下来通过一个需求来给大家演示下：

```html
 #（1）nginx的html目录下新建一个a.html
 <html>
   <head>
         <meta charset="utf-8">
         <title>跨域问题演示</title>
         <script src="jquery.js"></script>
         <script>
             $(function(){
                 $("#btn").click(function(){
                         $.get('http://192.168.200.133:8080/getUser',function(data){
                                 alert(JSON.stringify(data));
                         });
                 });
             });
         </script>
   </head>
   <body>
         <input type="button" value="获取数据" id="btn"/>
   </body>
 </html>
```

```nginx
#（2）在nginx.conf配置如下内容
 server{
     listen  8080;
     server_name localhost;
     location /getUser{
             default_type application/json;
             return 200 '{"id":1,"name":"TOM","age":18}';
     }
 }
 server{
     listen  80;
     server_name localhost;
     location /{
         root html;
         index index.html;
     }
 }
```

通过浏览器访问测试

<figure><img src="../.gitbook/assets/image (389).png" alt=""><figcaption></figcaption></figure>

**解决方案**

**add\_header指令**：该指令可以用来添加一些头信息

| 语法  | add\_header name value... |
| --- | ------------------------- |
| 默认值 | —                         |
| 位置  | http、server、location      |

此处用来解决跨域问题，需要添加两个头信息，一个是**Access-Control-Allow-Origin**,**Access-Control-Allow-Methods**

**Access-Control-Allow-Origin**: 直译过来是允许跨域访问的源地址信息，可以配置多个(多个用**逗号分隔**)，也可以使用**\*代表所有源**

**Access-Control-Allow-Methods**:直译过来是允许跨域访问的请求方式，值可以为 **GET POST PUT DELETE...**,可以全部设置，也可以根据需要设置，多个用**逗号分隔**

具体配置方式

```nginx
 location /getUser{
     add_header Access-Control-Allow-Origin *;
     add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE;
     default_type application/json;
     return 200 '{"id":1,"name":"TOM","age":18}';
 }
```

### 5.5、静态资源防盗链

**什么是资源盗链**

资源盗链指的是此内容不在自己服务器上，而是通过技术手段，绕过别人的限制将别人的内容放到自己页面上最终展示给用户。以此来盗取大网站的空间和流量。简而言之就是用别人的东西成就自己的网站。

{% code overflow="wrap" %}
```nginx
 #效果演示
 京东:
 https://img14.360buyimg.com/n7/jfs/t1/101062/37/2153/254169/5dcbd410E6d10ba22/4ddbd212be225fcd.jpg
 ​
 百度:
 https://pics7.baidu.com/feed/cf1b9d16fdfaaf516f7e2011a7cda1e8f11f7a1a.jpeg
```
{% endcode %}

我们自己准备一个页面，在页面上引入这两个图片查看效果

<figure><img src="../.gitbook/assets/image (390).png" alt=""><figcaption></figcaption></figure>

从上面的效果，可以看出来，下面的图片地址添加了**防止盗链**的功能，京东这边我们可以直接使用其图片。

**Nginx防盗链的实现原理：**

了解防盗链的原理之前，我们得先学习一个HTTP的头信息**Referer**，当浏览器向web服务器发送请求的时候，一般都会带上Referer,来告诉浏览器该网页是从哪个页面链接过来的。

后台服务器可以根据获取到的这个Referer信息来判断是否为自己信任的网站地址，如果是则放行继续访问，如果不是则可以返回403(服务端拒绝访问)的状态信息。

在本地模拟上述的服务器效果：

<figure><img src="../.gitbook/assets/image (391).png" alt=""><figcaption></figcaption></figure>

**Nginx防盗链的具体实现：**

**valid\_referers**：nginx会通就过查看**referer**自动和**valid\_referers**后面的内容进行匹配，如果匹配到了就将**$invalid\_referer变量置0**，如果没有匹配到，则将**$invalid\_referer变量置为1**，匹配的过程中不区分大小写。

| 语法  | valid\_referers none / blocked / server\_names / string... |
| --- | ---------------------------------------------------------- |
| 默认值 | —                                                          |
| 位置  | server、location                                            |

* **none**：如果Header中的Referer为空，允许访问
* **blocked**：在Header中的Referer不为空，但是该值被防火墙或代理进行伪装过，如不带http:、https:等协议头的资源允许访问
* **server\_names**：指定具体的域名或者IP
* **string**：可以支持正则表达式和\*的字符串。如果是正则表达式，需要以**\~开头**表示，例如

{% code overflow="wrap" lineNumbers="true" %}
```nginx
 location ~*\.(png|jpg|gif){
     valid_referers none blocked www.baidu.com 192.168.200.222 *.example.com example.*  www.example.org  ~\.google\.;
     if ($invalid_referer){
          return 403;
     }
     root /usr/local/nginx/html;
 }
```
{% endcode %}

遇到的问题:图片有很多，该如何批量进行防盗链？

**针对目录进行防盗链**

{% code overflow="wrap" lineNumbers="true" %}
```nginx
 location /images {
     valid_referers none blocked www.baidu.com 192.168.200.222 *.example.com example.*  www.example.org  ~\.google\.;
     if ($invalid_referer){
          return 403;
     }
     root /usr/local/nginx/html;
 }
```
{% endcode %}

这样我们可以对一个目录下的所有资源进行防盗链操作。

**遇到的问题**：Referer的限制**粒度比较粗**，比如随意加一个Referer，上面的方式是无法进行限制的。那么这个问题改如何解决？

此处我们需要用到Nginx的第三方模块**ngx\_http\_accesskey\_module**，第三方模块如何实现盗链，如何在Nginx中使用第三方模块的功能，这些在后面的**Nginx的模块篇**再进行详细探讨

### 5.6、Rewrite功能配置

Rewrite是Nginx服务器提供的一个**重要基本功能**，是Web服务器产品中几乎必备的功能。

主要的作用是用来实现**URL的重写**。

注意：Nginx服务器的Rewrite功能的实现依赖于**PCRE的支持**，因此在编译安装Nginx服务器之前，需要安装PCRE库。

Nginx使用的是**ngx\_http\_rewrite\_module模块**来解析和处理Rewrite功能的相关配置。

**"地址重写"与"地址转发"**

```
 # 重写和转发的区别:
 地址重写浏览器地址会发生变化而地址转发则不变
 一次地址重写会产生两次请求而一次地址转发只会产生一次请求
 地址重写到的页面必须是一个完整的路径而地址转发则不需要
 地址重写因为是两次请求所以request范围内属性不能传递给新页面而地址转发因为是一次请求所以可以传递值
 地址转发速度快于地址重写
```

**Rewrite规则**

**set指令**：该指令用来设置一个新的变量。

| 语法  | set $variable value; |
| --- | -------------------- |
| 默认值 | —                    |
| 位置  | server、location、if   |

**variable**：变量的名称，该变量名称要用"$"作为变量的第一个字符，且不能与Nginx服务器预设的全局变量同名。

**value**：变量的值，可以是字符串、其他变量或者变量的组合等。

**Rewrite常用全局变量**

| 变量                   | 说明                                                                                                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $args                | 变量中存放了请求URL中的请求指令。比如[http://192.168.200.133:8080?arg1=value1\&args2=value2](http://192.168.200.133:8080/?arg1=value1\&args2=value2) 中的"arg1=value1\&arg2=value2" 功能和$query\_string一样 |
| $http\_user\_agent   | 变量存储的是用户访问服务的代理信息(如果通过浏览器访问，记录的是浏览器的相关版本信息)                                                                                                                                          |
| $host                | 变量存储的是访问服务器的server\_name值                                                                                                                                                            |
| $document\_uri       | 变量存储的是当前访问地址的URI。比如[http://192.168.200.133/server?id=10\&name=zhangsan](http://192.168.200.133/server?id=10\&name=zhangsan) 中的"/server"，功能和$uri一样                                    |
| $document\_root      | 变量存储的是当前请求对应location的root值，如果未设置，默认指向Nginx自带html目录所在位置                                                                                                                               |
| $content\_length     | 变量存储的是请求头中的Content-Length的值                                                                                                                                                          |
| $content\_type       | 变量存储的是请求头中的Content-Type的值                                                                                                                                                            |
| $http\_cookie        | 变量存储的是客户端的cookie信息，可以通过add\_header Set-Cookie 'cookieName=cookieValue'来添加cookie数据                                                                                                    |
| $limit\_rate         | 变量中存储的是Nginx服务器对网络连接速率的限制，也就是Nginx配置中对limit\_rate指令设置的值，默认是0，不限制。                                                                                                                    |
| $remote\_addr        | 变量中存储的是客户端的IP地址                                                                                                                                                                      |
| $remote\_port        | 变量中存储了客户端与服务端建立连接的端口号                                                                                                                                                                |
| $remote\_user        | 变量中存储了客户端的用户名，需要有认证模块才能获取                                                                                                                                                            |
| $scheme              | 变量中存储了访问协议                                                                                                                                                                           |
| $server\_addr        | 变量中存储了服务端的地址                                                                                                                                                                         |
| $server\_name        | 变量中存储了客户端请求到达的服务器的名称                                                                                                                                                                 |
| $server\_port        | 变量中存储了客户端请求到达服务器的端口号                                                                                                                                                                 |
| $server\_protocol    | 变量中存储了客户端请求协议的版本，比如"HTTP/1.1"                                                                                                                                                        |
| $request\_body\_file | 变量中存储了发给后端服务器的本地文件资源的名称                                                                                                                                                              |
| $request\_method     | 变量中存储了客户端的请求方式，比如"GET","POST"等                                                                                                                                                       |
| $request\_filename   | 变量中存储了当前请求的资源文件的路径名                                                                                                                                                                  |
| $request\_uri        | 变量中存储了当前请求的URI，并且携带请求参数，比如[http://192.168.200.133/server?id=10\&name=zhangsan](http://192.168.200.133/server?id=10\&name=zhangsan) 中的"/server?id=10\&name=zhangsan"                  |

**if指令**：该指令用来支持条件判断，并根据条件判断结果选择不同的Nginx配置。

| 语法  | if (condition){...} |
| --- | ------------------- |
| 默认值 | —                   |
| 位置  | server、location     |

condition为判定条件，可以支持以下写法：

{% code overflow="wrap" lineNumbers="true" %}
```nginx
 # （1）变量名。如果变量名对应的值为空或者是0，if都判断为false,其他条件为true
 if ($param){
 }
 ​
 # （2）使用"="和"!="比较变量和字符串是否相等，满足条件为true，不满足为false
 if ($request_method = POST){nginx
     return 405;
 }
 # 注意：此处和Java不太一样的地方是字符串不需要添加引号。
 ​
 # （3）使用正则表达式对变量进行匹配，匹配成功返回true，否则返回false。变量与正则表达式之间使用"~","~*","!~","!~\*"来连接
     "~"代表匹配正则表达式过程中区分大小写
     "~\*"代表匹配正则表达式过程中不区分大小写
     "!~"和"!~\*"刚好和上面取相反值，如果匹配上返回false,匹配不上返回true
 if ($http_user_agent ~ MSIE){
     # $http_user_agent的值中是否包含MSIE字符串，如果包含返回true
 }
 # 注意：正则表达式字符串一般不需要加引号，但是如果字符串中包含"}"或者是";"等字符时，就需要把引号加上。
 ​
 # （4）判断请求的文件是否存在使用"-f"和"!-f"
     当使用"-f"时，如果请求的文件存在返回true，不存在返回false。
     当使用"!f"时，如果请求文件不存在，但该文件所在目录存在返回true,文件和目录都不存在返回false,如果文件存在返回false
 if (-f $request_filename){
     #判断请求的文件是否存在
 }
 if (!-f $request_filename){
     #判断请求的文件是否不存在
 }
 ​
 # （5）判断请求的目录是否存在使用"-d"和"!-d"
     当使用"-d"时，如果请求的目录存在，if返回true，如果目录不存在则返回false
     当使用"!-d"时，如果请求的目录不存在但该目录的上级目录存在则返回true，该目录和它上级目录都不存在则返回false,如果请求目录存在也返回false.
     
 # （6）判断请求的目录或者文件是否存在使用"-e"和"!-e"
     当使用"-e",如果请求的目录或者文件存在时，if返回true,否则返回false.
     当使用"!-e",如果请求的文件和文件所在路径上的目录都不存在返回true,否则返回false
     
 # （7）判断请求的文件是否可执行使用"-x"和"!-x"
     当使用"-x",如果请求的文件可执行，if返回true,否则返回false
     当使用"!-x",如果请求文件不可执行，返回true,否则返回false
 ​
```
{% endcode %}

**break指令**：该指令用于中断当前相同作用域中的其他Nginx配置。与该指令处于同一作用域的Nginx配置中，位于它前面的指令配置生效，位于后面的指令配置无效。

| 语法  | break;             |
| --- | ------------------ |
| 默认值 | —                  |
| 位置  | server、location、if |

```nginx
 # 例子:
 location /{
     if ($param){
         set $id $1;
         break;
         limit_rate 10k;  # limit_rate变量中存储的是Nginx服务器对网络连接速率的限制
     }
 }
```

**return指令**：该指令用于完成对请求的处理，直接向客户端返回响应状态代码。在return后的所有Nginx配置都是无效的。

| 语法  | return code \[text]; return code URL; return URL; |
| --- | ------------------------------------------------- |
| 默认值 | —                                                 |
| 位置  | server、location、if                                |

**code**：为返回给客户端的HTTP状态代理。可以返回的状态代码为0\~999的任意HTTP状态代理

**text**：为返回给客户端的响应体内容，支持变量的使用

**URL**：为返回给客户端的URL地址

**rewrite指令**：该指令通过正则表达式的使用来改变URI。可以同时存在一个或者多个指令，按照顺序依次对URL进行匹配和处理。

```
 # URL和URI的区别：
 URI:统一资源标识符
 URL:统一资源定位符
```

| 语法  | rewrite regex replacement \[flag]; |
| --- | ---------------------------------- |
| 默认值 | —                                  |
| 位置  | server、location、if                 |

**regex**：用来匹配URI的正则表达式

**replacement**：匹配成功后，用于替换URI中被截取内容的字符串。如果该字符串是以http:或者https:开头的，则不会继续向下对URI进行其他处理，而是直接返回重写后的URI给客户端。

**flag**：用来设置rewrite对URI的处理行为，可选值有如下：

* **last**：停止处理后续rewrite指令集，跳出location作用域，并开始搜索与更改后的URI相匹配的location，**URL地址不变**
* **break**：停止处理后续rewrite指令集，不会跳出location作用域，不再进行重新查找，终止匹配，**URL地址不变**
* **redirect**：返回302临时重定向，浏览器地址栏**会显示跳转后的URL地址**，爬虫不会更新URL
* **permanent**：返回301永久重定向，浏览器地址栏**会显示跳转后的URL地址**，爬虫会更新URL

**rewrite\_log指令**：该指令配置是否开启URL重写日志的输出功能。

| 语法  | rewrite\_log on / off;  |
| --- | ----------------------- |
| 默认值 | rewrite\_log off;       |
| 位置  | http、server、location、if |

开启后，URL重写的相关日志将以**notice级别**输出到**error\_log指令配置**的日志文件汇总。

### 5.7、Rewrite的案例

**域名跳转**

> 问题分析

先来看一个效果，如果我们想访问京东网站，大家都知道我们可以输入[**www.jd.com**](https://www.jd.com),但是同样的我们也可以输入[**www.360buy.com**](https://www.360buy.com)同样也都能访问到京东网站。

这个其实是因为京东刚开始的时候域名就是[www.360buy.com](https://www.360buy.com)，后面由于各种原因把自己的域名换成了[www.jd.com](https://www.jd.com), 虽然说域名变量，但是对于以前只记住了[www.360buy.com](https://www.360buy.com)的用户来说，我们如何把这部分用户也迁移到我们新域名的访问上来，针对于这个问题，我们就可以使用Nginx中Rewrite的域名跳转来解决。

> 环境准备

```
 # 准备两个域名  www.360buy.com | www.jd.com
 vim /etc/hosts
 192.168.200.133 www.360buy.com
 192.168.200.133 www.jd.com
```

> 在/usr/local/nginx/html/hm目录下创建一个访问页面

```html
 <html>
     <title></title>
     <body>
         <h1>欢迎来到我们的网站</h1>
     </body>
 </html>
```

> 通过Nginx实现当访问www.访问到系统的首页

```nginx
 server {
     listen 80;
     server_name www.hm.com;
     location /{
         root /usr/local/nginx/html/hm;
         index index.html;
     }
 }
```

> 通过Rewrite完成将[www.360buy.com](https://www.360buy.com)的请求跳转到[www.jd.com](https://www.jd.com)

```nginx
 server {
     listen 80;
     server_name www.360buy.com;
     rewrite ^/ http://www.jd.com permanent;
 }
```

> 问题描述：如何在域名跳转的过程中携带请求的URI？

```nginx
 # 修改配置信息
 server {
     listen 80;
     server_name www.360buy.com;
     rewrite ^(.*) http://www.jd.com$1 permanent;
 }
```

> 问题描述:我们除了上述说的[www.jd.com](https://www.jd.com) 、[www.360buy.com](https://www.360buy.com) 其实还有我们也可以通过 [www.jingdong.com](https://www.jingdong.com) 来访问，那么如何通过Rewrite来实现多个域名的跳转?

```nginx
 # 添加域名
 vim /etc/hosts
 192.168.200.133 www.jingdong.com
 ​
 # 修改配置信息
 server{
     listen 80;
     server_name www.360buy.com www.jingdong.com;
     rewrite ^(.*) http://www.jd.com$1 permanent;
 }
```

**域名镜像**

上述案例中，将 [www.360buy.com](https://www.360buy.com) 和 [www.jingdong.com](https://www.jingdong.com) 都能跳转到 [www.jd.com](https://www.jd.com)，那么 [www.jd.com](https://www.jd.com) 我们就可以把它起名叫**主域名**，其他两个就是我们所说的**镜像域名**，当然如果我们不想把整个网站做镜像，只想为其中某一个子目录下的资源做镜像，我们可以在location块中配置rewrite功能，比如:

```nginx
 server {
     listen 80;
     server_name rewrite.myweb.com;
     location ^~ /source1{
         rewrite ^/resource1(.*) http://rewrite.myweb.com/web$1 last;
     }
     location ^~ /source2{
         rewrite ^/resource2(.*) http://rewrite.myweb.com/web$1 last;
     }
 }
```

**独立域名**

一个完整的项目包含多个模块，比如购物网站有商品商品搜索模块、商品详情模块已经购物车模块等，那么我们如何为每一个模块设置独立的域名。

```nginx
 # 需求：
 http://search.yifan.com   访问商品搜索模块
 http://item.yifan.com     访问商品详情模块
 http://cart.yifan.com     访问商品购物车模块
 ​
 server{
     listen 80;
     server_name search.yifan.com;
     rewrite ^(.*) http://www.yifan.com/bbs$1 last;
 }
 server{
     listen 80;
     server_name item.yifan.com;
     rewrite ^(.*) http://www.yifan.com/item$1 last;
 }
 server{
     listen 80;
     server_name cart.yifan.com;
     rewrite ^(.*) http://www.yifan.com/cart$1 last;
 }
```

**目录自动添加"/"**

问题描述

```nginx
 # 通过一个例子来演示下问题:
 server {
     listen  80;
     server_name localhost;
     location / {
         root html;
         index index.html;
     }
 }
```

要想访问上述资源，很简单，只需要通过 [http://192.168.139.6](http://192.168.139.6) 直接就能访问，地址后面不需要加**/**,但是如果将上述的配置修改为如下内容:

```nginx
 server {
     listen  80;
     server_name localhost;
     location /a {
         root html;
         index index.html;
     }
 }
```

这个时候，要想访问上述资源，按照上述的访问方式，我们可以通过 [**http://192.168.139.6/a/**](http://192.168.139.6/a/) 来访问,但是如果地址后面不加斜杠，页面就会出问题。如果不加斜杠，Nginx服务器内部会自动做一个**301的重定向**，重定向的地址会有一个指令叫**server\_name\_in\_redirect on|off;**来决定重定向的地址：

```
 # 如果该指令为on
     重定向的地址为:  http://server_name/目录名/;
 # 如果该指令为off
     重定向的地址为:  http://原URL中的域名/目录名/;
```

所以就拿刚才的地址来说， [http://192.168.139.6/a](http://192.168.139.6/a) 如果不加斜杠，那么按照上述规则，如果指令server\_name\_in\_redirect为**on**，则301重定向地址变为 [http://localhost/a/](http://localhost/a/) 如果为**off**，则301重定向地址变为 [http://192.168.139.6/a/](http://192.168.139.6/a/) 后面这个是正常的，前面地址就有问题。

**注意**：server\_name\_in\_redirect指令在Nginx的**0.8.48版本之前默认都是on**，**之后改成了off**，所以现在我们这个版本不需要考虑这个问题，但是如果是0.8.48以前的版本并且server\_name\_in\_redirect设置为on，我们如何通过rewrite来解决这个问题？

**解决方案**：可以使用rewrite功能为末尾没有斜杠的URL自动添加一个斜杠

```nginx
 server {
     listen  80;
     server_name localhost;
     server_name_in_redirect on;
     location /a {
         if (-d $request_filename){
             rewrite ^/(.*)([^/])$ http://$host/$1$2/ permanent;
         }
     }
 }
```

**合并目录**

**搜索引擎优化(SEO)**是一种利用搜索引擎的搜索规则来提供目的网站的有关搜索引擎内排名的方式。我们在创建自己的站点时，可以通过很多中方式来有效的提供搜索引擎优化的程度。其中有一项就包含URL的目录层级一般不要超过三层，否则的话不利于搜索引擎的搜索也给客户端的输入带来了负担，但是将所有的文件放在一个目录下又会导致文件资源管理混乱并且访问文件的速度也会随着文件增多而慢下来，这两个问题是相互矛盾的，那么使用rewrite如何解决上述问题?

举例，网站中有一个资源文件的访问路径时 /server/11/22/33/44/20.html,也就是说20.html存在于第5级目录下，如果想要访问该资源文件，客户端的URL地址就要写成 [**http://www.web.name/server/11/22/33/44/20.html**](http://www.web.name/server/11/22/33/44/20.html)

```nginx
 server {
     listen 80;
     server_name www.web.name;
     location /server{
         root html;
     }
 }
```

但是这个是非常不利于SEO搜索引擎优化的，同时客户端也不好记.使用rewrite我们可以进行如下配置:

```nginx
 server {
     listen 80;
     server_name www.web.name;
     location /server{
         rewrite ^/server-([0-9]+)-([0-9]+)-([0-9]+)-([0-9]+)\.html$ /server/$1/$2/$3/$4/$5.html last;
     }
 }
```

这样的花，客户端只需要输入 [http://www.web.name/server-11-22-33-44-20.html](http://www.web.name/server-11-22-33-44-20.html) 就可以访问到20.html页面了。这里也充分利用了rewrite指令支持正则表达式的特性。

**防盗链**

防盗链之前我们已经介绍过了相关的知识，在rewrite中的防盗链和之前将的原理其实都是一样的，只不过通过rewrite可以将防盗链的功能进行完善下，当出现防盗链的情况，我们可以使用rewrite将请求转发到自定义的一张图片和页面，给用户比较好的提示信息。

```nginx
 # 根据文件类型实现防盗链的一个配置实例:
 server{
     listen 80;
     server_name www.web.com;
     locatin ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip)${
         valid_referers none blocked server_names *.web.com;
         if ($invalid_referer){
             rewrite ^/ http://www.web.com/images/forbidden.png;
         }
     }
 }
 ​
 # 据目录实现防盗链配置：
 server{
     listen 80;
     server_name www.web.com;
     location /file/{
         root /server/file/;
         valid_referers none blocked server_names *.web.com;
         if ($invalid_referer){
             rewrite ^/ http://www.web.com/images/forbidden.png;
         }
     }
 }
```

## 6、正向反向代理

正向代理和反向代理，通过一张图详细的介绍过，简而言之就是正向代理代理的对象是客户端，反向代理代理的是服务端，这是两者之间最大的区别

### 6.1、正向代理

<figure><img src="../.gitbook/assets/image (392).png" alt=""><figcaption></figcaption></figure>

**（1）服务端的设置**

```nginx
 http {
     log_format main 'client send request=>clientIp=$remote_addr serverIp=>$host';
     server{
         listen 80;
         server_name localhost;
         access_log logs/access.log main;
         location {
             root html;
             index index.html index.htm;
         }
     }
 }
```

**（2）使用客户端访问服务端，打开日志查看结果**

<figure><img src="../.gitbook/assets/image (393).png" alt=""><figcaption></figcaption></figure>

**（3）代理服务器设置**

```nginx
 server {
     listen  82;
     resolver 8.8.8.8;
     location /{
         proxy_pass http://$host$request_uri;
     }
 }
```

**（4）查看代理服务器的IP(192.168.200.146)和Nginx配置监听的端口(82)**

**（5）在客户端配置代理服务器**

<figure><img src="../.gitbook/assets/image (394).png" alt=""><figcaption></figcaption></figure>

**（6）设置完成后，再次通过浏览器访问服务端**

<figure><img src="../.gitbook/assets/image (395).png" alt=""><figcaption></figcaption></figure>

通过对比，上下两次的日志记录，会发现虽然我们是客户端访问服务端，但是如何使用了代理，那么服务端能看到的只是代理发送过去的请求，这样的化，就使用Nginx实现了正向代理的设置。

但是Nginx正向代理，在实际的应用中不是特别多，简单了解下，接下来学习Nginx的反向代理，这是Nginx比较重要的一个功能。

### 6.2、反向代理

**Nginx反向代理的配置语法**

Nginx反向代理模块的指令是由`ngx_http_proxy_module`模块进行解析，该模块在安装Nginx的时候已经自己加装到Nginx中了，接下来我们把反向代理中的常用指令一一介绍下：

**proxy\_pass**：该指令用来设置被代理服务器地址，可以是主机名称、IP地址加端口号形式。

| 语法  | proxy\_pass URL; |
| --- | ---------------- |
| 默认值 | —                |
| 位置  | location         |

**URL**：为要设置的被代理服务器地址，包含传输协议(`http`,`https://`)、主机名称或IP地址加端口号、URI等要素。

```nginx
 # 举例：
 proxy_pass http://www.baidu.com;
 location /server{}
 proxy_pass http://192.168.200.146;
     http://192.168.200.146/server/index.html
 proxy_pass http://192.168.200.146/;
     http://192.168.200.146/index.html
```

编写proxy\_pass的时候，后面的值要不要加"/"?

接下来通过例子来说明刚才我们提到的问题：

```nginx
 server {
     listen 80;
     server_name localhost;
     location /{
         # proxy_pass http://192.168.200.146;
         proxy_pass http://192.168.200.146/;
     }
 }
 # 当客户端访问 http://localhost/index.html,效果是一样的
 server{
     listen 80;
     server_name localhost;
     location /server{
         # proxy_pass http://192.168.200.146;
         proxy_pass http://192.168.200.146/;
     }
 }
 # 当客户端访问 http://localhost/server/index.html
 第一个proxy_pass就变成了 http://localhost/server/index.html
 第二个proxy_pass就变成了 http://localhost/index.html
```

**proxy\_set\_header**：该指令可以更改Nginx服务器接收到的客户端请求的请求头信息，然后将新的请求头发送给代理的服务器

| 语法  | proxy\_set\_header field value;                                            |
| --- | -------------------------------------------------------------------------- |
| 默认值 | proxy\_set\_header Host $proxy\_host; proxy\_set\_header Connection close; |
| 位置  | http、server、location                                                       |

需要注意的是，如果想要看到结果，必须在被代理的服务器上来获取添加的头信息。

```nginx
 # 被代理服务器： [192.168.200.146]
 server {
     listen  8080;
     server_name localhost;
     default_type text/plain;
     return 200 $http_username;
 }
 ​
 # 代理服务器: [192.168.200.133]
 server {
     listen  8080;
     server_name localhost;
     location /server {
         proxy_pass http://192.168.200.146:8080/;
         proxy_set_header username TOM;
     }
 }
 ​
 # 访问测试
```

**proxy\_redirect**：该指令是用来重置头信息中的"**Location**"和"**Refresh**"的值。

| 语法  | proxy\_redirect redirect replacement; proxy\_redirect default; proxy\_redirect off; |
| --- | ----------------------------------------------------------------------------------- |
| 默认值 | proxy\_redirect default;                                                            |
| 位置  | http、server、location                                                                |

```nginx
 # 服务端[192.168.200.146]
 server {
     listen  8081;
     server_name localhost;
     if (!-f $request_filename){
         return 302 http://192.168.200.146;
     }
 }
 ​
 # 代理服务端[192.168.200.133]
 server {
     listen  8081;
     server_name localhost;
     location / {
         proxy_pass http://192.168.200.146:8081/;
         proxy_redirect http://192.168.200.146 http://192.168.200.133;
     }
 }
 ​
 # proxy_redirect redirect replacement;
 redirect:目标,Location的值
 replacement:要替换的值
 ​
 # proxy_redirect default;
 default;
 将proxy_pass变量作为redirect进行替换
 将location块的uri变量作为replacement
 ​
 # proxy_redirect off;
 关闭proxy_redirect的功能
 ​
 # 举例
 假设被代理服务器返回Location字段为： http://localhost:8000/two/some/uri/
 proxy_redirect http://localhost:8000/two/ http://frontend/one/;
 将Location字段重写为http://frontend/one/some/uri/
 ​
 在代替的字段中可以不写服务器名：
 proxy_redirect http://localhost:8000/two/ /;
 这样就使用服务器的基本名称和端口，即使它来自非80端口
 ​
 # 指令中可使用变量：
 proxy_redirect http://localhost:8000/  http://$host:$server_port/;
 ​
```

### 6.3、反向代理实战

<figure><img src="../.gitbook/assets/image (396).png" alt=""><figcaption></figcaption></figure>

```nginx
 # 服务器1,2,3存在两种情况
 第一种情况: 三台服务器的内容不一样。
 第二种情况: 三台服务器的内容是一样。
 ​
 # 如果服务器内容不一样，那可以根据用户请求来分发到不同的服务器。
 # 代理服务器
 server {
     listen          8082;
     server_name     localhost;
     location /server1 {
         proxy_pass http://192.168.200.146:9001/;
     }
     location /server2 {
         proxy_pass http://192.168.200.146:9002/;
     }
     location /server3 {
         proxy_pass http://192.168.200.146:9003/;
     }
 }
 ​
 # 服务端
 server1
 server {
     listen          9001;
     server_name     localhost;
     default_type text/html;
     return 200 '<h1>192.168.200.146:9001</h1>'
 }
 server2
 server {
     listen          9002;
     server_name     localhost;
     default_type text/html;
     return 200 '<h1>192.168.200.146:9002</h1>'
 }
 server3
 server {
     listen          9003;
     server_name     localhost;
     default_type text/html;
     return 200 '<h1>192.168.200.146:9003</h1>'
 }
 ​
 # 如果服务器的内容是一样的，该如何处理?
 可以做负载均衡，详细请查看负载均衡小结
```

### 6.4、反向代理系统调优

反向代理指Buffer和Cache

Buffer翻译过来是"缓冲"，Cache翻译过来是"缓存"。

<figure><img src="../.gitbook/assets/image (397).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" lineNumbers="true" %}
```
 # 总结下：
 相同点:
     两种方式都是用来提供IO吞吐效率，都是用来提升Nginx代理的性能。
 不同点:
     缓冲主要用来解决不同设备之间数据传递速度不一致导致的性能低的问题，缓冲中的数据一旦此次操作完成后，就可以删除。
     缓存主要是备份，将被代理服务器的数据缓存一份到代理服务器，这样的话，客户端再次获取相同数据的时候，就只需要从代理服务器上获取，效率较高，缓存中的数据可以重复使用，只有满足特定条件才会删除。
```
{% endcode %}

**Proxy Buffer相关指令**

**proxy\_buffering** ：该指令用来开启或者关闭代理服务器的缓冲区；

| 语法  | proxy\_buffering on / off; |
| --- | -------------------------- |
| 默认值 | proxy\_buffering on;       |
| 位置  | http、server、location       |

**proxy\_buffers**：该指令用来指定单个连接从代理服务器读取响应的缓存区的个数和大小。

| 语法  | proxy\_buffers number size;        |
| --- | ---------------------------------- |
| 默认值 | proxy\_buffers 8 4k / 8K;(与系统平台有关) |
| 位置  | http、server、location               |

* **number**：缓冲区的个数
* **size**：每个缓冲区的大小，缓冲区的总大小就是number\*size

**proxy\_buffer\_size**：该指令用来设置从被代理服务器获取的第一部分响应数据的大小。保持与proxy\_buffers中的size一致即可，当然也可以更小。

| 语法  | proxy\_buffer\_size size;             |
| --- | ------------------------------------- |
| 默认值 | proxy\_buffer\_size 4k / 8k;(与系统平台有关) |
| 位置  | http、server、location                  |

**proxy\_busy\_buffers\_size**：该指令用来限制同时处于BUSY状态的缓冲总大小。

| 语法  | proxy\_busy\_buffers\_size size;     |
| --- | ------------------------------------ |
| 默认值 | proxy\_busy\_buffers\_size 8k / 16K; |
| 位置  | http、server、location                 |

**proxy\_temp\_path**：当缓冲区存满后，仍未被Nginx服务器完全接受，响应数据就会被临时存放在磁盘文件上，该指令设置文件路径，**注意path最多设置三层**。

| 语法  | proxy\_temp\_path path;        |
| --- | ------------------------------ |
| 默认值 | proxy\_temp\_path proxy\_temp; |
| 位置  | http、server、location           |

**proxy\_temp\_file\_write\_size**：该指令用来设置磁盘上缓冲文件的大小。

| 语法  | proxy\_temp\_file\_write\_size size;     |
| --- | ---------------------------------------- |
| 默认值 | proxy\_temp\_file\_write\_size 8K / 16K; |
| 位置  | http、server、location                     |

**通用网站的配置**

```nginx
 # 根据项目的具体内容进行相应的调节。
 proxy_buffering on;
 proxy_buffer_size 4 32k;
 proxy_busy_buffers_size 64k;
 proxy_temp_file_write_size 64k;
```

## 7、安全控制

关于web服务器的安全是比较大的一个话题，里面所涉及的内容很多，Nginx反向代理是如何来提升web服务器的安全呢？

**安全隔离**，什么是安全隔离?

通过代理分开了客户端到应用程序服务器端的连接，实现了安全措施。在反向代理之前设置防火墙，仅留一个入口供代理服务器访问。

<figure><img src="../.gitbook/assets/image (398).png" alt=""><figcaption></figcaption></figure>

**使用SSL对流量进行加密**

翻译成大家能熟悉的说法就是将我们常用的http请求转变成https请求，那么这两个之间的区别简单的来说两个都是HTTP协议，只不过https是身披SSL外壳的http.

HTTPS是一种通过计算机网络进行安全通信的传输协议。它经由HTTP进行通信，利用SSL/TLS建立全通信，加密数据包，确保数据的安全性。

SSL(Secure Sockets Layer)安全套接层

TLS(Transport Layer Security)传输层安全

上述这两个是为网络通信提供安全及数据完整性的一种安全协议，TLS和SSL在传输层和应用层对网络连接进行加密。

```
 # 总结来说为什么要使用https:
 http协议是明文传输数据，存在安全问题，而https是加密传输，相当于http+ssl，并且可以防止流量劫持。
```

Nginx要想使用SSL，需要满足一个条件即需要添加一个模块`--with-http_ssl_module`,而该模块在编译的过程中又需要OpenSSL的支持。

**Nginx添加SSL的支持**

```
 # 完成--with-http_ssl_module模块的增量添加
 将原有/usr/local/nginx/sbin/nginx进行备份
 拷贝nginx之前的配置信息
 在nginx的安装源码进行配置指定对应模块  ./configure --with-http_ssl_module
 通过make模板进行编译
 将objs下面的nginx移动到/usr/local/nginx/sbin下
 在源码目录下执行  make upgrade进行升级，这个可以实现不停机添加新模块的功能
```

**Nginx的SSL相关指令**

因为刚才我们介绍过该模块的指令都是通过ngx\_http\_ssl\_module模块来解析的。

**ssl**：该指令用来在指定的服务器开启HTTPS,可以使用 listen 443 ssl,后面这种方式更通用些。

| 语法  | ssl on / off; |
| --- | ------------- |
| 默认值 | ssl off;      |
| 位置  | http、server   |

```nginx
 server{
     listen 443 ssl;
 }
```

**ssl\_certificate**：为当前这个虚拟主机指定一个带有PEM格式证书的证书。

| 语法  | ssl\_certificate file; |
| --- | ---------------------- |
| 默认值 | —                      |
| 位置  | http、server            |

**ssl\_certificate\_key**：该指令用来指定PEM secret key文件的路径

| 语法  | ssl\_ceritificate\_key file; |
| --- | ---------------------------- |
| 默认值 | —                            |
| 位置  | http、server                  |

**ssl\_session\_cache**：该指令用来配置用于SSL会话的缓存

| 语法  | ssl\_sesion\_cache off / none / \[builtin\[:size]] \[shared:name:size] |
| --- | ---------------------------------------------------------------------- |
| 默认值 | ssl\_session\_cache none;                                              |
| 位置  | http、server                                                            |

* **off**：禁用会话缓存，客户端不得重复使用会话
* **none**：禁止使用会话缓存，客户端可以重复使用，但是并没有在缓存中存储会话参数
* **builtin**：内置OpenSSL缓存，仅在一个工作进程中使用。
* **shared**：所有工作进程之间共享缓存，缓存的相关信息用name和size来指定

**ssl\_session\_timeout**：开启SSL会话功能后，设置客户端能够反复使用储存在缓存中的会话参数时间。

| 语法  | ssl\_session\_timeout time; |
| --- | --------------------------- |
| 默认值 | ssl\_session\_timeout 5m;   |
| 位置  | http、server                 |

**ssl\_ciphers**：指出允许的密码，密码指定为OpenSSL支持的格式

| 语法  | ssl\_ciphers ciphers;          |
| --- | ------------------------------ |
| 默认值 | ssl\_ciphers HIGH:!aNULL:!MD5; |
| 位置  | http、server                    |

可以使用`openssl ciphers`查看openssl支持的格式。

**ssl\_prefer\_server\_ciphers**：该指令指定是否服务器密码优先客户端密码

| 语法  | ssl\_perfer\_server\_ciphers on / off; |
| --- | -------------------------------------- |
| 默认值 | ssl\_perfer\_server\_ciphers off;      |
| 位置  | http、server                            |

**生成证书**

方式一：使用阿里云/腾讯云等第三方服务进行购买。

方式二：使用openssl生成证书

```sh
 # 先要确认当前系统是否有安装openssl
 openssl version
 ​
 # 安装下面的命令进行生成
 mkdir /root/cert
 cd /root/cert
 openssl genrsa -des3 -out server.key 1024
 openssl req -new -key server.key -out server.csr
 cp server.key server.key.org
 openssl rsa -in server.key.org -out server.key
 openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

**开启SSL实例**

```nginx
 server {
     listen       443 ssl;
     server_name  localhost;
 ​
     ssl_certificate      server.cert;
     ssl_certificate_key  server.key;
 ​
     ssl_session_cache    shared:SSL:1m;
     ssl_session_timeout  5m;
 ​
     ssl_ciphers  HIGH:!aNULL:!MD5;
     ssl_prefer_server_ciphers  on;
 ​
     location / {
         root   html;
         index  index.html index.htm;
     }
 }
```

## Nginx进阶

## 1、Nginx负载均衡

### 1.1、负载均衡概述

早期的网站流量和业务功能都比较简单，单台服务器足以满足基本的需求，但是随着互联网的发展，业务流量越来越大并且业务逻辑也跟着越来越复杂，单台服务器的性能及单点故障问题就凸显出来了，因此需要多台服务器进行性能的水平扩展及避免单点故障出现。那么如何将不同用户的请求流量分发到不同的服务器上呢？

<figure><img src="../.gitbook/assets/image (399).png" alt=""><figcaption></figcaption></figure>

### 1.2、负载均衡的原理及处理流程

系统的扩展可以分为纵向扩展和横向扩展。

**纵向扩展**是从单机的角度出发，通过增加系统的硬件处理能力来提升服务器的处理能力

**横向扩展**是通过添加机器来满足大型网站服务的处理能力。

<figure><img src="../.gitbook/assets/image (400).png" alt=""><figcaption></figcaption></figure>

这里面涉及到两个重要的角色分别是"应用集群"和"负载均衡器"。

**应用集群**：将同一应用部署到多台机器上，组成处理集群，接收负载均衡设备分发的请求，进行处理并返回响应的数据。

**负载均衡器**：将用户访问的请求根据对应的负载均衡算法，分发到集群中的一台服务器进行处理。

### 1.3、负载均衡的作用

1. 解决服务器的高并发压力，提高应用程序的处理性能。
2. 提供故障转移，实现高可用。
3. 通过添加或减少服务器数量，增强网站的可扩展性。
4. 在负载均衡器上进行过滤，可以提高系统的安全性。

### 1.4、负载均衡常用的处理方式

**方式一：用户手动选择**

这种方式比较原始，只要实现的方式就是在网站主页上面提供不同线路、不同服务器链接方式，让用户来选择自己访问的具体服务器，来实现负载均衡。

<figure><img src="../.gitbook/assets/image (401).png" alt=""><figcaption></figcaption></figure>

**方式二：DNS轮询方式**

```
 DNS
 域名系统（服务）协议（DNS）是一种分布式网络目录服务，主要用于域名与 IP 地址的相互转换。
```

大多域名注册商都支持对同一个主机名添加多条A记录，这就是DNS轮询，DNS服务器将解析请求按照A记录的顺序，随机分配到不同的IP上，这样就能完成简单的负载均衡。DNS轮询的成本非常低，在一些不重要的服务器，被经常使用。

<figure><img src="../.gitbook/assets/image (402).png" alt=""><figcaption></figcaption></figure>

如下是我们为某一个域名添加的IP地址，用2台服务器来做负载均衡。

<figure><img src="../.gitbook/assets/image (403).png" alt=""><figcaption></figcaption></figure>

```sh
 验证:
 ping www.nginx521.cn
 清空本地的dns缓存
 ipconfig/flushdns
```

我们发现使用DNS来实现轮询，不需要投入过多的成本，虽然DNS轮询成本低廉，但是DNS负载均衡存在明显的缺点。

1. **可靠性低**

假设一个域名DNS轮询多台服务器，如果其中的一台服务器发生故障，那么所有的访问该服务器的请求将不会有所回应，即使你将该服务器的IP从DNS中去掉，但是由于各大宽带接入商将众多的DNS存放在缓存中，以节省访问时间，导致DNS不会实时更新。所以DNS轮流上一定程度上解决了负载均衡问题，但是却存在可靠性不高的缺点。

2. **负载均衡不均衡**

DNS负载均衡采用的是简单的轮询负载算法，不能区分服务器的差异，不能反映服务器的当前运行状态，不能做到为性能好的服务器多分配请求，另外本地计算机也会缓存已经解析的域名到IP地址的映射，这也会导致使用该DNS服务器的用户在一定时间内访问的是同一台Web服务器，从而引发Web服务器减的负载不均衡。

负载不均衡则会导致某几台服务器负荷很低，而另外几台服务器负荷确很高，处理请求的速度慢，配置高的服务器分配到的请求少，而配置低的服务器分配到的请求多。

**方式三:四/七层负载均衡**

介绍四/七层负载均衡之前，我们先了解一个概念，OSI(open system interconnection),叫开放式系统互联模型，这个是由国际标准化组织ISO指定的一个不基于具体机型、操作系统或公司的网络体系结构。该模型将网络通信的工作分为七层。

<figure><img src="../.gitbook/assets/image (404).png" alt=""><figcaption></figcaption></figure>

> 应用层：为应用程序提供网络服务。
>
> 表示层：对数据进行格式化、编码、加密、压缩等操作。
>
> 会话层：建立、维护、管理会话连接。
>
> 传输层：建立、维护、管理端到端的连接，常见的有TCP/UDP。
>
> 网络层：IP寻址和路由选择
>
> 数据链路层：控制网络层与物理层之间的通信。
>
> 物理层：比特流传输。

所谓四层负载均衡指的是OSI七层模型中的传输层，主要是基于IP+PORT的负载均衡

```
 实现四层负载均衡的方式：
 硬件：F5 BIG-IP、Radware等
 软件：LVS、Nginx、Hayproxy等
```

所谓的七层负载均衡指的是在应用层，主要是基于虚拟的URL或主机IP的负载均衡

```
 实现七层负载均衡的方式：
 软件：Nginx、Hayproxy等
```

四层和七层负载均衡的区别

{% code overflow="wrap" lineNumbers="true" %}
```
 四层负载均衡数据包是在底层就进行了分发，而七层负载均衡数据包则在最顶端进行分发，所以四层负载均衡的效率比七层负载均衡的要高。
 四层负载均衡不识别域名，而七层负载均衡识别域名。
```
{% endcode %}

处理四层和七层负载以为其实还有二层、三层负载均衡，二层是在数据链路层基于mac地址来实现负载均衡，三层是在网络层一般采用虚拟IP地址的方式实现负载均衡。

```
 实际环境采用的模式
 四层负载(LVS)+七层负载(Nginx)
```

### **1.5、Nginx七层负载均衡**

Nginx要实现七层负载均衡需要用到proxy\_pass代理模块配置。Nginx默认安装支持这个模块，我们不需要再做任何处理。Nginx的负载均衡是在Nginx的反向代理基础上把用户的请求根据指定的算法分发到一组【upstream虚拟服务池】。

**Nginx七层负载均衡的指令**

**upstream指令**：该指令是用来定义一组服务器，它们可以是监听不同端口的服务器，并且也可以是同时监听TCP和Unix socket的服务器。服务器可以指定不同的权重，默认为1。

| 语法  | upstream name {...} |
| --- | ------------------- |
| 默认值 | —                   |
| 位置  | http                |

**server指令**：该指令用来指定后端服务器的名称和一些参数，可以使用域名、IP、端口或者unix socket

| 语法  | server name \[paramerters] |
| --- | -------------------------- |
| 默认值 | —                          |
| 位置  | upstream                   |

**Nginx七层负载均衡的实现流程**

<figure><img src="../.gitbook/assets/image (405).png" alt=""><figcaption></figcaption></figure>

**服务端设置**

```nginx
 server {
     listen   9001;
     server_name localhost;
     default_type text/html;
     location /{
         return 200 '<h1>192.168.200.146:9001</h1>';
     }
 }
 server {
     listen   9002;
     server_name localhost;
     default_type text/html;
     location /{
         return 200 '<h1>192.168.200.146:9002</h1>';
     }
 }
 server {
     listen   9003;
     server_name localhost;
     default_type text/html;
     location /{
         return 200 '<h1>192.168.200.146:9003</h1>';
     }
 }
```

**负载均衡器设置**

```nginx
 upstream backend{
     server 192.168.200.146:9091;
     server 192.168.200.146:9092;
     server 192.168.200.146:9093;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

**负载均衡状态**

代理服务器在负责均衡调度中的状态有以下几个：

| 状态            | 概述                      |
| ------------- | ----------------------- |
| down          | 当前的server暂时不参与负载均衡      |
| backup        | 预留的备份服务器                |
| max\_fails    | 允许请求失败的次数               |
| fail\_timeout | 经过max\_fails失败后, 服务暂停时间 |
| max\_conns    | 限制最大的接收连接数              |

**down**：将该服务器标记为永久不可用，那么该代理服务器将不参与负载均衡。

```nginx
 upstream backend{
     server 192.168.200.146:9001 down;
     server 192.168.200.146:9002
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

该状态一般会对需要停机维护的服务器进行设置。

**backup**：将该服务器标记为备份服务器，当主服务器不可用时，将用来传递请求。

```nginx
 upstream backend{
     server 192.168.200.146:9001 down;
     server 192.168.200.146:9002 backup;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

此时需要将9094端口的访问禁止掉来模拟下唯一能对外提供访问的服务宕机以后，backup的备份服务器就要开始对外提供服务，此时为了测试验证，我们需要使用防火墙来进行拦截。

介绍一个工具`firewall-cmd`,该工具是Linux提供的专门用来操作firewall的。

```sh
 # 查询防火墙中指定的端口是否开放
 firewall-cmd --query-port=9001/tcp
 ​
 # 如何开放一个指定的端口
 firewall-cmd --permanent --add-port=9002/tcp
 ​
 # 批量添加开发端口
 firewall-cmd --permanent --add-port=9001-9003/tcp
 ​
 # 如何移除一个指定的端口
 firewall-cmd --permanent --remove-port=9003/tcp
 ​
 # 重新加载
 firewall-cmd --reload
 ​
 # 其中
  --permanent表示设置为持久
  --add-port表示添加指定端口
  --remove-port表示移除指定端口
```

**max\_conns**：max\_conns=number，用来设置代理服务器同时活动链接的最大数量，默认为0，表示不限制，使用该配置可以根据后端服务器处理请求的并发量来进行设置，防止后端服务器被压垮。

**max\_fails和fail\_timeout**：max\_fails=number，设置允许请求代理服务器失败的次数，默认为1。fail\_timeout=time，设置经过max\_fails失败后，服务暂停的时间，默认是10秒。

```nginx
 upstream backend{
     server 192.168.200.133:9001 down;
     server 192.168.200.133:9002 backup;
     server 192.168.200.133:9003 max_fails=3 fail_timeout=15;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

**负载均衡策略**

介绍完Nginx负载均衡的相关指令后，我们已经能实现将用户的请求分发到不同的服务器上，那么除了采用默认的分配方式以外，我们还能采用什么样的负载算法?

Nginx的upstream支持如下六种方式的分配算法，分别是:

| 算法名称        | 说明        |
| ----------- | --------- |
| 轮询          | 默认方式      |
| weight      | 权重方式      |
| ip\_hash    | 依据ip分配方式  |
| least\_conn | 依据最少连接方式  |
| url\_hash   | 依据URL分配方式 |
| fair        | 依据响应时间方式  |

**轮询**：是upstream模块负载均衡默认的策略。每个请求会按时间顺序逐个分配到不同的后端服务器。轮询不需要额外的配置。

```nginx
 upstream backend{
     server 192.168.200.146:9001 weight=1;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

**weight加权\[加权轮询]**：weight=number，用来设置服务器的权重，默认为1，权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器硬件配置进行调整的，所有此策略比较适合服务器的硬件配置差别比较大的情况。

```nginx
 upstream backend{
     server 192.168.200.146:9001 weight=10;
     server 192.168.200.146:9002 weight=5;
     server 192.168.200.146:9003 weight=3;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

**ip\_hash**：当对后端的多台动态应用服务器做负载均衡时，ip\_hash指令能够将某个客户端IP的请求通过哈希算法定位到同一台后端服务器上。这样，当来自某一个IP的用户在后端Web服务器A上登录后，在访问该站点的其他URL，能保证其访问的还是后端web服务器A。

| 语法  | ip\_hash; |
| --- | --------- |
| 默认值 | —         |
| 位置  | upstream  |

```nginx
 upstream backend{
     ip_hash;
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

需要额外多说一点的是使用ip\_hash指令无法保证后端服务器的负载均衡，可能导致有些后端服务器接收到的请求多，有些后端服务器接收的请求少，而且设置后端服务器权重等方法将不起作用。

<figure><img src="../.gitbook/assets/image (406).png" alt=""><figcaption></figcaption></figure>

**least\_conn**：最少连接，把请求转发给连接数较少的后端服务器。轮询算法是把请求平均的转发给各个后端，使它们的负载大致相同；但是，有些请求占用的时间很长，会导致其所在的后端负载较高。这种情况下，least\_conn这种方式就可以达到更好的负载均衡效果。

```nginx
 upstream backend{
     least_conn;
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。

<figure><img src="../.gitbook/assets/image (407).png" alt=""><figcaption></figcaption></figure>

**url\_hash**：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，要配合缓存命中来使用。同一个资源多次请求，可能会到达不同的服务器上，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。而使用url\_hash，就可以使得同一个url（也就是同一个资源请求）会到达同一台服务器，一旦缓存住了资源，再此收到请求，就可以从缓存中读取。

```nginx
 upstream backend{
     hash &request_uri;
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

访问如下地址：

```sh
 curl http://192.168.200.133:8083/a
 curl http://192.168.200.133:8083/b
 curl http://192.168.200.133:8083/c
```

<figure><img src="../.gitbook/assets/image (408).png" alt=""><figcaption></figcaption></figure>

**fair**：fair采用的不是内建负载均衡使用的轮换的均衡算法，而是可以根据页面大小、加载时间长短智能的进行负载均衡。那么如何使用第三方模块的fair负载均衡策略。

```nginx
 upstream backend{
     fair;
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

但是如何直接使用会报错，因为fair属于第三方模块实现的负载均衡。需要添加`nginx-upstream-fair`，添加对应的模块

```sh
 # 1.下载nginx-upstream-fair模块
 下载地址为:
     https://github.com/gnosek/nginx-upstream-fair
     
 # 2.将下载的文件上传到服务器并进行解压缩
 unzip nginx-upstream-fair-master.zip
 ​
 # 3.重命名资源
 mv nginx-upstream-fair-master fair
 ​
 # 4.使用./configure命令将资源添加到Nginx模块中
 ./configure --add-module=/root/fair
 ​
 # 5.编译
 make
```

编译可能会出现如下错误，ngx\_http\_upstream\_srv\_conf\_t结构中缺少default\_port

<figure><img src="../.gitbook/assets/image (409).png" alt=""><figcaption></figcaption></figure>

解决方案：

{% code overflow="wrap" lineNumbers="true" %}
```
 # 在Nginx的源码中 src/http/ngx_http_upstream.h,找到`ngx_http_upstream_srv_conf_s`，在模块中添加添加default_port属性
 in_port_t      default_port
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (410).png" alt=""><figcaption></figcaption></figure>

然后再进行make.

6. 更新Nginx

```sh
 # 6.更新Nginx
 mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginxold
 ​
 # 7.将安装目录下的objs中的nginx拷贝到sbin目录
 cd objs
 cp nginx /usr/local/nginx/sbin
 ​
 # 8.更新Nginx
 cd ../
 make upgrade
 ​
 # 编译测试使用Nginx
```

上面介绍了Nginx常用的负载均衡的策略，有人说是5种，是把轮询和加权轮询归为一种，也有人说是6种。那么在咱们以后的开发中到底使用哪种，这个需要根据实际项目的应用场景来决定的。

**负载均衡案例**

```nginx
 # 案例一：对所有请求实现一般轮询规则的负载均衡
 upstream backend{
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

```nginx
 # 案例二：对所有请求实现加权轮询规则的负载均衡
 upstream backend{
     server 192.168.200.146:9001 weight=7;
     server 192.168.200.146:9002 weight=5;
     server 192.168.200.146:9003 weight=3;
 }
 server {
     listen 8083;
     server_name localhost;
     location /{
         proxy_pass http://backend;
     }
 }
```

```nginx
 # 案例三：对特定资源实现负载均衡
 upstream videobackend{
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
 }
 upstream filebackend{
     server 192.168.200.146:9003;
     server 192.168.200.146:9004;
 }
 server {
     listen 8084;
     server_name localhost;
     location /video/ {
         proxy_pass http://videobackend;
     }
     location /file/ {
         proxy_pass http://filebackend;
     }
 }
```

```nginx
 # 案例四：对不同域名实现负载均衡
 upstream itcastbackend{
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
 }
 upstream itheimabackend{
     server 192.168.200.146:9003;
     server 192.168.200.146:9004;
 }
 server {
     listen  8085;
     server_name www.itcast.cn;
     location / {
         proxy_pass http://itcastbackend;
     }
 }
 server {
     listen  8086;
     server_name www.itheima.cn;
     location / {
         proxy_pass http://itheimabackend;
     }
 }
```

```nginx
 # 案例五：实现带有URL重写的负载均衡
 upstream backend{
     hash &request_uri;
     server 192.168.200.146:9001;
     server 192.168.200.146:9002;
     server 192.168.200.146:9003;
 }
 server {
     listen  80;
     server_name localhost;
     location /file/ {
         rewrite ^(/file/.*) /server/$1 last;
     }
     location / {
         proxy_pass http://backend;
     }
 }
```

### 1.6、Nginx四层负载均衡

Nginx在1.9之后，增加了一个stream模块，用来实现四层协议的转发、代理、负载均衡等。stream模块的用法跟http的用法类似，允许我们配置一组TCP或者UDP等协议的监听，然后通过proxy\_pass来转发我们的请求，通过upstream添加多个后端服务，实现负载均衡。

四层协议负载均衡的实现，一般都会用到LVS、HAProxy、F5等，要么很贵要么配置很麻烦，而Nginx的配置相对来说更简单，更能快速完成工作。

**添加stream模块的支持**

Nginx默认是没有编译这个模块的，需要使用到stream模块，那么需要在编译的时候加上`--with-stream`。

完成添加`--with-stream`的实现步骤:

```
 将原有/usr/local/nginx/sbin/nginx进行备份
 拷贝nginx之前的配置信息
 在nginx的安装源码进行配置指定对应模块  ./configure --with-stream
 通过make模板进行编译
 将objs下面的nginx移动到/usr/local/nginx/sbin下
 在源码目录下执行  make upgrade进行升级，这个可以实现不停机添加新模块的功能
```

**Nginx四层负载均衡的指令**

**stream指令**：该指令提供在其中指定流服务器指令的配置文件上下文。和http指令同级。

| 语法  | stream { ... } |
| --- | -------------- |
| 默认值 | —              |
| 位置  | main           |

**upstream指令**：该指令和http的upstream指令是类似的。

**四层负载均衡的案例**

**需求分析**

<figure><img src="../.gitbook/assets/image (411).png" alt=""><figcaption></figcaption></figure>

**实现步骤**

```nginx
 # (1)准备Redis服务器,在一条服务器上准备三个Redis，端口分别是6379,6378
 安装配置Redis,并启动两个Redis,端口为6379，6378
 ps -ef | grep redis
 ​
 # (2)准备Tomcat服务器
 安装并启动
 ​
 # (3)nginx.conf配置
 stream {
         upstream redisbackend {
                 server 192.168.200.146:6379;
                 server 192.168.200.146:6378;
         }
         upstream tomcatbackend {
                 server 192.168.200.146:8080;
         }
         server {
                 listen  81;
                 proxy_pass redisbackend;
         }
         server {
                 listen  82;
                 proxy_pass tomcatbackend;
         }
 }
 ​
 # 访问测试
```

**Nginx代理内部局域网Mysql服务**

```nginx
 # 1.安装stream模块
 yum install -y nginx-mod-stream
 # 可以看到，下载了/usr/lib64/nginx/modules/ngx_stream_module.so文件;
 ​
 # 2.编辑nginx配置，使用stream
 # 2.1.编辑nginx配置文件
 vim /sxapp/sxappopt/conf/nginx/nginx.conf
 ​
 # 2.2.首行导入stream模块
 load_module /usr/lib64/nginx/modules/ngx_stream_module.so;
  
 # 2.3.使用nginx代理mysql TCP
 stream{
     upstream mysql{
         # 提供的mysql服务器3306端口
         server 内部局域网IP:3306;
     }
     server{
         #监听的端口
         listen 8888;
         #连接超时时间
         proxy_connect_timeout 10s; 
         #设置客户端和代理服务之间的超时时间，如果5分钟内没操作将自动断开。
         proxy_timeout 300s;
         # 代理mysql
         proxy_pass mysql;
     }
 }
 ​
 # 一个完整的参考
 load_module /usr/lib64/nginx/modules/ngx_stream_module.so;
 user nobody;
 worker_processes auto;
 error_log /sxapp/sxappopt/logs/nginx/error.log;
 pid /sxapp/sxappopt/logs/nginx/nginx.pid;
 # Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
 include /sxapp/sxappopt/conf/nginx/modules/*.conf;
 events {
     worker_connections 1024;
 }
 # TCP
 stream{
     upstream mysql{
         server 172.31.32.233:3306;
     }
     server{
         #监听的端口
         listen 8888;
         #连接超时时间
         proxy_connect_timeout 10s; 
         #设置客户端和代理服务之间的超时时间，如果5分钟内没操作将自动断开。
         proxy_timeout 300s;
         # 代理mysql
         proxy_pass mysql;
     }
 }
 http {
     # ...
 }
 ​
 # 3.启动/重载nginx
 nginx -s reload
 ​
 # 4.navicat测试连接
```

## 2、Nginx缓存集成

### 2.1、缓存的概念

缓存就是数据交换的缓冲区(称作:Cache),当用户要获取数据的时候，会先从缓存中去查询获取数据，如果缓存中有就会直接返回给用户，如果缓存中没有，则会发请求从服务器重新查询数据，将数据返回给用户的同时将数据放入缓存，下次用户就会直接从缓存中获取数据。

<figure><img src="../.gitbook/assets/image (412).png" alt=""><figcaption></figcaption></figure>

缓存其实在很多场景中都有用到，比如：

| 场景       | 作用           |
| -------- | ------------ |
| 操作系统磁盘缓存 | 减少磁盘机械操作     |
| 数据库缓存    | 减少文件系统的IO操作  |
| 应用程序缓存   | 减少对数据库的查询    |
| Web服务器缓存 | 减少对应用服务器请求次数 |
| 浏览器缓存    | 减少与后台的交互次数   |

```
 # 缓存的优点
 1.减少数据传输，节省网络流量，加快响应速度，提升用户体验；
 2.减轻服务器压力；
 3.提供服务端的高可用性；
 ​
 # 缓存的缺点
 1.数据的不一致
 2.增加成本
```

<figure><img src="../.gitbook/assets/image (413).png" alt=""><figcaption></figcaption></figure>

本次课程注解讲解的是Nginx,Nginx作为web服务器，Nginx作为Web缓存服务器，它介于客户端和应用服务器之间，当用户通过浏览器访问一个URL时，web缓存服务器会去应用服务器获取要展示给用户的内容，将内容缓存到自己的服务器上，当下一次请求到来时，如果访问的是同一个URL，web缓存服务器就会直接将之前缓存的内容返回给客户端，而不是向应用服务器再次发送请求。web缓存降低了应用服务器、数据库的负载，减少了网络延迟，提高了用户访问的响应速度，增强了用户的体验。

### 2.2、Nginx的web缓存服务

Nginx是从0.7.48版开始提供缓存功能。Nginx是基于Proxy Store来实现的，其原理是把URL及相关组合当做Key,在使用MD5算法对Key进行哈希，得到硬盘上对应的哈希目录路径，从而将缓存内容保存在该目录中。它可以支持任意URL连接，同时也支持404/301/302这样的非200状态码。Nginx即可以支持对指定URL或者状态码设置过期时间，也可以使用purge命令来手动清除指定URL的缓存。

<figure><img src="../.gitbook/assets/image (414).png" alt=""><figcaption></figcaption></figure>

### 2.3、Nginx缓存设置的相关指令

Nginx的web缓存服务主要是使用`ngx_http_proxy_module`模块相关指令集来完成，接下来我们把常用的指令来进行介绍下。

**proxy\_cache\_path**：该指定用于设置缓存文件的存放路径

| 语法  | proxy\_cache\_path path \[levels=number] keys\_zone=zone\_name:zone\_size \[inactive=time]\[max\_size=size]; |
| --- | ------------------------------------------------------------------------------------------------------------ |
| 默认值 | —                                                                                                            |
| 位置  | http                                                                                                         |

**path**：缓存路径地址,如：`/usr/local/proxy_cache`

**levels**：指定该缓存空间对应的目录，最多可以设置3层，每层取值为1|2如 :

```
 levels=1:2   缓存空间有两层目录，第一次是1个字母，第二次是2个字母
 举例说明:
 itheima[key]通过MD5加密以后的值为 43c8233266edce38c2c9af0694e2107d
 levels=1:2   最终的存储路径为/usr/local/proxy_cache/d/07
 levels=2:1:2 最终的存储路径为/usr/local/proxy_cache/7d/0/21
 levels=2:2:2 最终的存储路径为/usr/local/proxy_cache/7d/10/e2
```

**keys\_zone**：用来为这个缓存区设置名称和指定大小，如：

```
 keys_zone=itcast:200m  
 缓存区的名称是itcast,大小为200M,1M大概能存储8000个keys
```

**inactive**：指定缓存的数据多次时间未被访问就将被删除，如：

```
 inactive=1d   
 缓存数据在1天内没有被访问就会被删除
```

**max\_size**：设置最大缓存空间，如果缓存空间存满，默认会覆盖缓存时间最长的资源，如:

```nginx
 max_size=20g
 最大缓存空间20g
```

{% code overflow="wrap" %}
```nginx
 # 配置实例:
 http{
     proxy_cache_path /usr/local/proxy_cache keys_zone=itcast:200m  levels=1:2:1 inactive=1d max_size=20g;
 }
```
{% endcode %}

**proxy\_cache**：该指令用来开启或关闭代理缓存，如果是开启则自定使用哪个缓存区来进行缓存。

| 语法  | proxy\_cache zone\_name / off; |
| --- | ------------------------------ |
| 默认值 | proxy\_cache off;              |
| 位置  | http、server、location           |

**zone\_name**：指定使用缓存区的名称

**proxy\_cache\_key**：该指令用来设置web缓存的key值，Nginx会根据key值MD5哈希存缓存。

| 语法  | proxy\_cache\_key key;                              |
| --- | --------------------------------------------------- |
| 默认值 | proxy\_cache\_key $scheme$proxy\_host$request\_uri; |
| 位置  | http、server、location                                |

**proxy\_cache\_valid**：该指令用来对不同返回状态码的URL设置不同的缓存时间

| 语法  | proxy\_cache\_valid \[code ...] time; |
| --- | ------------------------------------- |
| 默认值 | —                                     |
| 位置  | http、server、location                  |

```nginx
 proxy_cache_valid 200 302 10m;
 proxy_cache_valid 404 1m;
 为200和302的响应URL设置10分钟缓存，为404的响应URL设置1分钟缓存
 proxy_cache_valid any 1m;
 对所有响应状态码的URL都设置1分钟缓存
```

**proxy\_cache\_min\_uses**：该指令用来设置资源被访问多少次后被缓存

| 语法  | proxy\_cache\_min\_uses number; |
| --- | ------------------------------- |
| 默认值 | proxy\_cache\_min\_uses 1;      |
| 位置  | http、server、location            |

**proxy\_cache\_methods**：该指令用户设置缓存哪些HTTP方法

| 语法  | proxy\_cache\_methods GET / HEAD / POST; |
| --- | ---------------------------------------- |
| 默认值 | proxy\_cache\_methods GET HEAD;          |
| 位置  | http、server、location                     |

默认缓存HTTP的GET和HEAD方法，不缓存POST方法。

### 2.4、Nginx缓存设置案例

**需求分析**

<figure><img src="../.gitbook/assets/image (415).png" alt=""><figcaption></figcaption></figure>

**步骤实现**

**应用服务器的环境准备**

（1）在192.168.200.146服务器上的tomcat的webapps下面添加一个js目录，并在js目录中添加一个jquery.js文件

（2）启动tomcat

（3）访问测试 `http://192.168.200.146:8080/js/jquery.js`

**Nginx的环境准备**

（1）完成Nginx反向代理配置

```nginx
 http{
     upstream backend{
         server 192.168.200.146:8080;
     }
     server {
         listen       8080;
         server_name  localhost;
         location / {
             proxy_pass http://backend/js/;
         }
     }
 }
```

（2）完成Nginx缓存配置

{% code overflow="wrap" lineNumbers="true" %}
```nginx
 http{
     proxy_cache_path /usr/local/proxy_cache levels=2:1 keys_zone=itcast:200m inactive=1d max_size=20g;
     upstream backend{
         server 192.168.200.146:8080;
     }
     server {
         listen       8080;
         server_name  localhost;
         location / {
             proxy_cache cast;
             proxy_cache_key yifan;
             proxy_cache_min_uses 5;
             proxy_cache_valid 200 5d;
             proxy_cache_valid 404 30s;
             proxy_cache_valid any 1m;
             add_header nginx-cache "$upstream_cache_status";
             proxy_pass http://backend/js/;
         }
     }
 }
```
{% endcode %}

### 4.4、Nginx缓存的清除

**方式一**

```sh
 # 删除对应的缓存目录
 rm -rf /usr/local/proxy_cache/......
```

**方式二**

```sh
 # 使用第三方扩展模块 ngx_cache_purge
 （1）下载ngx_cache_purge模块对应的资源包，并上传到服务器上。
 ngx_cache_purge-2.3.tar.gz
 ​
 （2）对资源文件进行解压缩
 tar -zxf ngx_cache_purge-2.3.tar.gz
 ​
 （3）修改文件夹名称，方便后期配置
 mv ngx_cache_purge-2.3 purge
 ​
 （4）查询Nginx的配置参数
 nginx -V
 ​
 （5）进入Nginx的安装目录，使用./configure进行参数配置
 ./configure --add-module=/root/nginx/module/purge
 ​
 （6）使用make进行编译
 make
 ​
 （7）将nginx安装目录的nginx二级制可执行文件备份
 mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginxold
 ​
 （8）将编译后的objs中的nginx拷贝到nginx的sbin目录下
 cp objs/nginx /usr/local/nginx/sbin
 ​
 （9）使用make进行升级
 make upgrade
 ​
 （10）在nginx配置文件中进行如下配置
 server{
     location ~/purge(/.*) {
         proxy_cache_purge cast yifan;
     }
 }
```

### 4.5、Nginx设置资源不缓存

前面咱们已经完成了Nginx作为web缓存服务器的使用。但是我们得思考一个问题就是不是所有的数据都适合进行缓存。比如说对于一些经常发生变化的数据。如果进行缓存的话，就很容易出现用户访问到的数据不是服务器真实的数据。所以对于这些资源我们在缓存的过程中就需要进行过滤，不进行缓存。

Nginx也提供了这块的功能设置，需要使用到如下两个指令

**proxy\_no\_cache**：该指令是用来定义不将数据进行缓存的条件。

| 语法  | proxy\_no\_cache string ...; |
| --- | ---------------------------- |
| 默认值 | —                            |
| 位置  | http、server、location         |

```nginx
 # 配置实例
 proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
```

**proxy\_cache\_bypass**：该指令是用来设置不从缓存中获取数据的条件。

| 语法  | proxy\_cache\_bypass string ...; |
| --- | -------------------------------- |
| 默认值 | —                                |
| 位置  | http、server、location             |

```nginx
 # 配置实例
 proxy_cache_bypass $cookie_nocache $arg_nocache $arg_comment;
```

上述两个指令都有一个指定的条件，这个条件可以是多个，并且多个条件中至少有一个不为空且不等于"0",则条件满足成立。

上面给的配置实例是从官方网站获取的，里面使用到了三个变量，分别是$cookie\_nocache、$arg\_nocache、$arg\_comment。

**$cookie\_nocache、$arg\_nocache、$arg\_comment**

这三个参数分别代表的含义是:

```
 $cookie_nocache
 指的是当前请求的cookie中键的名称为nocache对应的值
 $arg_nocache和$arg_comment
 指的是当前请求的参数中属性名为nocache和comment对应的属性值
```

案例演示下:

```nginx
 log_format params $cookie_nocache | $arg_nocache | $arg_comment；
 server{
     listen  8081;
     server_name localhost;
     location /{
         access_log logs/access_params.log params;
         add_header Set-Cookie 'nocache=999';
         root html;
         index index.html;
     }
 }
```

**案例实现**

设置不缓存资源的配置方案

```nginx
 server{
     listen  8080;
     server_name localhost;
     location / {
         if ($request_uri ~ /.*\.js$){
            set $nocache 1;
         }
         proxy_no_cache $nocache $cookie_nocache $arg_nocache $arg_comment;
         proxy_cache_bypass $nocache $cookie_nocache $arg_nocache $arg_comment;
     }
 }
```

## 3、Nginx集群搭建

### 3.1、Nignx与Tomcat部署

前面课程已经将Nginx的大部分内容进行了讲解，我们都知道了Nginx在高并发场景和处理静态资源是非常高性能的，但是在实际项目中除了静态资源还有就是后台业务代码模块，一般后台业务都会被部署在Tomcat等web服务器上。那么如何使用Nginx接收用户的请求并把请求转发到后台web服务器？

<figure><img src="../.gitbook/assets/image (416).png" alt=""><figcaption></figcaption></figure>

步骤分析:

```
1.准备Tomcat环境，并在Tomcat上部署一个web项目
2.准备Nginx环境，使用Nginx接收请求，并把请求分发到Tomat上
```

**环境准备(Tomcat)**

```
 浏览器访问:
 http://192.168.200.146:8080/demo/index.html获
 ​
 取动态资源的链接地址:
 http://192.168.200.146:8080/demo/getAddress
```

采用Tomcat作为后台web服务器

```
 #（1）在Centos上准备一个Tomcat
 1.Tomcat官网地址:https://tomcat.apache.org/
 2.下载tomcat,本次课程使用的是apache-tomcat-8.5.59.tar.gz
 3.将tomcat进行解压缩
 mkdir web_tomcat
 tar -zxf apache-tomcat-8.5.59.tar.gz -C /web_tomcat
 ​
 #（2）准备一个web项目，将其打包为war
 1.将资料中的demo.war上传到tomcat8目录下的webapps包下
 2.将tomcat进行启动，进入tomcat8的bin目录下
 ./startup.sh
 ​
 #（3）启动tomcat进行访问测试。
 静态资源: http://192.168.200.146:8080/demo/index.html
 动态资源: http://192.168.200.146:8080/demo/getAddress
```

**环境准备(Nginx)**

使用Nginx的反向代理，将请求转给Tomcat进行处理。

```nginx
 upstream webservice {
     server 192.168.200.146:8080;
 }
 server{
     listen      80;
     server_name localhost;
     location /demo {
         proxy_pass http://webservice;
     }
 }
```

学习到这，可能大家会有一个困惑，明明直接通过tomcat就能访问，为什么还需要多加一个nginx，这样不是反而是系统的复杂度变高了么? 那接下来我们从两个方便给大家分析下这个问题，

第一个使用Nginx实现动静分离

第二个使用Nginx搭建Tomcat的集群

### 3.2、Nginx实现动静分离

**什么是动静分离**?

**动**：后台应用程序的业务处理

**静**：网站的静态资源(html,javaScript,css,images等文件)

**分离**：将两者进行分开部署访问，提供用户进行访问。举例说明就是以后所有和静态资源相关的内容都交给Nginx来部署访问，非静态内容则交个类似于Tomcat的服务器来部署访问。

**为什么要动静分离**?

前面我们介绍过Nginx在处理静态资源的时候，效率是非常高的，而且Nginx的并发访问量也是名列前茅，而Tomcat则相对比较弱一些，所以把静态资源交个Nginx后，可以减轻Tomcat服务器的访问压力并提高静态资源的访问速度。

动静分离以后，降低了动态资源和静态资源的耦合度。如动态资源宕机了也不影响静态资源的展示。

**如何实现动静分离**?

实现动静分离的方式很多，比如静态资源可以部署到**CDN**、Nginx等服务器上，动态资源可以部署到Tomcat上。本次使用Nginx+Tomcat来实现动静分离。

**动静分离实现步骤**

1. 将demo.war项目中的静态资源都删除掉，重新打包生成一个war包

2.将war包部署到tomcat中，把之前部署的内容删除掉

```
 进入到tomcat的webapps目录下，将之前的内容删除掉
 将新的war包复制到webapps下
 将tomcat启动
```

3.在Nginx所在服务器创建如下目录，并将对应的静态资源放入指定的位置

<figure><img src="../.gitbook/assets/image (417).png" alt=""><figcaption></figcaption></figure>

其中index.html页面的内容如下

```html
 <!DOCTYPE html>
 <html lang="en">
 <head>
     <meta charset="UTF-8">
     <title>Title</title>
     <script src="js/jquery.min.js"></script>
     <script>
         $(function(){
            $.get('http://192.168.200.133/demo/getAddress',function(data){
                $("#msg").html(data);
            });
         });
     </script>
 </head>
 <body>
     <img src="images/logo.png"/>
     <h1>Nginx如何将请求转发到后端服务器</h1>
     <h3 id="msg"></h3>
     <img src="images/mv.png"/>
 </body>
 </html>
```

4.配置Nginx的静态资源与动态资源的访问

```nginx
 upstream webservice{
    server 192.168.200.146:8080;
 }
 server {
         listen       80;
         server_name  localhost;
         #动态资源
         location /demo {
                 proxy_pass http://webservice;
         }
         #静态资源
         location ~/.*\.(png|jpg|gif|js){
                 root html/web;
                 gzip on;
         }
         location / {
             root   html/web;
             index  index.html index.htm;
         }
 }
```

5.启动测试 [http://192.168.139.6/index.html](http://192.168.139.6/index.html)

<figure><img src="../.gitbook/assets/image (418).png" alt=""><figcaption></figcaption></figure>

假如某个时间点，由于某个原因导致Tomcat后的服务器宕机了，我们再次访问Nginx,会得到如下效果，用户还是能看到页面，只是缺失了访问次数的统计，这就是前后端耦合度降低的效果，并且整个请求只和后的服务器交互了一次，js和images都直接从Nginx返回，提供了效率，降低了后的服务器的压力。

<figure><img src="../.gitbook/assets/image (419).png" alt=""><figcaption></figcaption></figure>

### 3.3、Nginx实现Tomcat集群搭建

在使用Nginx和Tomcat部署项目的时候，我们使用的是一台Nginx服务器和一台Tomcat服务器，如下:

<figure><img src="../.gitbook/assets/image (420).png" alt=""><figcaption></figcaption></figure>

那么问题来了，如果Tomcat的真的宕机了，整个系统就会不完整，所以如何解决上述问题，一台服务器容易宕机，那就多搭建几台Tomcat服务器，这样的话就提升了后的服务器的可用性。这也就是我们常说的集群，搭建Tomcat的集群需要用到了Nginx的反向代理和赋值均衡的知识，具体如何来实现?我们先来分析下原理

<figure><img src="../.gitbook/assets/image (421).png" alt=""><figcaption></figcaption></figure>

环境准备：

(1)准备3台tomcat,使用端口进行区分\[实际环境应该是三台服务器]，修改server.ml，将端口修改分别修改为8080,8180,8280

(2)启动tomcat并访问测试，

```
 http://192.168.200.146:8080/demo/getAddress
 http://192.168.200.146:8180/demo/getAddress
 http://192.168.200.146:8280/demo/getAddress
 ​
```

(3)在Nginx对应的配置文件中添加如下内容:

```nginx
 upstream webservice{
     server 192.168.200.146:8080;
     server 192.168.200.146:8180;
     server 192.168.200.146:8280;
 }
```

好了，完成了上述环境的部署，我们已经解决了Tomcat的高可用性，一台服务器宕机，还有其他两条对外提供服务，同时也可以实现后台服务器的不间断更新。但是新问题出现了，上述环境中，如果是Nginx宕机了呢，那么整套系统都将服务对外提供服务了，这个如何解决？

### 3.4、Nginx高可用解决方案

针对于上面提到的问题，我们来分析下要想解决上述问题，需要面临哪些问题?

<figure><img src="../.gitbook/assets/image (422).png" alt=""><figcaption></figcaption></figure>

需要两台以上的Nginx服务器对外提供服务，这样的话就可以解决其中一台宕机了，另外一台还能对外提供服务，但是如果是两台Nginx服务器的话，会有两个IP地址，用户该访问哪台服务器，用户怎么知道哪台是好的，哪台是宕机了的?

**Keepalived**

使用Keepalived来解决，Keepalived 软件由 C 编写的，最初是专为 LVS 负载均衡软件设计的，Keepalived 软件主要是通过 VRRP 协议实现高可用功能。

**VRRP介绍**

<figure><img src="../.gitbook/assets/image (423).png" alt=""><figcaption></figcaption></figure>

VRRP（Virtual Route Redundancy Protocol）协议，翻译过来为虚拟路由冗余协议。VRRP协议将两台或多台路由器设备虚拟成一个设备，对外提供虚拟路由器IP,而在路由器组内部，如果实际拥有这个对外IP的路由器如果工作正常的话就是MASTER,MASTER实现针对虚拟路由器IP的各种网络功能。其他设备不拥有该虚拟IP，状态为BACKUP,处了接收MASTER的VRRP状态通告信息以外，不执行对外的网络功能。当主机失效时，BACKUP将接管原先MASTER的网络功能。

从上面的介绍信息获取到的内容就是VRRP是一种协议，那这个协议是用来干什么的？

1.选择协议

{% code overflow="wrap" lineNumbers="true" %}
```
 VRRP可以把一个虚拟路由器的责任动态分配到局域网上的 VRRP 路由器中的一台。其中的虚拟路由即Virtual路由是由VRRP路由群组创建的一个不真实存在的路由，这个虚拟路由也是有对应的IP地址。
 而且VRRP路由1和VRRP路由2之间会有竞争选择，通过选择会产生一个Master路由和一个Backup路由。
```
{% endcode %}

2.路由容错协议

{% code overflow="wrap" lineNumbers="true" %}
```
 Master路由和Backup路由之间会有一个心跳检测，Master会定时告知Backup自己的状态，如果在指定的时间内，Backup没有接收到这个通知内容，Backup就会替代Master成为新的Master。
 Master路由有一个特权就是虚拟路由和后端服务器都是通过Master进行数据传递交互的，而备份节点则会直接丢弃这些请求和数据，不做处理，只是去监听Master的状态
```
{% endcode %}

用了Keepalived后，解决方案如下:

<figure><img src="../.gitbook/assets/image (424).png" alt=""><figcaption></figcaption></figure>

**环境搭建**

| VIP             | IP            | 主机名         | 主/从    |
| --------------- | ------------- | ----------- | ------ |
|                 | 192.168.139.5 | keepalived1 | Master |
| 192.168.139.222 |               |             |        |
|                 | 192.168.139.6 | keepalived2 | Backup |

## 4、Nginx制作下载站点

首先我们先要清楚什么是下载站点?

我们先来看一个网站`http://nginx.org/download/`这个我们刚开始学习Nginx的时候给大家看过这样的网站，该网站主要就是用来提供用户来下载相关资源的网站，就叫做下载网站。

<figure><img src="../.gitbook/assets/image (425).png" alt=""><figcaption></figcaption></figure>

如何制作一个下载站点:

nginx使用的是模块**ngx\_http\_autoindex\_module**来实现的，该模块处理以斜杠("/")结尾的请求，并生成目录列表。

nginx编译的时候会自动加载该模块，但是该模块默认是关闭的，需要使用下来指令来完成对应的配置

（1）**autoindex**：启用或禁用目录列表输出

| 语法  | autoindex on / off;  |
| --- | -------------------- |
| 默认值 | autoindex off;       |
| 位置  | http、server、location |

（2）**autoindex\_exact\_size**：对应HTLM格式，指定是否在目录列表展示文件的详细大小

默认为on，显示出文件的确切大小，单位是bytes。 改为off后，显示出文件的大概大小，单位是kB或者MB或者GB

| 语法  | autoindex\_exact\_size on / off; |
| --- | -------------------------------- |
| 默认值 | autoindex\_exact\_size on;       |
| 位置  | http、server、location             |

（3）**autoindex\_format**：设置目录列表的格式

| 语法  | autoindex\_format html / xml / json / jsonp; |
| --- | -------------------------------------------- |
| 默认值 | autoindex\_format html;                      |
| 位置  | http、server、location                         |

注意:该指令在1.7.9及以后版本中出现

（4）**autoindex\_localtime**：对应HTML格式，是否在目录列表上显示时间。

默认为off，显示的文件时间为GMT时间。 改为on后，显示的文件时间为文件的服务器时间

| 语法  | autoindex\_localtime on / off; |
| --- | ------------------------------ |
| 默认值 | autoindex\_localtime off;      |
| 位置  | http、server、location           |

配置方式如下:

```nginx
 location /download{
     root /usr/local;
     autoindex on;
     autoindex_exact_size on;
     autoindex_format html;
     autoindex_localtime on;
 }
```

## 5、Nginx的用户认证模块

对应系统资源的访问，往往需要限制谁能访问，谁不能访问。这块就是我们通常所说的认证部分，认证需要做的就是根据用户输入的用户名和密码来判定用户是否为合法用户，如果是则放行访问，如果不是则拒绝访问。

Nginx对应用户认证这块是通过**ngx\_http\_auth\_basic\_module**模块来实现的，它允许通过使用"HTTP基本身份验证"协议验证用户名和密码来限制对资源的访问。默认情况下nginx是已经安装了该模块，如果不需要则使用**--without-http\_auth\_basic\_module**。

该模块的指令比较简单，

（1）**auth\_basic**：使用“ HTTP基本认证”协议启用用户名和密码的验证

| 语法  | auth\_basic string / off;          |
| --- | ---------------------------------- |
| 默认值 | auth\_basic off;                   |
| 位置  | http,server,location,limit\_except |

开启后，服务端会返回401，指定的字符串会返回到客户端，给用户以提示信息，但是不同的浏览器对内容的展示不一致。

（2）**auth\_basic\_user\_file**：指定用户名和密码所在文件

| 语法  | auth\_basic\_user\_file file;      |
| --- | ---------------------------------- |
| 默认值 | —                                  |
| 位置  | http,server,location,limit\_except |

指定文件路径，该文件中的用户名和密码的设置，密码需要进行加密。可以采用工具自动生成

**实现步骤**

1.nginx.conf添加如下内容

```nginx
 location /download{
     root /usr/local;
     autoindex on;
     autoindex_exact_size on;
     autoindex_format html;
     autoindex_localtime on;
     auth_basic 'please input your auth';
     auth_basic_user_file htpasswd;
 }
```

2.我们需要使用`htpasswd`工具生成

```sh
 yum install -y httpd-tools
 # 创建一个新文件记录用户名和密码
 htpasswd -c /usr/local/nginx/conf/htpasswd username 
 ​
 # 在指定文件新增一个用户名和密码
 htpasswd -b /usr/local/nginx/conf/htpasswd username password
 ​
 # 从指定文件删除一个用户信息
 htpasswd -D /usr/local/nginx/conf/htpasswd username 
 ​
 # 验证用户名和密码是否正确
 htpasswd -v /usr/local/nginx/conf/htpasswd username 
```

<figure><img src="../.gitbook/assets/image (426).png" alt=""><figcaption></figcaption></figure>

上述方式虽然能实现用户名和密码的验证，但是大家也看到了，所有的用户名和密码信息都记录在文件里面，如果用户量过大的话，这种方式就显得有点麻烦了，这时候就得通过后台业务代码来进行用户权限的校验了。
