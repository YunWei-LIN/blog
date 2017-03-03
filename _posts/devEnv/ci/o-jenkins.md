title: jenkins集合
date: 2015-07-29 10:14:56
tags: jenkins
categories: developEnv
---

## 安装
[官方指南](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Red+Hat+distributions)

### 前提
JDK1.7以上 安装

### RedHat系 YUM安装
centOS 7 安装稳定版本如下：

```
    sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
    sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
    sudo yum install jenkins

```

### 安装结果
* daemon ： /etc/init.d/jenkins 
* user ： jenkins （查看passwd，获得工作路径）
* Log file ： /var/log/jenkins/jenkins.log
* configuration ： /etc/sysconfig/jenkins  (建议修改JENKINS_HOME到空间较大的路径)
* 初始密码 ：查看 /var/log/jenkins/jenkins.log



### 启动/停止
```
  systemctl start/stop/restart jenkins
  systemctl enable jenkins ##开机自动启动
```

ps: 别忘了关闭防火墙或者设置防火墙




## 配置
### JDK
一般yum安装的 JAVA_HOME 为 `/usr/lib/jvm/java`

### maven
安装maven
yum安装的 M2_HOME 为 `/usr/share/maven`

### 插件
这些必须安装：

```
Git plugin
Git client plugin
GitLab Plugin
maven plugin
```

### Manage Jenkins --> Configure Global Security

### Manage Jenkins --> Configure System
配置 JDK， Jenkins Location， Git plugin， Gitlab


### ssh-key
如果连接 Source Code Management 采用 ssh 连接， 需要先生成 ssh-key

    ssh-keygen -t rsa -C "jenkins@jenkinsServer"
注意生成目录， jenkins ssh-key的默认目录为： `/var/lib/jenkins/.ssh/`




## New Item
mavne project

### root project
Maven project name
Description

Discard Old Builds
Source Code Management

Build
	Root POM：pom.xml
	Goals and options： clean deploy -N

	
### sub project
其他 参照上方， 
Post Steps --> Excute shell (启动tomcat等Web server)
shell可能长的像下面这样
```sh
#!/bin/bash -ex
JAVA_OPTS="-Xms768m -Xmx1000m -XX:PermSize=512m -XX:MaxPermSize=1024m"
cd /data/apache-tomcat/
bin/catalina.sh stop 35 -force
rm -rf /data/apache-tomcat/webapps/xxxxxxxx.war /data/apache-tomcat/webapps/xxxxxxxx
cp $WORKSPACE/target/xxxxxxxx.war /data/apache-tomcat/webapps
## required, or tomcat will be killed by jenkins. The value can be set to anything.
BUILD_ID=please_do_not_kill_me 
sleep 20s
bin/catalina.sh start
```


	

## 项目权限
[项目权限](/2017/03/03/devEnv/ci/jenkinsRole/)




## 问题
### 第一次ssh连接 gitServer
Source Code Management 中 git 第一次ssh连接， 会有 `host error` 
解决方法： 
  * jenkins 用户登录， `/etc/passwd` 中 jenkins用户的默认shell 需要修改成 `bin/bash`
  * 执行命令 `ssh git@gitServerIP`, 接受yes
  * 还原 jenkins用户的默认shell为 `bin/false`

### 网络导致日志疯狂增长
一旦jenkins所在的服务器的网络出现问题， jenkins的日志文件（/var/log/jenkins/jenkins.log）大小会疯狂快速增长，直到撑爆硬盘
日志内容大概为

``` bash
Jul 29, 2015 5:04:59 AM javax.jmdns.impl.constants.DNSRecordClass classForIndex
WARNING: Could not find record class for index: -1
Jul 29, 2015 5:04:59 AM javax.jmdns.impl.DNSIncoming$MessageInputStream readName
SEVERE: bad domain name: possible circular name detected. Bad offset: 0xffffffff at 0xb6
Jul 29, 2015 5:04:59 AM javax.jmdns.impl.constants.DNSRecordType typeForIndex
SEVERE: Could not find record type for index: -1
Jul 29, 2015 5:04:59 AM javax.jmdns.impl.DNSIncoming readQuestion
SEVERE: Could not find record type: dns[query,108.61.99.169:53227, length=184, id=0x5c78, flags=0x3030, questions=6
questions:
        [DNSQuestion@84075318 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: 0\x00\x01\x00\x00\x00\x00\x00\x00\x09\x5F\x73\x6.\x72\x76\x69\x63\x65\x73\x07\x5F\x64\x6E\x73\x2D\x73\.4\x04\x5F\x75\x64\x70\x05\x6C\x6F\x63\x61\x6C\x00\x00\.C\x00\x01ϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿ.]
        [DNSQuestion@9629900 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        [DNSQuestion@788455775 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        [DNSQuestion@1602108435 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        [DNSQuestion@1486500959 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        [DNSQuestion@1944352362 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]]
        question:      [DNSQuestion@84075318 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: 0\x00\x01\x00\x00\x00\x00\x00\x00\x09\x5F\x73\x6.\x72\x76\x69\x63\x65\x73\x07\x5F\x64\x6E\x73\x2D\x73\.4\x04\x5F\x75\x64\x70\x05\x6C\x6F\x63\x61\x6C\x00\x00\.C\x00\x01ϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿϿ.]
        question:      [DNSQuestion@9629900 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        question:      [DNSQuestion@788455775 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        question:      [DNSQuestion@1602108435 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        question:      [DNSQuestion@1486500959 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
        question:      [DNSQuestion@1944352362 type: TYPE_IGNORE index 0, class: CLASS_UNKNOWN index 0, name: ]
   0: 5c7830305c783030 5c7830305c783030 5c7830305c783031 5c7830305c783030     \x00\x00 \x00\x00 \x00\x01 \x00\x00
  20: 5c7830305c783030 5c7830305c783030 5c7830395c783546 5c7837335c783635     \x00\x00 \x00\x00 \x09\x5F \x73\x65
  40: 5c7837325c783736 5c7836395c783633 5c7836355c783733 5c7830375c783546     \x72\x76 \x69\x63 \x65\x73 \x07\x5F
  60: 5c7836345c783645 5c7837335c783244 5c7837335c783634 5c7830345c783546     \x64\x6E \x73\x2D \x73\x64 \x04\x5F
  80: 5c7837355c783634 5c7837305c783035 5c7836435c783646 5c7836335c783631     \x75\x64 \x70\x05 \x6C\x6F \x63\x61
  a0: 5c7836435c783030 5c7830305c783043 5c7830305c783031                      \x6C\x00 \x00\x0C \x00\x01
```

jenkins官方的issue，目前尚未fixed
``` bash
https://issues.jenkins-ci.org/browse/JENKINS-10160
```

临时解决方案：
修改jenkins的java启动参数，编辑 /etc/sysconfig/jenkins， 看看效果
``` bash
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dhudson.DNSMultiCast.disabled=true -Dhudson.udp=-1"
```
