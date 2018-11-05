---
title: （转）centos7安装最新docker
categories: docker
date: 2018-10-9 09:56:14
tags: 
     - docker
     - centos7
comments: true
---
要求: 机器需要能访问外网

<!-- more -->

### 配置docker阿里云yum源
>直接执行下面的命令即可

```bash
cat >>/etc/yum.repos.d/docker.repo<<EOF
[docker-ce-edge]
name=Docker CE Edge - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF
```

### yum 方式安装 docker

```bash
yum -y install docker-ce


### 查看docker版本
docker --version  

### 启动docker
systemctl enable docker
systemctl start docker
```

原文地址： [https://blog.csdn.net/hehailiang_dream/article/details/79983229](https://blog.csdn.net/hehailiang_dream/article/details/79983229)

