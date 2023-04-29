---
title: "Docker容器数据卷"
date: 2023-04-29T10:45:36+08:00
draft: false
tags: ['Linux', 'Docker']
category: '学习之路'
---

# 什么是容器数据卷？

## Docker理念回顾

将应用和环境打包成一个镜像！
保存数据？如果数据在容器中，那么我们容器删除，数据就会丢失。需求：**数据持久化！**
容器之间可以有一个数据共享的技术！Docker容器中产生的数据同步到本地！
这就是一个`卷`技术。目录的挂载，将我们容器中的目录挂载到Linux上。
总结：容器的持久化和同步操作！容器间也是可以数据共享的！

# 使用数据卷

> 方式一：直接使用命令来挂载 -v

```shell
docker run -it -v 主机目录：容器内目录

# 测试
[root@VM-4-5-centos ~]# docker run -it -v /home/centostest:/home centos /bin/bash 

# 通过 docker inspect 容器id 查看挂载信息
```

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682575510147-bec5ff91-a8a9-479b-9505-1ee881a1d48b.png)
测试同步：
主机：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682575694586-e43176d7-7db7-4213-891c-1e4029d268ff.png)
容器：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682575697642-39c4871d-d6dc-4a57-8a55-1a6326aa1c8d.png)
测试二：先停止容器，在主机上修改文件
容器：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682575953273-ffb390c1-bca4-49fa-88d9-7c65e690552d.png)
主机：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682575962986-f276b607-41fe-4018-a104-7e618cf47a02.png)

# 实战：安装MySQL

思考：MySQL的数据持久化问题，`data`

```shell
docker pull mysql

# 运行的时候 需要挂载，需要注意的是，运行MySQL的时候需要配置密码！
# 官方：docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动挂载MySQL：
-d     后台运行
-p     端口映射
-v     卷挂载
-e     环境配置
--name 容器名字
[root@VM-4-5-centos ~]# docker run -d -p 3307:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql
```

启动成功之后，本地使用工具连接。

# 具名挂载，匿名挂载

```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx

# 查看所有volume的情况 匿名卷
docker volume ls

# 这里发现，我们-v的时候只写了容器内的路径，没有写主机路径

# 具名挂载 通过-v卷名：容器内路径
docker run -d -P --name nginx01 -v test-nginx:/etc/nginx nginx
# 查看这个卷
docker volume inspect test-nginx
```

所有docker容器内的卷，没有指定目录的情况下都是在 `/var/lib/docker/volumes/xxx/_data`下

```shell
[root@VM-4-5-centos home]# docker run -d -P --name nginx02 -v test-nginx:/etc/nginx nginx
cd53fa44710df853dd0b228c8e0a63fe107ece61d763111f777facbe37d2ea5b
[root@VM-4-5-centos home]# docker volume inspect test-nginx
[
    {
        "CreatedAt": "2023-04-27T14:43:14+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/test-nginx/_data",
        "Name": "test-nginx",
        "Options": null,
        "Scope": "local"
    }
]

```

我们通过具名挂载可以方便的找到一个卷，大多数情况下使用的是`具名挂载`

```shell
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载
-v 容器内路径 # 匿名挂载
-v 卷名：容器内路径 # 具名挂载
-v /主机路径：容器内路径 # 指定路径挂载
```

拓展：

```shell
# 通过-v 容器内路径：ro rw 改变读写权限
ro  readonly 只读
rw  readwrite 可读可写


docker run -d -P --name nginx02 -v test-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v test-nginx:/etc/nginx:rw nginx

# 只要看到 ro 就说明这个路径只能通过主机来操作，容器内部是无法操作的！
```

# 初识Dockerfile

Dockerfile就是用来构建docker镜像的构建文件！命令脚本。
通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个一个命令！
先体验一下！

```dockerfile
# 创建一个dockerfile文件
# 文件中的内容 指令（大写） 参数

FROM centos

VOLUME ["volume01", "volume02"]

CMD echo "----end----"
CMD /bin/bash

# 这里的每个命令，就是镜像的一层一层！
```

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682578924703-16110cde-e940-4e81-a5aa-2c821b9187f7.png)

```shell
# 启动一下自己的镜像（匿名挂载）
docker run -it 镜像id /bin/bash
```

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682579096782-441227b3-0ed4-4776-811c-3c211da90c9b.png)

```shell
# 查看卷挂载的路径
docker inspect 容器id
```

![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682579396029-b8274af9-89cf-4746-956e-12ad2282746a.png)
测试文件同步：
镜像：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682579477421-0c83e840-bd5d-4e45-aa67-8253f770ff1f.png)
主机：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682579488281-8be04a59-124c-4b7d-abf0-22f58a12606b.png)
这种方式我们为来使用的十分多！
假设构建镜像时没有挂载卷，要手动镜像挂载 -v卷名：容器内路径！

# 数据卷容器

多个MySQL同步数据。
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682579812826-1946502b-e005-4900-8464-e6f37796cbca.png)

```shell
# 启动三个容器，通过我们自己写的镜像

```

启动`docker01`（父容器）：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682580160981-2ad8180d-d6b7-4b62-9e44-b44b9a944220.png)
启动`docker02`：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682580275803-c32f794b-5e84-45cc-8435-af32b1353cf5.png)
测试：docker01中在`volume01`中添加文件，在docker02中查看：
docker01：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682580416334-f1080c8e-62c6-41af-88d5-a27cd63f305d.png)
docker02：
![image.png](https://img-1315662121.cos.ap-guangzhou.myqcloud.com/img/1682580418664-29c3599d-e7e9-4440-8f0a-4ba0f9f5bded.png)

```shell
# 测试：可以删除docker01，查看docker02是否还能访问数据
# 结论：依旧能够访问！
```

结论：
容器之间配置信息的传递，数据卷容器的生命周期一致持续到没有容器使用为止！
但是一旦你持久化到了本地，这个时候本地的数据是不会删除的！
