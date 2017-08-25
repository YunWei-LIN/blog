title: Android UI
date: 2017-8-24 23:29:44
tags: [android, Android, UI, Material Design]
categories: Android
---

## 主要内容
* Material Design

*更新历史*


<!-- more -->


## 传统布局
### 线性布局
`LinearLayout`。

#### *material design* 
+ `AppBarLayout` 实际上是一个垂直方向的LinearLayout， AppBarLayout接收到滚动事件的时候,它内部的子控件可以指定通过app:layout_scrollFlags 属性去影响这些事件的
如 scroll|enterAlways|snap 。其中,scroll 表示当向上滚动的时候,Toolbar会跟着一起向上滚动并实现隐藏;enterAlways 表示当向下滚动的时候,Toolbar会跟着一起向下滚动并重新显示。snap 表示当Toolbar还没有完全隐藏或显示的时候,会根据当前滚动的距离,自动选择是隐藏还是显示。

### 相对布局


### 帧布局
`FrameLayout`， 默认所有控件在不进行明确定位的情况下,都会摆放在布局的左上角。

#### *material design* 
+ `CoordinatorLayout` 是一个增强型FrameLayout, 可以监听其所有子控件的各种事件,然后自动帮助我们做出最为合的响应。
+ `CardView` 也是一个FrameLayout, 只是额外提供了圆角和阴影等效果, 看上去会有立体的感觉。 


### 表格布局


### 绝对布局


## Material Design
材料主题
![](/images/android_ThemeColors.png)

### Toolbar
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

### 滑动菜单
滑动菜单就是将一些菜单选项隐藏起来,而不是放置在主屏幕上,然后可以通过滑动的方将菜单显示出来。
Google 提供 DrawerLayout， NavigationView 帮助我们简单就可实现滑动菜单的功能
![](/images/android_moveMenu.png)

#### DrawerLayout
它是一个布局,在布局中允许放入两个直接子控件,第一个子控件是主屏幕中显示的内容,第二个子控件是滑动菜单中显示的内容。

布局文件

```xml
<android.support.v4.widget.DrawerLayout 
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:id="@+id/drawer_layout"
android:layout_width="match_parent"
android:layout_height="match_parent">
<FrameLayout android:layout_width="match_parent" 
    android:layout_height="match_parent">
    
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

#### NavigationView
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
    android:layout_height="match_parent">
    <FrameLayout
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

### 悬浮按钮和可交互提示
#### 悬浮按钮 FloatingActionButton
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


#### 可交互提示 Snackbar
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

#### CoordinatorLayout
加强版的 FrameLayout ， 可以监听其所有子控件的各种事件，然后自动帮助我们做出最为合理的响应。

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

### 卡片式布局
列表 内卡片式 布局 ， 涉及以下控件：
+ CoordinatorLayout : 加强版的FrameLayout, 可以监听其所有子控件的各种事件，然后自动帮助我们做出最为合理的响应。
+ AppBarLayout : 垂直方向的LinearLayout， 子控件可以指定通过app:layout_scrollFlags 属性去影响AppBarLayout接受到的滚动事件
+ RecyclerView : 列表
+ CardView : FrameLayout, 额外提供了圆角和阴影等效果, 看上去会有立体的感觉



<div style="float:left;border:solid 1px 000;margin:2px;">
<img src="/images/android_cardview1.png" alt=""> 
</div>
<div style="float:left;border:solid 1px 000;margin:2px;">
 <img src="/images/android_cardview2.png" alt="">
</div>
<div style="clear:both;"></div>
<br>


#### 主布局

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
        
        <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    
            <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            app:layout_scrollFlags="scroll|enterAlways|snap"/>
            
        </android.support.design.widget.AppBarLayout>
        
        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior" />
            ...
    </android.support.design.widget.CoordinatorLayout>
    ...
</android.support.v4.widget.DrawerLayout>
```
- Toolbar 放置在了AppBarLayout里面,然后在RecyclerView中使用app:layout_behavior 属性指定一个布局行为。其中appbar_scrolling_view_behavior 这个字符串也是由Design Support库提供的。 
- 在Toolbar中添加了一个app:layout_scrollFlags 属性,并将这个属性的值指定成scroll|enterAlways|snap 。其中,scroll 表示当RecyclerView向上滚动的时候,Toolbar会跟着一起向上滚动并实现隐藏;enterAlways 表示当RecyclerView向下滚动的时候,Toolbar会跟着一起向下滚动并重新显示。snap 表示当Toolbar还没有完全隐藏或显示的时候,会根据当前滚动的距离,自动选择是隐藏还是显示。

