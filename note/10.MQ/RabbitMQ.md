# 概念

## 消息队列协议

### 01、什么是协议

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002321.png)

我们知道消息中间件负责数据的传递，存储，和分发消费三个部分，数据的存储和分发的过程中肯定要遵循某种约定成俗的规范，你是采用底层的TCP/IP，UDP协议还是其他的自己取构建等，而这些约定成俗的规范就称之为：协议。

> 所谓协议是指：
> 1：计算机底层操作系统和应用程序通讯时共同遵守的一组约定，只有遵循共同的约定和规范，系统和底层操作系统之间才能相互交流。
> 2：和一般的网络应用程序的不同它主要负责数据的接受和传递，所以性能比较的高。
> 3：协议对数据格式和计算机之间交换数据都必须严格遵守规范。

### 02、网络协议的三要素

1.语法。语法是用户数据与控制信息的结构与格式,以及数据出现的顺序。
2.语义。语义是解释控制信息每个部分的意义。它规定了需要发出何种控制信息,以及完成的动作与做出什么样的响应。
3.时序。时序是对事件发生顺序的详细说明。

比如我MQ发送一个信息，是以什么数据格式发送到队列中，然后每个部分的含义是什么，发送完毕以后的执行的动作，以及消费者消费消息的动作，消费完毕的响应结果和反馈是什么，然后按照对应的执行顺序进行处理。如果你还是不理解：大家每天都在接触的http请求协议:

> 1：语法：http规定了请求报文和响应报文的格式。
> 2：语义：客户端主动发起请求称之为请求。（这是一种定义，同时你发起的是post/get请求）
> 3：时序：一个请求对应一个响应。（一定先有请求在有响应，这个是时序）

而消息中间件采用的并不是http协议，而常见的消息中间件协议有：OpenWire、AMQP、MQTT、Kafka，OpenMessage协议。

> **面试题：为什么消息中间件不直接使用http协议呢？**
> 1:  因为http请求报文头和响应报文头是比较复杂的，包含了cookie，数据的加密解密，状态码，响应码等附加的功能，但是对于一个消息而言，我们并不需要这么复杂，也没有这个必要性，它其实就是负责数据传递，存储，分发就行，一定要追求的是高性能。尽量简洁，快速。
> 2:大部分情况下http大部分都是短链接，在实际的交互过程中，一个请求到响应很有可能会中断，中断以后就不会就行持久化，就会造成请求的丢失。这样就不利于消息中间件的业务场景，因为消息中间件可能是一个长期的获取消息的过程，出现问题和故障要对数据或消息就行持久化等，目的是为了保证消息和数据的高可靠和稳健的运行。

### 03：AMQP协议

AMQP：(全称：Advanced Message Queuing Protocol)  是高级消息队列协议。由摩根大通集团联合其他公司共同设计。是一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。Erlang中的实现有RabbitMQ等。
特性：
1：分布式事务支持。
2：消息的持久化支持。
3：高性能和高可靠的消息处理优势。

AMQP协议的支持者：
![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002318.png)

### 04：MQTT协议

MQTT协议：（Message Queueing Telemetry Transport）消息队列是IBM开放的一个即时通讯协议，物联网系统架构中的重要组成部分。
特点：
1：轻量
2：结构简单
3：传输快，不支持事务
4：没有持久化设计。
应用场景：
1：适用于计算能力有限
2：低带宽
3：网络不稳定的场景。
支持者：

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002314.png)

### 05、OpenMessage协议

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002312.png)

是近几年由阿里、雅虎和滴滴出行、Stremalio等公司共同参与创立的分布式消息中间件、流处理等领域的应用开发标准。
特点：
1：结构简单
2：解析速度快
3：支持事务和持久化设计。

### 06、Kafka协议

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002310.png)

Kafka协议是基于TCP/IP的二进制协议。消息内部是通过长度来分割，由一些基本数据类型组成。
特点是：
1：结构简单
2：解析速度快
3：无事务支持
4：有持久化设计

### 07、小结

协议：是在tcp/ip协议基础之上构建的一种约定成俗的规范和机制、它的主要目的可以让客户端（应用程序 java，go）进行沟通和通讯。并且这种协议下规范必须具有持久性，高可用，高可靠的性能。

## 消息队列持久化

### 01、持久化

简单来说就是将数据存入磁盘，而不是存在内存中随服务器重启断开而消失，使数据能够永久保存。

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002305.png)

