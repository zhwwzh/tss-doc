Installation and configuration tomcat v7.0 on CentOS7
=====================================================

## Install

### Install java

``` text
# cd /tmp/soft
# tar -zxvf jdk1.7.0_80.tar.gz
# mv jdk1.7.0_80 /usr/local

# vim /etc/profile
…
export JAVA_HOME=/usr/local/jdk1.7.0_80
export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
…

# source /etc/profile
# java -version
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)
```

### Install tomcat

``` text
# cd /tmp/soft
# tar -zxvf apache-tomcat-7.0.103.tar.gz

# mkdir -p /usr/local/tomcat1
# mv apache-tomcat-7.0.103 /usr/local/tomcat1/
```

## Configure

### user configuration

``` text
# vim /usr/local/tomcat01/apache-tomcat-7.0.103/conf/tomcat-users.xml
...
  <role rolename="manager-gui"/>
  <user username="tomcatadmin" password="passw0rd" roles="manager-gui"/>
...
```

用户名/密码 -> tomcatadmin/passw0rd

Tomcat7，admin变成了admin角色转为admin-gui和admin-script两个角色：

* admin-gui: 允许访问HTML GUI，可以避免CSRF攻击
* admin-script: 允许访问文本接口

manager角色变更为下面的4个角色：

* manager-gui: 允许访问HTML GUI和状态页
* manager-script: 允许访问文本接口和状态页
* manager-jmx: 允许访问JMX代理和状态页
* manager-status: 仅允许访问状态页

## Deploy

``` text
# mkdir -p /usr/local/tomcat1/apache-tomcat-7.0.103/mywebapps
# mkdir -p /usr/local/tomcat1/apache-tomcat-7.0.103/mywebapps/mytomcat-helloworld

# mv mytomcat-helloworld.war /usr/local/tomcat1/apache-tomcat-7.0.103/mywebapps/mytomcat-helloworld

# cd /usr/local/tomcat1/apache-tomcat-7.0.103/mywebapps/mytomcat-helloworld
# jar xvf mytomcat-helloworld.war

# vim /usr/local/tomcat1/apache-tomcat-7.0.103/conf/server.xml
...
<Context path="/mytomcat-helloworld" docBase="usr/local/tomcat1/apache-tomcat-7.0.103/mywebapps/mytomcat-helloworld" debug="0" reloadable="true"></Context>
...

# systemctl restart tomcat1.service; systemctl status tomcat1.service
```

## Maintain

### Create tomcat service using systemctl

``` text
# vim /usr/local/tomcat1/apache-tomcat-7.0.103/bin/catalina.sh
...
# setting tomcat.pid
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
...

# touch /usr/lib/systemd/system/tomcat1.service
# chmod 644 /usr/lib/systemd/system/tomcat1.service
# vim /usr/lib/systemd/system/tomcat1.service
...
[Unit]
Description=Tomcat1
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking

Environment="JAVA_HOME=/usr/local/jdk1.7.0_80"

PIDFile=/usr/local/tomcat1/apache-tomcat-7.0.103/tomcat.pid
ExecStart=/usr/local/tomcat1/apache-tomcat-7.0.103/bin/startup.sh
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
...

# systemctl enable tomcat1.service
Created symlink from /etc/systemd/system/multi-user.target.wants/tomcat1.service to /usr/lib/systemd/system/tomcat1.service.
```

### Start foreground

``` text
# /usr/local/tomcat1/apache-tomcat-7.0.103/bin/catalina.sh run
```

### Start

``` text
# systemctl start tomcat1.service
```

### Stop

``` text
# systemctl stop tomcat1.service
```

### Check status

``` text
# systemctl status tomcat1.service
● tomcat1.service - Tomcat1
   Loaded: loaded (/usr/lib/systemd/system/tomcat1.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2020-03-28 12:32:27 CST; 5s ago
  Process: 2821 ExecStart=/usr/local/tomcat1/apache-tomcat-7.0.103/bin/startup.sh (code=exited, status=0/SUCCESS)
 Main PID: 2829 (java)
    Tasks: 30
   CGroup: /system.slice/tomcat1.service
           └─2829 /usr/local/jdk1.7.0_80/bin/java -Djava.util.logging.config.file=/usr/local/tomcat1/apache-tomcat-7.0.103/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemer...
```

### Check version

``` text
# /usr/local/tomcat2/apache-tomcat-7.0.103/bin/version.sh 
Using CATALINA_BASE:   /usr/local/tomcat2/apache-tomcat-7.0.103
Using CATALINA_HOME:   /usr/local/tomcat2/apache-tomcat-7.0.103
Using CATALINA_TMPDIR: /usr/local/tomcat2/apache-tomcat-7.0.103/temp
Using JRE_HOME:        /usr/local/jdk1.7.0_80
Using CLASSPATH:       /usr/local/tomcat2/apache-tomcat-7.0.103/bin/bootstrap.jar:/usr/local/tomcat2/apache-tomcat-7.0.103/bin/tomcat-juli.jar
Using CATALINA_PID:    /usr/local/tomcat2/apache-tomcat-7.0.103/tomcat.pid
Server version: Apache Tomcat/7.0.103
Server built:   Mar 16 2020 08:34:15 UTC
Server number:  7.0.103.0
OS Name:        Linux
OS Version:     3.10.0-1062.el7.x86_64
Architecture:   amd64
JVM Version:    1.7.0_80-b15
JVM Vendor:     Oracle Corporation
```
