---
title: Netlify托管静态博客hexo
categories: hexo
date: 2018-03-21 16:00:00
tags: [hexo,netlify]
comments: true
---
## 介绍
Netlify是一个现代网站自动化系统，其JAM架构代表了现代网站的发展趋势。所谓JAM，就是指基于客户端JavaScript、可重用API和预构建Markup标记语言的三者结合。Netlify完美且免费支持的ssl、域名绑定、http/2和TLS。最重要的就是，管理方式用git方法传递给github、gitlab或者是Bitbucket，然后Netlify就能自动编译并生成静态网站，支持自动编译Jekyll、Hexo、Hugo等常见的静态博客程序。

<!-- more -->

### 注册使用Netlify
1. 官方首页：https://www.netlify.com/
2. 注册账号：可以使用Github、Gitlab、Bitbucket授权登陆，然后登录到空间管理中心，点击右上角的“New site from Git”添加网站
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fplftfrz6vj20xa0araak.jpg)
3. 使用github托管，选择一个仓库，此仓库存放博客程序
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853gy1fplg2brzsxj20k00c8q3m.jpg)
4. 绑定域名，我这里用dnspod解析，添加CNAME到brave-tereshkova-***.netlify.com
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853gy1fplgba6evqj20mf013a9y.jpg)
5. https可以一键开启，还可以强制https
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853gy1fplgaaah1dj20pi0kywfm.jpg)
6. Build command中填写部署命令
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853gy1fplgcgdpc1j20lx0b0dge.jpg)
7. 开源博客程序：https://www.staticgen.com/

### hexo安装
 > 参考官方的安装文档: https://hexo.io/zh-cn/docs/
      修改后push到github，Netlify会自动部署更新。
	
**Netlify的访问速度不错，其他功能大家自行摸索~**
