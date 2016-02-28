Kotlin
======

第一次知道kotlin这个语言是在JakeWharton的这个[dex-method-list][1] 项目里，本来这个项目用的是java后来大神又用Kotlin实现了一下，前两天1.0正式版发布了，被各种新闻刷了一下后有必要学习一下。

花了半天看完了[《Kotlin for Android》][2]这份文档，之前很多人说像Swift，看完这份文档后才发现它更像C#，比如带问号的可null类型，委托，lambda和的Linq类似，还有 explicit，implicit cast (协变逆变),sealed(密封类),里面还提供了和Linq To Sql 很像的Anko框架,以及getter，setter。

# 一. Android下的Kotlin
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


# 二. Kotlin 介绍

下面介绍一下Kotlin语言的一些细节。内容是从官方文档和其他文章里总结得来的，目标是可以快速的学习Kotlin语言。

## 1. 基础
### 1.1 基本类型
Kotlin中没有像Java中的原始基本类型，基本类型都被当做对象形式存在。
#### 数值类型

Kotlin提供的数值类型如下：


| Type   | Bitwidth |
| ------ | -------- |
| Double | 64       |
| Float  | 32       |
| Long   | 64       |
| Int    | 32       |
| Short  | 16       |
| Byte   | 8        |

**需注意Char在Kotlin中不属于数值类型**

#### 字面值常量
```kotlin
整型：123
  长整型：123L //需要增加L后缀
16进制：0x0f
二进制：0b00001011
Double：123.5，123.5e10//默认是Double类型
Float：123.5f //Float类型需要增加F或f后缀

```
**注：不支持8进制字面值常量**

```kotlin
val i = 12 // An Int
val iHex = 0x0f // 一个十六进制的Int类型
val l = 3L // A Long
val d = 3.5 // A Double
val f = 3.5F // A Float

```

#### 显式转换

数值类型不会自动转型，必须要显示的进行类型转换，例子
```kotlin
//Byte -> Int
val b:Byte =1
val i:Int=b.toInt()
```

比如下面这种情况：

```kotlin
fun pow(a:Int):Double{
    return Math.pow(a.toDouble(),2);//此处会出现Kotlin: The integer literal does not conform to the expected type kotlin.Double 的编译错误
}
```
上面例子`pow(double a, double b) ` 传入的是两个Double类型，但是直接写入Int类型并不会进行隐式转换。需要做一次显示转换

```kotlin
fun pow(a:Int):Double{
    return Math.pow(a.toDouble(),2.toDouble());
}

```

每个数值类型都支持下面转换：

```kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```


隐式转换，基于运算符重载技术，例子：

```kotlin
val l=1.toLong() + 1 //Long+Int=>Long

```
#### 字符类型Char

字符类型使用Char表示，不能当成数值使用。

和Java一样，字符类型值是用单引号包起来的`1`,'\n'，可以显示转换成`Int`类型

```kotlin
val c:Char='c'
val i: Int = c.toInt()
```

#### 布尔值

```kotlin
val booltest=true
```

#### 字符串

使用`String`声明，Kotlin中有两种样式的String

一种是和Java类似的声明：

```kotlin
val s="Hello world"

```

一种是多行形式的表示

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""

```
运行结果如下图：
![enter description here][9]


可以像数组那样访问，并且可以被迭代：

```kotlin
val s = "Example"
val c = s[2] // 这是一个字符'a'
// 迭代String
val s = "Example"
for(c in s){
    print(c)
}
```

#### 模板

字符串可以包含模板表达式，模板表达式由一个 $ 开始并包含另一个简单的名称：

```kotlin
val i = 10
val s = "i = $i" 
//结果为 i = 10

```

或者是一个带大括号的表达式：+

```kotlin
val s = "abc"
val str = "$s.length is ${s.length}"
//结果为 abc.length is 3
```
## 1.2 数组

### 创建数组

指定长度 

```kotlin
    val sizeArray=arrayOfNulls<Int>(5)//sizeArray: Integer[5]{null}
    val sizeArray1=IntArray(5)//sizeArray1: {0,0,0,0,0}
    //同样的类型还有:BooleanArray,ByteArray,IntArray,CharArray,DoubleArray,FloatArray,LongArray,ShortArray,
