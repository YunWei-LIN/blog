title: git
date: 2016-05-03 11:44:58
tags: git
categories: developEnv
---
## git 的后悔药

* 场景1：直接丢弃工作区的修改（还没有add）

`git checkout -- file`

* 场景2： 丢弃暂存区（已经add，未commit）
分两步，第一步用命令 `git reset HEAD file` ，就回到了场景1，第二步按场景1操作。

* 场景3：已经提交了不合适的修改到版本库（已经commit， 未push）
`git reset --hard HEAD^ #还原到上个`
`git reset --hard commitId`

* 场景4：已经推送到远程（已经push）
`revert commitId`
