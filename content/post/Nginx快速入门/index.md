---
title: "Nginx快速入门"
date: 2023-04-09T20:15:22+08:00
draft: false
---

公司项目刚上线，并发量少，只需一个jar包启动即可，<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040244337-74e4343c-1f43-4c1c-87ca-425c7c0be289.png#averageHue=%23f7f7f7&clientId=u4980bad8-10a3-4&from=paste&height=244&id=u10365bdf&name=image.png&originHeight=244&originWidth=582&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10408&status=done&style=none&taskId=ub750129e-75a4-449e-9718-187938908b7&title=&width=582)<br />但慢慢的，使用的用户越来越多，并发量增大，一台服务器根本不能满足！<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040274363-3914548b-848f-4b48-b793-03d9a472d008.png#averageHue=%23f6f2f2&clientId=u4980bad8-10a3-4&from=paste&height=336&id=u1fbe7bdb&name=image.png&originHeight=336&originWidth=544&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22032&status=done&style=none&taskId=u9811dad2-8932-42a7-89ba-c77ad8131f8&title=&width=544)<br />这时候，可以通过扩展服务器的数量，这时候几个项目启动在不同的服务器上，用户要访问，就需要增加一个代理服务器来帮我们解决和处理请求。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040383105-b3158b30-78bd-4b04-bc4c-2d0f53190da9.png#averageHue=%23f4f4f4&clientId=u4980bad8-10a3-4&from=paste&height=402&id=ud1fd5651&name=image.png&originHeight=402&originWidth=721&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36054&status=done&style=none&taskId=u0ee1b123-a2d3-4da6-b7d9-bf16b61315c&title=&width=721)<br />我们希望这个过程是用户无感知的，所以选择了**Nginx**！
<a name="GFmaQ"></a>

# 什么是Nginx？

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。2011年6月1日，nginx 1.0.4发布。<br />其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。在全球活跃的网站中有12.18%的使用比率，大约为2220万个网站。<br />Nginx 是一个安装非常的简单、配置文件非常简洁（还能够支持perl语法）、Bug非常少的服务。Nginx 启动特别容易，并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。你还能够不间断服务的情况下进行软件版本的升级。<br />Nginx代码完全用C语言从头写成。官方数据测试表明能够支持高达 50,000 个并发连接数的响应。
<a name="FSC2X"></a>

# Nginx作用？

> Http代理，反向代理：作为web服务器最常用的功能之一，尤其是反向代理。

正向代理<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040543569-2db3f5cb-e76f-4d02-ace3-5928336adc03.png#averageHue=%23f8f8f8&clientId=u4980bad8-10a3-4&from=paste&height=380&id=u17e2b039&name=image.png&originHeight=380&originWidth=627&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30887&status=done&style=none&taskId=u2e95e6fe-fecf-49f8-aa61-33d1fe1c1bc&title=&width=627)<br />反向代理<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040554158-aa3e76b5-018a-4985-8a5f-4aecc505f3e8.png#averageHue=%23f9f9f9&clientId=u4980bad8-10a3-4&from=paste&height=402&id=ub00c0a4e&name=image.png&originHeight=402&originWidth=620&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52177&status=done&style=none&taskId=u23f0cc9c-b428-475a-b6d7-5ffe41673de&title=&width=620)

> Nginx提供的负载均衡策略有两种：内置策略和扩展策略。内置策略为轮询，加权轮询，Iphash扩展策略。

轮询：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040601065-c048d9f2-5c26-4055-9aee-a0f3f30ad29d.png#averageHue=%23f9f9f9&clientId=u4980bad8-10a3-4&from=paste&height=363&id=u4a77d30b&name=image.png&originHeight=363&originWidth=630&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46598&status=done&style=none&taskId=u1d0d4e2c-cc00-4bb4-82fc-c2a55742d3e&title=&width=630)<br />加权轮询：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040610665-feee5185-7f38-4faa-9011-68dec46e68bf.png#averageHue=%23f9f9f9&clientId=u4980bad8-10a3-4&from=paste&height=326&id=u0eb23f93&name=image.png&originHeight=326&originWidth=618&originalType=binary&ratio=1&rotation=0&showTitle=false&size=47546&status=done&style=none&taskId=ubf8037b2-4c6b-4f0f-a0b7-8aa6239ee6e&title=&width=618)<br />iphash对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040660465-a7b4fe09-4bab-47b7-8a87-fd4d4e76d8bc.png#averageHue=%23f8f8f8&clientId=u4980bad8-10a3-4&from=paste&height=331&id=u4bfa7f28&name=image.png&originHeight=331&originWidth=620&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53682&status=done&style=none&taskId=u09f460a5-767a-45b5-8162-638f32eae92&title=&width=620)

