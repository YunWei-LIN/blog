title: 1-8-RHEL7软件包管理
date: 2015-12-6 15:29:44
tags: GOD linux
categories: linux
---

## 主要内容 
* rpm管理软件
* yun管理软件
* 源码管理软件


## rpm管理软件
rpm （Redhat   package  manager）
* 作用：管理rpm包
* 语法：rpm  【选项】  包名
  参数
  -i, --install                    install package(s)
  -v, --verbose 详细             provide more detailed output
  -h, --hash                     print hash marks as packageinstalls (good with -v) #安装时打印散列标记#号
  -q， --query                   查询选项
  -U, --upgrade=<packagefile>+      升级软件包
  -e, --erase=<package>+            清除 (卸载) 软件包


### rpm包
* 来源
独立下载或光盘镜像

```bash
#挂载光盘：
[root@localhost ~]# umount /dev/sr0
[root@localhost ~]# mount /dev/sr0 /mnt/
```

* 包名意义
zsh-5.0.2-7.el7.x86_64.rpm

|zsh    |-5| .0|.2 |  -7 | x86  | 64|
|--|--|--|--|--|--|--|
|软件名 |主版本号 |此版本号 |修订号 | release（第几次发布版本） | CPU架构系统平台 | 支持的系统位数|


### rpm安装
    [root@localhost ~]# rpm -ivh /mnt/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm

安装时，如果需要解决依赖关系。上rpm包相关的网站上找
http://rpmfind.net/
http://rpm.pbone.net/
http://www.rpmseek.com/index.html


### rpm查询
* 查询zsh软件是否安装
      [root@localhost ~]# rpm -q zsh
      zsh-5.0.2-7.el7.x86_64

* 查询系统所有安装过的rpm软件
      [root@localhost ~]# rpm -qa
	
      [root@localhost ~]# rpm -qa | grep zsh
      zsh-5.0.2-7.el7.x86_64

* 查询zsh这个软件安装后，产生了那些文件和目录
      [root@localhost ~]# rpm -ql zsh | more

* zsh这个文件是那个软件安装的
      [root@localhost ~]# rpm -qf  `which zsh` 
      zsh-5.0.2-7.el7.x86_64

* 在软件没有安装之前进行查看
```
[root@localhost ~]# rpm -qpl /mnt/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm
      warning:/mnt/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature,key ID fd431d51: NOKEY
      /usr/bin/rb
      /usr/bin/rx
```

查看一个包的作用，凡是加上p则表示查询的包未安装。若查询已安装的包则去掉p

### rpm升级
    [root@localhost ~]# rpm-Uvh /mnt/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm

### rpm卸载
    [root@localhost ~]# rpm -e zsh                      只写软件包的名字，不用写版本号

### rpm签名验证
导入RPM-GPG-KEY安装rpm包时，对rpm的签名进行验证。验证的原理是：非对称加密。 导入公钥。验证rpm中的签名是否是对的。

    [root@localhost ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release 





## YUM管理软件
YUM，自动装软件包（软件包管理），解决依赖关系问题、自动下载软件包。基于C/S架构。

### YUM源
YUM 基于C/S架构， 软件需要来源，即 YUM源。 可以分为  `ftp    http   file（本地yum源）` 三类。

例如 ： 配置本地yum源
在 `/etc/yum.repos.d/` 目录下新建 `xxxx.repo` 文件
```bash
[root@localhost ~]# vim /etc/yum.repos.d/rhel7.repo
[rhel7-yum]                  #yum源名称，唯一的，用来区分不同的yum源
name=rhel7-source            #对yum源描述信息
baseurl=file:///mnt          #yum源的路径（repodata目录所在的目录）
enabled=1                    #为1，表示启用yum源
gpgcheck=0                   #为1，使用公钥检验rpm的正确性
```

### YUM使用
* yum安装

      [root@localhost ~]# yum install zsh

  选项 ：-y  #回答yes  取消交互

      [root@localhost ~]# yum -y install mariadb-server


* yum安装一组软件包

      [root@localhost ~]# yum group list
      [root@localhost ~]# yum group install "Security Tools"

 

* yum查询

      [root@localhost ~]# yum list z*
      [root@localhost ~]# yum search zsh

* yum删除

      [root@localhost ~]# yum remove  zsh

* 清空yum缓存

      [root@localhost ~]# yum clean all

* yum生成列表

      [root@localhost ~]# yum list




## 源码管理软件
### make
前提：系统必须安装：开发工具、开发库

步骤：
1. 获得源码包
2. 解压
   tar
3. 配置，检测安装环境
  ./configure --prefix=path  #检查安装环境是否符合需求，如果没有问题，生成：Makefile文件，  --prefix 参数 指定安装路径。这样删除或备份时，直接对删除这个目录操作就可以了。
4. 编译
  make -j n (编译时会读取Makefile文件) n 为电脑cpu核数 -2 ， 并发编译
5. 安装
  make install

6. 删除
  make uninstall

7. 再次编译
  make clean

例：安装软件包extundelete-0.2.4.tar.bz2
```bash
[root@localhost ~]# tar jxvf extundelete-0.2.4.tar.bz2
  
[root@localhost ~]# cd extundelete-0.2.4/
  
#指定安装路径
[root@localhostextundelete-0.2.4]# ./configure --prefix=/usr/local/extundelet
Configuring extundelete0.2.4
configure: error: Can'tfind ext2fs library
  
[root@localhostextundelete-0.2.4]# yum -y install e2fsprogs-devel
  
[root@localhostextundelete-0.2.4]# ./configure --prefix=/usr/local/extundelet
Configuring extundelete0.2.4
Writing generated filesto disk
  
#编译安装
[root@localhostextundelete-0.2.4]# make && make install
  
[root@localhostextundelete-0.2.4]# make uninstall
Making uninstall in src
 ( cd '/usr/local/bin' && rm -fextundelete )
  
#删除时只需删除路径即可：
[root@localhost local]#rm -rf extundelet-rm/
```

### 安装.src.rpm源码包的方法
* 编译
将src.rpm中源码文件编译成可执行的二进制文件。

      rpmbuild --rebuild  lrzsz-0.12.20-27.1.el6.src.rpm
 
若顺利执行成功则会在root用户根目录下生成一个: rpmbuild目录。在/root/rpmbuild/RPMS/x86_64/目录下生成lrzsz-0.12.20-27.1.el6.x86_64.rpm这个rpm文件。

* 安装

接下来就是rpm的安装过程。

      [root@localhostx86_64]# rpm  -e lrzsz
      [root@localhostx86_64]# rpm -vih lrzsz-0.12.20-27.1.el7.x86_64.rpm



## 软件安装方法比较
rpm＋yum：方便，软件版本低。稳定性好、管理方便。性能稍差。
源码手动：麻烦，软件版本新。稳定性稍差、管理稍差。性能好。  LAMP，LNMP






