# 1.消息中间件概述

## 1.1 什么是消息中间件

**消息队列**是指利用**高效可靠**的**消息传递机制**进行与平台无关的**数据交流**,并基于**数据通信**来进行分布式系统的集成

## 1.2 消息队列的特点



## 1.3 AMQP和JMS

### 1.3.1 AMQP

### 1.3.2 JMS

### 1.3.3 AQMP与JMS的区别

## 1.4 消息队列产品





# 2. 安装与配置 rabbitmq

## Windows下安装

1.下载并安装Erlang

rabbitmq服务端代码是使用Erlang语言编写的，安装rabbitmq需要先安装Erlang

下载地址：[https://www.erlang.org/downloads](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.erlang.org%2Fdownloads)

根据自己电脑的配置和rabbitmq的版本下载对应的Erlang环境

安装版本：Erlang的版本的为23.0 rabbitmq的版本为3.8.5

安装步骤 下一步就行

配置Erlang环境变量

![image-20200712200335057](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712200335057.png)

在path下新建，输入%ERLANG_HOME%\bin

![image-20200712200408372](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712200408372.png)

查看配置是否生效

![image-20200712200709570](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712200709570.png)

2.下载并安装rabbitmq

下载地址：https://www.rabbitmq.com/download.html

安装完成之后会默认在服务中创建rabbitmq服务

![image-20200712200955963](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712200955963.png)

RabbitMQ安装好后接下来安装RabbitMQ-Plugins。打开命令行cd，输入RabbitMQ的sbin目录。
 然后在后面输入rabbitmq-plugins enable rabbitmq_management命令进行安装

![image-20200712201046088](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712201046088.png)

启动方式：

1.打开sbin目录，双击rabbitmq-server.bat

![image-20200712201229825](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712201229825.png)

2.在服务列表中启动rabbitmq服务

![image-20200712201322133](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712201322133.png)

访问 [http://localhost:15672](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A15672)进行web端管理界面

默认账号 guest guest

![image-20200712201447086](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712201447086.png)

![image-20200712201502745](https://raw.githubusercontent.com/fontStep/photos/master/img/image-20200712201502745.png)