#Java新特性#

##switch语句中的String##

在Java 6 之前 case 语句的常量只能是 byte、char、short和int或者枚举常量。Java7规范中增加了String。

```java
 private void testSwitch(String val) {
        switch (val) {
            case "doc":
                System.out.println("doc");
                break;
            case "music":
                System.out.println("music");
                break;
            case "video":
                System.out.println("video");
                break;
        }
    }
```

##数值文本表示法##

###二进制文本##
Java7之前要处理二进制，必须调用`Integer.parseInt()`方法

```java
int x=Integer.parseInt("1100100",2);
```

Java7 中可以写成：

```java
int x=0b1100100;
```
###数字中的下划线###

由于很长的数字不易阅读，java7 中增加了数字的分隔线来增加长数字的可阅读性

比如10000000可以写成10_000_000


##异常处理增强##

异常处理有两处改进 `multicatch`和`final`重抛。

在java7之前一个代码块里可能抛出多个可检测异常，则需要写多个catch，举例：



在java7 中可以将多个合并到一个catch里 ，举例：





















