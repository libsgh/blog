---
title: java配置环境变量
categories: java
date: 2018-3-24 10:37:30
tags: [java]
comments: true
---

### 根据系统下载jdk 

- jdk8: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
- jdk9: http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html
- jdk10: http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html

<!-- more -->

### windows下安装配置

- 根据系统位宽下载32/64位jdk并安装
- 配置环境变量，以win10为例
**我的电脑->属性->高级系统设置->环境变量**

| 变量名 | 变量值 |
| :-: | :-: |
| Path | %JAVA_HOME%\bin|
| CLASSPATH | .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar; |
| JAVA_HOME | C:\Program Files\Java\jdk1.8.0_121 |
>**注意：**加在用户变量里只对当前用户有效

用下面的命令验证是否安装成功

```
$ java
$ javac
$ java -version
```
### linux 下安装配置（解压版）

- 解压jdk-7u79-linux-x64.tar.gz

```
$ tar -zxvf jdk-7u79-linux-x64.gz
```

- 在/etc/profile文件末尾添加下面几行

```
$ vi /etc/profile
```

```
JAVA_HOME=/usr/local/jdk1.7.0_79
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
JRE_HOME=$JAVA_HOME/jre
export JAVA_HOME JRE_HOME PATH CLASSPATH
```
- 执行以下命令让修改生效：

```
$ source /etc/profile
```

- java、javac、java -version验证配置是否正确

### linux 下安装openjdk
```
$ sudo apt-get install openjdk-7-jre
$ sudo apt-get install openjdk-7-jdk
```
然后按照之前的步骤配置环境变量

### CentOS yum 安装JDK

- 查看yum库中都有哪些jdk版本

```
$ yum search java|grep jdk
```

- 选择版本,进行安装

```
$ yum install java-1.7.0-openjdk

```

>安装完之后，默认的安装目录是在: /usr/lib/jvm/java-1.7.0-openjdk-1.7.0.75.x86_64

- 设置环境变量（同上）

### 用rpm安装JDK

- 下载rpm安装文件

```
$ curl -O http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.rpm
```

- 使用rpm命令安装

```
$ rpm -ivh jdk-7u79-linux-x64.rpm
```

- 设置环境变量（同上）

**End~**