---
title: "Docker镜像"
date: 2023-04-29T10:44:26+08:00
draft: false
tags: ['Linux', 'Docker']
category: '学习之路'
---

# 镜像是什么？

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。
所有的应用，直接打包docker镜像，就可以直接跑起来！
如何得到镜像：

- 从远程操控下载
- 朋友拷贝
- 自己制作镜像 DockerFile

# Docker镜像加载原理

> UnionFS（联合文件系统）

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682572951653-3c46e48f-dc68-4868-aafb-428bb4b3b2c0.png)
我们在下载的时候看到的一层层安装就是这个

> Docker镜像加载原理

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682573251105-da2498cb-be79-4f80-9488-460aa36d09fd.png)

# 分层理解

我们在下载某个镜像时，可以看到都是分层按需下载的，意思就是已有的东西就不再安装，只需要安装没有的。
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682573350049-e0bfe949-9720-420a-b491-b4793965d283.png)
为什么Docker安装要分层安装？
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682573466247-fbd8f0e9-e844-4f85-a561-1334c4508c5f.png)
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682573585950-8053d805-8e29-4c9e-96d5-dbe46600e362.png)
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682573599041-1d26a1dd-6d44-4fab-8b8c-283b9a560943.png)
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682573605792-5b664494-632a-423a-844f-4d99344ae353.png)

> 特点

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。
这一层就是我们通常说的容器曾，容器之下的都叫镜像层。

# commit镜像

```shell
docker commit 提交容器称为一个新的副本

# 命令和git类似
docker commit -m="提交的描述信息"-a="作者" 容器id 目标镜像名：[tag]
```

实战测试：

```shell
1、# 启动一个默认的tomcat

2、# 发现这个tomcat的webapps目录下没有任何东西

3、# 我们就需要将webapps.dist下的东西拷贝到webapps下

4、# 将我们的镜像提交（此时是一个新的镜像），我们以后就使用修改后的镜像即可
```

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682574586923-13e6c7d9-f962-47ed-a82a-4d65f261b2e1.png)
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682574639391-cbaa382f-0cdc-42cc-9cd1-c83aca9e375f.png)
如果你想保存当前容器的状态，就可以通过`commit`来提交，得到一个镜像。
