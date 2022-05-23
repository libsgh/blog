---
title: Linux du及df命令的使用
categories: linux
date: 2018-3-24 10:25:05
tags: [linux]
comments: true
---
>du 和 df 命令都是 Linux 系统的重要工具，来显示 Linux 文件系统的磁盘使用情况。

<!-- more -->

## du命令

- <code>du</code>（disk usage 的简称）是用于查找文件和目录的磁盘使用情况的命令。du 命令在与各种选项一起使用时能以多种格式提供结果。
- 下面是一些例子

### 得到一个目录下所有子目录的磁盘使用概况

>$ du /home

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fpnovaaft8j20cl04it8s.jpg)

- 该命令的输出将显示 <code>home</code> 中的所有文件和目录以及显示块大小。

### 以人类可读格式也就是 kb、mb 等显示文件/目录大小

>$ du -h /home

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fpnovqw89oj20dn04j0st.jpg)

### 目录的总磁盘大小

>$ du -sh /home

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fpnowpiqkyj20g801pmx0.jpg)

- 它是<code>/home</code> 目录的总大小

## df 命令

- <code>df</code>（disk filesystem 的简称）用于显示 Linux 系统的磁盘利用率。（LCTT 译注：df 可能应该是 disk free 的简称。）
- 下面是一些例子

### 显示设备名称、总块数、总磁盘空间、已用磁盘空间、可用磁盘空间和文件系统上的挂载点。

>$ df

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fpnoyh3rruj20hq04d74j.jpg)

### 人类可读格式的信息

>$ df -h

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fpnoxw9hd1j20ht04574i.jpg)

- 上面的命令以人类可读格式显示信息。

### 显示特定分区的信息

>$ df -hT /etc在

![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1fpnoxgb4y0j20hq01q748.jpg)

<code>-hT</code> 加上目标目录将以可读格式显示<code>/etc</code> 的信息。

**虽然 <code>du</code> 和  <code>df</code> 命令有更多选项，<code>--help</code>查看**

原文地址：https://linux.cn/article-9457-1.html
