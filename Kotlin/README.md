Kotlin
======

第一次知道kotlin这个语言是在JakeWharton的这个[dex-method-list][1] 项目里，本来这个项目用的是java后来大神又用Kotlin实现了一下，前两天1.0正式版发布了，被各种新闻刷了一下后有必要学习一下。

花了半天看完了[《Kotlin for Android》][2]这份文档，之前很多人说像Swift，看完这份文档后才发现它更像C#，比如带问号的可null类型，委托，lambda和的Linq类似，还有 explicit，implicit cast (协变逆变),sealed(密封类),里面还提供了和Linq To Sql 很像的Anko框架,以及getter，setter。

# Android下的Kotlin
按照惯例我们做一个"Helloworld"项目来尝尝鲜

## 环境配置
使用Kotlin开发Android应用需要安装Android Studio 和其配套的插件
需要安装Kotlin插件，是使Android Studio支持Kotlin的基础插件，在《Kotlin for Android》一书中说还需要 Kotlin Extensions For Android 插件，这个插件在最近的版本中已经obsolote，这个插件已经被集成到Kotlin插件里，所以不需要安装这个插件了。

![enter description here][3]

## 1. 使用AndroidStudio创建一个新项目

我是用的Android Studio是最新的2.0 Beta5，Gradle是2.10

![enter description here][4]

## 2. 将MainActivity类转换成Kotlin代码
这是一个很给力的特性，可以把Java代码无缝的转换成Kotlin代码。这种转换并不是最优化的，但是在学习Kotlin的时候这个方法很有用，可以对比之前java代码和Kotlin代码语法的转换。
先打开MainActivity类，然后有三种方式进行转换：
1. 选择`code`->`Convert File To Kotlin File`
2. 选择`Help`->`Find Action`或者使用快捷键 在弹出的对话框里输入`convert` 就会有提示，如下图
3. 直接使用快捷键`shift+alt+command`可以直接触发转换工具

![enter description here][5]

## 3. 配置Gradle
代码转换完成后会提示配置Kotlin，可以手动配置也可以使用Android Studio自带的配置工具进行配置。
暂不介绍手动配置，自动配置会在第一次代码转换完成后有个`configure`提示(如下图)，选择ok即可自动更新Gradle配置文件，非常方便
![enter description here][6]

配置完成后Gradle配置文件里会自动加上kotlin gradle的插件配置
![enter description here][7]

之后选择sync项目，将gradle插件下载下来即可运行项目了。
![enter description here][8]

## 4. 运行项目

Kotlin里有个对Android开发来说很好用的特性就是它剔除了findViewById()方法的调用。
### 配置gralde
使用这个特性需要在app的build.gradle文件配置里增加下面这个plugin引用。
```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
//增加这句
apply plugin: 'kotlin-android-extensions'

```
### 修改代码

将`activity_main.xml`修改如下

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="dodola.kotlinandroid1.MainActivity">

    <TextView
        android:id="@+id/helloText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />
</RelativeLayout>
```

修改MainActivity.kt代码如下：
```kotlin
package dodola.kotlinandroid1

import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        helloText.text="hello dodola"
    }
}

```
我们需要import进一个特殊的包，`import kotlinx.android.synthetic.main.activity_main.*` 这个包有个命名规则，后面再介绍。
从`helloText.text="hello dodola"`代码中可以看出不需要调用findViewById，也没有调用setText方法，而是直接使用text来进行赋值，这个使用的是Kotlin的一个叫做`Getter Setter`技术，这种机制类似C#里的`Accessors(访问器)`，后面再详细介绍，会发现很多特性都可以在C#里找到相应的技术，我之前是做.Net开发的，所以对C#相对敏感，后面会拿C#和Swift和Kotlin做对比，这样也有助于理解Kotlin里的一些技术。

















  [1]: https://github.com/JakeWharton/dex-method-list
  [2]: https://wangjiegulu.gitbooks.io/kotlin-for-android-developers-zh/content/index.html
  [3]: ./images/QQ20160218-7.png "QQ20160218-7.png"
  [4]: ./images/QQ20160218-0.png "QQ20160218-0.png"
  [5]: ./images/QQ20160218-1.png "QQ20160218-1.png"
  [6]: ./images/QQ20160218-4.png "QQ20160218-4.png"
  [7]: ./images/QQ20160218-5.png "QQ20160218-5.png"
  [8]: ./images/QQ20160218-6.png "QQ20160218-6.png"