---
layout:     post
title:      "nginx + uwsgi + centos 部署 django过程全记录"
subtitle:   " \"django\""
date:       2017-08-23 11:02:20
author:     "Daisy"
header-img: "img/post_django.jpg"
catalog: true
tags:
    - django
---

> “django ”

这段时间一直在学**Python**，得益于**Py**(蜜汁微笑)的语言风格，写起代码来神速，曾经使用py爬豆瓣top250和知乎，用py的轮子爽的飞起。

> 先介绍一个django是什么。 django是基于python的restful风格的 **web** 开发框架。让你只用很少的代码就可以写出一个web网站或者一个提供API支持的后台。

## 为什么要使用nginx + uWSGI 部署服务 ##


1. django自己内置了一套遵循wsgi协议的web服务器，关于什么是wsgi，请看[这里](https://baike.baidu.com/item/wsgi/3381529?fr=aladdin "这里")，但是，注意：django自带的web服务器仅限于开发使用，说白了也就是性能不够，不能支持线上生产环境的并发。
2. nginx + uwsgi 是现在市面上主流的部署django服务的做法，使用nginx + uwsgi实现动静分离，动态请求分发给uwsgi,静态请求也就是静态的文件给nginx处理。结构图如下所示
 ![图片来自网络，侵删](http://i.imgur.com/ra7Auuu.png)


## 开始正题   ##
前面是一些概念性的废话，下面我们开始正题

### 服务器环境 ###
    centos 7.3 + python 3.5.2（环境可能不同，但原理是一样的）  vps ： vultr

### 客户端环境 ###
    win10 64位 ，xshell 连接服务器

### python 源码 ###
    一套简单的博客系统，教程地址： http://zmrenwu.com/category/django-blog-tutorial/

### 开始部署 ###
#### 安装Python依赖包 ####
	yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make

#### 安装python版本 ####
centos7自带python版本 2.7，我们安装3.5.2,

    wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz # 下载python包
	tar -xzvf Python-3.5.2.tgz -C /usr/local/src/  # src目录是存放源码的目录解压到src目录
	cd /usr/local/src/Python-3.5.2 # 跳转到解压目录
	./configure --prefix=/usr/local/python3 # 检查
	make && make install  #安装

#### 将python添加到linux环境变量 ####
    cd /etc/profile.d/
    export PATH="$PATH:/usr/local/python3/bin"
    source ../profile  # 重载文件
    echo $PATH  # 查看当前环境变量是否添加 如果发现后面有python3出现，代表成功

#### 代码上传准备工作 ####
- 我将源码托管在github上，再在linux中用git拉下来，所以我们需要安装git
- git安装命令`yum -y install git`

#### mysql安装 ####
centos7中推荐使用mariadb，如果你项目中没有用到mysql你可以不安装

	yum install mariadb-server mariadb  安装
 	systemctl start mariadb 	#启动mysql
	mysql -u root -p #登录mysql 密码默认为空 执行下方的创建数据库语句
	CREATE DATABASE `blog` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci; # 创建数据库，后面我们会用到
	exit # 退出数据库


#### 拉取blog代码 并使用django自带的服务器运行####
前面都是准备工作，下面我们来点看得见的东西。将我们的项目运行起来。如果你不是使用我的源码，请保证你的代码可以通过runserver跑起来，这样才可以谈得上在服务器上部署，下面请跟着一起做，每一步都尽量相同

	cd /opt/
	mkdir blog
	cd blog
	mkdir script # 新建一个文件夹，我们后面有用，现在先不用管
	git clone https://github.com/loopq/django_blog.git # 拉取源码

前面我们已经拉取了代码，接下来我们运行一下，看看我们的代码是不是好的！
	
	cd django_blog    # 来到django_blog工程目录下
	rm -rf static_all # 删除这个文件夹，后面我们再创建
	pip3 install -r requirements.txt # 安装项目所需要的依赖
	python3 manage.py makemigrations # 迁移数据库
	python3 manage.py migrate        # 生成数据库
	python3 manage.py createsuperuser #生成超级管理员，输入账户和密码。密码需要两次相同
	python3 manage.py runserver 0:8000 #运行django服务器，浏览器页面上访问你的ip:8000/blog/，如果出错，请对照上方的步骤一步一步来(注意，请保证你的8000端口是开放给外部的，或者直接把防火墙给关掉)

这时候如果你的web界面是正常的话(何为正常，类似下方)，没有数据是正常的，因为数据库是全新的，我们来添加点数据，浏览器输入你的ip:8000/admin/进入管理员界面，账号和密码是上方生成超级管理员的账号和密码。
![](http://i.imgur.com/ui8XagH.png)

添加数据的步骤我就不演示了（懒，手动逃跑），这是django自带的后台管理，很强大是不是。多玩玩就会了。添加完数据之后回首页刷新，数据就出来了,回xshellctrl+c 停止django服务

OK,现在我们可以保证django代码是正确的。接下来进入正菜

### uwsgi安装 和 配置 ###
uwsgi安装

	pip3 install uwsgi # 安装

uwsgi 启动服务
其实仅仅使用uwsgi也是可以发布django的，只不过因为uwsgi又要处理动态请求，又要处理静态文件，效率没有加入nginx高，所以才加入nginx。下方是uwsgi启动的命令，使用这个可以得到和上方一样的效果。

	uwsgi --http :8000 --module blogproject.wsgi --static-map=/static=static

但是实际情况下，不会直接用命令行启动，都是使用配置文件，下面开始。<br>

	cd /opt/blog/script/
	nano uwsgi.ini # 编辑配置文件 nano是我使用的编辑器，vi，vim都可以使用
	
	# 配置文件内容 ，如果你的工程目录和名字和我的不一样，需要修改下方的 blog 字段 和 django_blog 字段，blog和django_blog是文件夹,blogproject是项目名
	[uwsgi]
	# 项目目录
	chdir=/opt/blog/django_blog/
	# 启动uwsgi的用户名和用户组
	uid=root
	gid=root
	# 指定项目的application
	module=blogproject.wsgi:application
	# 指定sock的文件路径
	socket=/opt/blog/script/uwsgi.sock
	# 启用主进程
	master=true
	# 进程个数
	workers=5
	pidfile=/opt/blog/script/uwsgi.pid
	# 自动移除unix Socket和pid文件当服务停止的时候
	vacuum=true
	# 序列化接受的内容，如果可能的话
	thunder-lock=true
	# 启用线程
	enable-threads=true
	# 设置自中断时间
	harakiri=30
	# 设置缓冲
	post-buffering=512
	# 设置日志目录
	daemonize=/opt/blog/script/uwsgi.log
	# 配置文件内容结束


	# uwsgi 命令
	uwsgi --ini uwsgi.ini  启动
	uwsgi --stop uwsgi.pid  停止
	
	ps -ef | grep -i uwsgi 检查uwsgi有没有启动

检查uwsgi已经启动的情况下，我们进行下一步nginx的安装，如果没有启动，或者出了问题，一步一步排查原因吧


### nginx 安装及设置 ###
#### nginx安装 ####

	yum -y install nginx

访问 ip:80端口出现nginx欢迎页 代表 nginx安装成功

#### nginx配置 ####

	cd /etc/nginx/conf.d
	
为什么一定要在这里？因为在nginx的配置文件中有这么一行

	include /etc/nginx/conf.d/*.conf

**配置文件详细编写**<br>
这里就很重要了，是配置文件的编写

	server {
	listen 80; # 监听哪个端口
	server_name 45.32.71.174 ;  # 你的ip地址 或者是域名，如果是域名需要解析dns，这个自行搞定
	access_log /var/log/nginx/access.log main;  # 日志记录
	error_log /var/log/nginx/error.log;   # nginx错误日志，可自行设置，但必须保证提前建立好该目录和文件
	charset utf-8;
	gzip on;
	gzip_types text/plain application/x-javascript text/css text/javascript application/x-httpd-
	php application/json text/json image/jpeg image/gif image/png application/octet-stream;
	error_page 404 /404.html;
	error_page 500 502 503 504 /50x.html;
	# 指定项目路径uwsgi
	location / {
		include uwsgi_params;
		uwsgi_connect_timeout 30;
		uwsgi_pass unix:/opt/blog/script/uwsgi.sock; # 这里需要改为你的目录(如果你和我的目录一致不需要改动)
	}
	# 指定静态文件路径
	location /static/ {
		alias /opt/blog/django_blog/static/;   # 这里需要改为你的目录(如果你和我的目录一致不需要改动)
		index index.html index.htm;
	}
	}


在这里编写的配置文件都会被自动包含，编写完成之后使用

	nginx -t 

确认编写正确，出来的信息尾部带有successful表示成功，有错误去错误日志里面看详细日志，一点点改就行。

#### nginx + uwsgi 运行 及 检查 ####

	service nginx stop
	
	uwsgi --stop uwsgi.pid(如果已经启动 停止)
	
	uwsgi --ini uwsgi.ini

	service nginx start


### 以为到这里就完了？ ###

访问admin页面试试？
![](http://i.imgur.com/kCFCOlS.png)


![](http://i.imgur.com/VRFGGem.jpg)

为什么我们的CSS和js都失效了？我们的首页可都是好的呀。这是怎么肥四！？	

原来啊是因为admin是django自带的，css和js等静态文件都在源码里面，是找不到的。


### 收集静态文件 ###
setting文件加入：

	STATIC_ROOT = os.path.join(BASE_DIR, 'static_all')
	
在项目路径下执行

	python3 manage.py collectstatic --noinput # 收集文件

你可以在项目目录下看到static_all文件夹，里面就包含整个项目的静态文件 和 admin的相关静态文件

别忘了在 nginx的配置文件中配置 静态文件所在地，我们还需要他处理静态请求呢。

nginx配置文件更改完之后最好重启，因为nginx对静态文件是有缓存的。

ok，到这里就全部结束。That's DONE!

参考链接：<br>
[https://docs.djangoproject.com/en/1.11/howto/static-files/deployment/]()<br>
[http://www.jianshu.com/p/80393ae41a5f]()<br>
[http://www.jianshu.com/p/6d4e0cad4bf9]()<br>
[这个是视频，51cto的，讲师很不错](http://edu.51cto.com/course/9037.html)




历时一天半，搞定。  莫名的自豪感哈  ^~^





	
	
	
	
	








