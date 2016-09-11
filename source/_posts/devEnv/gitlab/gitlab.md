title: gitlab
date: 2016-06-20 13:46:15
tags: gitlab
categories: developEnv
---
gitlab 一个开源的版本管理系统,实现一个自托管的Git项目仓库,可通过Web界面进行访问公开的或者私人项目。基本拥有GitHub类似的功能。

<!-- more -->

## 安装
[免费开源的CE版本](https://about.gitlab.com/downloads/)
请选择需要的版本， 这里有个 [大中国专用的mirror](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)

然后按照提示执行就ok。

初始账户： `root`
password： `5iveL!fe`

## 管理和配置
[官方文档](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md)


* 配置

GitLab 7.6 以后， 默认的配置文件 `/etc/gitlab/gitlab.rb`， 修改后，使用 `sudo gitlab-ctl reconfigure` 使配置生效并启动gitlab。

配置文件的内容有点像这样：
```
  # Disable the built-in nginx
  #nginx['enable'] = false
  #web_server['external_users'] = ['nginx']


  #
  # # Disable the built-in unicorn
  #unicorn['enable'] = false


  # Check and change the external_url to the address your users will type in their browser
  external_url 'http://IP:70'

  #port
  #redis['port'] = 7001
  #postgresql['port'] = 7002
  unicorn['port'] = 7004

  #git data path
  git_data_dir "/data/git-data"
  gitlab_rails['backup_path'] = '/data/git-data-backups'

  # limit backup lifetime to 7 days - 604800 seconds
  gitlab_rails['backup_keep_time'] = 604800
```
重点关注 `external_url` `port` `git data path`。



* 管理命令
  启动所有gitlab组件：
      gitlab-ctl start
    
  停止所有gitlab组件：
      gitlab-ctl stop
    
  重启所有gitlab组件：
      gitlab-ctl restart

## 使用

### 管理员
如图可以进入admin特别区域。多用于管理 `group`， `project`， `user`， 更高级的话管理 `job` 和 `hook`
![](http://amadis.qiniudn.com/gitlab1.png)

### 普通用户
* SSH KEY
统一使用ssh协议类操作git repository， 所以登陆以后，请将自己的public key添加到gitlab账户。

![](http://amadis.qiniudn.com/gitlab2.png)
![](http://amadis.qiniudn.com/gitlab3.png)
![](http://amadis.qiniudn.com/gitlab4.png)

ssh key 可由管理员方式发送到各位，
包括2个文件，一个是.pub结尾的，这个就是public key，另一个无后缀名的，这是各位的private key，请妥善保管，以后将凭这个文件来获得对git服务器的访问权。

private key需要存放在办公电脑路径：
Linux ： `/home/你的用户/.ssh`
Windows: `C:\Users\你的用户\.ssh`

如需自己生成：`ssh-keygen -t rsa -C "your_email@your_email.com"`

* 操作
每个人在自己的dashborad里可以看到自己加入的project
有权限的人，可以自己创建project

![](http://amadis.qiniudn.com/gitlab5.png)


版本操作完全是git的内容。Linux的话直接命令行就很好了； Windows的话 推荐 [SourceTree](https://www.sourcetreeapp.com/) or [smartgit](http://www.syntevo.com/smartgithg/)