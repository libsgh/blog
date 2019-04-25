---
title: 贴吧签到助手
categories: 原创
date: 2018-6-25 16:30:08
tags: 
     - spring boot
comments: true
---
 [贴吧签到云](https://sign.iicm.tk)是我在工作之余开发的一个程序，旨在帮助百度贴吧用户进行离线签到，回帖，封禁，查ID等web工具，你可以免费自由的使用[它](https://gitee.com/iicm/TieBa-Tool)，喜欢的话在码云上fork一下，您的支持是我维护和更新的动力

<!-- more -->

## 架构及原理介绍

1. 基于spring boot的spring mvc 构建的java web程序，htppclient模拟http请求,jsoup对html进行简单处理抓取信息，前端展示用了基于bootstrap的adminlte模板
2. [tieba-api](https://github.com/libsgh/tieba-api)中封装了一些贴吧基本的请求操作，包括登录获取cookie，回帖，签到，获取用户信息，获取用户关注的贴吧，获取头像，知道签到，文库签到等。

## 开发调试
1. 从git下载源码，导入到eclipse
	![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fsnivdmpl7j20e80ezgm3.jpg)
2. 程序启动的入口<code>App.java</code>，资源目录说明
	![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fsnj3f4hnnj20ap03zmx5.jpg)
    
## 打包部署
1. 从码云上直接下载编译好的jar包:[releases](https://gitee.com/iicm/TieBa-Tool/releases)
2. 自行编译打包：eclipse中选中项目，右键-->run-->Maven build ,Goals输入<code>clean package -DskipTests</code>
	或跳到程序目录下执行<code>mvn clean package -DskipTests</code>，最终在程序所在目录的target下生成jar包
3. 命令部署执行   

``` bash
$ java -jar TieBa-Tool.jar --spring.config.location=application.yml &
```
> <code>spring.config.location</code>指定配置文件的路径

4. 服务方式启动，这里以centos7为例

``` bash
$ cd /etc/systemd/system/
$ vi tt.service
```

```
[Unit]
Description=tb

[Service]
WorkingDirectory=/opt/boot/
PrivateTmp=true
Restart=always
Type=simple
ExecStart=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-0.b14.el7_4.x86_64/bin/java -jar /opt/boot/TieBa-Tool.jar --spring.config.location=/opt/boot/application.yml
ExecStop=/usr/bin/kill -15  $MAINPID

[Install]
WantedBy=multi-user.target
```
``` bash
$ systemctl stop tt
$ systemctl status tt
$ systemctl start tt
```

## 配置文件说明
```
logging:
  level:
    com:
      github:
        tieba: debug
    org:
      springframework: warn
mybatis:
  mapperLocations: classpath:/mapper/**Mapper.xml
  typeAliasesPackage: com.libs.bean
mapper:
  style: normal
server:
  port: '8080'
  tomcat:
    uri-encoding: UTF-8
#https相关配置
    #remote-ip-header: x-forwarded-for
  #use-forward-headers: true
  #address: 127.0.0.1
spring:
  datasource:
    driver-class-name: org.sqlite.JDBC
    password: '1234'
    #数据库路径
    url: jdbc:sqlite:C:\\Users\\Single\\Desktop\\data (2).db
    username: root
  freemarker:
    cache: 'false'
    charset: UTF-8
    check-template-location: 'true'
    content-type: text/html
    expose-request-attributes: 'true'
    expose-session-attributes: 'true'
    request-context-attribute: request
    suffix: .html
    template-loader-path: classpath:views
  http:
    encoding:
      force: true
      charset: UTF-8
      enabled: true
#邮件通知相关配置
  mail: 
   default-encoding: UTF-8
   host: smtp.126.com
   password: xxx
   protocol: smtp
   username: xxx@126.com
   port: 465
   properties:
     mail: 
       smtp:
         auth: true
         ssl:
           enable: true
#版本号
version: 2.0.1
#站点名称
website_name: 贴吧签到云
#是否邀请注册
isInvite: true
#程序首页，用于发送邮件的模板
home_url: https://sign.iicm.tk
```
**参考[spring boot应用启动打包部署](https://iicm.tk/2018/03/22/spring-boot-deploy/)**

## 站点贴图
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fsnk5m6jb9j21ha0q7ahw.jpg)
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fsnk6koqebj21h00qagty.jpg)
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fsnk8lbqs2j21gu0q8tj7.jpg)

**相关连接**
~~[TieBa-Cloud](https://gitee.com/iicm/TieBa-Cloud)~~
[TieBa-Tool](https://gitee.com/iicm/TieBa-Tool)
[tieba-api](https://github.com/libsgh/tieba-api)
[示例站点](https://sign.iicm.tk)
