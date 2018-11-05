---
title: win10下使用Eclipse + scala + sbt开发spark应用
categories: 大数据
date: 2018-7-9 11:50:25
tags: 
     - 大数据
     - spark
     - scala
comments: true
---

一般认为，项目文件夹中包含 build.sbt 文件，即为 sbt 项目。

<!-- more -->

## 环境配置

### 安装java（略）

### 安装并配置SBT

1. 下载安装包: https://www.scala-sbt.org/download.html ， 下载msi安装包直接安装，如果下载ZIP包，需要配置环境变量，将sbt的bin目录添加到PATH中。

2. 设置好后，打开DOS窗口(R + Win -> cmd)，输入<code>sbt</code>，在第一次运行时，sbt会联网下载一些jar包。
``` bash
$ sbt
```
3. sbt本地仓库的设置
> sbt 使用 ivy 作为库管理工具。ivy 默认把 library repository 建在user home下面。
定制 library local repository 的位置的方法是：
修改sbt配置文件：[sbt安装目录]\conf\sbtconfig.txt；
文件中添加
<code>-Dfile.encoding=UTF8</code>
<code>-Dsbt.boot.directory=D:/sbt-repository/root/</code>
<code>-Dsbt.ivy.home=D:/sbt-repository</code>
4. 配置maven仓库
默认下载包是从中央仓库下载，速度比较慢，可以配置阿里云或是私服地址，执行过sbt之后会在个人目录下生成一个.sbt的文件夹，例如我的在<code>C:\Users\Single\.sbt</code>下新建repositories文件
![](https://ws1.sinaimg.cn/large/6b162853ly1ft3lmqumv8j20k7044jrh.jpg)
> [repositories]
	local
   maven-local: file:////D:/repository/
   isuwang-public: http://10.4.1.2:8081/nexus/content/groups/guttv/
5. sbteclipse的配置
sbteclipse是eclipse的sbt插件，但与一般eclipse插件的配置及使用并不相同。
sbteclipse项目源码托管在github上：https://github.com/typesafehub/sbteclipse。
全局插件定义在<code>~/.sbt/1.0/plugins/plugins.sbt</code>文件中。
**【以上路径在Windows下默认存在于C盘中的用户文件夹中。】**
局部插件定义在具体项目文件夹下 project/folder/plugins.sbt文件中。
plugins.sbt文件中的内容如下：
```
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "5.2.4")
```
如果需要描述多个插件，则应保留不同设置语句之间一个空白行。
添加sbteclipse插件之后，在命令行中输入sbt，如果sbt已经启动则输入reload，等待系统自动解析并下载jar。此后，得到新的指令eclipse。

### Eclipse安装scala插件
在Eclipse Marketplace中搜索scala安装，或直接使用[scala-ide](http://scala-ide.org/)
![](https://ws1.sinaimg.cn/large/6b162853ly1ft3lygkqmkj20jp0om403.jpg)

### 创建eclipse sbt-scala项目
1. 在eclipse-workspace中新建文件夹
2. dos中进入到这个文件夹，输入命令<code>sbt</code>
3. 在sbt状态下输入<code>eclipse</code>初始化项目
 ![](https://ws1.sinaimg.cn/large/6b162853ly1ft3m9p5oigj20py0ax3yx.jpg)
 以及目录结构
 ![](https://ws1.sinaimg.cn/large/6b162853ly1ft3mawlshrj20iv03jglo.jpg)
4. eclipse中导入刚才创建的项目
 ![](https://ws1.sinaimg.cn/large/6b162853ly1ft3mccanq3j20du06u0sw.jpg)
5. 在项目根目录下创建<code>build.sbt</code>文件,参考示例添加了spark依赖和assembly解决一些冲突,这里基于spark2.2.1，scala2.11，jdk1.8

```scala
val spark_v = "2.2.1"
lazy val root = (project in file(".")).
  settings(
    inThisBuild(List(
   	  name := "xxx",//项目名称
      version := "1.0",
      organization    := "com.ecample",
      scalaVersion    := "2.11.11"
    )),
    name := "xxx",
    javacOptions++=Seq("-source","1.8","-target","1.8"),
    //添加依赖
    libraryDependencies ++= Seq(
      "org.apache.spark" %% "spark-core" % spark_v % "provided" exclude ("com.google.guava", "guava"),
	  "org.apache.spark" %% "spark-sql" % spark_v % "provided",
	  "org.apache.spark" %% "spark-hive" % spark_v % "provided",
	  "org.apache.spark" %% "spark-graphx" % spark_v,
	  //"org.apache.spark" %% "spark-streaming" % spark_v exclude ("com.google.guava", "guava"),
	  "org.apache.spark" %% "spark-sql-kafka-0-10" % "2.2.1" % "provided",
	  "org.apache.hbase" % "hbase-client" % "1.2.0",
	  "org.apache.hbase" % "hbase-server" % "1.2.0",
	  "org.apache.hbase" % "hbase-common" % "1.2.0"
    )
)

//打包合并冲突
assemblyMergeStrategy in assembly := {
  case PathList("javax", "inject", xs @ _*) => MergeStrategy.first
  case PathList("org", "apache", "commons", "beanutils", xs @ _*) => MergeStrategy.first
  case PathList("org", "apache", "commons", "collections", xs @ _*) => MergeStrategy.first
  case PathList("org", "apache", "spark", "unused", xs @ _*) => MergeStrategy.first
  case PathList("org", "aopalliance", xs @ _*) => MergeStrategy.first
  case PathList("org", "apache", "hadoop", "yarn", xs @ _*) => MergeStrategy.first
  case x =>
      val oldStrategy = (assemblyMergeStrategy in assembly).value
      oldStrategy(x)
}
```
**在project文件夹下创建<code>build.properties</code>、<code>plugins.sbt</code>**

**sbt的版本**

```
sbt.version=1.1.6
```
**sbt-assembly打包插件**
```
logLevel := Level.Warn

resolvers += Resolver.url("artifactory", url("http://scalasbt.artifactoryonline.com/scalasbt/sbt-plugin-releases"))(Resolver.ivyStylePatterns)

resolvers += "bintray-sbt-plugins" at "http://dl.bintray.com/sbt/sbt-plugin-releases"

addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "5.2.4")

addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.7")
```
6. 在项目上右键<code>build path</code>-><code>source</code>-><code>Add Floder</code>,并修改scala版本为2.11
![](https://ws1.sinaimg.cn/large/6b162853ly1ft3nti9xz0j20b10b14ln.jpg)
7. sbt中执行compile，使得项目关联到jar，最终得到的目录结构，类似于maven项目
![](https://ws1.sinaimg.cn/large/6b162853ly1ft3ng7wvmsj209u082glq.jpg)

### 编译打包
<code>sbt clean compile assembly</code>，详细使用参考官方文档：https://github.com/sbt/sbt-assembly

相关连接
scala官网：https://www.scala-lang.org/
scala-ide：http://scala-ide.org/
sbt：https://www.scala-sbt.org/
sbteclipse：https://github.com/typesafehub/sbteclipse
assembly：https://github.com/sbt/sbt-assembly

**End~~**