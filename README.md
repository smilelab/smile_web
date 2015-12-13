# 实验室网站搭建手册
## 0. 简介
网站使用Hexo进行静态网页生成，使用七牛做图片的CDN，同时挂载在gitcafe和github上，但是smilelab.uestc.edu.cn指向smilelab.gitcafe.io，smilelab.github.io仅作发布测试用。

使用Hexo生成静态网页有以下好处：

1. 使用Markdown便捷内容，简单便捷
2. 使用ejs模板语言做布局，容易入门


## 1. 安装

### 环境：
ubuntu14.04 x64

### 安装过程：
1. 安装NodeJS，推荐从官网下载二进制包，解压到/usr/local/目录下
2. 安装git：sudo apt-get install git
3. cd smile_lab/
4. 安装hexo：sudo npm install hexo --no-optional             （这一步通常需要翻墙，可以使用shadowsocks+tsocks配合）
5. npm install                 （这一步同样需要翻墙）
6. 测试是否安装成功：hexo g，生成静态网页，如果这一步没有出错，则说明已经安装成功

### 附：使用shadowsocks+tsocks翻墙方法：
因为shadowsocks是socks5技术，系统并不支持socks5的全局转发，所以需要tsocks工具把所有的网络请求转换成socks5后再出去。

	sudo add-apt-repository ppa:hzwhuang/ss-qt5
	sudo apt-get update 
	sudo apt-get install shadowsocks-qt5 tsocks

然后在dash中打开shadowsocks-qt5，配置并连接到远程，本地端口设置为1080；编辑/etc/tsocks.conf文件：

	local = 192.168.0.0/255.255.255.0
	local = 10.0.0.0/255.0.0.0
	server = 127.0.0.1
	server_type = 5
	server_port = 1080
	
在需要翻墙的命令前加上tsocks。如npm install这一条命令需要翻墙，则变成**tsocks npm install**.

## 2. 结构
### 总体结构
以下是整个网站的所有文件目录：

	.
	├── README.md			本文件
	├── _config.yml			关于网站的全局变量配置
	├── db.json				生成静态网页时需要的db，不需要理会
	├── debug.log
	├── logo-header.png	logo
	├── node_modules		安装的hexo环境，不必理会
	├── package.json		hexo环境，不必理会
	├── public					生成静态网页后，静态网页的存放目录
	├── scaffolds			hexo预定义的模板，不必理会
	├── source					网站所有内容的存放目录，这是最关键的目录
	└── themes					网站主题存放目录，对于需要修改主题的，可以在这个目录下完成

基本上，主要关注两个目录即可：即**source和themes**。下面分别介绍这两个目录。

### source目录结构
这是source目录的结构**（注意与网站的链接对应）**：

		.
	├── contact					联系我们
	│   └── index.md			主页/contact的主要内容
	├── en							英文版面，结构与上一层完全一样
	│   ├── contact
	│   ├── images
	│   ├── index.md			英文版面的主页
	│   ├── joining_us
	│   ├── members
	│   ├── news
	│   ├── publications
	│   └── research
	├── google15ebd55a83bf26a9.html	用来做google搜索优化的文件，不必理会
	├── images
	├── index.md					这是我们网站主页的内容
	├── joining_us
	│   └── index.md
	├── members
	├── news
	│   └── index.md
	├── publications
	│   └── index.md
	├── research
	│   └── index.md
	└── robots.txt				为搜索引擎提供支持

这里的所有文件都是使用markdown语言进行编辑，关于markdown语言，稍后会有相应的解释。

**但是需要注意到en/目录下，所有的md文件在front-matter区域都有layout: layout_en来标记该md文件使用layout_en模板进行生成。**至于为什么需要这么做，稍后会有解释。

### themes目录结构
themes目录下，在我们的网站中是这样的结构：

	.
	└── hueman
	    ├── Gruntfile.js
	    ├── LICENSE
	    ├── README.md
	    ├── _config.yml				该主题的全局设置
	    ├── _config.yml.example	一个example，不必理会
	    ├── languages					多语言支持
	    ├── layout						各种布局模板
	    ├── package.json
	    ├── scripts						各种js脚本集合
	    └── source						布局相关的css文件

因为我们使用了hueman这个主题，因此在themes目录下仅有这个目录。当然如果我们需要更换主题，只需要把新主题拷入该目录，然后在根目录的_config.yml中修改themes的值即可。

需要注意的是，为了做多语言支持，除了在languages目录下定义对应的字符串外，我对layout/layout.ejs文件做了修改。因为layout.ejs是模板生成的入口，也就是说其他任何模板都会在最外层套上layout.ejs。

注意到我们网站最顶部的menu button与/source目录中的md文件没有任何关系，那么这些button是如何识别/source目录中的md文件对应的语言呢？

在/source目录下，所有的md文件默认使用page.ejs模板进行生成，在page.ejs外层再套上layout.ejs，拼装后生成html文件。于是，我们可以这样做：

1. 在layout.ejs文件中，仅作为入口：<%- body %>
2. 修改page.ejs文件，这是默认的中文页面模板，注意到menu button在_partial/header.ejs中的定义
3. 新增layout_en.ejs文件，这是默认的英文模板，注意到menu button在_partial/header_en.ejs文件中的定义
4. 修改languages目录中的文件，新增不同语言的menu button的链接定义

## 3. 编辑
编辑网站内容主要针对source目录。

关于markdown语言，可以简单的从这个网页了解到markdown的一切：<http://wowubuntu.com/markdown/>

这里我仅列出几个需要注意的地方：

* 在我们的环境中，所有的**标题**，在若干个`#`的后面，**需要留出一个空格**
* source目录中，可以看到几乎每个文件头部都有front-matter，这官方的解释：
	Front-matter 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量，举例来说：

```
title: Hello World
date: 2013/7/13 20:46:25
---
```
这就是一个简单的front-matter，用于定义该文件的一些变量，具体有什么变量可以从这里查看：<https://hexo.io/zh-cn/docs/front-matter.html>

**需要注意的是，所有的英文版面，都需要加上`layout: layout_en`变量声明！**

*  为了加快网站的访问速度，目前把所有的图片都传上了**七牛**，然后在七牛中生成对应的外链，最后把外链插入到markdown文本中，具体过程在稍后解释
*  目前所有的markdown文件的文件名都是**index.md**，具体是根据文件夹做链接映射

## 4. Todo-List
1. 英文版面的内容
2. 规范图片尺寸
3. google、baidu的搜索优化