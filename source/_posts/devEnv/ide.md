title: idea 和 eclipse 快捷键
date: 2015-09-16 11:44:58
tags: idea eclipse 快捷键
categories: developEnv
---
eclipse 和 idea 常用快捷键 对比 (转载)

|Eclipse | IntelliJ IDEA | Description|
| ------------------ |:---------------------:| :-----------------------------------------------------------------------|
|F4 | ctrl+h | show the type hierarchy|
|ctrl+alt+g | ctrl+alt+F7 | find usages|
|ctrl+shift+u | ctrl+f7 | finds the usages in the same file|
|alt+shift+r | shift+F6 | rename|
|ctrl+shift+r | ctrl+shift+N | find file / open resource|
|ctrl+shift+x, j | ctrl+shift+F10 | run (java program)|
|ctrl+shift+o | ctrl+alt+o | organize imports|
|ctrl+o | ctrl+F12 | show current file structure / outline|
|ctrl+shift+m | ctrl+alt+V | create local variable refactoring|
|syso ctrl+space | sout ctrl+j | System.out.println(“”)|
|alt + up/down | ctrl + shift + up/down | move lines|
|ctrl + d | ctrl + y | delete current line|
|??? | alt + h | show subversion history|
|ctrl + h | ctrl + shift + f | search (find in path)|
|“semi” set in window-> preferences | ctrl + shift + enter | if I want to add the semi-colon at the end of a statement|
|ctrl + 1 or ctrl + shift + l | ctrl + alt + v | introduce local variable|
|alt + shift + s | alt + insert | generate getters / setters|
|ctrl + shift + f | ctrl + alt + l | format code|
|ctrl + y | ctrl + shift + z | redo|
|ctrl + shift + c | ctrl + / | comment out lines|
|ctrl + alt + h | ctrl + alt + h (same!) | show call hierarchy|
|none ? | ctrl + alt + f7 | to jump to one of the callers of a method|
|ctrl + shift + i | alt + f8 | evaluate expression (in debugger)|
|F3 | ctrl + b | go to declaration (e.g. go to method)|
|ctrl + l | ctrl + g | go to line|
|alt + /  | ctrl + space| eclipse:Content Assist ; idea:completion 跟输入法冲突，自己改|
|？？？|ctrl + shift + a |查找所有Intellij的命令|
|？？？|ctrl + shift + alt + t|重构所有|

Shortcut Translator插件， 安装后，按Ctrl+Shift+K调出快捷键翻译对话框，选定你惯用的IDE keymap和需要学习的keymap，按下惯用keymap的快捷键，即可看到学习keymap上的对应快捷键。


## IDEA 初始化
### 插件
* JReble
* IdeaVim
默认开启/关闭快捷是 Ctrl+Alt+v， 有冲突，可以改成Shift+Alt+v

### 快捷键
* 修改 completion ： shift + space
* 增加 close all 和 close other
* 修改自动完成大小写敏感
setting-->Editor-->General-->Code completion : Case sensitive completion 修改为 none

### server
新建好server， before launch ： Maven Goal -》 clean package


### 注释
* Live templates
增加 java method comment
jmc ： /**

* File and Code templates
Includs 可以修改成如下内容：

      /**
      * Created with ${PRODUCT_NAME}.
      * User: ${USER}
      * Date: ${DATE}
      * Time: ${TIME}
      * description: 
      */ 
      
### 
* 编辑 .properties 文件
settting --》  File Encodings --》 Transparent native-to-ascii conversion 打勾

* 限制一行代码的宽度
File->settings->Code Style->General中，修改“Right margin (columns)”的值即可改变代码行宽度的限制。
自动将代码换行:
第一种，在上述的“Right margin (columns)”的下方，有“Wrap when typing reaches right margin”选项，选中它，是什么效果呢？如下图所示，随着输入的字符的增加，当代码宽度到达界线时，IDEA会自动将代码换行。 
第二种，在File->settings->Code Style->Java中，选中“Wrapping and Braces”选项卡，在“Keep when reformatting”中有一个“Ensure rigth margin is not exceeded”，选中

* quick doc
eclipse 光标放类或方法上， 默认有javaDoc 等 doc 显示；
idea 默认没有， 可以用 `CTRL + Q` 手动显示， 或者 `setting -> editor -> general` 页面， other 部分， 将 `show quick documentation` 打上勾