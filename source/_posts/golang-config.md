---
title: go安装、开发环境配置
categories: go
date: 2021-03-08 17:23:00
tags: 
     - go
comments: true
---
![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/ae757299d8afb757830359579bda077c.png)

<!-- more -->

### go相关网址

- 官网：https://golang.google.cn/
- 中文社区：https://studygolang.com/
- 全球代理：https://goproxy.io/zh/
- 中国区代理：https://goproxy.cn/

### 开发工具

- https://www.jetbrains.com/go/

### go安装

1. 下载对应平台的版本：https://golang.google.cn/dl/

2. 官方安装教程：https://golang.google.cn/doc/install

   - 以Linux系统安装为例

     ```bash
     $ wget https://golang.org/dl/go1.16.linux-amd64.tar.gz
     ```

   - 以root身份运行

     ```bash
     $ rm -rf / usr / local / go && tar -C / usr / local -xzf go1.16.linux-amd64.tar.gz
     ```

   - 将`/usr/local/go/bin`添加到`PATH`环境变量

     > 通过将以下行添加到`$HOME/.profile`或`/etc/profile`或`~/.bashrc`（用于系统范围的安装）中来执行此操作

      ```bash
     export PATH=$PATH:/usr/local/go/bin
      ```

     

   - 验证是否安装成功，查看安装版本

     ```bash
     $ go version
     ```

### 配置

1. 查看环境变量

   ```bash
   $ go env
   GO111MODULE="on"
   GOARCH="amd64"
   GOBIN=""
   GOCACHE="/home/single/.cache/go-build"
   GOENV="/home/single/.config/go/env"
   GOEXE=""
   GOFLAGS=""         
   GOHOSTARCH="amd64"                       
   GOHOSTOS="linux"                     
   GOINSECURE=""                        
   GOMODCACHE="/home/single/GoEnv/pkg/mod"                       
   GONOSUMDB=""                       
   GOOS="linux"                       
   GOPATH="/home/single/GoEnv"                        
   GOPRIVATE=""                       
   GOPROXY="https://goproxy.io"                       
   GOROOT="/usr/local/go"                       
   GOSUMDB="sum.golang.org"                       
   GOTMPDIR=""                        
   GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"                       
   GOVCS=""                       
   GOVERSION="go1.16"                       
   GCCGO="gccgo"                        
   AR="ar"                                        
   CC="gcc"                                      
   CXX="g++"                                   
   CGO_ENABLED="1"                                      
   GOMOD="/dev/null"                                     
   CGO_CFLAGS="-g -O2"                                       
   CGO_CPPFLAGS=""                               
   CGO_CXXFLAGS="-g -O2"                               
   CGO_FFLAGS="-g -O2"                                
   CGO_LDFLAGS="-g -O2"                                      
   PKG_CONFIG="pkg-config"                                         
   GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build4160819105=/tmp/go-build -gno-record-gcc-switches" 
   ```

2. 开启go module，国内还需要设置代理，解决有些包无法下载的问题

   ```bash
   go env -w GO111MODULE=on
   go env -w GOPROXY=https://goproxy.cn,direct
   ```

### 创建go module项目

* 执行下面命令将生成`go.mod`文件，包含模块名称和版本

  ```bash
  $ mkdir goapp
  $ cd goapp
  $ go mod init goapp
  ```

  

### 使用goland开发go应用

#### File >> Settings >> Go

1. **GOROOT**：go的安装目录，这里会从系统自动读取，也可以单独指定

   ![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/48fddbee2e7ccf0a6c4d17892ea4f8fc.png)

2. **GOPATH**：用来设置个人工作区间对应的目录。里面可以存放编写的代码、编译文件、编译后的可执行文件。

   ![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/a98f7e99ddbee2e5f151d06029f089a6.png)

3. **Go Modules**：开启go modules，并设置`GOPROXY=https://goproxy.io`

   ![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/318c9b4445335533927f7ec74701bc83.png)

#### 运行配置

环境变量、传入参数、构建后运行（指定构建后的文件目录）等

![](https://cdn.jsdelivr.net/gh/libsgh/blog/themes/material-x/source/img/article/dd88199dede13311d20511838fde2c52.png)