### 02、常见的持久化方式

|          | ActiveMQ | RabbitMQ | Kafka | RocketMQ |
| -------- | -------- | -------- | ----- | -------- |
| 文件存储 | 支持     | 支持     | 支持  | 支持     |
| 数据库   | 支持     | /        | /     | /        |

## 消息的分发策略

### 01、消息的分发策略

MQ消息队列有如下几个角色
1：生产者
2：存储消息
3：消费者
那么生产者生成消息以后，MQ进行存储，消费者是如何获取消息的呢？一般获取数据的方式无外乎推（push）或者拉（pull）两种方式，典型的git就有推拉机制，我们发送的http请求就是一种典型的拉取数据库数据返回的过程。而消息队列MQ是一种推送的过程，而这些推机制会适用到很多的业务场景也有很多对应推机制策略。

### 02、场景分析一

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002303.png)

> 比如我在APP上下了一个订单，我们的系统和服务很多，我们如何得知这个消息被那个系统或者那些服务或者系统进行消费，那这个时候就需要一个分发的策略。这就需要消费策略。或者称之为消费的方法论。

### 03、场景分析二

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002301.png)

> 在发送消息的过程中可能会出现异常，或者网络的抖动，故障等等因为造成消息的无法消费，比如用户在下订单，消费MQ接受，订单系统出现故障，导致用户支付失败，那么这个时候就需要消息中间件就必须支持消息重试机制策略。也就是支持：出现问题和故障的情况下，消息不丢失还可以进行重发。

### 04、消息分发策略的机制和对比

|          | ActiveMQ | RabbitMQ | Kafka | RocketMQ |
| -------- | -------- | -------- | ----- | -------- |
| 发布订阅 | 支持     | 支持     | 支持  | 支持     |
| 轮询分发 | 支持     | 支持     | 支持  | /        |
| 公平分发 | /        | 支持     | 支持  | /        |
| 重发     | 支持     | 支持     | /     | 支持     |
| 消息拉取 | /        | 支持     | 支持  | 支持     |

## 消息队列高可用和高可靠

### 01、什么是高可用机制

所谓高可用：是指产品在规定的条件和规定的时刻或时间内处于可执行规定功能状态的能力。
当业务量增加时，请求也过大，一台消息中间件服务器的会触及硬件（CPU,内存，磁盘）的极限，一台消息服务器你已经无法满足业务的需求，所以消息中间件必须支持集群部署。来达到高可用的目的。

### 02、集群模式1 - Master-slave主从共享数据的部署方式

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002259.png)

解说：生产者讲消费发送到Master节点，所有的都连接这个消息队列共享这块数据区域，Master节点负责写入，一旦Master挂掉，slave节点继续服务。从而形成高可用，

### 03、集群模式2 - Master- slave主从同步部署方式

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002257.png)

解释：这种模式写入消息同样在Master主节点上，但是主节点会同步数据到slave节点形成副本，和zookeeper或者redis主从机制很类同。这样可以达到负载均衡的效果，如果消费者有多个这样就可以去不同的节点就行消费，以为消息的拷贝和同步会暂用很大的带宽和网络资源。在后续的rabbtmq中会有使用。

### 04、集群模式3 - 多主集群同步部署模式

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002256.png)

解释：和上面的区别不是特别的大，但是它的写入可以往任意节点去写入。

### 05、集群模式4 - 多主集群转发部署模式

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002254.png) 

解释：如果你插入的数据是broker-1中，元数据信息会存储数据的相关描述和记录存放的位置（队列）。
它会对描述信息也就是元数据信息就行同步，如果消费者在broker-2中进行消费，发现自己几点没有对应的消息，可以从对应的元数据信息中去查询，然后返回对应的消息信息，场景：比如买火车票或者黄牛买演唱会门票，比如第一个黄牛有顾客说要买的演唱会门票，但是没有但是他会去联系其他的黄牛询问，如果有就返回。

### 06、集群模式5 Master-slave与Breoker-cluster组合的方案

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002251.png)

解释：实现多主多从的热备机制来完成消息的高可用以及数据的热备机制，在生产规模达到一定的阶段的时候，这种使用的频率比较高。

这么集群模式，具体在后续的课程中会进行一个分析和讲解。他们的最终目的都是为保证：消息服务器不会挂掉，出现了故障依然可以抱着消息服务继续使用。

反正终归三句话：
1：要么消息共享，
2：要么消息同步
3：要么元数据共享

