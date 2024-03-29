---
title: linux wine 32 & 64
date: 2017-09-26 13:00:10
tags: [linux, wine]
categories: linux
---
## 主要内容
  wine在linux中很常用，目前是32位和64位的软件都有；在此讨论在64位centos 7中安装32位和64位的wine。
还有wine的中文环境下一些乱码问题。

*更新历史*
2017-09-26 增加 同时 wine 32和64 的应用

<!-- more -->

## 安装
### 删除已有版本
    yum erase wine wine-*
    #zsh中yum erase wine "wine-*"

### 编译环境
    yum install samba-winbind-clients -y
    yum groupinstall 'Development Tools' -y
    yum install libjpeg-turbo-devel libtiff-devel freetype-devel -y
    yum install libgcc.i686 libX11-devel.i686 freetype-devel.i686 gnutls-devel.i686 libxml2-devel.i686 libjpeg-turbo-devel.i686 libpng-devel.i686 libXrender-devel.i686 glibc-devel.i686 glibc-devel ibstdc++-devel.i686  -y 
[参考](http://www.cyberciti.biz/tips/compile-32bit-application-using-gcc-64-bit-linux.html)

### 下载wine源码
    ver=1.9.21
    cd /usr/src
    wget http://dl.winehq.org/wine/source/1.9/wine-${ver}.tar.bz2 -O wine-${ver}.tar.bz2
    tar xjf wine-${ver}.tar.bz2
    cd wine-${ver}/
    mkdir -p wine32 wine64
### wine 64 编译安装
    cd wine64
    ../configure --enable-win64
    make -j 4
    make install

### wine 32 编译安装
    cd ../wine32
    ../configure --with-wine64=../wine64
    make -j 4
    make install

### 校验
    $ file `which wine`
    /usr/local/bin/wine: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=a83b9f0916e6c0d5427e2c38a172c93bd8023d98, not stripped
    $ file `which wine64`
    /usr/local/bin/wine64: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=4d8e8468402bc63bd2a72c59c57fcad332235d41, not stripped

## 中文字体库
### 安装字体管理工具
    yum install fontconfig mkfontscale
### 字体库
    mkdir -pv /usr/share/fonts/chinese/TrueType
从Windows字体目录C:\Windows\Fonts 拷贝需要的字体库到刚刚建立目录

### 建立字体缓存
    cd /usr/share/fonts/chinese/TrueType
    mkfontscale
    mkfontdir
    fc-cache -fv
    fc-list |grep SimSun
## 中文乱码
[参考](www.oschina.net/question/12_2853)


## wine 32和64 的应用
[WINEPREFIX 和 WINEARCH ](https://wiki.archlinux.org/index.php/Wine_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#WINEPREFIX)

### WINEPREFIX
通过设置WINEPREFIX环境变量，可以更改Wine系统目录的位置。如果希望让不同的Windows程序使用不同的系统环境或配置，这一变量会非常有用。
比如同时需要wine 32位和64位的程序， 就需要设置32位和64位的Wine系统目录。

```sh
env WINEPREFIX=~/.win-32  wine32  32A程序.exe
env WINEPREFIX=~/.win-64  wine64  64B程序.exe
```

### WINEARCH
对于64位用户，如果使用[multilib]仓库里的Wine，默认创建的系统目录是64位环境的。若想使用纯32位环境，修改WINEARCH 变量win32为即可
```sh
WINEARCH=win32 WINEPREFIX=~/win32 winecfg 
WINEPREFIX=~/win64 winecfg
```

 

