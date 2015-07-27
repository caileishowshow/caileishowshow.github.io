---
layout: post
title: "搭建Octopress博客相关(一)"
date: 2015-07-27 22:44:50 +0800
comments: true
categories: 
---
Octopress是一个现在比较流行的博客解决方案，相对于其他网站的博客，设置起来更为麻烦，但是整体效果要比其他的要好，至少没有乱七八糟的广告，简单记录下在MAC平台下配置的相关流程，也记录下遇到的各种坑
###安装前提工具
MAC OS系统的好处就是很多工具都是现成的，像一些开发工具等等，一般在安装过xcode之后，很多工具都相应的安装好了
git（mac自带）
ruby 1.9.3以上版本。在mac os 10.10上，默认ruby版本已经是2.0以上的版本，因此，默认是可以用的。
如果想要升级ruby版本，可以参考[升级ruby](http://blog.csdn.net/lissdy/article/details/9191351)，前提是安装了xcode。

###下载Octopress，通过Git开始下载
git clone git://github.com/imathis/octopress.git octopress
下载下来的项目包括配置文件和一些插件等，目前来说还不能使用，需要安装主题模板等

###安装相关工具
####安装bundler

sudo gem install bundler
通过gem下载bundler文件，再执行 
bundle install 进行安装
第一次安装时碰到了一个错误

	Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://api.rubygems.org）
后来发现，原来是当初配置gem时，由于GFW的原因，将境gem源改为了淘宝的ruby源，而当前bundler配置文件中还是只想rubygems，所以导致无法找到链接，具体的做法就是做一个重定向，将api.rubygems.org指向taobao.rubygems.org

	gemsources−−removehttps://rubygems.org/ gem sources -a https://ruby.taobao.org/

之后使用rake工具安装默认的主题以及配置工具

	rake install
安装之后目录中多出三个文件夹 **source** **public** **sass**,其中source是存放各种配置生成的html文件以及各种静态文件的

至此为止，我们已经做好了Octopress的各种配置

###查看成果
输入命令

	rake preview
接下来在浏览器中输入以下内容，就可以进行预览
	
    http://localhost:4000

###首页配置
在写博客之前，首先当然是要给博客起个名字了，在默认主题中，包括一个名字和一行简介，默认情况下是 **My Octopress Blog**和**A blogging framework for hackers.**
具体的修改方式是打开_config.yml,在第五行到第十行的内容
	url: http://yoursite.com
	title: My Octopress Blog
	subtitle: A blogging framework for hackers.
	author: Your Name
	simple_search: https://www.google.com/search
	description:
第一行是访问博客的URL，第二行是博客名称，也就是显示的大字，第三行副标题，第四行作者，用来显示在底部Copyright,第五行站内搜索引擎，默认是google，当然也可以改成baidu，第六行描述，显示在右边相关。

####其他内容
配置文件中还包括好多其他信息，例如时间显示形式，rss格式，各种静态文件配置等信息。此外还有一些第三方内容的配置，包括github，twitter，google+等，由于国内等的原因（你懂得），建议将twitter和google相关的都删去。

###新博客
通过rake new_post['title']生成一个markdown文件。
该文件存放在source/_posts/ 之中，命名格式是是时间-名称.markdown。
注意名称中的title和内容并无任何关系。
编写博客通过markdown的方式进行编写。
当编写完之后可以通过上面rake preview的方式进行预览。

###打包
Octopress实际上是通过ruby进行打包，将资源文档和各种markdown打包成html在网页中进行展示。通过命令
	rake generate
可以将当前的博客文件打包成html静态文件，保存在**public**目录中，点开就可以看到各种html和资源文件等。而以后我们上传到github的也都是这个public文件夹中的静态文件。

现在Octopress的基本用法（安装，设置，打包生成静态文件）都已进行了解，接下来还有很多内容，包括主题设置，样式设置等，以后再慢慢探索。

