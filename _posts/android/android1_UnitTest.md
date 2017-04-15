title: Android入门 一 ： unit test
date: 2016-5-10 19:29:44
tags: 
 - android
 - junit
categories: Android
---

主要内容
* Local Unit Test
* Instrumented Unit Test

本文主要以idea 说明如何进行 Android Unit Test。
分为与Android环境相关的`Instrumented Unit Test` ， 与Android环境无关的`Local Unit Test`

<!-- more -->



## Local Unit Test
`Local Unit Test` 用来做跟android环境无关的Unit Test。
### 配置依赖
`build.gradle` 中增加必要的依赖

```
dependencies {
    ...
    testCompile 'junit:junit:4.12'
    androidTestCompile 'junit:junit:4.12'
}
```

### Test java class
以 `JUnit 4` 为基础的话， 仅需要 用 `@Test` 注解测试method。
android官方例子像这样的：

```
import org.junit.Test;
import java.util.regex.Pattern;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;

public class EmailValidatorTest {

    @Test
    public void emailValidator_CorrectEmailSimple_ReturnsTrue() {
        assertThat(EmailValidator.isValidEmail("name@email.com"), is(true));
    }
    ...
}
```

### 运行
别忘记将 `/src/test/java` 设置成 `Test Source Root`
* 同步
`project` 视图下， 右键工程， `synchronize ${your project name}`

* 配置BuildVariants
`Test Artifact` 选择 `Unit Tests`

* 运行
右键测试class， run ！

![](http://amadis.qiniudn.com/androidJunit1a1.png)




## Instrumented Unit Test
如果你的测试需要访问 `instrumentation information` (例如 app's Context) 或者需要 `Android framework component` (例如 Parcelable  SharedPreferences)，那么你可以使用 `Instrumented Unit Test`。
目前有2中方式实现： `AndroidJUnit4` 以及 `TestCase`


### AndroidJUnit4
这是Android官方的例子， 需要注意的是如需要取 `context` 必须使用 `getTargetContext()`
> InstrumentationRegistry is an exposed registry instance that holds a reference to the instrumentation running in the process and it's arguments and allows injection of the following instances:

> * InstrumentationRegistry.getInstrumentation(), returns the Instrumentation currently running.
> * InstrumentationRegistry.getContext(), returns the Context of this Instrumentation’s package.
> * InstrumentationRegistry.getTargetContext(), returns the application Context of the target application.
> * InstrumentationRegistry.getArguments(), returns a copy of arguments Bundle that was passed to this Instrumentation. This is useful when you want to access the command line arguments passed to Instrumentation for your test.


#### 配置依赖
* 在顶层`build.gradle` 中增加必要的依赖

```
dependencies {
    androidTestCompile 'com.android.support:support-annotations:23.0.1'
    androidTestCompile 'com.android.support.test:runner:0.4.1'
    androidTestCompile 'com.android.support.test:rules:0.4.1'
    // Optional -- Hamcrest library
    androidTestCompile 'org.hamcrest:hamcrest-library:1.3'
    // Optional -- UI testing with Espresso
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
    // Optional -- UI testing with UI Automator
    androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.1'
}
```

* 在模块`build.gradle` 中配置 `instrumentation runner`

```
android {
    defaultConfig {
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}
```

#### Test java class
`instrumentation Test` 应该使用 Junit4 的方式编写， 代码默认路径为 `src/androidTest/java`。
Android 的官方例子如下：

```
import android.os.Parcel;
import android.support.test.runner.AndroidJUnit4;
import android.util.Pair;
import org.junit.Test;
import org.junit.runner.RunWith;
import java.util.List;
import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

@RunWith(AndroidJUnit4.class)
@SmallTest
public class LogHistoryAndroidUnitTest {

    public static final String TEST_STRING = "This is a string";
    public static final long TEST_LONG = 12345678L;
    private LogHistory mLogHistory;

    //前处理
    @Before
    public void createLogHistory() {
        mLogHistory = new LogHistory();
    }

    @Test
    public void logHistory_ParcelableWriteRead() {
        // Set up the Parcelable object to send and receive.
        mLogHistory.addEntry(TEST_STRING, TEST_LONG);

        // Write the data.
        Parcel parcel = Parcel.obtain();
        mLogHistory.writeToParcel(parcel, mLogHistory.describeContents());

        // After you're done with writing, you need to reset the parcel for reading.
        parcel.setDataPosition(0);

        // Read the data.
        LogHistory createdFromParcel = LogHistory.CREATOR.createFromParcel(parcel);
        List<Pair<String, Long>> createdFromParcelData = createdFromParcel.getData();

        // Verify that the received data is correct.
        assertThat(createdFromParcelData.size(), is(1));
        assertThat(createdFromParcelData.get(0).first, is(TEST_STRING));
        assertThat(createdFromParcelData.get(0).second, is(TEST_LONG));
    }
    
    //后处理
    @After
    
}
```

### TestCase
继承 `android.test` 包下面一系列 `xxxxTestCase`。
IDEA 自动生存的例子：
```
package com.example.sam.myapplication;

import android.app.Application;
import android.test.ApplicationTestCase;
import org.junit.Test;

/**
 * <a href="http://d.android.com/tools/testing/testing_android.html">Testing Fundamentals</a>
 */
public class ApplicationTest extends ApplicationTestCase<Application> {
    public ApplicationTest() {
        super(Application.class);
    }

    //前处理
    @Override
    protected void setUp() throws Exception {
        super.setUp();
        
        // 增加你自己的前处理
    }
    
    @Test
    public void test(){
        System.out.println("ok              ok");
    }
    
    //后处理
        @Override
    protected void tearDown() throws Exception {
        super.tearDown();
        
        //增加你自己的后处理
    }

    
    
}
```

### 运行
别忘记将 `src/androidTest/java` 设置成 `Test Source Root`
* 同步
`project` 视图下， 右键工程， `synchronize ${your project name}`

* 配置BuildVariants
`Test Artifact` 选择 `Android instrumentation Tests`

* 运行
右键测试class，  `Android Test` 方式运行！ 在 `Local Unit Test` 的 下一格，带Android机器人图标的。
run ！

![](http://amadis.qiniudn.com/androidJunit1a2.png)


## 官方链接
[Best Practices for Testing](http://developer.android.com/training/testing/index.html)

