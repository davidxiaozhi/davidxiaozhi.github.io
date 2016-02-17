---
layout:     post
title:      "Hello 2015"
subtitle:   " \"Hello World, Hello Blog\""
date:       2015-01-29 12:00:00
author:     "davidxiaozhi"
header-img: "img/post-bg-rwd.jpg"
tags:
    - tutorial
    - jekyll
---

#Jekyll安装指南

首先介绍一下jekyll
>  Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的

这使我们只需要专注与书写文档本身，熟悉基本的markdown语法即可，而不用过多的关注页面布局样式等，文档书写完成后也可以本地预览相应模板渲染的效果。
</br>


####官方提供的安装命令

```shell
 $ gem install jekyll
~ $ jekyll new my-awesome-site
~ $ cd my-awesome-site
~/my-awesome-site $ jekyll serve
# => 打开浏览器 http://localhost:4000
```
***

幸福的是，你依据官方提供的安装命令顺利的安装成功了,不幸的是，由于每个人使用的linux系统内核以及内核版本不同，预安装软件版本不同，那么你可能遇到的报错信息也不同。首先**gem**命令存在与ruby当中，而jekyll命令依赖的ruby版本在2.0以上。我在初期 执行jekyll 安装命令时 ，通过gem安装好相关依赖后，缺发现ruby版本必须2.0以上，而使用sudo apt-get install ruby 后安装的是1.9.3版本，因此建议大家使用rvm安装ruby，下面总结一下安装过程以及遇到的各种问题

-  第一步安装rvm
   我们可以采用官方提供的安装命令安装
[官方网站 https://rvm.io/](https://rvm.io/)

```shell
	gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3；
	curl -sSL https://get.rvm.io | bash -s stable
```
   如果由于网络原因，不能通过以上命令安装，我们可以下载rvm的安装包，本地安装执行，rvm最后安装在***～/.rvm*** 目录,如果经常使用可以将～/.rvm/bin/rvm追加进/etc/profile当中 PATH变量

- 第二部安装ruby 2.0.0版本以上的ruby，我是卸载后重新安装的

```shell
  		$ sudo apt-get remove ruby ruby-dev ;
		#安装2.2.0版本 jekyll要求
		rvm install 2.2.0；
		rvm use 2.2.0 --default
```
-	安装jekyll
```shell
	 gem install jekyll #顺利完成哦
```

有的时候我们会发现rvm以及ruby不能使用
>You need to change your terminal emulator preferences to allow login shell.
Sometimes it is required to use `/bin/bash --login` as the command.
Please visit https://rvm.io/integration/gnome-terminal/ for an example.

就是有的时候需要登陆shell才能运行，我们可以使用上文提示的`/bin/bash --login` ，或者在ubuntu终端配置以登陆shell方式运行

登陆shell和非登陆shell的区别可以参考<http://smilejay.com/2012/10/interactive-shell-login-shell>

---

#### jekyll的使用指南

>待后续补全
