---
title: "Redis入门"
date: 2023-04-09T20:16:42+08:00
draft: false
tags: ['Linux', 'Redis']
category: '学习之路'
---

# Redis是什么？

Redis（remote dictionary server），远程字典服务<br />开源的使用C语言编写，支持网络、可基于内存亦可支持持久化、k-v数据库、提供多种语言等等

# Redis能干嘛？

1. 内存存储、持久化，内存中是断电即失、持久化很重要（RDB、AOF）
2. 效率高，可以用作高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器、计数器（浏览量）
6. ...

# 特性

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务

# Redis安装

## Windows安装

直接GitHub下载压缩包解压即可。<br />地址：[https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100](https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100)<br />开启Redis：双击`redis-server.exe`文件即可。<br />使用客户端连接Redis：打开`redis-cli.exe`即可。输入`ping`命令，返回`PONG`即ok。

## Linux安装

### 1、进入官网下载安装包

地址：[https://redis.io/download/](https://redis.io/download/)
<a name="LeFRj"></a>

### 2、将安装包传到服务器上

使用`xftp`工具传输到服务器上。这里我放在`/opt`目录下

### 3、进行解压

使用`tar -zxvf redis-7.0.10.tar.gz`进行解压。
```sh
tar -zxvf redis-7.0.10.tar.gz
```



### 4、安装gcc编译

`yum install gcc-c++`<br />

```sh
yum install gcc-c++
```

通过`gcc -v`查看版本，有版本号即成功。<br />

```sh
gcc -v
```

但是这里有个问题，因为`redis 6.0`以上的版本需要gcc高版本。所以使用以下命令升级为`gcc 9.*`

```sh
yum -y install centos-release-scl
```

```<br />
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
```

```sh
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

然后进入 `/opt`目录下使用`make`，然后等待环境配好。<br />然后再次输入`make install`，等待配好即可。

```sh
make install
```

### 5、默认路径

`redis`默认安装位置为`/usr/local/bin`
<a name="cyDDt"></a>

### 6、拷贝配置文件

将`/opt/redis-7.0.10/redis.conf`拷贝到`/usr/local/bin`下面新建的`redisconfig`目录下。

```sh
# 在/usr/local/bin 下新建redisconfig
mkdir redisconfig

# 拷贝文件到redisconfig
cp /opt/redis-7.0.10/redis.conf /usr/local/bin/redisconfig
```

### 7、修改配置文件

修改`/usr/local/bin/redisconfig/redis.conf`文件<br />

```sh
vim /usr/local/bin/redisconfig/redis.conf
```

将其中的`daemonize`的`no`改为`yes`

```sh
daemonize no  -> yes
```

### 8、运行redis

在`/usr/local/bin`目录下使用命令`redis-server redisconfig/redis.conf`运行，<br />

```sh
redis-server rediscongig/redis.conf
```

然后使用`redis-cli -p 6379`可以连接redis服务器。<br />

```sh
redis-cli -p 6379
```

输入`ping`，若返回为`PONG`即为成功。

```sh
6379> ping
PONG
```

### 9、查看redis进程

`ps -ef|grep redis`

```sh
ps-ef | grep redis
```

### 10、退出服务

在连接界面输入`shutdown`关闭redis连接，然后输入`exit`即可。

```sh
shutdown

exit
```

可以再次查看redis进程看是否关闭。
```sh
ps -ef | grep redis
```

# 测试性能

使用`redis-benchmark`加指定参数测试。

首先打开redis服务，输入下面命令测试：

`redis-benchmark -h localhost -p 6379 -c 100 -n 10000`

```sh
redis-benchmark -h localhost -p 6379 -c 100 -n 10000
```

随机截取一点来看看：

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1681036770515-5c576693-ef14-4dd6-ada5-6a5e2170d026.png)

可以看到10000条请求在0.19秒就完毕！100个客户端并发！可以看到redis的速度真的非常恐怖。

# 基础知识

redis，默认有16个数据库

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1681112343086-f25fb47f-5b01-4da4-a118-bcbca7bdd6ab.png)

默认使用的是第9个

可以使用`select`进行切换不同的数据库

```shell
select 3 # 切换数据库

dbsize # 查看数据库大小

keys * # 查看所有key

flushdb # 清空当前数据库

flushall # 清空全部数据库


```

**Redis是单线程的！**

Redis是很快的，基于内存操作，CPU不是性能瓶颈，Redis瓶颈是根据机器的内存和网络带宽。

为什么单线程还这么快？

1. 误区1：高性能的服务器一定是多线程的？
2. 误区2：多线程（CPU上下文会切换）一定比单线程效率高

核心：Redis多有数据都放在内存中，所以说使用单线程区操作效率就是最高的，因为多线程需要CPU上下文切换，对于内存系统来说，没有上下文切换，效率就是最高的。多次读写都在一个CPU下。端并发！可以看到redis的速度真的非常恐怖。
