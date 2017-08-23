title: Android入门 图谱
date: 2017-8-22 19:29:44
tags: [android, Android]
categories: Android
---

## 主要内容
Android UI

* Material Design

*更新历史*
2017-8-22 logcat

<!-- more -->

材料主题
![](/images/android_ThemeColors.png)

## Toolbar
Toolbar 继承了ActionBar的所有功能,而且灵活性很高,可以配合其控件来完成一些Material Design的效果
![](/images/android_Toolbar.png)

+ 隐藏ActionBar
`res/values/styles.xml` 中，修改 AppTheme的parent主题， 通常有Theme.AppCompat.NoActionBar 和Theme.AppCompat.Light.NoActionBar这两种主题可选

+ 增加 Toolbar
在布局文件中增加 Toolbar
```xml
...
<android.support.v7.widget.Toolbar
android:id="@+id/toolbar"
android:layout_width="match_parent"
android:layout_height="?attr/actionBarSize"
android:background="?attr/colorPrimary"
android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
...
```
  在 activity增加 ActionBar 支持
```java
    public class MainActivity extends AppCompatActivity {


        @Override
        public boolean onCreateOptionsMenu(Menu menu) {
            return super.onCreateOptionsMenu(menu);
        }

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
            toolbar.setTitle("Fruits"); //Toolbar 显示内容
            setSupportActionBar(toolbar);
            
            ActionBar actionBar = getSupportActionBar();
            if (actionBar != null) {
                actionBar.setDisplayHomeAsUpEnabled(true);
                actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
            }
        }
    }
```

+ 增加菜单
右击res目录→New→Directory,创建一个menu文件夹。然后右击menu文件→New→Menu resource file,创建一个toolbar.xml文件
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto">
<item
    android:id="@+id/backup"
    android:icon="@drawable/ic_backup"
    android:title="Backup"
    app:showAsAction="always" />
<item
    android:id="@+id/delete"
    android:icon="@drawable/ic_delete"
    android:title="Delete"
    app:showAsAction="ifRoom" />
<item
    android:id="@+id/settings"
    android:icon="@drawable/ic_settings"
    android:title="Settings"
    app:showAsAction="never" />
</menu>
```
    修改activity
```java
public class Activity extends AppCompatActivity {
    ...
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.toolbar, menu);
        return true;
    }
        @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.backup:
                Toast.makeText(this, "You clicked Backup",
                        Toast.LENGTH_SHORT).
                        show();
                break;
            case R.id.delete:
                Toast.makeText(this, "You clicked Delete",
                        Toast.LENGTH_SHORT).
                        show();
                break;
            case R.id.settings:
                Toast.makeText(this, "You clicked Settings",
                        Toast.LENGTH_SHORT).
                        show();
                break;
            default:
        }
        return true;
    }
}

```
+ 设置导航按钮
activity 的 onCreate 中
```java
ActionBar actionBar = getSupportActionBar();
if (actionBar != null) {
    actionBar.setDisplayHomeAsUpEnabled(true); //导航按钮显示
    actionBar.setHomeAsUpIndicator(R.drawable.ic_menu); // 导航按钮 图标
}
```

## 滑动菜单
滑动菜单就是将一些菜单选项隐藏起来,而不是放置在主屏幕上,然后可以通过滑动的方将菜单显示出来。
Google 提供 DrawerLayout， NavigationView 帮助我们简单就可实现滑动菜单的功能
![](/images/android_moveMenu.png)

### DrawerLayout
它是一个布局,在布局中允许放入两个直接子控件,第一个子控件是主屏幕中显示的内容,第二个子控件是滑动菜单中显示的内容。

布局文件

```xml
<android.support.v4.widget.DrawerLayout 
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:id="@+id/drawer_layout"
android:layout_width="match_parent"
android:layout_height="match_parent">
<FrameLayout android:layout_width="match_parent" android:layout_height="match_parent">
    <android.support.v7.widget.Toolbar 
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimary"
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
</FrameLayout>
<TextView android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    android:text="This is menu"
    android:textSize="30sp"
    android:background="#FFF" />
