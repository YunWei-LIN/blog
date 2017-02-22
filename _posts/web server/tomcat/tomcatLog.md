title: tomcat  日志分割
date: 2017-02-21 17:07:07
tags: 
 - tomcat
 - cronolog
categories: server
---

主要内容
tomcat的catalina.out会不断增长，很麻烦，看了多张解决方案，包括官方的，决定采用cronolog的方案。

<!-- more -->

## cronolog

### 环境
```
# cat /etc/redhat-release
CentOS release 5.4 (Final)
```

### 安装yum源

```
# mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup 
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

### 安装cronolog
```
# yum install -y cronolog
```


查看cronolog安装后所在目录（验证安装是否成功）

```
# which cronolog
正常情况下显示：
/usr/sbin/cronolog
```

## Tomcat7以后的版本配置
`bin/catalina.sh`

###  第一步
大概213行
将

```
if [ -z "$CATALINA_OUT" ] ; then

CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out

fi
```

修改为

```
if [ -z "$CATALINA_OUT" ] ; then

CATALINA_OUT="$CATALINA_BASE"/logs/catalina.%Y-%m-%d.out

fi
```

### 第二步 
大概414行左右
将

```
touch "$CATALINA_OUT"
```

改为
```
#touch "$CATALINA_OUT"
```

### 第三步
大概 436～437
将

```
org.apache.catalina.startup.Bootstrap "$@" start \
>> "$CATALINA_OUT"  2>&1 "&"
```

修改为

```
org.apache.catalina.startup.Bootstrap "$@" start 2>&1 \
| /usr/sbin/cronolog "$CATALINA_OUT" >> /dev/null &
```
