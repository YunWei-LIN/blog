title: 十年回头望
date: 2016-06-14 21:28:38
tags: 十年回头望
categories: 杂谈
---

十年之前，刚刚毕业；十年之后，回头看看；低头思考，抬头前行。

<!-- more -->

以下是个人对开发框架整体的思考和一些优秀实践，简单整理，陆续补充，详细的看是否有时间完善。

##  系统架构
### 网络拓扑
中小型系统架构网络拓扑
<a class="fancybox" title="" href="http://amadis.qiniudn.com/structure.jpg" rel="article0">
<img  width="700px"  alt="" src="http://amadis.qiniudn.com/structure.jpg">
</a>


### 基础框架
以Spring的一系列框架为基础（如 Spring MVC， SpringData），配合自己开发的开发脚手架。
表现层：Css3， JavaScript6，JQuery ， Jsp2
控制层：spring mvc 
服务层：spring core
持久层：spring data JPA 
安全层：shiro
批处理：quartz
单元测试： Junit + H2数据库
数据库：Mysql + Redis

## 开发流程
<p>
<a class="fancybox" title="开发流程" href="http://amadis.qiniudn.com/process.png" rel="article0">
<img alt="开发流程" width="400px" height="600px" src="http://amadis.qiniudn.com/process.png">
</a>
<span class="caption">开发流程</span>
</p>


## 第三方管理系统
  
  
### git gitlab 源码管理
  
  
* git
分布式源码管理，gitFlow是其一大特色。
  相关链接
  * [git](/tags/git/)


* gitlab
一个开源的版本管理系统,实现一个自托管的Git项目仓库,可通过Web界面进行访问公开的或者私人项目。基本拥有GitHub类似的功能。
建议采用ssh协议
  * 方便 ： 不用每次输入密码
  * 安全
  * 减少传送数据，节约流量
  [gitlab](/tags/gitlab/)

* gitFlow
gitFlow模型：一个组织软件开发活动的版本迭代模型，定义围绕项目发布的严格分支模型，提供了一个健壮的用于管理大型项目的框架。
5大分支
[git-flow 备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

* jgitflow-maven-plugin

  [jgitflow](http://jgitflow.bitbucket.org/)

### maven 项目管理 
* 管理系统依赖
统一依赖的来源和版本，保持开发环境一致，避免莫名错误

* 管理项目生命周期
构建，编译，测试，发布，更新版本。。。。

[maven](/tags/maven/)




### nexus 本地资源库
使用 Sonatype Nexus Repository Manager OSS版本；
nexus可以帮助你与其他开发人员和最终用户共享组件。它大大简化了内部存储库的维护和访问外部存储库，可以从单一地点完全控制访问和部署每个组件。
主要配合maven 和 gradle 工作。

* [相关链接](/tags/nexus/)

### jira 任务与缺陷管理

多人协助也好， 一个人玩也行；
自己给自己安排任务，自己安排时间，完了自己查看统计数据，挺好。


### code review
crucible系统
窃以为： code review是成本最低且有效提高代码质量的手段，特别是几个技术高深的人员一起review

### jenkins 持续集成
* 测试
测试人员随时或的最新版本，构建测试版本，且不打扰开发人员；

* 发布
配合`maven`的多环境参数配置，图形化一键发布，方便高效

* 相关链接
[jenkins](/tags/jenkins/)


## 开发辅助系统
自己开发，开发测试人员内部使用， 持续改进ing


### 开发沙箱
* 接口文档在线化
* 开发环境/测试环境/生产环境的切换
* 保留接口请求和服务器返回内容的现场数据
* 模拟接口仿真服务，方便客户端开发和测试
* 接口版本化管理（待改进）

### 2维码管理系统
个性化定制2维码，并跟踪该2维码后续行为数据。比如给某推广人员定制一个专属2维码，并收集和分析该2维码的后续行为数据。

### 数据收集分析系统
收集用户行为数据，根据运营需求分析；
友盟等提供的功能直接用友盟；
这里可以做一些对友盟等第三方数据平台的补充。

### 安装包管理系统
统一管理APP安装包， 提供图像界面给测试人员和运营人员操作，记录保存每个版本。
配合CI使用。
iOS 生产环境未实现。。。。。


### 源码规范检查工具
* 检查多余图片资源、语言包
* 检查项目中命名、编码规范


### 意见反馈管理系统
联合运营人员或者客服人员，聆听用户反馈，改进产品。


## 数据库
生产最近使用较多的是 mysql及其分支；junit中多用H2

### Multi-Source Replication
### 热备
### 冷备

## 开发脚手架
注重代码积累，将常用代码模块化，需要时可插拔，提高开发效率和代码质量。
从技术层面来说，产品化需要做到可配置、可定制、可灵活组装、可快速开发.
* 认证授权模块
* 消息短信模块
* 用户基础模块
* 第三方支付模块
* 静态资源模块
* 加解密模块





