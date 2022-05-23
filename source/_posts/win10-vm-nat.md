---
title: 解决win10下vm上不去网的问题
categories: 其他
date: 2018-7-9 16:02:24
tags: 
     - 杂
     - 虚拟机
comments: true
---

### 目标
1. 解决win10 vm虚拟机中无法上网的问题
2. 让虚拟机ip是和宿主机相同网段的ip，以便局域网中的其他人能访问你的虚拟机

<!-- more -->

### 方法步骤
1. 先查看当前网络配置，截图记录，备用
![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1ft3o8w09p9j20cs0e474r.jpg)
2. 关闭vm中的虚拟机，再将虚拟机的网络配置还原成默认值
![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1ft3ocmdxsnj20gg0elq3l.jpg)
3. 宿主机中打开网络适配器,将以太网ip，dns均设置为自动获取，选中以太网2和VMate8创建网桥
![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1ft3ofuieykj207703owek.jpg)
4. 设置网桥的IPv4为1步骤中的配置，并测试主机是否能连外网
5. vm的虚拟网络编辑器中设置子网ip，与主机的子网ip相同，并**关闭dhcp服务**
![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1ft3ojqnqdij20g3055t8t.jpg)
6. 设置虚拟机的网络连接
![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1ft3olohjerj20k90ia3za.jpg)
7. 打开虚拟机设置虚拟机中的ip为静态ip和dns
![](https://fastly.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/6b162853ly1ft3oqdm7udj20gb0c73yx.jpg)

### 注意
在上面的步骤<code>5</code>中记得关闭dhcp服务，否则在二级网络环境里会对局域网内的其他机器产生影响

**End~~**
