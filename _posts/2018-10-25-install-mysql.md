---
layout: default_post
title : Linux 下安装 MySQL
category: 安装教程
---




前言：安个最新版本的 MySQL 尝尝鲜。

#### 导航
1. 几种安装方式
1. 目录组成
1. 安装
1. 初始化
1. 启动，登录
1. 配置文件
1. 工具链接
1. 总结


<br>
# 1. 几种安装方式

使用二进制进行安装：

> https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html

使用 yum 等工具进行安装： 

> https://dev.mysql.com/doc/refman/8.0/en/linux-installation.html

使用 docker 进行安装：

> https://dev.mysql.com/doc/refman/8.0/en/linux-installation-docker.html

当然，也可以使用源码安装：

> https://dev.mysql.com/doc/refman/8.0/en/source-installation.html


这里我选择第一种，二进制码安装，即可以省去源码编译的过程，又可以自定义目录。
至于 docker 安装，更加方便，后面再讲。 


<br>
# 2. 目录组成

照了官方文档：
> https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html#binary-installation-layout

可以发现目录组成如下：

![image]({{site.data.photo_url}}18-10-25@01.png)


<br>
# 3. 安装

<br>
#### 01. 卸载残留

如果用 yum 安装过，记得卸载后看有什么没删除干净，比如/etc/my.cnf 配置。

#### 02. 准备依赖

看一下需要的依赖 libaio  有没有安装
rpm -qa | grep libaio  

#### 03.下载
到下载页面

> https://dev.mysql.com/downloads/mysql/

选择版本，复制下载网站，使用 wget 命令
> wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.13-linux-glibc2.12-i686.tar.

#### 04.分配用户组（MySQL不允许分到 root 用户，php 其实也一样，服务器安全问题。）
> groupadd mysql

> useradd -r -g mysql -s /bin/false mysql

- 说明: useradd 命令使用 -r 和 -s /bin/false 选项来创建对服务器主机没有登录权限的用户。

#### 05. 解压 修改名称
> tar xvf mysql-8.0.13-linux-glibc2.12-x86_64.tar.xz 

> mv  mysql-8.0.13-linux-glibc2.12-x86_64 mysql-8.0.13

#### 06.可创建软链接，这样后面就不用输长路径

> ln -s  mysql-8.0.13 mysql


<br>
# 4. 初始化
<br>
#### 01. 使用命令

> bin/mysqld --initialize --user=mysql

一定要记住下面第四行提到的初始密码：

![image]({{site.data.photo_url}}18-10-25@02.png)

初始化之后会多出一个 data 文件

#### 02. 安装SSL的安全访问配置

> bin/mysql_ssl_rsa_setup

这样做以后可以采用加密传输的方式去访问数据库


#### 03. 可能会出现

```
Failed to access directory pointed by --datadir. 
Please make sure that directory exists and is accessible by mysql_ssl_rsa_setup. Supplied value : /usr/local/mysql/data
```

改为指定 data 路径即可
> bin/mysql_ssl_rsa_setup --datadir=/home/install/mysql/mysql-8.0.13/data


<br>
# 5. 启动，登录

<br>
#### 01. 复制一下服务启动的命令
> cp support-files/mysql.server /etc/init.d/mysql.server

这样后面就能用 service mysql.server 去启动、重启服务了。

#### 02. 启动
有三种方式

- 使用 mysqld 脚本启动：bin/mysqld start 
（这是直接启动mysqld进程，不管其之前是否被启动）
- 使用 service 启动：service mysql.server restart
- 使用 mysqld_safe 启动 bin/mysqld_safe --user=mysql &
 （启动MySQL服务器后，继续监控其运行情况，并在其死机时重新启动它）

#### 03. 登录

> mysql -uroot -p'上面提到的初始密码'

<br>
# 6. 配置文件

在 /etc 下新建一个my.cnf，加入一些配置。如果没有这个文件，MySQL 的配置都是默认的，会有一些问题。

```
[mysqld]
port      = 3306
socket    = /tmp/mysql.sock
#安装目录
basedir   = /home/install/mysql/mysql-8.0.13
#存放数据的目录
datadir   = /home/install/mysql/mysql-8.0.13/data
#在data目录下生成一个错误日志
log-error = mysql_err.log

```

重启服务配置才能生效哟。。！！


<br>
# 7. 工具连接

8.0 以上采用 caching_sha2_password，作为密码存储方式，想用 navicat 连接需要先修改为普通密码登录模式：


> alter user 'root'@'localhost' identified with mysql_native_password by '你的密码'

然后修改为可以远程访问：

```
use mysql；
select host, user from user;
update user set host = '%' where user ='root';
```

重启即可！！！！



<br>
# 7. 总结

其实就是要弄懂：
1. mysql 安装后会有一个服务端 和 客户端
1. 新版本的密码存储方式有所不同
1. 可以自动配置 ssl 安全连接
1. 知道如何输入初始化密码，更改初始化密码
1. 知道如何使用工具连接
1. 知道如何增加服务，如何增加快捷命令

<br>
##### 后话
接下来就是 docker 下的 LNMP 搞一搞。
