title: nexus 不能自动下载 restlet jars 的解决方案
date: 2015-10-15 11:22:59
tags: nexus
categories: developEnv
---

今天用到了 activiti 5.16；它依赖 `org.restlet.jee:org.restlet:2.2.1` 等 jar。奇怪的事情发现了， 一向运行良好的 nexus 出问题，`org.restlet.jee:org.restletxxx:2.2.1` 一系列的jars都无法下载。这里吐草下，`org.restlet.jee:org.restletxxx` 看着就别扭，so nexus 也找不到它，自然也无法下载。
解决方案如下：

* 增加restlet的Repository
管理员账户登录nexus 控制台， 增加restlet的Repository，类型为proxy
![](http://7xklqw.com1.z0.glb.clouddn.com/nexus1.png)
具体内容参照下图，重要 `Remote Storage Location` 设置为 `http://maven.restlet.com/`
![](http://7xklqw.com1.z0.glb.clouddn.com/nexus2.png)

* restlet 加入 Public repositories group
![加入前](http://7xklqw.com1.z0.glb.clouddn.com/nexus3.png)
![加入后](http://7xklqw.com1.z0.glb.clouddn.com/nexus4.png)

* 清除本地 Repositories
别忘了删除 maven 的本地 repositories 里的 restlet 文件夹。



