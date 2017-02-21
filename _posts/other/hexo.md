title: github hexo 多说 构建自己的blog
date: 2015-07-03 
tags: 
- hexo 
- duoshuo 
- github

categories: hexo
---

7月5日左右搭建成功的，算比较新鲜吧：） 记录一下操作步骤，感谢前人，希望能帮到其他人。主要涉及github，hexo，多说。
github： 我想你懂得，万一你不懂，百度可以帮助你;
hexo  ： 博客博客模板框架 - 基于nodejs 
多说   ： 一个现成的评论模块，据说是国内最好的

下面每个讲讲，nodejs和git不说了。


## github
首先你得有一个github的账户， ssh链接也配置好。

### GitHub Pages 建立博客
GitHub Pages分两种，一种是你的GitHub用户名建立的username.github.io这样的用户&组织页（站），另一种是依附项目的pages。
建立个人的博客使用第一种，像 sam2099.github.io 。
[GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/)



## hexo

### 安装Hexo
``` bash
$ npm install -g hexo
```

### 初始化
找个干净好找的地方建个文件夹hexo，然后执行
``` bash
$ hexo init
```
hexo会自动建好目录和文件。

### 看看初始效果
在当前目录（hexo）运行，然后到浏览器输入localhost:4000看看。
``` bash
$ hexo g
$ hexo s
```
哈哈，正常的话，可以看到了。

### 主题
复制wuchong的修改的主题， 他的主题集成了多说，rss等，比较方便
``` bash
$ git clone https://github.com/wuchong/jacman.git themes/jacman
```
Hexo目录下的config.yml配置文件中的theme属性，将其设置为jacman。
``` bash
theme: jacman
```

### Quick Start
下面是hexo init 时生成的操作命令，够用了。
Welcome to [Hexo](http://hexo.io/)! This is your very first post. Check [documentation](http://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](http://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).


### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](http://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](http://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](http://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](http://hexo.io/docs/deployment.html)


### hexo 设置
根据自己爱好来对Hexo生成的网站进行设置了，对整站的设置，只要修改项目目录的_config.yml就可以了。
注意空格， 冒号后面一定跟一个空格；层级关系也需要空格。
我所有的 [博客源文件](https://github.com/sam2099/bolg) 可以和blog对照着看。
主要设置的地方：
``` bash
Site
plugins
theme
deploy
```

## 插件

### RSS
安装
``` bash
$ npm install hexo-generator-feed -g  --save   
```

配置
参考 hexo 设置；
然后修改 themes/jacman下的 _config.yml，在 menu section 添加：
Rss: /atom.xml


### sitemap
安装
``` bash
$ npm install hexo-generator-sitemap -g  --save   
```

配置
参考 hexo 设置

## 多说
去[多说](http://duoshuo.com/) 注册个账户，然后建立个应用；
然后修改 themes/jacman下的 _config.yml，在duoshuo_shortname： 后面添加你应用的名称

## 域名映射
在source根路径下，创建文件CNAME（无后缀，大写）;内容就一行，是你需要映射的域名，比如我的
```
giveme5.top
```

## 主题
目前来说 NexT 这个主题最符合我的口味。

[NexT](http://theme-next.iissnan.com/getting-started.html#description-setting)
[NexT](http://zhiho.github.io/2015/09/29/hexo-next/)

## 藏在最后
这是第一篇，我想这个作为笔记，有需要就可以拿出来看看，省得记载纸上找不到；所以这个大概是给自己看的。