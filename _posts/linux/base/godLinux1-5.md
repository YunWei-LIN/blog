title: 1-5 vim编辑器
date: 2015-12-4 16:29:44
tags: GOD linux
categories: linux
---
写在一起之前： 命令行输入 `vimtutor` 你会有惊喜， 全部做完，恭喜你，入门了！！！！！！！！！

主要内容：
vim主要模式介绍
vim命令模式
vim插（输）入模式
vim命令（末）行模式
设置vim开发环境


##  主要模式
vim 主要有 `命令模式` `插（输）入模式` `命令（末）行模式` 三种模式， 意义和转换如下图：
![](http://7xklqw.com1.z0.glb.clouddn.com/vimmodels.png)
<!--more-->

## 命令模式
此状态下敲击键盘动作会被Vim识别为命令，而非输入字符。

### 字符操作
* `i` 当前字符之前插入
* `I` 行首插入
* `a` 当前字符之后插入
* `A` 行尾插入
* `esc` 退出当前模式
* `o` 下一行插入
* `O` 上一行插入
* `x` 向后删除一个字符del
* `X` 向前删除一个字符
* `u` 撤销一步 

### 定位
* `gg` 定位到行首
* `G` 定位到最后一行，行首
* `nG` 定位到某一行
* `:n` 定位到某一行
* `ngg` 定位到某一行 
n 代表行号

### 行操作
* `home键或^` 行首 
* `end或$` 行尾
* `dd` 删除一行 
  `ndd` 删除以当前行开始的n行
  `ddp` 上下两行的内容互换
  `dG` 删除从当前行至文件未尾的所有行。
* `yy` 复制一行 
  `Nyy` 复制N行
* `p` 将复制行粘贴 P上粘
* 扩展：剪切=先删除，再粘贴
* `d + HOME 或^` 删除到行首  
* `d + END 或$` 删除到行尾 

### 词操作
* `dw` 删除一个词，删除时要将光标移动到这个词的行首。 另外，如果光标不在行首，则删除光标之后的字母。
* `yw` 复制一个词
* `w` 切换单词

### 块操作
* `大D 或d+$` 删至行尾 
  `d+^` 删至行首
* `y+$` 复制至尾 
  `y+^` 复制至首
  `"+y`  复制选中内容到＋寄存器，也就是系统的剪贴板，供其他程序用 +需要显式输入。 ：reg 可查看支付支持＋寄存器

### v 模式
进入v模式 移动光标选择区域、
编程的时候需要进行多行注释：
1、注释：ctrl+v 进入列编辑模式
2向下或向上移动光标
3把需要注释的行的开头标记起来
4然后按大写的I
5再插入注释符,比如"#"。
6再按Esc,就会全部注释了。
  
  删除多行注释：
  删除：再按ctrl+v 进入列编辑模式；向下或向上移动光标 ；选中注释部分,然后按d, 就会删除注释符号。

## 输入模式
字符就是字符本身，用来编辑内容

## 命令（末）行模式
* 保存退出
`:w` 保存 save
`:q` 没有进行任何修改，退出 quit
`:q!` 修改了，不保存，强制退出
`:wq` 保存并退出 
`:wq!` 强制保存并退出。

* 替换
`:%s /this/that` #每一行的第一个this被替换成that  
`:s /emacs/vim` #在当前行中把第一个emacs替换成vim。
`:%s /this/that/g` #将文本中所有的this替换成that
`:2,5s /this/that/g` #替换第二行到第五行中sbin
注意： `替换内容中本身有/符号， 外面的/可以用#替换。 比如 :%s #this#that#g`

* `:set nu/nonu`  #显示行号
* 查找
`/` 正向查找 ：/target   
`n` 往下查找，
`N` 往上查找
去消高亮显示： noh 或 随便查找一组没有的字符

* `:! command` 运行shell命令

* `:r /etc/ssh/sshd_config.bak`  读取其他文件


## 设置vim开发环境
在命令模式下用set命令设置的东西是不能保存的，下次打开vim时又要重新设置。所以vim提供了一个配置文件叫vimrc，可以保存你的配置信息。
  ~/.vimrc
输入：
`set nu` #显示行数 
`set history=10` #
`set autoindent` #自动缩排，如当前行是从第3个字符的位置开始编辑的,按回车后光标会自动定位在下一行第三3个字符的位置。
`set paste` #置粘贴模式，这样粘贴过来的程序代码就不会错位了。
等等

## vim同时打开多个文件
用法： vim -[o/O] filename
大写 `O` 左右分屏，小写的 `o` 上下分屏， `ctrl+WW` 在文件之间进行切换


## 参考
[鸟哥linux](http://linux.vbird.org/linux_basic/0310vi.php)