### 07、什么是高可靠机制

所谓高可用是指：是指系统可以无故障低持续运行，比如一个系统突然崩溃，报错，异常等等并不影响线上业务的正常运行，出错的几率极低，就称之为：高可靠。
在高并发的业务场景中，如果不能保证系统的高可靠，那造成的隐患和损失是非常严重的。
如何保证中间件消息的可靠性呢？可以从两个方面考虑：
1：消息的传输：通过协议来保证系统间数据解析的正确性。
2：消息的存储可靠：通过持久化来保证消息的可靠性。

# 安装

## RabbitMQ安装

### 01、概述

官网：https://www.rabbitmq.com/
什么是RabbitMQ,官方给出来这样的解释：

> RabbitMQ is the most widely deployed open source message broker.
> With tens of thousands of users, RabbitMQ is one of the most popular open  source message brokers. From T-Mobile to Runtastic, RabbitMQ is used  worldwide at small startups and large enterprises.
> RabbitMQ is  lightweight and easy to deploy on premises and in the cloud. It supports multiple messaging protocols. RabbitMQ can be deployed in distributed  and federated configurations to meet high-scale, high-availability  requirements.
> RabbitMQ runs on many operating systems and cloud  environments, and provides a wide range of developer tools for most  popular languages.
> 翻译以后：
> RabbitMQ是部署最广泛的开源消息代理。
> RabbitMQ拥有成千上万的用户，是最受欢迎的开源消息代理之一。从T-Mobile 到Runtastic，RabbitMQ在全球范围内的小型初创企业和大型企业中都得到使用。
> RabbitMQ轻巧，易于在内部和云中部署。它支持多种消息传递协议。RabbitMQ可以部署在分布式和联合配置中，以满足大规模，高可用性的要求。
> RabbitMQ可在许多操作系统和云环境上运行，并为大多数流行语言提供了广泛的开发人员工具。

简单概述：
RabbitMQ是一个开源的遵循AMQP协议实现的基于Erlang语言编写，支持多种客户端（语言）。用于在分布式系统中存储消息，转发消息，具有高可用，高可扩性，易用性等特征。

### 02、安装RabbitMQ

1：下载地址：https://www.rabbitmq.com/download.html
2：环境准备：CentOS7.x+ / Erlang
RabbitMQ是采用Erlang语言开发的，所以系统环境必须提供Erlang环境，第一步就是安装Erlang。

> erlang和RabbitMQ版本的按照比较: https://www.rabbitmq.com/which-erlang.html
> ![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002231.png)

### 03、 Erlang安装

查看系统版本号

```
[root@iZm5eauu5f1ulwtdgwqnsbZ ~]# lsb_release -aLSB Version:    :core-4.1-amd64:core-4.1-noarchDistributor ID: CentOSDescription:    CentOS Linux release 8.3.2011Release:        8.3.2011Codename:       n/a
```

#### 3-1:安装下载

参考地址：https://www.erlang-solutions.com/downloads/

```
wget https://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpmrpm -Uvh erlang-solutions-2.0-1.noarch.rpm
```

#### 3-2：安装成功

```
yum install -y erlang
```

#### 3-3：安装成功

```
erl -v
```

### 04、安装socat

```
yum install -y socat
```

### 05、安装rabbitmq

下载地址：https://www.rabbitmq.com/download.html
![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002240.png)

#### 5-1:下载rabbitmq

```
> wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.13/rabbitmq-server-3.8.13-1.el8.noarch.rpm> rpm -Uvh rabbitmq-server-3.8.13-1.el8.noarch.rpm
```

#### 5-2:启动rabbitmq服务

```
# 启动服务> systemctl start rabbitmq-server# 查看服务状态> systemctl status rabbitmq-server# 停止服务> systemctl stop rabbitmq-server# 开机启动服务> systemctl enable rabbitmq-server
```

### 06、RabbitMQ的配置

RabbitMQ默认情况下有一个配置文件，定义了RabbitMQ的相关配置信息，默认情况下能够满足日常的开发需求。如果需要修改需要，需要自己创建一个配置文件进行覆盖。
参考官网：
1:https://www.rabbitmq.com/documentation.html
2:https://www.rabbitmq.com/configure.html
3:https://www.rabbitmq.com/configure.html#config-items
4：https://github.com/rabbitmq/rabbitmq-server/blob/add-debug-messages-to-quorum_queue_SUITE/docs/rabbitmq.conf.example