```
使用值创建

```kotlin
 //使用装箱操作
    val sizeArray2=arrayOf(1,2,3,4,5)
    /**
    sizeArray2: Integer[]
    0 = {Integer@447} "1"
    1 = {Integer@448} "2"
    2 = {Integer@449} "3"
    3 = {Integer@450} "4"
    4 = {Integer@451} "5"
    **/
    //下面方法创建的数组是未装箱的
    val sizeArray3=intArrayOf(1,2,3,4,5)//sizeArray3:int[5] {1,2,3,4,5}
    //同类的方法还有
    //booeanArrayOf,byteArrayOf,doubleArrayOf,floatArrayOf,intArrayOf,longArrayOf,shortArrayOf
```
空数组

```kotlin
    val emptyArray=emptyArray<Int>()//emptyArray: Integer[0]
```
### 访问数组

```kotlin
val arr = arrayOf(1, 2, 3)
println(asc[1])         //  1
println(asc.get(1))     //  1
```

### 遍历数组

```kotlin
for(i in arr){
    println(i)
}
//1 2 3
for(j in asc.indices){
    println(j)
}
//0 1 2 
```


## 1.3 基本语法

### 定义包名

```kotlin
package my.demo //在源文件的最开头
import java.util. 
```

**包名不必和文件夹路径一致**

### 定义类

```kotlin
class MainActivity{
//包含一个默认的构造器
}
```

构造方法

```kotlin
class Datas{
    
}
```

### 定义方法

```kotlin
//第一种形态
fun pow(a:Int):Double{
    return Math.pow(a.toDouble(),2.toDouble());
}
//第二种形态，一个表达式函数体和一个可推断类型
fun pow(a:Int)= Math.pow(a.toDouble(),2.toDouble());

```

### 定义局部变量

用`val`声明只读变量：

```kotlin

//-------in kotlin---------
val a:Int=1
val b = 1
val c:Int //声明
c=1
c=2//ERROR: val 只能赋值一次，这里会造成编译错误

//--------in java---------

final int a=1;

```

使用`var`声明可变变量：

```kotlin
//--------in kotlin-------
var x=5
x+=1


//--------in java ---------
int x=5;
```

### 字符串模板

Kotlin允许在字符串中嵌入**变量**和**表达式**：

```kotlin
val name = "Bob"
println("My name is ${name}") //打印"My name is Bob"

val a = 10
val b = 20
println("The sum is ${a+b}") //打印"The sum is 30"
```

### if 表达式

```kotlin
fun max(a: Int, b: Int): Int {
    if (a > b)
        return a
    else
        return b
}

fun max(a: Int,  b: Int) = if (a > b) a else b

```

### when表达式

用于替代`switch`，但功能更强大

```kotlin
val age = 17

val typeOfPerson = when(age){
    0 -> "New born"
    in 1..12 -> "Child"
    in 13..19 -> "Teenager"
    else -> "Adult"
}

fun cases(obj: Any) {
    when (obj) {
        1 -> print("one")
        "hello" -> print("Greeting")
        is Long -> print("Long")
        !is Long -> print("Not a string")
        else -> print("Unknown")
    }
}

```
**注：else是必须的，相当于switch里的default**

### for 循环

```kotlin
fun sum(a: IntArray): Int {
    var sumValue=0;
    for(i in a){
        sumValue+=i
    }
    return sumValue
}
//或者

fun sum(a: IntArray): Int {
    var sumValue=0;
    for(i in a.indices){
        sumValue+=a[i]
    }
    return sumValue
}
```

### while do ..while 循环

while和do..while循环的语法与Java完全相同。


```kotlin

fun sum(a: IntArray): Int {
    var sumValue=0;
    
    var i=0
    while(i<a.size){
        sumValue+=a[i++]
    }
    
    return sumValue
}

```

### 范围表达式

在Kotlin中，范围创建只需要`..`操作符，例如：

```kotlin
val r1 = 1..5
//该范围包含数值1,2,3,4,5

```
如果创建一个降序范围，则需要使用downTo函数

```kotlin
val r2 = 5 downTo 1
//该范围包含数值5,4,3,2,1
```
默认范围的步长是1，可使用step方法修改步长：

```kotlin
val r3 = 5 downTo 1 step 2
//该范围包含数值5,3,1
```

可使用`in`操作符检查数值是否在某个范围内

```kotlin
if (x in 1..y-1)
    print("OK")

