---
title: linux下常用中间件搭建 - jboss部署(十二)
categories: [原创, 教程]
toc: true
date: 2019-08-30 12:13:40
tags: [linux, jboss]
---
## 概况
**JBoss版本：** 7.1.1  
**JBoss安装包：** jboss-as-7.1.1.Final.tar.gz
<!--more-->
**其他依赖环境：**  
> jdk环境（前面章节已安装）


**服务器（1台）**  
> A：192.168.0.1  
> B：192.168.0.2   
> 此章节未做集群，任选一台



## 步骤
### 解压


```bash
# 新建文件夹jboss（规划该目录下可能存在多个jboss应用）
mkdir /u01/jboss
# 解压
tar -zxvf /u01/setup/jboss-as-7.1.1.Final.tar.gz -C /u01/jboss
# 改名成demo1
mv /u01/jboss/jboss-as-7.1.2.Final /u01/jboss/demo1  
```
### 修改配置
视具体情况而定，我的项目中是这么配的
```bash
vim /u01/jboss/demo1/bin/standalone.conf
```
```bash
# 最后添加上这么一些配置
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.port=33024"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
JAVA_OPTS="$JAVA_OPTS -Djava.rmi.server.hostname=192.168.0.1"
JAVA_OPTS="$JAVA_OPTS -Xrunjdwp:transport=dt_socket,address=43024,server=y,suspend=n"
JAVA_OPTS="$JAVA_OPTS -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
JAVA_OPTS="$JAVA_OPTS -Djboss.modules.system.pkgs=org.jboss.byteman,org.jboss.logmanager"
JAVA_OPTS="$JAVA_OPTS -Xbootclasspath/p:/u01/jboss/demo1/bin/../modules/org/jboss/logmanager/main/jboss-logmanager-1.2.2.GA.jar"
JAVA_OPTS="$JAVA_OPTS -Xbootclasspath/p:/u01/jboss/demo1/bin/../modules/org/jboss/logmanager/log4j/main/jboss-logmanager-log4j-1.0.0.GA.jar"
JAVA_OPTS="$JAVA_OPTS -Xbootclasspath/p:/u01/jboss/demo1/bin/../modules/org/apache/log4j/main/log4j-1.2.16.jar"
JAVA_OPTS="$JAVA_OPTS -Dorg.jboss.logging.Logger.pluginClass=org.jboss.logging.logmanager.LoggerPluginImpl"
JAVA_OPTS="$JAVA_OPTS -Dlogging.configuration=file:/u01/jboss/demo1/bin/../standalone/configuration/logging.properties"
```
```bash
# 修改另外一个配置文件
vim /u01/jboss/demo1/standalone/configuration/standalone.xml
```
```xml
<!--配置从上到下-->

<!-- 增加配置 与extensions标签同级-->
<system-properties>
    <property name="org.apache.catalina.connector.URI_ENCODING" value="UTF-8"/>
    <property name="org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING" value="true"/>
</system-properties>

<!-- 修改1 -->
<local default-user="$local" allowed-users="*"/>
<!-- 改成 -->
<local default-user="$local"/>

<!-- 删除配置 -->
<authorization>
    <properties path="application-roles.properties" relative-to="jboss.server.config.dir"/>
</authorization>

<!-- 修改2 -->
<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000"/>
<!-- 改成 -->
<deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000" deployment-timeout="300"/>

<!-- 删除配置 -->
<spec-descriptor-property-replacement>false</spec-descriptor-property-replacement>
<jboss-descriptor-property-replacement>true</jboss-descriptor-property-replacement>

<!-- 修改3 -->
<core-threads count="50"/>
<queue-length count="50"/>
<max-threads count="50"/>
<!-- 改成 -->
<core-threads count="2000"/>
<queue-length count="2000"/>
<max-threads count="2000"/>

<!-- 修改4 -->
<login-module code="RealmDirect" flag="required">
    <module-option name="password-stacking" value="useFirstPass"/>
</login-module>
<!-- 改成 -->
<login-module code="RealmUsersRoles" flag="required">
    <module-option name="usersProperties" value="${jboss.server.config.dir}/application-users.properties"/>
    <module-option name="rolesProperties" value="${jboss.server.config.dir}/application-roles.properties"/>
    <module-option name="realm" value="ApplicationRealm"/>
    <module-option name="password-stacking" value="useFirstPass"/>
</login-module>

<!-- 修改5 -->
<virtual-server name="default-host" enable-welcome-root="true">
<!-- 改成 -->
<virtual-server name="default-host" enable-welcome-root="false">

<!-- 修改6 -->
<interfaces>
    <interface name="management">
        <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
    </interface>
    <interface name="public">
        <inet-address value="${jboss.bind.address:127.0.0.1}"/>
    </interface>
    <!-- TODO - only show this if the jacorb subsystem is added  -->
    <interface name="unsecure">
        <!--
          ~  Used for IIOP sockets in the standard configuration.
          ~                  To secure JacORB you need to setup SSL 
          -->
        <inet-address value="${jboss.bind.address.unsecure:127.0.0.1}"/>
    </interface>
</interfaces>
<!-- 改成 -->
<interfaces>
    <interface name="management">
        <any-address/>
    </interface>
    <interface name="public">
        <any-address/>
    </interface>
    <interface name="unsecure">
        <any-address/>
    </interface>
</interfaces>

<!-- 修改7 （修改偏移量，多应用需要配置）-->
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
<!-- 改成（偏移量的值只要端口不冲突即可） -->
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:1}">
```

### war包部署（部署方式二选一）
```bash
# 将war包放到指定目录下
mv demo1 /u01/jboss/demo1/standalone/deployments/
# 后台启动，并将启动日志保存到指定文件
nohup /u01/jboss/demo1/bin/standalone.sh >> /u01/jboss/demo1/standalone/log/web.log 2>&1 &
```
### 文件夹部署（部署方式二选一）
```bash
# 新建文件夹demo1.war
mkdir /u01/jboss/demo1/standalone/deployments/demo1.war
# 将准备部署的war包解压到上述文件夹下
unzip -q demo1.war -d /u01/jboss/demo1/standalone/deployments/demo1.war
# 建一个空的启动文件
touch /u01/jboss/demo1/standalone/deployments/demo1.war.dodeploy
# 启动
nohup /u01/jboss/demo1/bin/standalone.sh >> /u01/jboss/demo1/standalone/log/web.log 2>&1 &
```


## 总结
本文只涉及到我们生产上使用的配置，并未对各配置项做具体的说明。若配置中出现异常情况实属正常，google一下你就知道。
