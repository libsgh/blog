---
title: v2ray + heroku 实现免费国际穿越（暂时失效）
categories: 其他
date: 2018-7-12 19:31:46
tags: 
     - 杂
comments: true
---

### 简介

> [Heroku](https://www.heroku.com/) 是 Salesforce 旗下云服务商，提供方便便捷的各种云服务，如服务器，数据库，监控，计算等等。并且他提供了免费版本，这使得我们这些平时想搞一些小东西的人提供了莫大的便捷，虽然他有时长和宕机的限制，但是对于个人小程序来说已经足够了。
[V2Ray（Project V）](https://www.v2ray.com/)是一个优秀的开源网络代理工具，可以帮助你畅爽体验互联网，目前已经全平台支持Windows、Mac、Android、IOS、Linux等操作系统的使用。相对起Shadowsocks来说属于后起之秀，在混淆能力、兼容性、速度上有着独到的优点。目前在ss,ssr被大范围封杀的情况下，V2Ray是一个不错的选择。

<!-- more -->

### 注册登录

很简单，根据官网的提示操作就行

### 一键部署v2ray 到 heroku

1. 首先访问  https://github.com/wangyi2005/v2ray-heroku ，点击<code>Deploy to Heroku</code>
![](https://ws1.sinaimg.cn/large/6b162853ly1ft7a48pgqij20qw0dmgmp.jpg)
2. 部署时配置 v2ray core 的版本、Vmess协议的UUID（"alterId"默认为64）和连接缓存,UUID可以用客户端配置工具生成，这里相当于连接的密钥
![](https://ws1.sinaimg.cn/large/6b162853ly1ft7a9o5kbij20k90e8dg9.jpg)
3. 服务端部署后，应 open app ，显示 Bad Request，表示部署成功。
4. 更新 v2ray 版本，修改 app settings-->Config Vars-->VER，程序自动重启，通过view Logs确认。
5. 客户端配置 client_config.json, 建议使用 cn_sniproxy+websocket+tls 传输协议。

### 客户端安装

1. 各种客户端：https://www.v2ray.com/ui_client/
2. 这里以<code>v2rayN</code>为例： https://github.com/2dust/v2rayN/releases 下载.exe可执行程序
3. 下载<code>v2ray-core</code>： https://github.com/v2ray/v2ray-core/releases
4. 将<code>v2rayN.exe/<code>和<code>v2ray-core</code>放在同一文件夹内
5. 运行<code>v2rayN.exe</code>

### 客户端配置

1. 注意uuid（**与服务器端配置的Vmess协议的UUID一致**），alterId，传输协议用**ws**,也就是websocket协议
![](https://ws1.sinaimg.cn/large/6b162853ly1ft7alydlsjj20e50cv3yr.jpg)

2. tls配置（推荐），注意修改端口
![](https://ws1.sinaimg.cn/large/6b162853ly1ftbq2rvh0yj20ii0dijrt.jpg)

3. 手机安装客户端，扫描二维码或是通过url分享导入配置
![](https://ws1.sinaimg.cn/large/6b162853ly1ft7alyfip0j20cq0zk756.jpg)

4. Ubuntu环境下使用V2ray，参考：https://deng55.github.io/2017/12/04/Ubuntu%E7%8E%AF%E5%A2%83%E4%B8%8B%E4%BD%BF%E7%94%A8V2ray/

### 使用

1. chrome或firefox配置SwitchyOmega插件
2. 开启系统代理+pac，基本和ss相同
3. 如果本地同时开启了ss，可以修改本地监听端口避免冲突
![](https://ws1.sinaimg.cn/large/6b162853ly1ft7awspzewj20ek042746.jpg)

### 测试总结

用来上google查资料是没问题的，看youtube可能稍慢，速度整体来说还可以，毕竟免费

### 相关连接

heroku官网：https://www.heroku.com/
v2ray官网：https://www.v2ray.com/
一键部署脚本：https://github.com/wangyi2005/v2ray-heroku
客户端：https://www.v2ray.com/ui_client/

**End~**