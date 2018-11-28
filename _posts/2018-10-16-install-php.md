---
layout: default_post
title : Linux 下安装 PHP（分类：安装教程）
---




前言：初学的时候总是什么什么集成开发环境，或者用 docker 安装，还没好好在 Linux 下用源码安装 PHP。今天就搞一把 PHP 安装先。

#### 导航
1. 下载
1. 相关依赖一览
1. 安装前的配置
1. 编译
1. 安装
1. 相关配置
1. 总结


<br>
# 1. 下载

- 官网地址：

> http://php.net

- 下载地址：

> http://php.net/downloads.php

- 下载命令

> wget http://am1.php.net/get/php-7.2.12.tar.gz/from/this/mirror

- 进入下载目录，解压：

> tar -xzf mirror


<br>
# 2. 相关依赖一览

#### gcc   

> yum -y install gcc-c++

#### zlib 和 zlib-devel 

> yum install zlib

> yum install zlib-devel

#### libxml2 以及 libxml2-devel 

> yum install  libxml2

> yum install libxml2-devel 

#### libxslt 以及 libxslt-devel 

> yum install libxslt 

> yum install libxslt-devel 

#### openssl 以及 openssl-devel
> yum install openssl 

> yum install  openssl-devel

#### jpeg 和 png

> yum install libjpeg libpng ibjpeg-devel libpng-devel 

#### freetype 以及 freetype-devel
> yum install freetype 

> yum install  freetype-devel

#### gd 
> yum install gd 

#### curl

> yum install curl

>yum install curl-devel

```
注：
使用 yum list installed  可以查看使用 yum 安装了哪些依赖包
使用 rpm -qa | grep xxx 可以查看是否安装了 xxx
xxx-devel 是 xxx 软件的开发包，包含头文件以及静态库甚至源码。
```


<br>
# 3. 安装前的配置

<br>
- 查看可以自定义的配置：

> ./configure --help

- 需要注意的有如下几个：

```
--prefix  =  安装目录

--with-xxx    xxx 是要安装的拓展【如果拓展已经在本地安装了，也可也用 = 去指定目录】

--enable-xxx   xxx是需要激活的拓展功能

--with-config-file-path=  php.ini文件位置

--sysconfdir= 其他配置文件的位置，不指定默认是在 安装目录 / etc 下
```

- 最终执行命令如下（我自定义了安装目录 和 php.ini 的位置）：

```
./configure \
--prefix=/home/install/php/php7 \
--with-config-file-path=/home/install/php/php7-ini \
--with-curl \
--with-freetype-dir \
--with-gd \
--with-gettext \
--with-iconv-dir \
--with-kerberos \
--with-libdir=lib64 \
--with-libxml-dir \
--with-mysqli \
--with-openssl \
--with-pcre-regex \
--with-pdo-mysql \
--with-pdo-sqlite \
--with-pear \
--with-png-dir \
--with-xmlrpc \
--with-xsl \
--with-zlib \
--enable-fpm \
--enable-bcmath \
--enable-libxml \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-opcache \
--enable-pcntl \
--enable-shmop \
-enable-soap \
--enable-sockets \
--enable-sysvsem \
--enable-xml \
--enable-zip \
--disable-fileinfo


```

成功则提示

![image]({{site.data.photo_url}}18-10-16@01.png)



<br>
# 4. 编译

#### 执行 编译

> make

```
注意：可能会出现 

make: *** [ext/fileinfo/libmagic/apprentice.lo] Error 1 

是因为服务器内存不足1G，在上面配置时  ./configure 时加参数 --disable-fileinfo 即可。
```

#### 成功提示
![image]({{site.data.photo_url}}18-10-16@02.png)


<br>


# 5. 安装

#### 执行 安装

> make install

#### 安装成功
![image]({{site.data.photo_url}}18-10-16@03.png)


<br>
# 6. 相关配置 和 启动

<br>	
#### 拷贝一下配置示例文件 php.ini 

> cp php.ini-development  /home/install/php7-ini/php.ini


#### 进到安装目录的 etc 中

> cp  php-fpm.conf.default  php-fpm.conf

#### 进到安装目录的 etc/php-fpm.d 中 

> cp www.conf.default www.conf（配置用户的文件）

#### 进到 sbin 中， 启动 php-fpm 

> ./php-fpm 

#### 查看是否启动 (其实 php-fpm 模式，就是启动监听了 9000 端口的进程。)

> ps -ef \| grep php



#### 添加 php 到环境变量  

````
vi  /etc/profile

在末尾加入：
PATH=$PATH:/home/install/php/php7/bin
export PATH

使改动立即生效：
source /etc/profile
````

#### 查看版本

> php -v

<br>
# 7. 总结

总的来说，需要：
1. 知道有哪些依赖
1. 知道如何自定义配置
1. 执行编译安装
1. 按提示解决一些依赖、环境问题
1. 知道各种配置文件的位置，知道如何启动

<br>
##### 后话
其实和安装 Nginx 差不多，或者说 Linux 下的安装都是这个样的。接下来需要去熟悉一下一些基本的配置。
