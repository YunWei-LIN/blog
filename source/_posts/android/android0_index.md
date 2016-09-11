title: Android入门 总纲
date: 2016-5-13 19:29:44
tags: Android
categories: Android
---

# 开发环境
## logcat


# 布局
## 线性布局
## 相对布局
## 帧布局
## 表格布局
## 绝对布局


# 存储
## 内部存储
## SD卡
## SharedPreference
## XML

# SQLite
## 建库
## 建表
## 原生 CRUD
## API CRUD
## 事务

# 控件
## Toast

## ListView
导入局部布局
条目点击监听 ： `setOnItemClickListener`


### 缓存
* convertView
* ViewHolder

## Dialog 对话框
### 确定取消Dialog
### 单选Dialog
### 多选Dialog



# 网络访问
## 堵塞
## 消息队列


# 四大组件 Activity，Service服务,Content Provider内容提供者，BroadcastReceiver广播接收器
# Activity
## 页面跳转
显式效率高
### 显式跳转
### 隐式跳转

## 传递数据
### 跳转
### 返回
* 子Activity
```
Intent data = new Intent();
data.putExtra("key", value);
setRestult(resultCode, data) //set result vaule
finish(); //销毁当前Activity, 返回父Activity
```

* 父Activity
启动 ： `StartActivityForResult(Intent, requestCode)` 
接收 ： `OnActivityResult(int requestCode, int resultCode, Intent data)` 打开多个的情况可以采用requestCode和resultCode来区别每个子Activity及其返回数据的目的

## 生命周期

## 启动模式
### launchMode
* standard
* singleTop : 栈顶已经有一个相同类型的Activity实例，Intent不会再创建一个Activity，而是通过onNewIntent()被发送到现有的Activity
* singleTask : singleTask模式的Activity只允许在一个APP栈中有一个实例。如果系统中已经有了一个实例，持有这个实例的任务将移动到顶部，（将其上方的所有Activity销毁）同时intent将被通过onNewIntent()发送。如果没有，则会创建一个新的Activity并置放在合适的任务中。
* singleInstance : 非常接近于singleTask，系统中只允许一个Activity的实例存在。每个singleInstance会有自己独立的栈。实际开发中只有平台型应用（qq，微信，微博），提供给其他应用使用的情况，才采用。

## 横竖切换
* 默认切换会调用生命周期方法；
配置`AndroidManifest.xml` ， 增加`android:configChanges="orientation|keyboardHidden"` 时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

* 锁定
配置`AndroidManifest.xml` ， 增加`android:orientation`
或者 在 `OnCreate`方法中 增加 `setRequestOrientation(Activity.)`


# BroadcastReceiver广播接收器

# 服务
