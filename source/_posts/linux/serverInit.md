title: linux server 配置选项
date: 2016-1-26 19:29:44
tags: linux server
categories: linux
---

主要内容
* 环境准备
* 用户管理
* sshd管理
* 防暴力破解

......<!-- more -->

## 环境准备
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
```

## 用户管理
* 增加一root组用户
* 禁止root登录

## sshd管理
* 修改服务端口
* 公钥登录
* 禁止密码登录

## 防暴力破解
使用 `fail2ban` 防止暴力破解， 将对方IP放入防火墙.











