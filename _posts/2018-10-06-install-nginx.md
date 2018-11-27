---
layout: default_post
title : Linux 下安装 Nginx（分类：安装教程）
---




前言：写一下 LNMP 环境的搭建，从 Nginx 开始

#### 导航
1. 下载
1. 目录介绍
1. 编译安装
1. 常见的一些组合命令
1. 总结


<br>
# 1. 下载

- 访问官网：
> https://time.geekbang.org/  (开源版)

- 下载地址
>http://nginx.org/en/download.html

- 选择稳定版的（双数） 
> http://nginx.org/download/nginx-1.14.1.tar.gz

- 使用 wget 命令
> wget http://nginx.org/download/nginx-1.14.1.tar.gz

- 进到源码目录，解压
> tar -xzf nginx-1.14.1.tar.gz 


<br>
# 2. 目录介绍

#### auto目录
- cc ：用于编译
- lib : 库文件
- os ： 判断所有操作系统特性
- 其他：辅助上面三个功能

####  CHANGES  
- 每一个版本的特性 和 修复的 bug

####  CHANGES.ru  
- 作者是俄罗斯人，所以有一个俄语版的CHANGES  

####  conf  
- 配置示例文件，安装完后会被自动拷贝变成我们的配置文件

####  configure  
- 脚本文件，生成中间文件，执行编译前的必备动作

####  contrib  
- 提供 vim 工具，可以让 vim 识别 nginx语法 
- 使用 cp -r contrib/vim/* /usr/share/vim/vim74/ 拷贝一下即可

####  html 
- 两个标准的 html5 文件，500 错误的默认界面，以及欢迎界面
 
####  man  
- 帮助文件

####  src
- 源代码


<br>
# 3. 编译安装

#### 01. 使用 ./configure --help | more 查看一些安装前需要的配置

```
--prefix      指定安装路径
--with-xxx 使用哪些模块   （不加的话就不编译 xxx 模块 到 Nginx 中）
--without-xxx 不使用哪些模块 （不加的话就默认编译 xxx 模块到 Nginx 中）
```

#### 02. 这里使用默认安装即可

> ./configure --prefix=/home/install/nginx

#### 03. 可能会遇到

- 缺少依赖
- 依赖重复（冲突）
- yum 源需要更换

等等情况，按提示去解决即可。

#### 04. 配置成功会提示
![image]({{site.data.photo_url}}18-10-06@01.png)

#### 05. 然后可以看到新生成了一个目录 objs，文件下有
```
autoconf.err  Makefile  ngx_auto_config.h  ngx_auto_headers.h  ngx_modules.c  src
```

其中 ngx_modules.c 决定了接下来编译有哪些模块会被编译进去

#### 06. 然后就可以执行 make命令 去编译了

> make

![image]({{site.data.photo_url}}18-10-06@02.png)

#### 07. 再次进入 objs 中

可以看到生成大量的编译生成的目标文件（.o结尾的），这些文件还可以用于 Nginx 升级。以及还可以看到动态模块编译生成的 so 结尾的文件。


#### 08. 然后首次安装需要执行 
> make install

这个命令链接所有的.o 以及其他杂七杂八，生成可执行文件

#### 09. 安装成功，提示：

![image]({{site.data.photo_url}}18-10-06@03.png)

#### 10. 可以到安装目录查看：
```
conf 
html  
logs  
sbin
```

搞定收工！！！


<br>
# 4. 总结

大致思路：
1. 先要知道每个文件夹大概的作用
1. 知道如何自定义安装配置
1. 补全需要的依赖
1. 进行编译安装

<br>

##### 后话
其实就像在 window 下安装一个软件一样，只不过 window 的图形化界面把这些简化成了下一步、下一步。