</android.support.v4.widget.DrawerLayout>
```
DrawerLayout中放置了两个直接子控件,第一个子控件是FrameLayout,用于作为主屏幕显示的内容,当然里面还有我们刚刚定义的Toolbar。第二个子控件这里使用了一个TextView,于作为滑动菜单中显示的内容,其实使用什么都可以,DrawerLayout并没有限制只能使用固定控件。 
第二个子控件有一点需要注意,layout_gravity 这个属性是必须指定的,因为我们要告诉DrawerLayout滑动菜单是在屏幕的左边还是右边,指定left表示滑动菜单在左边,指定right表示滑动菜单在右边。这里我指定了start,表示会根据系统语言进行判断,如果系统语言从左往右的,比如英语、汉语,滑动菜单就在左边,如果系统语言是从右往左的,比如阿拉伯语滑动菜单就在右边。 

### NavigationView
NavigationView是Design Support库中提供的一个控件,可以将滑动菜单页面的实现变得非常简单。

+ menu
menu是用来在NavigationView中显示具体的菜单项的,右击menu文件夹→New→Menu resource file，创建一个nav_menu.xml文件
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_call"
            android:icon="@drawable/nav_call"
            android:title="Call" />
        <item
            android:id="@+id/nav_friends"
            android:icon="@drawable/nav_friends"
            android:title="Friends" />
        <item
            android:id="@+id/nav_location"
            android:icon="@drawable/nav_location"
            android:title="Location" />
        <item
            android:id="@+id/nav_mail"
            android:icon="@drawable/nav_mail"
            android:title="Mail" />
        <item
            android:id="@+id/nav_task"
            android:icon="@drawable/nav_task"
            android:title="Tasks" />
    </group>
</menu>
```
group表示一个组，checkableBehavior 指定为single 表示组中的所有菜单项只能单选





+ headerLayout
用来在NavigationView中显示头部布局的,可以随意定制布局.
右击layout文件夹→New→Layout resource file，创建一个nav_header.xml文件
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:padding="10dp"
    android:background="?attr/colorPrimary">

    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/icon_image"
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true" />

    <TextView
        android:id="@+id/mail"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:text="tonygreendev@gmail.com"
        android:textColor="#FFF"
        android:textSize="14sp" />
	<TextView
        android:id="@+id/username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/mail"
        android:text="Tony Green"
        android:textColor="#FFF"
        android:textSize="14sp" />
</RelativeLayout>
```

+ NavigationView
修改布局文件
```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">“<FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

    </FrameLayout>

    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header "/>
</android.support.v4.widget.DrawerLayout>
```

+ activity
修改activity， 增加点击事件
```java
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        
        ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.setDisplayHomeAsUpEnabled(true);
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
        }

        NavigationView  navView = (NavigationView) findViewById(R.id.nav_view);
        navView.setCheckedItem(R.id.nav_call); //默认选中

        navView.setNavigationItemSelectedListener(new NavigationView.OnNavigation
            ItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(MenuItem item) {
                mDrawerLayout.closeDrawers(); //关闭
                return true;
            }
        });
    }

    ...

}
```

## 悬浮按钮和可交互提示
### 悬浮按钮 FloatingActionButton
实现悬浮按钮的效果
![](/images/android_fab.png)

```xml
<android.support.design.widget.FloatingActionButton
    android:id="@+id/fab"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|end"
    android:layout_margin="16dp"
    android:src="@drawable/ic_done"/>
```

app:elevation 属性来给FloatingActionButton指定一个高度值，高度值越大，投影范围也越大，但是投影效果越淡，高度值越小，投影范围也越小，但是投影效果越浓。
用默认值就可以


### 可交互提示 Snackbar
![](/images/android_snackbar.png)

可交互的Toast， 使用场景不同
```java
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ...
        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Data deleted", Snackbar.LENGTH_SHORT)
                        .setAction("Undo", new View.OnClickListener() {
                            @Override
                            public void onClick(View v) {
                                Toast.makeText(MainActivity.this, "Data restored",
                                    Toast.LENGTH_SHORT).show();
                            }
                        })
                        .show();
            }
        });
    }

    ...

}
```

Snackbar可能将我们的悬浮按钮给遮挡住, 可以使用CoordinatorLayout包裹fab

### CoordinatorLayout
加强版的FrameLayout， 可以监听其所有子控件的各种事件，然后自动帮助我们做出最为合理的响应。

```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

<android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:src="@drawable/ic_done" />
    </android.support.design.widget.CoordinatorLayout>

    ...

</android.support.v4.widget.DrawerLayout> 
```

## 卡片式布局











## 线性布局
## 相对布局
## 帧布局
## 表格布局
## 绝对布局


