title: git
date: 2017-08-18 16:44:58
tags: [git, github, gitlab]
categories: developEnv
---

## 主要内容
git , github, gitlab 相关

* git 的后悔药

*更新历史*
2017-08-18 增加 删除/回滚 多次远程提交

<!-- more -->

## git 的后悔药

### 场景1：直接丢弃工作区的修改（还没有add）

`git checkout -- file`

### 场景2： 丢弃暂存区（已经add，未commit）
分两步，第一步用命令 `git reset HEAD file` ，就回到了场景1，第二步按场景1操作。

### 场景3：已经提交了不合适的修改到版本库（已经commit， 未push）
`git reset --hard HEAD^ #还原到上个`
`git reset --hard commitId`

### 场景4：已经推送到远程（已经push）
+ 删除某一次远程提交
`git revert commitId`
`git push origin master`

+ 删除/回滚 多次远程提交
`
 git rebase -i "commit id"^
`
![](/images/git1.jpg)
在编辑框中删除相关commit，如pick 5b3ba7a test2，然后保存退出（如果遇到冲突需要先解决冲突）！
`
git push origin master -f
`
这里需要强制推送， 如果是保护branch， 需要临时解除保护。
