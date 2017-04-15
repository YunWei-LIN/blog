title: DDOS
date: 2015-11-17 11:34:57
tags: DDOS
categories: linux
---
## 参考是否受到 ddos
* sh
```
netstat -ntu | awk '{print $5}' | cut -d: -f1          | sort | uniq -C | sort -n
netstat -ntu | awk '{print $5}' | cut -d: -f4          | sort | uniq -C | sort -n
#             截取外网ip 端口号   | 截取外网ip 以：为分割符 | 排序  |排除相同记录|排序并统计
```
<!--more-->
* 测试模拟ddos
ab 命令 ： 做压力测试的工具和性能的监控工具
语法： ab -n 要产生的链接总数 -c 同时打开的客户端数量 http：//链接 

## 防止 DDOS
* 手动 iptables
* 自动检测，自动添加
fail2ban  或 linux ddos deflate（deflate.medialayer.com）

* 安装 deflate

可以 参照deflate ， 编写木马（自动下载，自动执行）
重要的是 计划任务查不到