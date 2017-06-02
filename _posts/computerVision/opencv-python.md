---
title: Opencv3.2 python2.7 的CentOS7安装 
date: 2017-06-01 18:48:06
tags: [opencv, centos, python]
categories: 机器视觉
---
主要内容
* 系统环境
* 软件源
* 软件环境
* 源码安装

Opencv3.2 python2.7 的CentOS7 完整安装过程。 yum安装的opencv 是2.4 版本；为使用最新功能，下面介绍 源码安装 Opencv3。

......

<!-- more -->

## 系统环境
```sh

╭─root@localhost ~  
╰─# uname -a                                                                1 ↵
Linux localhost.localdomain 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
╭─root@localhost ~  
╰─# cat /etc/redhat-release 
CentOS Linux release 7.3.1611 (Core) 

```

## 软件源
* [epel]()
epel 源， 阿里的镜像

```sh
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

* [Nux Dextop](http://li.nux.ro/repos.html)
vlc 播放器 和 视频解码器 等

```sh
yum -y install epel-release && rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
```

## 软件环境
编译opencv需要的软件包

+ GCC 4.4.x or later
+ CMake 2.8.7 or higher
+ Git
+ GTK+2.x or higher
+ pkg-config
+ Python 2.6 or later and Numpy 1.5 or later with developer packages (python-dev, python-numpy)
+ ffmpeg and development packages
+ [optional] libtbb2 libtbb-dev
+ [optional] libdc1394 2.x
+ [optional] libdc1394-devel libv4l-devel gstreamer-plugins-base-devel
+ [optional] CUDA Toolkit 6.5 or higher

### 更新系统
```sh
yum -y update
```

### 开发工具
```sh
yum groupinstall 'Development Tools'
```

```sh
yum install cmake cmake-gui git pkgconfig autoconf automake  freetype-devel gcc gcc-c++  libtool make mercurial nasm pkgconfig zlib-devel
```

### 视频和图像格式
#图像
```sh
yum install libpng-devel libjpeg-turbo-devel jasper-devel openexr-devel libtiff-devel libwebp-devel 
```

#视频
```sh
yum install libdc1394-devel libv4l-devel gstreamer-plugins-base-devel ffmpeg ffmpeg-devel
```

### GUI特征
```sh
yum install gtk2-devel
```

### Threading Building Blocks (TBB)
```sh
yum install tbb-devel eigen3-devel
```

### cuda安装
//TODO

### vlc
```sh
yum install vlc
```


### python
centos 7 默认安装的是 python 2.7

#### pip (python package manager)
```sh
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py
```

或者yum安装，然后升级
```sh
yum install -y python-pip
pip install --upgrade pip
```

#### python 2.7 development tools
```sh
yum install python-devel
```

#### numpy
```py
pip install numpy
```

## Opencv3.2 源码安装
工作目录为 `～`， 按照你的喜爱调整，记得下面的目录都要调整

### 获取源码
+ opencv

```sh
cd ~
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.2.0
```

+ opencv_contrib
从 opencv 3.0 开始， opencv_contrib 单独发布， 这里有很多有意思的功能

```sh
cd ~
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.2.0
```

### 编译安装

#### 编译目录

```sh
cd ~/opencv
mkdir build
cd build
```

#### cmake

```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
    -D INSTALL_C_EXAMPLES=OFF \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D BUILD_EXAMPLES=ON \
    -D BUILD_OPENCV_PYTHON2=ON ..
```

特保注意：
+ ippicv
cmake 会停留在 如下类似位置：

    ``` sh
    -- checking for module 'libgphoto2'
    --   package 'libgphoto2' not found
    -- ICV: Downloading ippicv_linux_2017xxxx.tgz...
    ```

    cmake 需要下载 `ippicv`， 会持续一段时间，耐心等待下。
    如果超时或者你想手动下载， 参照你 `build` 目录下 `CMakeDownloadLog.txt` 文件
    ```sh
    > cat CMakeDownloadLog.txt 
    use_cache "/home/alalek/projects/opencv/dev/.cache"
    do_unpack "ippicv_2017u2_lnx_intel64_20170418.tgz" "87cbdeb627415d8e4bc811156289fa3a" "https://raw.githubusercontent.com/opencv/opencv_3rdparty/a62e20676a60ee0ad6581e217fe7e4bada3b95db/ippicv/ippicv_2017u2_lnx_intel64_20170418.tgz" "/home/alalek/projects/opencv/build/opencv/3rdparty/ippicv"
    #cmake_download "/home/alalek/projects/opencv/dev/.cache/ippicv/87cbdeb627415d8e4bc811156289fa3a-ippicv_2017u2_lnx_intel64_20170418.tgz" "https://raw.githubusercontent.com/opencv/opencv_3rdparty/a62e20676a60ee0ad6581e217fe7e4bada3b95db/ippicv/ippicv_2017u2_lnx_intel64_20170418.tgz"
    ```

+ python
检查 python相关环境是否一致， 类似如下

    ```sh
--   Python 2:
--     Interpreter:                 /usr/bin/python2.7 (ver 2.7.5)
--     Libraries:                   /lib64/libpython2.7.so (ver 2.7.5)
--     numpy:                       /usr/lib64/python2.7/site-packages/numpy/core/include (ver 1.12.1)
--     packages path:               lib/python2.7/site-packages
    ```

    可以通过 图像工具 `cmake-gui` 来调整
    ```
    cmake-gui ..
    ```

#### make
```sh
make -j4 # 4是你的电脑cpu线程
make install
ldconfig
```

检查 `/usr/local/lib/python2.7/site-packages` 目录下 是否已经成功安装 `cv2.so`

#### 链接给python

```sh
cd /usr/lib64/python2.7/site-packages 
ln -s /usr/local/lib/python2.7/site-packages/cv2.so cv2.so
```

### 测试opencv-python

```sh
python
>>>import cv2
>>>cv2.__version__
3.2.0'
```

congratulation！！！
