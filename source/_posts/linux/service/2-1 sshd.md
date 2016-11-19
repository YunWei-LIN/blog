title: 2-1 sshd
date: 2016-1-26 16:29:44
tags: [linux, sshd]
categories: linux
---

主要内容
* 安装
* 配置
* 公钥登录

<!-- more -->

## 公钥登录

### 客户端生成公钥密钥对
`ssh-keygen -t rsa -C "your_email@your_email.com"`

```
[root@server ~]# ssh-keygen -b 1024 -t rsa
Generating public/private rsa key pair.     #提示正在生成rsa密钥对
Enter file in which to save the key (~/.ssh/id_dsa):     #询问公钥和私钥存放的位置，回车用默认位置即可
Enter passphrase (empty for no passphrase):     #询问输入私钥密语，输入密语
Enter same passphrase again:     #再次提示输入密语确认
Your identification has been saved in ~/.ssh/id_dsa.     #提示公钥和私钥已经存放在/root/.ssh/目录下
Your public key has been saved in ~/.ssh/id_dsa.pub.
The key fingerprint is:
x6:68:xx:93:98:8x:87:95:7x:2x:4x:x9:81:xx:56:94 root@server     #提示key的指纹
```
注意保护好 `私钥密语`


### 上传公钥
公钥放到用户目录的 .ssh 这个目录下（如果目录不存在，需要创建~/.ssh目录，并把目录权限设置为700）。

```
$ mkdir ~/.ssh     #如果当前用户目录下没有 .ssh 目录，请先创建目录
$ chmod 700 ~/.ssh
$ 
$ #客户端上传公钥
$ 
$ cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
$ rm -f ~/.ssh/id_rsa.pub
$ chmod 600 ~/.ssh/*
``` 

### 关闭密码认证
` vim /etc/ssh/sshd_config`

`PasswordAuthentication yes`
-->
`PasswordAuthentication no`

`systemctl restart sshd`