> 动静分离，在我们的软件开发中，有些请求是需要后台处理的，有些请求是不需要经过后台处理的（如css，html，jpg，js等文件），这些不需要经过后台处理的文件称为静态文件。让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其左缓存操作，提高资源相应的速度。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681040782256-f604f1bb-c452-47fc-b33f-9e4d5088ba52.png#averageHue=%23fafafa&clientId=u4980bad8-10a3-4&from=paste&height=310&id=u407dffe3&name=image.png&originHeight=310&originWidth=638&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37917&status=done&style=none&taskId=u54946675-2fc5-492e-821a-b5a5f1c90cf&title=&width=638)<br />目前，通过使用Nginx大大提高了我们网站的响应速度，优化了用户体验，让网站的健壮性更上一层楼！
<a name="ZSBAV"></a>

# Nginx的安装

<a name="UGSFO"></a>

## Windows下安装

<a name="JB1gU"></a>

### 1、下载

地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html) 下载稳定版本<br />下载后直接解压即可，但是要注意路径不能有中文！！！
<a name="MW27y"></a>

### 2、启动nginx

再解压目录下，打开cmd，执行`nginx.exe`即可运行
<a name="rxwcm"></a>

### 3、访问

浏览器输入：`http://localhost:80`，能看到下面的页面即成功<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/35204765/1681041024171-3d9f5faa-a7a8-41e1-a081-7011cc519e7b.png#averageHue=%23f8f6f5&clientId=u4980bad8-10a3-4&from=paste&height=211&id=u7a89477f&name=image.png&originHeight=211&originWidth=557&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42728&status=done&style=none&taskId=ufaa2efe9-eba3-4db2-9d44-a2266890bb2&title=&width=557)
<a name="uHYoa"></a>

### 4、配置监听

nginx的配置文件是`conf`目录下的`nginx.conf`，默认端口为`80`，若被占用只需要更改即可。<br />修改配置文件之后，需要执行`nginx -s reload`命令让其生效
<a name="c4N7D"></a>

### 5、关闭nginx

使用cmd命令窗口启动nginx，关闭窗口是不能结束nginx进程的，可以有两种方法关闭：

1. 输入命令：`nginx -s stop`快速停止或者`nginx -s quit`完整有序的停止
2. 使用`taskkill /f /t /im nginx.exe`

> taskkill 是用来终止进程的。
> /f 是强制终止
> /t 终止指定的进程和任何由此启动的子进程
> /im 是指定的进程名称

<a name="oUvgQ"></a>

## Linux下安装

<a name="jZVU8"></a>

### 1、安装gcc

`yum install gcc-c++`
<a name="i9bfR"></a>

### 2、下载安装包

地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)<br />将下载好的安装包（tar.gz）通过xftp工具传输到服务器上。我这里放在`/opt`下
<a name="CWR1G"></a>

### 3、解压

通过`tar -zxvf nginx-1.22.0.tar.gz`<br />进入解压好的目录下：<br />`cd nginx-1.22.0`
<a name="rj9Kj"></a>

### 4、配置

使用默认配置，在nginx根目录下执行<br />`./configure`<br />`make`<br />`make install`<br />通过命令`whereis nginx`即可看到nginx的安装目录在`/usr/local/nginx`下
<a name="cl70T"></a>

### 5、启动nginx

进入到nginx安装目录的`sbin`下。<br />`cd /usr/local/nginx/sbin`<br />通过`./nginx`运行nginx，然后浏览器输入`ip地址:80`即可访问。
<a name="OyOhq"></a>

### 6、关闭nginx

还是在`/usr/local/nginx/sbin`目录下：<br />`./nginx -s stop`或者`./nginx -s quit`
<a name="WHies"></a>

### 7、配置文件

配置文件在`/usr/local/nginx/conf/nginx.conf`<br />修改配置文件之后需要返回目录`/usr/local/nginx/sbin`使用命令`./nginx -s -reload`重启服务。
<a name="eEzeQ"></a>

# nginx常用命令

```shell
cd /usr/local/nginx/sbin/
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件
ps aux|grep nginx  查看nginx进程
```

如果连接不上：则可以通过下面命令检查：

```shell
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
# 查看防火墙规则
firewall-cmd --list-all
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```
