title: Android入门 图谱
date: 2017-8-22 19:29:44
tags: [android, Android]
categories: Android
---

# 主要内容
Android 相关一览

* log

*更新历史*
2017-8-22 logcat

<!-- more -->


# 开发环境
## log
Android中的日志工具类是Log(android.util.Log),这个类中提供了如下5个方法来供我们打日志。 
+ Log.v() 。用于打印那些最为琐碎的、意义最小的日志信息。对应级别verbose,Android日志里面级别最低的一种。 
+ Log.d() 。用于打印一些调试信息,这些信息对你调试程序和分析问题应该是有帮助的对应级别debug,比verbose高一级。 
+ Log.i() 。用于打印一些比较重要的数据,这些数据应该是你非常想看到的、可以帮你析用户行为数据。对应级别info,比debug高一级。 
+ Log.w() 。用于打印一些警告信息,提示程序在这个地方可能会有潜在的风险,最好去复一下这些出现警告的地方。对应级别warn,比info高一级。 
+ Log.e() 。用于打印程序中的错误信息,比如程序进入到了catch语句当中。当有错误息打印出来的时候,一般都代表你的程序出现严重问题了,必须尽快修复。对应级error,比warn高一级。 


logcat 里面可以定制过滤器

### 定制日志工具
用过`log4j`的同学一定知道日志等级，可以在配置文件中配置；郭霖大大的方案是自己定制下log

```

    public class LogUtil {
        public static final int VERBOSE = 1;
        public static final int DEBUG = 2;
        public static final int INFO = 3;
        public static final int WARN = 4;
        public static final int ERROR = 5;
        public static final int NOTHING = 6;
        public static int level = VERBOSE;
        public static void v(String tag, String msg) {
            if (level <= VERBOSE) {
                Log.v(tag, msg);
            }
        }
        public static void d(String tag, String msg) {
            if (level <= DEBUG) {Log.d(tag, msg);
            }
        }
        public static void i(String tag, String msg) {
            if (level <= INFO) {
                Log.i(tag, msg);
            }
        }
        public static void w(String tag, String msg) {
            if (level <= WARN) {
                Log.w(tag, msg);
            }
        }
        public static void e(String tag, String msg) {
            if (level <= ERROR) {
                Log.e(tag, msg);
            }
        }
    }
```

然后我们只需要修改level 变量的值,就可以自由地控制日志的打印行为了。比如让level=VERBOSE 就可以把所有的日志都打印出来,让level=WARN 就可以只打印警告以上级别的志,让level=NOTHING 就可以把所有日志都屏蔽掉。

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
