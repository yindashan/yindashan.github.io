---
layout: post
title: "像写程序一样写博客：搭建基于github的博客"
date: 2014-07-12 14:42:04 +0800
comments: true
categories: [Octopress, 测试]
tags: [Linux, Github, jekyll]
---
github真是无所不能。其Pages功能支持上传html，并且在页面中显示。于是有好事者做了一个基于github的博客管理工具：octopress，基本原理是用git来管理你的文章，然后最终发布到github上成为一个独立博客站点。由于github支持CNAME域名指向，所以如果有独立域名的话，可以基于这些做出一个专业的博客站点出来。

本博客就是完全基于此搭建起来的，在使用了2个月之后，我将原系统根据中国人的需求做了一些配置，去掉了GFW会挡住的google font api，以及替换掉了速度超慢的国外的评论系统，也加上了分享到国内的微博的功能。现在把这些都总结出来，希望大家都可以方便地搭建基于github的博客来。

安装

首先说说怎么安装相应的工具。其实这些内容在 http://octopress.org/docs/setup/ 上都有，我只是把它大概翻译了一下。

安装rbenv

```
brew update
brew install rbenv
brew install ruby-build

rbenv install 1.9.3-p0
rbenv local 1.9.3-p0
rbenv rehash
```
你有可能需要安装老版本的GCC编译器才能顺利安装Ruby 1.9.3:

```
brew tap homebrew/dupes
brew install apple-gcc42
```
安装Octopress

首先从github上将源码clone下来：



```
git clone git://github.com/imathis/octopress.git octopress
cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).
ruby --version  # Should report Ruby 1.9.2
```
然后安装依赖:



```
gem install bundler
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
bundle install
```
最后安装Octopress

```
rake install
```
配置

安装好之后可以简单配置一下：

主要是修改文件：_config.yml ，这个配置文件都有相应的注释。主要就是改一些博客头，作者名之类的东西。 注意最好把里面的twitter相关的信息全部删掉，否则由于GFW的原因，将会造成页面load很慢。
修改定制文件/source/_includes/custom/head.html 把google的自定义字体去掉，原因同上。
设置github账号

基于github的博客当然需要先注册github账号，Github的账号注册地址是：https://github.com/signup/free 。申请好github账号后，建一个名为 username.github.io 的代码仓库。这里注意username必须是和你的账号名一致。

写博客方法

然后就可以写博客啦～ 写博客主要是用以下几个命令，这里有详细介绍：

```
rake new_post[‘article name’] 生成博文框架，然后修改生成的文件即可
rake generate 生成静态文件
rake watch 检测文件变化，实时生成新内容
rake preview 在本机4000端口生成访问内容
rake deploy 发布文件
```
博文是采用markdown语法，另外增加了一些扩充的插件，markdown的介绍文章网上可以搜到很多，比如这个。

高级配置

我主要介绍一下如何配置评论和分享到微博功能。步骤如下：

在_config.yml中增加一项： weibo_share: true
修改 source/_includes/post/sharing.html ，增加：



```
  // 下面的大括号是全角的，如果复制，请自行改成半角
 ｛% if site.weibo_share %｝
     ｛% include post/weibo.html %｝
 ｛% endif %｝
```
增加文件：source/_includes/post/weibo.html
访问 http://www.jiathis.com/ ，获取分享的代码
访问 http://uyan.cc/ ，获得评论的代码
将上面2步的代码都加入到weibo.html中即可
修改sass/base/_typography.scss，将其中的article blockquote的font-style由italic改为normal,因为中文的引用文字用斜体显示其实并不好看。再将其中的ul, ol 的margin-left: 1.3em;修改为margin-bottom: 0em;。
其它

对于国内的用户来说，Github因为服务器在国外，访问速度上不可避免有些慢。我在2014年5月尝试将博客同时放到Github和GitCafe上（GitCafe提供博客服务，而Github作为备份服务器），使得国内访问速度非常理想，感兴趣的朋友可以参考这篇文章：《将博客从GitHub迁移到GitCafe》

Tips

从wordpress迁移到github

这儿有一篇文章介绍了如何做迁移： http://blog.xupeng.me/2011/12/14/migrate-to-octopress/

图片

如果要在文章中上传图片，直接copy到 /source/images目录下即可。在文章中可以直接引用。也可以选一些大的图床站点，例如flickr之类的放在那边。

域名

如果你象我一样有自己的域名，可以将域名指向这个博客，具体步骤是：

在域名管理中，建立一个CNAME指向，将你的域名指向 yourname.github.io
建一个名为CNAME的文件在source目录下，然后将自己的域名输入进去。
将内容push到github后，第一次生效大概等1小时，之后你就可以用自己的域名访问了。
原理

Octopress其实为你建立了2个分支，一个是master分支，用于存放生成的最终网页。另一个是source分支，用于存放最初的原始markdown文件。
平时写作和提交都在source分支下，当需要发布时，rake deploy命令会将内容生成到 public 这个目录，然后将这个目录的内容当作master分支的内容同步到github上面。
参考

这儿还有一些参考的文章：

http://www.yangzhiping.com/tech/octopress.html
http://blog.xupeng.me/2011/12/14/migrate-to-octopress/
Posted by 唐巧 Feb 10th, 2012   summary

原创文章，版权声明：自由转载-非商用-非衍生-保持署名 | Creative Commons BY-NC-ND 3.0