```
范围外：

```kotlin
if (x !in 0..array.lastIndex)
    print("Out")
```

### Lambda

```kotlin
val sumLambda: (Int, Int) -> Int = {x,y -> x+y}
```

## 1.4 包

### 声明包
```kotlin
package foo.bar

```
如果没有指定包名，那这个文件的内容就从属于没有名字的 "default" 包。

### import

```kotlin
import foo.Bar //Bar 现在可以不用条件就可以使用
//或者范围内的所有可用的内容 (包，类，对象，等等):
import foo.*/ /foo 中的所有都可以使用
//如果命名有冲突，我们可以使用 as 关键字局部重命名解决冲突
import foo.Bar // Bar 可以使用
import bar.Bar as bBar // bBar 代表 'bar.Bar'
```

## 1.5 控制流

### if表达式

```kotlin
//传统用法
var max = a
if (a < b)
    max = b

//带 else 
var max: Int
if (a > b)
    max = a
else
    max = b

//作为表达式
val max = if (a > b) a else b

//if分之可以作为块，最后一个表达式是该块的值

val max = if (a > b){
    print("Choose a")
    a
}
else{
    print("Choose b")
    b
}

```

### when 表达式

一般用法,用来替代 switch,需注意的是里面的条件可以重复,如果条件重复的话则按照最先匹配的条件处理
```kotlin
	val x=2
	when(x){
		1->print("1")
		2->print("2")
		else->print("else")//文档里写else 是mandatory的,但是去掉也可以
	}
	
```

如果多个条件的处理方式一样,可以写作下面方式:

```kotlin
when (x) {
    0,1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

可以用任意表达式作为分支的条件
```kotlin
when (x) {
    parseInt(s) -> print("s encode x")
    else -> print("s does not encode x")
}
```
可以用 in 或者 !in 检查值是否值在一个集合中：
```kotin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```
可以使用 is 判断是否是某个类型:

```kotlin
//官方的这个例子写的很奇怪,感觉为了使用 when 来故意这么写的

val hasPrefix = when (x) {
    is String -> x.startsWith("prefix")
    else -> false
}
//上面的例子可以直接写成
val hasPrefix=x.startsWith("prefix")


//下面的例子可能更好一些
	var a=B()
		when(a){
			is C->print("A")
			is B->print("B")
			is A->print("C")
		}

    open class A{
    	
    }
    open class B:A(){
    	
    }
    class C:B(){
    	
    }


```
when 也可以用来代替 if-else, 如果没有提供任何参数则分支的条件就是表达式
```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```
测试例子:

```kotlin
fun main(args: Array<String>) {
	var x=3
	when(x){
		
		1->println("1->1")
		2->{
			x=3
			println("2->${x}")
		}
		3,4,5->println("3,4,5->")
		test()->println("test()->${test()}")
		in 7..10->println("6..10->${x}")
		}	
		
		var a=B()
		when(a){
			is C->print("A")
			is B->print("B")
			is A->print("C")
		}
		

}

open class A{
	
}
open class B:A(){
	
}
class C:B(){
	
}

fun test():Int{
	println("=====test====")
	return 6
}

```
### for 
```kotlin
for (item in collection)
    print(item)
    
for (i in array.indices)
    print(array[i])
```

### while
```kotlin
```


## 返回与跳转

##标签

标签相当于 C语言 中的 label,通过@结尾来表示,比如 `abc@`

```kotlin
loop@ for(i in 1..100){
    //
}
```
声明了标签之后可以使用 break 或者 continue 进行跳转:







  [1]: https://github.com/JakeWharton/dex-method-list
  [2]: https://wangjiegulu.gitbooks.io/kotlin-for-android-developers-zh/content/index.html
  [3]: ./images/QQ20160218-7.png "QQ20160218-7.png"
  [4]: ./images/QQ20160218-0.png "QQ20160218-0.png"
  [5]: ./images/QQ20160218-1.png "QQ20160218-1.png"
  [6]: ./images/QQ20160218-4.png "QQ20160218-4.png"
  [7]: ./images/QQ20160218-5.png "QQ20160218-5.png"
  [8]: ./images/QQ20160218-6.png "QQ20160218-6.png"