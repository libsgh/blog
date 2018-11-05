---
title: Fiddler4-安卓手机抓包
categories: 杂
date: 2018-4-9 18:57:48
tags: [抓包,Fiddler4]
comments: true
---

![](https://ws1.sinaimg.cn/large/6b162853ly1fq6lv1xtamj20dc08wdfr.jpg)

>有的时候我们需要手机中的应用进行抓包，分析http请求协议，Fiddler4是一个很好的选择。

<!-- more -->

## 下载并安装Fiddler4

官网：https://www.telerik.com/fiddler

## fiddler手机抓包原理

在本机开启了一个http的代理服务器，然后它会转发所有的http请求和响应。Fiddler 是以代理web 服务器的形式工作的，它使用代理地址:127.0.0.1，端口:8888。网络请求走fiddler，fiddler从中拦截数据，由于fiddler充当中间人的角色，所以可以解密https。因此，它比一般的firebug或者是chrome自带的抓包工具要好用的多。不仅如此，它还可以支持请求重放等一些高级功能。它还可以支持对手机应用进行http抓包的。

>如果chrome或firefox安装了代理插件记得屏蔽，否则无法抓取

## 设置

- 我这里用笔记本做了wifi热点共享，360wifi，然后手机连接wifi，使电脑和笔记本在同一局域网环境中即可
- Fiddler4设置https和远端代理，默认为8888
![](https://ws1.sinaimg.cn/large/6b162853ly1fq6lid31j7j20f30abwev.jpg)
![](https://ws1.sinaimg.cn/large/6b162853ly1fq6ljoiapvj20f60abgm4.jpg)
- <code>win</code>+<code>R</code>,<code>cmd</code>打开命令窗口，输入<code>ipconfig</code>,查看无线网络的ip（wifi热点）或是局域网ip（路由器）
![](https://ws1.sinaimg.cn/large/6b162853ly1fq6ln13ns0j20eu03oa9y.jpg)
- 打开手机，连接wifi，代理设置成手动，ip为pc端的ip，端口是fiddler4代理的端口，如图所示
![](https://ws1.sinaimg.cn/large/6b162853ly1fq6lqdggosj20u01hcjud.jpg)
- 手机浏览器输入：http://172.27.35.1:7432/，下载证书并安装
![](https://ws1.sinaimg.cn/large/6b162853ly1fq7cfhdzjtj20u01hctbp.jpg)
- **这时可以看到fiddler4成功抓到http请求信息了**
![](https://ws1.sinaimg.cn/large/6b162853ly1fq7cjfv36fj20a60buwf8.jpg)

**Enjoy it~**
