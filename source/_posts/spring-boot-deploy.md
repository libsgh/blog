---
title: spring boot应用启动打包部署
categories: spring boot
date: 2018-03-22 08:00:00
tags: [java,spring boot]
comments: true
---
## 打包

 * <code>mvn clean package -Dmaven.test.skip=true</code>
 > 生成jar或是war包只需修改pom.xml<packaging>war</packaging>
  
<!-- more -->
 
## 内嵌容器

### war、jar包直接启动

 * 以<code>java -jar</code>的方式启动项目 <br>
 <code>--spring.config.location=xxx.yml</code> 指定外部配置文件路径 <br>
 <code>--server.port=9999</code> 指定端口 <br>
 例：<code>nohup $JAVA -jar $JVM_OPTS $JAR_FILE --spring.config.location=$APP_CONF --server.port=$port  > $LOG_PATH 2>&1 &</code> <br>
 <code>$JVM_OPTS</code>指定jvm参数 <br>
 <code>$JAVA</code> 指定jdk <br>
 
** 参考脚本 **

```Shell
#!/bin/bash
# description: Starts and stops the App.
# author:libs
# ./boot.sh start/stop/restart/status oms_console 1234

APP_HOME=/home/libosong
LOG_HOME=$APP_HOME/logs
APP_NAME=$2
port=$3
APP_CONF=$APP_HOME/application.properties
JAVA_HOME=/opt/guttv/docker_images/tomcat_8.5/jdk1.8.0_121
JAVA=$JAVA_HOME/bin/java

dirname $0|grep "^/" >/dev/null
if [ $? -eq 0 ];then
   APP_HOME=`dirname $0`
else
    dirname $0|grep "^\." >/dev/null
    retval=$?
    if [ $retval -eq 0 ];then
        APP_HOME=`dirname $0|sed "s#^.#$APP_HOME#"`
    else
        APP_HOME=`dirname $0|sed "s#^#$APP_HOME/#"`
    fi
fi

if [ ! -d "$LOG_HOME" ];then
  mkdir $LOG_HOME
fi
#JMX监控需用到
JMX="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1091 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
#JVM参数
JVM_OPTS="-Dname=$APP_NAME -Djava.ext.dirs=$JAVA_HOME/lib -Duser.timezone=Asia/Shanghai -Dlogging.path=$LOG_HOME -Djava.security.egd=file:/dev/./urandom -Xms256M -Xmx256M"

JAR_FILE=$APP_HOME/$APP_NAME.war
pid=0
start(){
  checkpid
  if [ ! -n "$pid" ]; then
    $JAVA -jar $JVM_OPTS $JAR_FILE --spring.config.location=$APP_CONF --server.port=$port --logging.path=$LOG_HOME  >/dev/null 2>&1 &
    checkpid
    echo "---------------------------------"
    echo "启动完成,PID:$pid>>>>>"
    echo "---------------------------------"
    #sleep 2s
    #tail -f $LOG_PATH
  else
      echo "$APP_NAME is runing PID: $pid"   
  fi

}


status(){
   checkpid
   if [ ! -n "$pid" ]; then
     echo "$APP_NAME not runing"
   else
     echo "$APP_NAME runing PID: $pid"
   fi 
}

checkpid(){
    pid=`ps -ef |grep $JAR_FILE |grep -v grep |awk '{print $2}'`
}

stop(){
    checkpid
    if [ ! -n "$pid" ]; then
     echo "$APP_NAME not runing"
    else
      echo "$APP_NAME stop..."
      kill -9 $pid
    fi 
}

restart(){
    stop 
    sleep 5s
    start
}

case $1 in  
          start) start;;  
          stop)  stop;; 
          restart)  restart;;  
          status)  status;;   
              *)  echo "require start|stop|restart|status"  ;;  
esac
```

### Docker中启动

** DockerFile **

```
FROM frolvlad/alpine-oraclejdk8:slim
MAINTAINER Libs
WORKDIR /workdir
ENV LANG en_US.UTF-8    
ENV LANGUAGE en_US:en    
ENV LC_ALL en_US.UTF-8 
ENTRYPOINT [ "sh", "-c", "java -Djava.security.egd=file:/dev/./urandom -Xms256M -Xmx256M -jar $war_name.war --spring.config.location=$config_location --server.port=$port"]
```

** 启动脚本 **

```Shell
docker run -itd --ulimit nofile=20480:40960 --name $docker_name --net=host \
-e TZ="Asia/Shanghai" \
-e war_name=$project_name \
-e config_location=/opt/application.properties \
-e port=$port \
-v $logs_path:/var/guttv/logs \
-v $root_path:/workdir \
-v /opt/guttv/dev/share:/data/share \
-v $config_path:/opt/application.properties \
 spring/boot:slim
```

### Linux服务的方式启动、停止、重启

** pom.xml配置插件 **

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
            </configuration>
        </plugin>
    </plugins>
</build>
```

** 将jar、war软连接到 /etc/init.d 目录 **

```
ln -s /var/apps/myapp.jar /etc/init.d/myapp
```
** 给jar、war设置执行权限 **

```
chmod +x myapp.jar
```
** 以服务方式重启、停用<code>service myapp start|stop|restart|status</code> **
***

** 如果你是CentOS 7或红帽7以上，你还可以用下面的方法处理 ** 
<code>vim /usr/lib/systemd/system/myapp.service</code>

```
[Unit]
Description=myapp
After=network.target

[Service]
WorkingDirectory=/var/apps/myapp
ExecStart=/usr/local/java/bin/java -Dsun.misc.URLClassPath.disableJarChecking=true -jar /var/apps/myapp.jar
ExecStop=kill $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target
```

**使用Linux 7 以后服务新的启动方式，相关命令 ** 

```
启动
systemctl start myapp
停止
systemctl stop myapp
重启
systemctl restart myapp
查看日志
journalctl -u myapp
关于更多 systemctl 命令的使用方法，度娘有很多。
```

>以上均可以通过jenkins配置持续集成，下面是一个示例

 * 源码管理用gitlab
![](https://ws1.sinaimg.cn/large/6b162853ly1fpllzhbefoj213x0o5ta5.jpg)
 * maven构建
![](https://ws1.sinaimg.cn/large/6b162853ly1fpllzp5rjgj213y0g0aaw.jpg)
 * 远程部署到服务器
![](https://ws1.sinaimg.cn/large/6b162853ly1fpllzukrv7j212w0nmwg1.jpg)

## 外部容器

** 排除tomcat插件 **

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**修改启动类，继承SpringBootServletInitializer，重写configure方法 **

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer{
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(Application.class);
	}
	
	public static void main(String[] args) throws UnknownHostException {
		SpringApplication application = new SpringApplication(Application.class);
		application.run(args);
	}
}
```

>这里需要注意不能设置打包插件executable设置为true,否则jar包将无法在tomcat中解压

部分参考：http://blog.csdn.net/catoop/article/details/50588851

**End~**