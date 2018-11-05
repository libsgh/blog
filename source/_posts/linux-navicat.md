---
title: Navicat Premium 12.0.11 for linux 安装与激活
categories: Linux mint
date: 2018-04-27 19:59:29
tags: [linux Mint,navicat]
comments: true
---
[Navicat](http://www.navicat.com.cn)是一款非常好用的sql管理工具,功能强大,使用简单.下面介绍win&linux下如何安装和激活

### 下载/安装/配置

1. 下载地址:[naicat Premium 12](http://www.navicat.com.cn/download/navicat-premium)
> linux版本其实是wine,所以激活方法win和linux都是通用的,linux版本下载下来是个tar包，目前linux版本是12.0.11
2. 解压
``` bash
$ tar -zxvf  /usr/local/navicat120_mysql_cs_x64.tar.gz
```
3. 解压后  cd进入解压后的目录运行命令
``` bash
$ ./start_navicat
```
4. 启动后会乱码,编辑start_navicat将<code>export LANG="en_US.UTF-8"</code>改为<code>export LANG="zh_CN.UTF-8"</code>
5. 添加快捷方式,网上找一个ico的图标,我是在win用提取的exe图标
``` bash
$ cd /user/share/applications
$ sudo touch navicat.desktop
$ sudo vim navicat.desktop
```
 **添加内容如下，注意修改Exec和Icon参数:**
```
[Desktop Entry]
Encoding=UTF-8
Name=Navicat
Comment=Navicat Premium
Exec=/home/single/soft/navicat120_premium_cs_x64/start_navicat
Icon=/home/single/soft/navicat120_premium_cs_x64/Navicat.ico
Terminal=false
StartupNotify=true
Type=Application
Categories=Application;Development;
```

6. 复制到桌面
```bash
$ cp navicat.desktop /home/single/Desktop
```
**这时打开是需要注册的,有14天的试用期**
 
 <!-- more -->
 
### 破解

1. [破解补丁](https://github.com/DoubleLabyrinth/navicat-keygen/releases)
2. 下载解压 之后有个说明是win下面的破解方法,按照步骤来
> linux破解执行命令前面加上<code>wine</code>
3. 最后一步需要输入request code(in Base64),这时我们断网直至激活成功,打开navicat,输入上面得到的SnKey,点击激活,会有一个错误提示
4. 选择手动激活,将步骤2中生成的激活码复制到下面,点激活,出现Navicat现已激活的提示就成功了
![](https://ws1.sinaimg.cn/large/6b162853ly1fqrg60cx3nj20fi0ctdgp.jpg)

### 资源下载

打包上传到百度云了,懒人去拿
链接: https://pan.baidu.com/s/1VW9UeTq5j94FpAwa4qFdHQ 密码: <code>26th</code>

### 注意事项

1. 之前傻了尝试在虚拟机中,或是win中激活,然后压缩包复制到linux中,发现还是需要重新激活,这种方式行不通,包括网上有一种补丁直接能拦截注册提示窗口的注册方式,在win下面是可以的,但是linux下无效
2. linux下删除/home/single/.navicat64/system.reg,这样快要到期的时候删除一次,会重新计算,这种方式我没来及尝试,如果可行,可以设置crontab定时删除这个文件,**删除整个.navicat64文件夹会丢失数据库连接信息**
3. 在激活过程中有什么问题,可以删除<code>RegPrivateKey.pem</code>文件及安装目录下的<code>navicat.exe.backup</code>文件,重新按步骤激活
4. 成功激活的版本12.0.11,其他版本自行尝试,有问题可以去[](https://github.com/DoubleLabyrinth/navicat-keygen)寻找答案
5. windows最新版本是12.0.29，可以尝试网盘中另一款激活工具<code>Navicat_Keygen_Patch_v3.4_By_DFoX_URET.exe</code>，操作更加简便,亲测有效
![](https://ws1.sinaimg.cn/large/6b162853ly1fs0lkgbxdej20fw0fp0ub.jpg)
**Enjoy~**
