---
title: Mac使用Hexo+Github构建自己的博客
date: 2016-03-04 15:01:11
tags:
	- Hexo
	- Mac
	- Github
---


## 一、环境

* 电脑环境：OS 10.11.2
* 拥有Github账号

## 二、Hexo 安装

安装Hexo之前需要安装Nodejs。

在[官网](https://nodejs.org/en/)下载Download for OS X (x64) v5.7.1稳定版。

安装时直接保持默认配置即可。

安装完之后，还需要为npm设置一下镜像:
```
$ sudo npm install -g hexo  --registry=https://registry.npm.taobao.org
```
<!-- more -->

设置完之后，就可以通过国内的镜像下载了。设置的信息会写到$HOME的.npmrc文件里面。否则：
```
npm ERR! network socket hang up
npm ERR! network This is most likely not a problem with npm itself
npm ERR! network and is related to network connectivity.
npm ERR! network In most cases you are behind a proxy or have bad network settings.
npm ERR! network 
npm ERR! network If you are behind a proxy, please make sure that the
npm ERR! network 'proxy' config is set properly.  See: 'npm help config'

```
同时，还需要注意：/usr/local/bin/文件夹的权限。

总之，一顿折腾。

```
hexo-cli: 1.0.1
os: Darwin 15.2.0 darwin x64
http_parser: 2.6.2
node: 5.7.1
v8: 4.6.85.31
uv: 1.8.0
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 56.1
modules: 47
openssl: 1.0.2g

```


***Hexo 安装完成！！***


## 三、Hexo 初始化

安装完成就要开始我们的博客的搭建了，找个地方（例如：~/hexo/）创建一个文件夹用来存放博客。进入该文件夹。

```
$ sudo hexo init
```
或则
```
$ sudo hexo init blog
```
前者在当前文件夹下（~/hexo/）初始化，后者在（~/hexo/blog/）下初始化。

初始化完成之后。Hexo会在这个文件下生成初始文件。

然后**进入**init的文件夹（前者：~/hexo/，后者：~/hexo/blog/）

这里我还多了一个步骤（有些教程没有这一步，有些没有）

```
$ npm install
```
目录中安装 node_modules

当然如果还要安装插件也是在这里执行命令[参考](http://www.chinaz.com/web/2015/1016/458004.shtml):

```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save

```
我安装了：

```
npm install hexo-deployer-git --save # 提交到github时需要
# 生成 RSS 和站点地图需要
npm install hexo-generator-feed@1 --save 
npm install hexo-generator-sitemap@1 --save

```

好了看看效果

```
$ hexo server #启动hexo服务

```

打开浏览器，输入网址 http://localhost:4000/ 

## 四、配置自己的博客信息

初始化完成之后，就要配置属于自己的博客信息了。

修改文件下的_config.yml文件：

```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 青林亦
subtitle: 世界如此喧嚣
description:
author: 青林亦
language: zh-CN #语言
timezone: Asia/Shanghai #时区

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://www.qinglinyi.com # 我的域名
root: /
#permalink: :year/:month/:day/:title/
permalink: posts/:title/
permalink_defaults:

# Directory #目录
source_dir: source #源文件
public_dir: public #生成的网页文件
tag_dir: tags #标签
archive_dir: archives #归档
category_dir: categories #分类
code_dir: downloads/code
i18n_dir: :lang #国际化
skip_render:

# Writing #写作
new_post_name: :title.md #新文章标题
default_layout: post #默认模板(post page photo draft)
titlecase: false #标题转换成大写
external_link: true #新标签页里打开连接
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight: #语法高亮
  enable: true
  line_number: true #显示行号
  auto_detect: true
  tab_replace:

# Category & Tag #分类和标签
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination #分页
per_page: 10 #每页文章数, 设置成 0 禁用分页
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape

  
# Deployment #部署, 同时发布在 GitHub 注意格式 空格
deploy:
  type: git   
  repository: https://github.com/qinglinyi/qinglinyi.github.io.git
  branch: master

# Disqus #Disqus评论系统
disqus_shortname: 

plugins: #插件，例如生成 RSS 和站点地图的
 hexo-generator-feed
 hexo-generator-sitemap

```
***特别注意***
配置文件的里面格式，比如
deploy:（顶格）
 type: git（type空一格并且冒号也要空一格；部署type这里确实是git，据说与Hexo版本相关，老版本是github）
 
当然这个部署格式是没问题的，**亲测可用**

假如你发现还有问题的话:

* 保证你的git可以连接到github。[我是通过这个设置的SSH连接github](http://www.cnblogs.com/zhcncn/p/3787849.html)
* 保证hexo-deployer-git插件可用

## 五、部署到Github

这个很多[教程](http://jingyan.baidu.com/article/d8072ac47aca0fec95cefd2d.html)都有。

主要是注意创建的Repository的名字是：qinglinyi.github.io。保持和你的Github账户一致。

使用命令行测试一下连接Github

```
ssh -T git@github.com
```

然后部署项目

```
$ hexo generate
$ hexo deploy
```

当然是用简写的命令也是可以。

```
$ hexo g
$ hexo s
```
部署完成之后，到[https://qinglinyi.github.io/](https://qinglinyi.github.io/)就可以访问博客了。

## 六、主题

我是使用[hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia)。

## 七、绑定域名

首先申请域名，设置域名解析详细可以移步[这里](http://www.jianshu.com/p/1d427e888dda)

当然，在_config.yml文件下我也设置了url。

## 最后

构建过程肯定不是一帆风顺的，都是一路折腾，所谓道路是曲折的，前途是光明的！



