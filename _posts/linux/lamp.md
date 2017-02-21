title: lamp
date: 2015-11-17 11:34:57
tags: lamp
categories: linux
---
## 安装

## 调优
<!--more-->
### 屏蔽 apache/nginx 的版本
* 查看web 服务器的类型和版本
```
curl -I ip/domain
```

* 应该隐藏 服务器的类型和版本
apache 主配置文件  httpd-default.conf 打开注释

serverTokens prod
serverSignature off

* 提升：让版本彻底消失
需要重新编译源码， 修改incloude目录下的ap_release.h 文件
-- prefix 
-- enable-so
-- enable-rewrite
-- enable-ssl

### 启用压缩模块 mod_deflate