#### RecyclerView的子项

```xml
<android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp">
    
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        
        <ImageView
            android:id="@+id/fruit_image"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop" />
        
        <TextView
            android:id="@+id/fruit_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp" />
    </LinearLayout>

</android.support.v7.widget.CardView>
```

- CardView来作为子项的最外层布局,从而使得RecyclerView中的每个元素都是在卡片当中的。CardView由于是一个FrameLayout,因此它没有什么方便的定位方式,这里我们只好CardView中再嵌套一个LinearLayout,然后在LinearLayout中放置具体的内容。
- 过app:cardCornerRadius 属性指定卡片圆角的弧度,数值越大,圆角的弧度也越大。另外还可以通过app:elevation 属性指定卡片的高度





### 下拉刷新 SwipeRefreshLayout
把想要实现下拉刷新功能的控件放置到SwipeRefreshLayout中,就可以迅速让这个控件支持下拉刷新。

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
        ...
        
        <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/swipe_refresh"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">
        
        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </android.support.v4.widget.SwipeRefreshLayout>
    ...
    </android.support.design.widget.CoordinatorLayout>
    ...
</android.support.v4.widget.DrawerLayout>
```

### 可折叠式标题栏  CollapsingToolbarLayout
CollapsingToolbarLayout是一个作用于Toolbar基础之上的布局,可以让Toolbar的效果变得更加丰富,不仅仅是展一个标题栏,而是能够实现非常华丽的效果。 不能独立存在的,只能作AppBarLayout的直接子布局。而AppBarLayout又必须是CoordinatorLayout的子布局.
![](/images/android_Ctb1.png)

```xml
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <android.support.design.widget.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp">
        
        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">
        
	        <ImageView
	            android:id="@+id/fruit_image_view"android:layout_width="match_parent"
	            android:layout_height="match_parent"
	            android:scaleType="centerCrop"
	            app:layout_collapseMode="parallax" />
	        
	        <android.support.v7.widget.Toolbar
	            android:id="@+id/toolbar"
	            android:layout_width="match_parent"
	            android:layout_height="?attr/actionBarSize"
	            app:layout_collapseMode="pin" />
        
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>
    
    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
            <android.support.v7.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="15dp"
            android:layout_marginLeft="15dp"
            android:layout_marginRight="15dp"
            android:layout_marginTop="35dp"
            app:cardCornerRadius="4dp">
            
            <TextView
                android:id="@+id/fruit_content_text"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_margin="10dp" />
            </android.support.v7.widget.CardView>
        </LinearLayout>
    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_comment"
        app:layout_anchor="@id/appBar"
        app:layout_anchorGravity="bottom|end" />

</android.support.design.widget.CoordinatorLayout>
 
 ```

+ app:contentScrim 属性
用于指定CollapsingToolbarLayout在趋于折叠状态以及折叠之后的背景色,CollapsingToolbarLayout在折叠之后就是一个普通的Toolbar,那么背景色肯定应该是colorPrimary了。
+ app:layout_scrollFlags 属性
scroll表示CollapsingToolbarLayout会随着水果内容详情的滚动一起滚动,exitUntilCollapsed 表示CollapsingToolbarLayout随着滚动完成折叠之后就保留在界面上,不再移出屏幕。
+ app:layout_collapseMode
它用于指定当控件在CollapsingToolbarLayout折叠过程中的折叠模式,其中Toolbar指定成pin,表示在折叠过程中位置始终保持不变,ImageView指定成parallax,表示会在折叠的过程中产生一定的错位偏移,这种模式的视觉效果会非常好。 
+ NestedScrollView
和AppBarLayout是平级, app:layout_behavior 属性指定了一个布局行为
+ FloatingActionButton
app:layout_anchor 属性指定了一个锚点,我们将锚点设置为AppBarLayout,这样悬浮按钮就会出现在水果标题栏的区域内,接着又使app:layout_anchorGravity 属性将悬浮按钮定位在标题栏区域的右下角。


### 充分利用系统状态栏空间
+ 背景图和状态栏融合到一起， 借助android:fitsSystemWindows=true。
+ 嵌套结构的布局中,所有父布局都设上这个属性才可以;
+ 程序的主题将状态栏颜色指定成透明色。android:statusBarColor 属性的值指定成@android:color/transparent;
+ AndroidManifest.xml中使用修改后的主题

![](/images/android_Ctb2.png)
title: Android入门 图谱
