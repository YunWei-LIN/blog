---
title: jenkins 管理项目权限
date: 2017-03-03 11:30:08
tags: jenkins 
categories: developEnv
---
主要内容

利用 插件`Role-based Authorization Strategy` 管理项目权限

<!-- more -->

## 安装插件
安装插件`Role-based Authorization Strategy`

![Role-based Authorization Strategy](/images/jenkins_role_1.png)


## 授权策略
`Manage Jenkins` -> `Configure Global Security` -> `Access Control` -> `Authorization` -> 选择“Role-Based Strategy”

![](/images/jenkins_role_2.png)

注意下拉到最后 `保存`

## 创建用户
`Manage Jenkins` -> `Manage Users` -> `Create User`， 
创建用户`productManager` 
![](/images/jenkins_role_3.png)


## 管理角色
### 创建角色
`Manage Jenkins` -> `Manage and Assign Roles` -> `Manage Roles`， 
需要分别在 `Global roles` 全局级别， `Project roles` 项目级别 操作

![](/images/jenkins_role_4.png)

2个级别：
+ `Global roles` 全局级别
创建角色`member`，给`member`角色 赋予 `Overall.Read`  `Job.Create` 这2个全局权限。`Overall.Read` 必须要赋予。
  
  
+ `Project roles` 项目级别
创建角色`pm`，例如定义`pm`角色为该项目管理员，所以赋予其项目的全部权限， 看你们的实际情况。

注意下拉到最后 `保存`

### 赋予角色
`Manage Jenkins` -> `Manage and Assign Roles` -> `Assign Roles`， 
为上面创建的用户`productManager` 赋予角色， 这里也需要分别在 `Global roles` 全局级别， `Project roles` 项目级别 操作。
![](/images/jenkins_role_5.png)

2个级别：
+ `Global roles` 全局级别
用户`productManager` 赋予角色`member`
  
  
+ `Project roles` 项目级别
用户`productManager` 赋予角色`pm`。

注意下拉到最后 `保存`

## 验证角色
+ 管理员视图：
![](/images/jenkins_role_6.png)


+ 用户`productManager` 视图：
![](/images/jenkins_role_7.png)