#### 06-1、相关端口

> 5672:RabbitMQ的通讯端口
> 25672:RabbitMQ的节点间的CLI通讯端口是
> 15672:RabbitMQ HTTP_API的端口，管理员用户才能访问，用于管理RabbitMQ,需要启动Management插件。
> 1883，8883：MQTT插件启动时的端口。
> 61613、61614：STOMP客户端插件启用的时候的端口。
> 15674、15675：基于webscoket的STOMP端口和MOTT端口

一定要注意：RabbitMQ 在安装完毕以后，会绑定一些端口，如果你购买的是阿里云或者腾讯云相关的服务器一定要在安全组中把对应的端口添加到防火墙。

## RabbitMQWeb管理界面及授权操作

### 01、RabbitMQ管理界面

#### 01-1：默认情况下，rabbitmq是没有安装web端的客户端插件，需要安装才可以生效

```
rabbitmq-plugins enable rabbitmq_management
```

> 说明：rabbitmq有一个默认账号和密码是：`guest` 默认情况只能在localhost本机下访问，所以需要添加一个远程登录的用户。

#### 01-2：安装完毕以后，重启服务即可

```
systemctl restart rabbitmq-server
```

> 一定要记住，在对应服务器(阿里云，腾讯云等)的安全组中开放`15672`的端口。

#### 01-3：在浏览器访问

http://ip:15672/ 如下：
![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210515002242.png)

### 02、授权账号和密码

#### 2-1：新增用户

```
rabbitmqctl add_user admin admin
```

#### 2-2:设置用户分配操作权限

```
rabbitmqctl set_user_tags admin administrator
```

用户级别：

- 1、administrator 可以登录控制台、查看所有信息、可以对rabbitmq进行管理
- 2、monitoring  监控者 登录控制台，查看所有信息
- 3、policymaker  策略制定者  登录控制台,指定策略
- 4、managment 普通管理员 登录控制台

#### 2-3：为用户添加资源权限

```
rabbitmqctl.bat set_permissions -p / admin ".*" ".*" ".*"
```

### 03、小结

```
rabbitmqctl add_user 账号 密码rabbitmqctl set_user_tags 账号 administratorrabbitmqctl change_password Username Newpassword 修改密码rabbitmqctl delete_user Username 删除用户rabbitmqctl list_users 查看用户清单rabbitmqctl set_permissions -p / 用户名 ".*" ".*" ".*" 为用户设置administrator角色rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
```

## RabbitMQ之Docker安装

### 01、Docker安装RabbitMQ

#### 1-1、虚拟化容器技术—Docker的安装

```
（1）yum 包更新到最新> yum update（2）安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的> yum install -y yum-utils device-mapper-persistent-data lvm2（3）设置yum源为阿里云> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo（4）安装docker> yum install docker-ce -y（5）安装后查看docker版本> docker -v (6) 安装加速镜像 sudo mkdir -p /etc/docker sudo tee /etc/docker/daemon.json <<-'EOF' {  "registry-mirrors": ["https://0wrdwnn6.mirror.aliyuncs.com"] } EOF sudo systemctl daemon-reload sudo systemctl restart docker
```

#### 1-2、docker的相关命令

```
# 启动docker：systemctl start docker# 停止docker：systemctl stop docker# 重启docker：systemctl restart docker# 查看docker状态：systemctl status docker# 开机启动：  systemctl enable dockersystemctl unenable docker# 查看docker概要信息docker info# 查看docker帮助文档docker --help
```

#### 1-3、安装rabbitmq

参考网站：
1：https://www.rabbitmq.com/download.html
2：https://registry.hub.docker.com/_/rabbitmq/

#### 1-4、获取rabbit镜像：

```
docker pull rabbitmq:management
```

#### 1-5、创建并运行容器

```
docker run -di --name=myrabbit -p 15672:15672 rabbitmq:management
```

—hostname：指定容器主机名称
—name:指定容器名称
-p:将mq端口号映射到本地
或者运行时设置用户和密码

```
docker run -di --name myrabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management
```

查看日志

```
docker logs -f myrabbit
```

#### 1-6、容器运行正常

使用 `http://你的IP地址:15672` 访问rabbit控制台

### 02、额外Linux相关排查命令

```
> more xxx.log  查看日记信息> netstat -naop | grep 5672 查看端口是否被占用> ps -ef | grep 5672  查看进程> systemctl stop 服务
```

