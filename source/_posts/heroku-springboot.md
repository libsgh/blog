---
title: heroku上部署spring boot 应用
categories: heroku
date: 2018-10-12 18:14:59
tags: 
     - heroku
     - spring boot
comments: true
---

### 前期准备

- [注册 Heroku 账户](https://signup.heroku.com/dc) 需要扶墙
- [安装Heroku](https://devcenter.heroku.com/articles/heroku-cli) Toolbelt Heroku Command Line Interface (CLI)

<!-- more -->

### 本地登陆Heroku heroku login 

输入用户名密码

``` bash
$ heroku login
Enter your Heroku credentials.
Email: java@example.com
Password: (typing will be hidden)
Authentication successful.
```

### 创建spring boot项目

省略

### 将代码提交到本地仓库

``` bash
$ git init
$ git add .
$ git commit -m "first commit"
```

### 创建一个Heroku应用xxx

``` bash
$ heroku create xxx
```

### 推送到heroku 远程git仓库

```bash
$ git push heroku master
```

### 日志查看及bash访问

```bash
$ heroku logs --tail
```
```bash
$ heroku run bash
```

### 设置编码

```bash
$ heroku config:add LANG=en_US.UTF-8
```

### 注意事项

![](https://ws1.sinaimg.cn/large/6b162853ly1fw5lt1u95fj20bo0l4wfg.jpg)

>1. 30分钟无人访问就休眠（解决办法： https://uptimerobot.com 定时访问你的应用）
2. 绑定了信用卡之后每个月有1000小时（所有应用加和），普通用户只有550小时（所有应用加和），而一个月有720小时，所以...而且无法自定义域名（解决办法：绑定visa或是mastercard）
3. 如果要通过程序在heroku上保存文件，需要放到/app下面，重启后，目录重置
4. 中文文件名乱码，需要改变编码格式
5. 内存512M，如果超出200%会停止应用···