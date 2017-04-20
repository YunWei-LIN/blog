title: linux server 配置选项
date: 2016-11-18 19:29:44
tags: [linux, server]
categories: server
---

主要内容
* 环境准备
* 用户管理
* sshd管理
* 防暴力破解

主要以Centos为主。

......<!-- more -->

## 安装Centos
### U盘安装。
`df -l` U盘挂载设备路径 
`umount sdb1` 
`dd if=iso镜像文件 of=U盘挂载设备名称sdx (一定不要加上数字) bs=8M `

恢复
`umount U盘挂载设备路径`
`mkfs.ntfs U盘挂载设备路径`  (mkfs.vfat)

### hostname
为了方便管理，可以设置服务器的机器名称

		hostnamectl set-hostname `your hostname`
		reboot

修改 `/etc/hosts` 将 `your hostname` 写入到 ipv4 和 ipv6


## 环境准备

### yum源
[阿里CentOS 镜像](http://mirrors.aliyun.com/help/centos)

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 

yum makecache #生成缓存
```

### 科学上网
[SS](/2016/05/05/other/shadowsocks)


### git (git-flow)
		yum -y install git

[git的后悔药](/2016/05/03/devEnv/git/git/)

`git-flow` 看情况安装。


### zsh
```
  echo $SHELL  #查看系统当前的shell

  yum install zsh
  
  cat /etc/shells #查看本地所有的shell
  
  chsh -s /bin/zsh #切换到zsh
```

###  oh-my-zsh
```
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
  
#修改主题，非必须
vim ~/.zshrc
ZSH_THEME="bira"

#插件
plugins=(git git-flow git-flow-completion )
```

## 用户管理
* 增加一root组用户
* 禁止root登录

## sshd管理
* 公钥登录
* 修改服务端口
* 禁止密码登录
[ssh](/2016/01/26/linux/service/2-1%20sshd/)

## firewall
[firewall](http://giveme5.cc/2017/03/28/linux/Firewalld-md/)


## 防暴力破解
使用 `fail2ban` 防止暴力破解， 将对方IP放入防火墙.
[fail2ban](http://giveme5.cc/2017/03/28/linux/fail2ban-md/)

## VNC
[VNC](/2016/10/10/linux/vnc/)

## Sonatype nexus
[nexus](/)

## gitlab
[gitlab](/tags/gitlab/)


## CI
[jenkins](/tags/jenkins/)


## [nginx](/2016/01/22/web server/nginx/nginx/)


## [mysql](/categories/mysql/)




