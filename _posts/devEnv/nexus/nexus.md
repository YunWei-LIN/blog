title: Sonatype nexus
date: 2016-06-20 15:22:59
tags: nexus
categories: developEnv
---

永久免费的 Sonatype Nexus Repository Manager OSS版本，
可以帮助你与其他开发人员和最终用户共享组件。它大大简化了内部存储库的维护和访问外部存储库，可以从单一地点完全控制访问和部署每个组件。
主要配合maven 和 gradle 工作。   

<!-- more -->

## 安装
前提要JRE7以上

### [下载](http://www.sonatype.com/download-oss-sonatype/)
配合maven使用， 下载 `Nexus Repository Manager OSS 2.xx`

### [安装](http://books.sonatype.com/nexus-book/reference/installing.html)
主要步骤像这样滴

* 文件

```
$ sudo cp nexus-2.13.0-02-bundle.tar.gz /usr/local
$ cd /usr/local
$ sudo tar xvzf nexus-2.13.0-02-bundle.tar.gz
$ sudo ln -s nexus-2.13.0-02 nexus
```

`/usr/local` 这个安装路径根据自己的实际情况调整。

* 环境变量

```
vim /etc/profile
NEXUS_HOME=/usr/local/nexus
export NEXUS_HOME

source /etc/profile
```

* sonatype-work

安装同级目录有`sonatype-work`文件夹，用来保存 repository and configuration data；和安装文件分离，方便升级。


* [系统服务](http://books.sonatype.com/nexus-book/reference/install-sect-service.html)

增加用户 nexus  : useradd nexus
修改nexus的权限 chown -R nexus.nexus nexus



## 配置和管理
* 配置文件
`$NEXUS_HOME/conf/nexus.properties`， 大概像这样的

```
  #
  # Sonatype Nexus (TM) Open Source Version
  # Copyright (c) 2007-2014 Sonatype, Inc.
  # All rights reserved. Includes the third-party code listed at http://links.sonatype.com/products/nexus/oss/attributions.
  #
  # This program and the accompanying materials are made available under the terms of the Eclipse Public License Version 1.0,
  # which accompanies this distribution and is available at http://www.eclipse.org/legal/epl-v10.html.
  #
  # Sonatype Nexus (TM) Professional Version is available from Sonatype, Inc. "Sonatype" and "Sonatype Nexus" are trademarks
  # of Sonatype, Inc. Apache Maven is a trademark of the Apache Software Foundation. M2eclipse is a trademark of the
  # Eclipse Foundation. All other trademarks are the property of their respective owners.
  #

  # Sonatype Nexus
  # ==============
  # This is the most basic configuration of Nexus.

  # Jetty section
  application-port=8081
  application-host=0.0.0.0
  nexus-webapp=${bundleBasedir}/nexus
  nexus-webapp-context-path=/nexus

  # Nexus section
  nexus-work=${bundleBasedir}/../sonatype-work/nexus
  runtime=${bundleBasedir}/nexus/WEB-INF
```

* 启动

```
  su nexus
  cd /data/iho/nexus/bin/
  ./nexus start
```

## 使用
[官方文档](http://books.sonatype.com/nexus-book/reference/config.html)

### 后台管理
* 地址账户
  地址    ：端口和context 参照 你自己配置的 `$NEXUS_HOME/conf/nexus.properties`
  初始账户 ：`admin`
  初始密码 ：`admin123`

* Repository
  [参照](/2015/10/15/devEnv/nexus/restlet/)

### 客户端
[参照Maven配置]()
