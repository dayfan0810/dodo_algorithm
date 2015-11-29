<!-- toc -->
转载自：

[Java NIO 系列教程](http://ifeve.com/java-nio-all/)

[NIO入门](http://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)

部分内容从《Java 编程思想》(第四版)中第十八章内容补充的。
#Java NIO 介绍
##I/O 简介
I/O ? 或者输入/输出 ? 指的是计算机与外部世界或者一个程序与计算机的其余部分的之间的接口。它对于任何计算机系统都非常关键，因而所有 I/O 的主体实际上是内置在操作系统中的。单独的程序一般是让系统为它们完成大部分的工作。

在 Java 编程中，直到最近一直使用 流 的方式完成 I/O。所有 I/O都被视为单个的字节的移动，通过一个称为 Stream 的对象一次移动一个字节。流 I/O 用于与外部世界接触。它也在内部使用，用于将对象转换为字节，然后再转换回对象。

##什么是Java NIO
NIO 是 New IO 的简称，在JDK 1.4 里提供的新 API， NIO 弥补了原来的I/O的不足，它在标准Java代码中提供了高速的、面向块的 I/O 。
通过定义包含数据的类，以及通过以块的形式处理这些数据，NIO  不用使用本机代码就可以利用低级优化，这是原来的 I/O 包所无法做到的。

##为什么要使用 NIO?
NIO 的创建目的是为了让 Java 程序员可以实现高速 I/O 而无需编写自定义的本机代码。NIO 将最耗时的 I/O 操作(即填充和提取缓冲区)转移回操作系统，因而可以极大地提高速度。

##流与块的比较
原来的 I/O 库(在 java.io.*中) 与 NIO 最重要的区别是数据打包和传输的方式。正如前面提到的，原来的 I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。
面向流 的 I/O 系统一次一个字节地处理数据。一个输入流产生一个字节的数据，一个输出流消费一个字节的数据。为流式数据创建过滤器非常容易。链接几个过滤器，以便每个过滤器只负责单个复杂处理机制的一部分，这样也是相对简单的。不利的一面是，面向流的 I/O 通常相当慢。
一个 面向块 的 I/O 系统以块的形式处理数据。每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按(流式的)字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

##NIO相关组件

Java NIO 主要由下面几个核心组件组成：

 - Channels（通道）
 - Buffers（缓冲区）
 - Selectors（）

Java NIO 有比这更多的类和组件，但是Channel，Buffer和Selector组成核心API。其余的组件，像 Pipe 和 FileLock 仅仅是用来结合核心组件使用的实用工具类，因此我会将描述重点放在这三个核心组件上，其他的组件会在其他章节中单独介绍。

##Channels and Buffers

通常NIO里所有的输入输出(IO)都通过一个Channel，Channel和流类似，从Channel里可以读取数据到一个Buffer中，数据也可以通过Buffer写入到一个Channel中，如下图：
![Channel到Buffer，Buffer到Channel](http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png)

Channel和Buffer有很多中不同的类型，下面是NIO里主要的几个类：

 - FileChannel
 - DatagramChannel
 - SocketChannel
 - ServerSocketChannel

见名思意，这几个类覆盖了UDP+TCP 的网络IO和文件IO。

下面是NIO里一个常用的 Buffer 类：

 - ByteBuffer
 - CharBuffer
 - DoubleBuffer
 - FloatBuffer
 - IntBuffer
 - LongBuffer
 - ShortBuffer

这些类覆盖了你能发送接收的基础类型：byte,short,int,long,float,double，char
Java NIO里也包括一种名为MappedByteBuffer的类。[TODO]

##Selectors

一个Selector 允许一个线程去操作多个 Channel，如果你的应用里有很多连接(Channels)打开，并且每个连接流量比较低的情况下比较方便操作。比如聊天服务。

下面这个图描述了一个Selector 操作三个Channel：
![enter image description here](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)

要在Selector中注册一个Channel你需要调用`select()` 方法，这个方法会阻塞住直到准备好一个已经注册的通道。当方法返回后线程可以处理这些事件，比如建立一个连接，接收数据等等。

#Java NIO Channel

##什么是通道？
Channel是一个对象，可以通过它读取和写入数据。拿 NIO 与原来的 I/O 做个比较，通道就像是流。
正如前面提到的，所有数据都通过 Buffer 对象来处理。您永远不会将字节直接写入通道中，相反，您是将数据写入包含一个或者多个字节的缓冲区。同样，您不会直接从通道中读取字节，而是将数据从通道读入缓冲区，再从缓冲区获取这个字节。

Java NIO Channel与IO中的流相似，但是以下区别：
- 通道与流的不同之处在于通道是双向的。而流只是在一个方向上移动(一个流必须是 `InputStream` 或者 `OutputStream` 的子类)， 而 通道 可以用于读、写或者同时用于读写。
- 通道可以异步地读写。
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。

正如上面所说，从通道读取数据到缓冲区，从缓冲区写入数据到通道。如下图所示：
![enter image description here](http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png)

##Channel的实现

这些是Java NIO中最重要的通道的实现：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel
- FileChannel 从文件中读写数据。

DatagramChannel 能通过UDP读写网络中的数据。
SocketChannel 能通过TCP读写网络中的数据。
ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

##基本的 Channel 示例
下面是一个使用FileChannel读取数据到Buffer中的示例：
```java
 RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
    FileChannel inChannel = aFile.getChannel();

    ByteBuffer buf = ByteBuffer.allocate(48);

    int bytesRead = inChannel.read(buf);
    while (bytesRead != -1) {

      System.out.println("Read " + bytesRead);
      buf.flip();

      while(buf.hasRemaining()){
          System.out.print((char) buf.get());
      }

      buf.clear();
      bytesRead = inChannel.read(buf);
    }
    aFile.close();
```
注意 buf.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据

# Buffer

##什么是缓冲区

Buffer 是一个对象， 它包含一些要写入或者刚读出的数据。 在 NIO 中加入 Buffer 对象，体现了新库与原 I/O 的一个重要区别。在面向流的 I/O 中，您将数据直接写入或者将数据直接读到 Stream 对象中。
在 NIO 库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的。在写入数据时，它是写入到缓冲区中的。任何时候访问 NIO 中的数据，您都是将它放到缓冲区中。
缓冲区实质上是一个数组。通常它是一个字节数组，但是也可以使用其他种类的数组。但是一个缓冲区不 仅仅 是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。
##缓冲区类型
最常用的缓冲区类型是 ByteBuffer。一个 ByteBuffer 可以在其底层字节数组上进行 get/set 操作(即字节的获取和设置)。
ByteBuffer 不是 NIO 中唯一的缓冲区类型。事实上，对于每一种基本 Java 类型都有一种缓冲区类型：
- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer
每一个 Buffer 类都是 Buffer 接口的一个实例。 除了 ByteBuffer，每一个 Buffer 类都有完全一样的操作，只是它们所处理的数据类型不一样。因为大多数标准 I/O 操作都使用 ByteBuffer，所以它具有所有共享的缓冲区操作以及一些特有的操作。

##Buffer的基本用法


使用Buffer读写数据一般遵循以下四个步骤：

1. 写入数据到Buffer
2. 调用 `buffer.flip()`方法
3. 从Buffer中读取数据
4. 调用`buffer.clear()`方法或者`buffer.compact()`方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

下面是一个使用Buffer的例子：
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

##Buffer内部细节
缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

###概述
 NIO 中有两个重要的缓冲区组件：状态变量和访问方法 (accessor)。
状态变量是前一节中提到的"内部统计机制"的关键。每一个读/写操作都会改变缓冲区的状态。通过记录和跟踪这些变化，缓冲区就可能够内部地管理自己的资源。
在从通道读取数据时，数据被放入到缓冲区。在有些情况下，可以将这个缓冲区直接写入另一个通道，但是在一般情况下，您还需要查看数据。这是使用 访问方法 get() 来完成的。同样，如果要将原始数据放入缓冲区中，就要使用访问方法 put()。

###状态变量
可以用三个值指定缓冲区在任意时刻的状态：
* position
* limit
* capacity

这三个变量一起可以跟踪缓冲区的状态和它所包含的数据

**Position**

您可以回想一下，缓冲区实际上就是美化了的数组。在从通道读取时，您将所读取的数据放到底层的数组中。 position 变量跟踪已经写了多少数据。更准确地说，它指定了下一个字节将放到数组的哪一个元素中。因此，如果您从通道中读三个字节到缓冲区中，那么缓冲区的 position 将会设置为3，指向数组中第四个元素。
同样，在写入通道时，您是从缓冲区中获取数据。 position 值跟踪从缓冲区中获取了多少数据。更准确地说，它指定下一个字节来自数组的哪一个元素。因此如果从缓冲区写了5个字节到通道中，那么缓冲区的 position 将被设置为5，指向数组的第六个元素。

**Limit**

limit 变量表明还有多少数据需要取出(在从缓冲区写入通道时)，或者还有多少空间可以放入数据(在从通道读入缓冲区时)。
position 总是小于或者等于 limit。

**Capacity**

缓冲区的 capacity 表明可以储存在缓冲区中的最大数据容量。实际上，它指定了底层数组的大小 ― 或者至少是指定了准许我们使用的底层数组的容量。
limit 决不能大于 capacity。

###观察变量

我们首先观察一个新创建的缓冲区。出于本例子的需要，我们假设这个缓冲区的 总容量 为8个字节。 Buffer 的状态如下所示：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure1.gif)
回想一下 ，limit 决不能大于 capacity ，此例中这两个值都被设置为 8。我们通过将它们指向数组的尾部之后(如果有第8个槽，则是第8个槽所在的位置)来说明这点。
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure2.gif)

position 设置为0。如果我们读一些数据到缓冲区中，那么下一个读取的数据就进入 slot 0 。如果我们从缓冲区写一些数据，从缓冲区读取的下一个字节就来自 slot 0 。 position 设置如下所示：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure3.gif)

由于 capacity 不会改变，所以我们在下面的讨论中可以忽略它。

**第一次读取**

现在我们可以开始在新创建的缓冲区上进行读/写操作。首先从输入通道中读一些数据到缓冲区中。第一次读取得到三个字节。它们被放到数组中从 position 开始的位置，这时 position 被设置为 0。读完之后，position 就增加到 3，如下所示：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure4.gif)
limit 没有改变。

**第二次读取**

在第二次读取时，我们从输入通道读取另外两个字节到缓冲区中。这两个字节储存在由 position 所指定的位置上， position 因而增加 2：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure5.gif)

limit 没有改变。

**flip**

现在我们要将数据写到输出通道中。在这之前，我们必须调用 flip() 方法。这个方法做两件非常重要的事：
它将 limit 设置为当前 position。
它将 position 设置为 0。
前一小节中的图显示了在 flip 之前缓冲区的情况。下面是在 flip 之后的缓冲区：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure6.gif)

我们现在可以将数据从缓冲区写入通道了。 position 被设置为 0，这意味着我们得到的下一个字节是第一个字节。 limit 已被设置为原来的 position，这意味着它包括以前读到的所有字节，并且一个字节也不多。

**第一次写入**

在第一次写入时，我们从缓冲区中取四个字节并将它们写入输出通道。这使得 position 增加到 4，而 limit 不变，如下所示：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure7.gif)

**第二次写入**

我们只剩下一个字节可写了。 limit在我们调用 flip() 时被设置为 5，并且 position 不能超过 limit。所以最后一次写入操作从缓冲区取出一个字节并将它写入输出通道。这使得 position 增加到 5，并保持 limit 不变，如下所示：
![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure8.gif)


**clear**

最后一步是调用缓冲区的 clear() 方法。这个方法重设缓冲区以便接收更多的字节。 Clear 做两种非常重要的事情：
它将 limit 设置为与 capacity 相同。
它设置 position 为 0。
下图显示了在调用 clear() 后缓冲区的状态：

![](http://www.ibm.com/developerworks/cn/education/java/j-nio/figure9.gif)
缓冲区现在可以接收新的数据了。

###访问方法

到目前为止，我们只是使用缓冲区将数据从一个通道转移到另一个通道。然而，程序经常需要直接处理数据。例如，您可能需要将用户数据保存到磁盘。在这种情况下，您必须将这些数据直接放入缓冲区，然后用通道将缓冲区写入磁盘。
或者，您可能想要从磁盘读取用户数据。在这种情况下，您要将数据从通道读到缓冲区中，然后检查缓冲区中的数据。
在本节的最后，我们将详细分析如何使用 ByteBuffer 类的 get() 和 put() 方法直接访问缓冲区中的数据。

**get() 方法**

ByteBuffer 类中有四个 get() 方法：
```java
byte get();
ByteBuffer get( byte dst[] );
ByteBuffer get( byte dst[], int offset, int length );
byte get( int index );
```
第一个方法获取单个字节。第二和第三个方法将一组字节读到一个数组中。第四个方法从缓冲区中的特定位置获取字节。那些返回 ByteBuffer 的方法只是返回调用它们的缓冲区的 this 值。

此外，我们认为前三个 get() 方法是相对的，而最后一个方法是绝对的。 相对 意味着 get() 操作服从 limit 和 position 值 ― 更明确地说，字节是从当前 position 读取的，而 position 在 get 之后会增加。另一方面，一个 绝对 方法会忽略 limit 和 position 值，也不会影响它们。事实上，它完全绕过了缓冲区的统计方法。

上面列出的方法对应于 ByteBuffer 类。其他类有等价的 get() 方法，这些方法除了不是处理字节外，其它方面是是完全一样的，它们处理的是与该缓冲区类相适应的类型。

**put()方法**

ByteBuffer 类中有五个 put() 方法：
```java
ByteBuffer put( byte b );
ByteBuffer put( byte src[] );
ByteBuffer put( byte src[], int offset, int length );
ByteBuffer put( ByteBuffer src );
ByteBuffer put( int index, byte b );
```
第一个方法 写入（put） 单个字节。第二和第三个方法写入来自一个数组的一组字节。第四个方法将数据从一个给定的源 ByteBuffer 写入这个 ByteBuffer。第五个方法将字节写入缓冲区中特定的 位置 。那些返回 ByteBuffer 的方法只是返回调用它们的缓冲区的 this 值。
与 get() 方法一样，我们将把 put() 方法划分为 相对 或者 绝对 的。前四个方法是相对的，而第五个方法是绝对的。
上面显示的方法对应于 ByteBuffer 类。其他类有等价的 put() 方法，这些方法除了不是处理字节之外，其它方面是完全一样的。它们处理的是与该缓冲区类相适应的类型。
类型化的 get() 和 put() 方法
除了前些小节中描述的 get() 和 put() 方法， ByteBuffer 还有用于读写不同类型的值的其他方法，如下所示：
* getByte()
* getChar()
* getShort()
* getInt()
* getLong()
* getFloat()
* getDouble()
* putByte()
* putChar()
* putShort()
* putInt()
* putLong()
* putFloat()
* putDouble()

事实上，这其中的每个方法都有两种类型 ― 一种是相对的，另一种是绝对的。它们对于读取格式化的二进制数据（如图像文件的头部）很有用。

缓冲区的使用：一个内部循环
下面的内部循环概括了使用缓冲区将数据从输入通道拷贝到输出通道的过程。
```java
while (true) {
     buffer.clear();
     int r = fcin.read( buffer );

     if (r==-1) {
       break;
     }

     buffer.flip();
     fcout.write( buffer );
}
```
read() 和 write() 调用得到了极大的简化，因为许多工作细节都由缓冲区完成了。 clear() 和 flip() 方法用于让缓冲区在读和写之间切换。



## 关于缓冲区的更多内容

### 概述
本节将讨论使用缓冲区的一些更复杂的方面，比如缓冲区分配、包装和分片。我们还会讨论 NIO 带给 Java 平台的一些新功能。您将学到如何创建不同类型的缓冲区以达到不同的目的，如可保护数据不被修改的 只读 缓冲区，和直接映射到底层操作系统缓冲区的 直接 缓冲区。我们将在本节的最后介绍如何在 NIO 中创建内存映射文件。

### 缓冲区分配和包装

在能够读和写之前，必须有一个缓冲区。要创建缓冲区，您必须 分配 它。我们使用静态方法 allocate() 来分配缓冲区：
```java
ByteBuffer buffer = ByteBuffer.allocate( 1024 );
```
allocate() 方法分配一个具有指定大小的底层数组，并将它包装到一个缓冲区对象中 ― 在本例中是一个 ByteBuffer。
您还可以将一个现有的数组转换为缓冲区，如下所示：
```java
byte array[] = new byte[1024];
ByteBuffer buffer = ByteBuffer.wrap( array );
```
本例使用了 wrap() 方法将一个数组包装为缓冲区。必须非常小心地进行这类操作。一旦完成包装，底层数据就可以通过缓冲区或者直接访问。

### 缓冲区分片

slice() 方法根据现有的缓冲区创建一种 子缓冲区 。也就是说，它创建一个新的缓冲区，新缓冲区与原来的缓冲区的一部分共享数据。
使用例子可以最好地说明这点。让我们首先创建一个长度为 10 的 ByteBuffer：

`ByteBuffer buffer = ByteBuffer.allocate( 10 );`

然后使用数据来填充这个缓冲区，在第 n 个槽中放入数字 n：

```java
for (int i=0; i<buffer.capacity(); ++i) {
     buffer.put( (byte)i );
}
```

现在我们对这个缓冲区 分片 ，以创建一个包含槽 3 到槽 6 的子缓冲区。在某种意义上，子缓冲区就像原来的缓冲区中的一个 窗口 。
窗口的起始和结束位置通过设置 position 和 limit 值来指定，然后调用 Buffer 的 slice() 方法：

```java
buffer.position( 3 );
buffer.limit( 7 );
ByteBuffer slice = buffer.slice();
```

片 是缓冲区的 子缓冲区 。不过， 片段 和 缓冲区 共享同一个底层数据数组，我们在下一节将会看到这一点。

### 缓冲区份片和数据共享
我们已经创建了原缓冲区的子缓冲区，并且我们知道缓冲区和子缓冲区共享同一个底层数据数组。让我们看看这意味着什么。
我们遍历子缓冲区，将每一个元素乘以 11 来改变它。例如，5 会变成 55。

```java
for (int i=0; i<slice.capacity(); ++i) {
     byte b = slice.get( i );
     b *= 11;
     slice.put( i, b );
}
```

最后，再看一下原缓冲区中的内容：

```java
buffer.position( 0 );
buffer.limit( buffer.capacity() );

while (buffer.remaining()>0) {
     System.out.println( buffer.get() );
}
```
结果表明只有在子缓冲区窗口中的元素被改变了：
```
$ java SliceBuffer
0
1
2
33
44
55
66
7
8
9
```
缓冲区片对于促进抽象非常有帮助。可以编写自己的函数处理整个缓冲区，而且如果想要将这个过程应用于子缓冲区上，您只需取主缓冲区的一个片，并将它传递给您的函数。这比编写自己的函数来取额外的参数以指定要对缓冲区的哪一部分进行操作更容易。



只读缓冲区非常简单 ― 您可以读取它们，但是不能向它们写入。可以通过调用缓冲区的 asReadOnlyBuffer() 方法，将任何常规缓冲区转换为只读缓冲区，这个方法返回一个与原缓冲区完全相同的缓冲区(并与其共享数据)，只不过它是只读的。
只读缓冲区对于保护数据很有用。在将缓冲区传递给某个对象的方法时，您无法知道这个方法是否会修改缓冲区中的数据。创建一个只读的缓冲区可以 保证 该缓冲区不会被修改。
不能将只读的缓冲区转换为可写的缓冲区。

###直接和间接缓冲区

另一种有用的 ByteBuffer 是直接缓冲区。 直接缓冲区 是为加快 I/O 速度，而以一种特殊的方式分配其内存的缓冲区。
实际上，直接缓冲区的准确定义是与实现相关的。Sun 的文档是这样描述直接缓冲区的：
给定一个直接字节缓冲区，Java 虚拟机将尽最大努力直接对它执行本机 I/O 操作。也就是说，它会在每一次调用底层操作系统的本机 I/O 操作之前(或之后)，尝试避免将缓冲区的内容拷贝到一个中间缓冲区中(或者从一个中间缓冲区中拷贝数据)。
您可以在例子程序 FastCopyFile.java 中看到直接缓冲区的实际应用，这个程序是 CopyFile.java 的另一个版本，它使用了直接缓冲区以提高速度。
还可以用内存映射文件创建直接缓冲区。
###内存映射文件 I/O

内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。
内存映射文件 I/O 是通过使文件中的数据神奇般地出现为内存数组的内容来完成的。这其初听起来似乎不过就是将整个文件读到内存中，但是事实上并不是这样。一般来说，只有文件中实际读取或者写入的部分才会送入（或者 映射 ）到内存中。
内存映射并不真的神奇或者多么不寻常。现代操作系统一般根据需要将文件的部分映射为内存的部分，从而实现文件系统。Java 内存映射机制不过是在底层操作系统中可以采用这种机制时，提供了对该机制的访问。
尽管创建内存映射文件相当简单，但是向它写入可能是危险的。仅只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

###将文件映射到内存

了解内存映射的最好方法是使用例子。在下面的例子中，我们要将一个 FileChannel (它的全部或者部分)映射到内存中。为此我们将使用 FileChannel.map() 方法。下面代码行将文件的前 1024 个字节映射到内存中：
```java
MappedByteBuffer mbb = fc.map( FileChannel.MapMode.READ_WRITE,
     0, 1024 );
```
map() 方法返回一个 MappedByteBuffer，它是 ByteBuffer 的子类。因此，您可以像使用其他任何 ByteBuffer 一样使用新映射的缓冲区，操作系统会在需要时负责执行行映射。


#Scatter/Gather

Java NIO开始支持scatter/gather，scatter/gather用于描述从Channel（译者注：Channel在中文经常翻译为通道）中读取或者写入到Channel的操作。
分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。
聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

scatter/gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

##Scattering Reads

Scattering Reads是指数据从一个channel读取到多个buffer中。如下图描述：

![](http://ifeve.com/java-nio-scattergather/scatter/)

代码示例如下：
```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```
注意buffer首先被插入到数组，然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(译者注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

##Gathering Writes

Gathering Writes是指数据从多个buffer写入到同一个channel。如下图描述：

![](http://ifeve.com/wp-content/uploads/2013/06/gather.png)

代码示例如下：
```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers
ByteBuffer[] bufferArray = { header, body };
channel.write(bufferArray);
```

buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。


完整例子：

```java
// $Id$

import java.io.*;
import java.net.*;
import java.nio.*;
import java.nio.channels.*;

public class UseScatterGather
{
  static private final int firstHeaderLength = 2;
  static private final int secondHeaderLength = 4;
  static private final int bodyLength = 6;

  static public void main( String args[] ) throws Exception {
    if (args.length!=1) {
      System.err.println( "Usage: java UseScatterGather port" );
      System.exit( 1 );
    }

    int port = Integer.parseInt( args[0] );

    ServerSocketChannel ssc = ServerSocketChannel.open();
    InetSocketAddress address = new InetSocketAddress( port );
    ssc.socket().bind( address );

    int messageLength =
      firstHeaderLength + secondHeaderLength + bodyLength;

    ByteBuffer buffers[] = new ByteBuffer[3];
    buffers[0] = ByteBuffer.allocate( firstHeaderLength );
    buffers[1] = ByteBuffer.allocate( secondHeaderLength );
    buffers[2] = ByteBuffer.allocate( bodyLength );

    SocketChannel sc = ssc.accept();

    while (true) {

      // Scatter-read into buffers
      int bytesRead = 0;
      while (bytesRead < messageLength) {
        long r = sc.read( buffers );
        bytesRead += r;

        System.out.println( "r "+r );
        for (int i=0; i<buffers.length; ++i) {
          ByteBuffer bb = buffers[i];
          System.out.println( "b "+i+" "+bb.position()+" "+bb.limit() );
        }
      }

      // Process message here

      // Flip buffers
      for (int i=0; i<buffers.length; ++i) {
        ByteBuffer bb = buffers[i];
        bb.flip();
      }

      // Scatter-write back out
      long bytesWritten = 0;
      while (bytesWritten<messageLength) {
        long r = sc.write( buffers );
        bytesWritten += r;
      }

      // Clear buffers
      for (int i=0; i<buffers.length; ++i) {
        ByteBuffer bb = buffers[i];
        bb.clear();
      }

      System.out.println( bytesRead+" "+bytesWritten+" "+messageLength );
    }
  }
}


```


#通道之间的数据传输

在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel（译者注：channel中文常译作通道）传输到另外一个channel。

##transferFrom()

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中（译者注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个简单的例子：
```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。
此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

##transferTo()

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子：

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

是不是发现这个例子和前面那个例子特别相似？除了调用方法的FileChannel对象不一样外，其他的都一样。
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。



#Selector


Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。


##为什么使用Selector?

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。

但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。不管怎么说，关于那种设计的讨论应该放在另一篇不同的文章中。在这里，只要知道使用Selector能够处理多个通道就足够了。

下面是单线程使用一个Selector处理3个channel的示例图：

![Java NIO: A Thread uses a Selector to handle 3 Channel's](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)


##Selector的创建

通过调用Selector.open()方法创建一个Selector，如下：
```java
Selector selector = Selector.open();
```
向Selector注册通道

为了将Channel和Selector配合使用，必须将channel注册到selector上。通过SelectableChannel.register()方法来实现，如下：
```java
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

1. Connect
2. Accept
3. Read
4. Write

通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

这四种事件用SelectionKey的四个常量来表示：

1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：
```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```
##SelectionKey

在上一小节中，当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。这个对象包含了一些你感兴趣的属性：

* interest集合
* ready集合
* Channel
* Selector
* 附加的对象（可选）


下面我会描述这些属性。

###interest集合

就像向Selector注册通道一节中所描述的，interest集合是你所选择的感兴趣的事件集合。可以通过SelectionKey读写interest集合，像这样：

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;

```

可以看到，用“位与”操作interest 集合和给定的SelectionKey常量，可以确定某个确定的事件是否在interest 集合中。

###ready集合

ready 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个ready set。Selection将在下一小节进行解释。可以这样访问ready集合：

```java
int readySet = selectionKey.readyOps();
```
可以用像检测interest集合那样的方法，来检测channel中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型：
```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

###Channel + Selector

从SelectionKey访问Channel和Selector很简单。如下：

```java
Channel  channel  = selectionKey.channel();

Selector selector = selectionKey.selector();
```
###附加的对象
可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加 与通道一起使用的Buffer，或是包含聚集数据的某个对象。使用方法如下：

```java
selectionKey.attach(theObject);

Object attachedObj = selectionKey.attachment();
```
还可以在用register()方法向Selector注册Channel的时候附加对象。如：

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);

```

###通过Selector选择通道

一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。

下面是select()方法：

```java
int select()
int select(long timeout)
int selectNow()
```

select()阻塞到至少有一个通道在你注册的事件上就绪了。

select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。

selectNow()不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。

select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。

**selectedKeys()**

一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：
```java
Set selectedKeys = selector.selectedKeys();
```
当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。

可以遍历这个已选择的键集合来访问就绪的通道。如下：
```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
}

```


这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。

注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。

SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。

**wakeUp()**

某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。

如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法上，下个调用select()方法的线程会立即“醒来（wake up）”。

**close()**

用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。

###完整的示例

这里有一个完整的示例，打开一个Selector，注册一个通道注册到这个Selector上(通道的初始化过程略去),然后持续监控这个Selector的四种事件（接受，连接，读，写）是否就绪。

```java
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```

#FileChannel

Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。

FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。


##打开FileChannel

在使用FileChannel之前，必须先打开它。但是，我们无法直接打开一个FileChannel，需要通过使用一个InputStream、OutputStream或RandomAccessFile来获取一个FileChannel实例。下面是通过RandomAccessFile打开FileChannel的示例：
```java
RandomAccessFile aFile     = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel      inChannel = aFile.getChannel();
```

##从FileChannel读取数据

调用多个read()方法之一从FileChannel中读取数据。如：
```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```

首先，分配一个Buffer。从FileChannel中读取的数据将被读到Buffer中。

然后，调用FileChannel.read()方法。该方法将数据从FileChannel读取到Buffer中。read()方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1，表示到了文件末尾。

##向FileChannel写数据

使用FileChannel.write()方法向FileChannel写数据，该方法的参数是一个Buffer。如：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}

```

注意FileChannel.write()是在while循环中调用的。因为无法保证write()方法一次能向FileChannel写入多少字节，因此需要重复调用write()方法，直到Buffer中已经没有尚未写入通道的字节。

##关闭FileChannel

用完FileChannel后必须将其关闭。如：
```java
channel.close();

```

##FileChannel的position()方法

有时可能需要在FileChannel的某个特定位置进行数据的读/写操作。可以通过调用position()方法获取FileChannel的当前位置。

也可以通过调用position(long pos)方法设置FileChannel的当前位置。

这里有两个例子:

```java
long pos channel.position();

channel.position(pos +123);
```
如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 —— 文件结束标志。

如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“文件空洞”，磁盘上物理文件中写入的数据间有空隙。

##FileChannel的size方法

FileChannel实例的size()方法将返回该实例所关联文件的大小。如:
```java
long fileSize = channel.size();
```

这个例子截取文件的前1024个字节。

##FileChannel的force方法

FileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到FileChannel里的数据一定会即时写到磁盘上。要保证这一点，需要调用force()方法。

force()方法有一个boolean类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

下面的例子同时将文件数据和元数据强制写到磁盘上：
```java
channel.force(true);
```

#SocketChannel

Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel：

打开一个SocketChannel并连接到互联网上的某台服务器。
一个新连接到达ServerSocketChannel时，会创建一个SocketChannel。

##打开 SocketChannel

下面是SocketChannel的打开方式：
```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

##关闭 SocketChannel

当用完SocketChannel之后调用SocketChannel.close()关闭SocketChannel：
```java
socketChannel.close();
```

##从 SocketChannel 读取数据

要从SocketChannel中读取数据，调用一个read()的方法之一。以下是例子：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);
```

首先，分配一个Buffer。从SocketChannel读取到的数据将会放到这个Buffer中。

然后，调用SocketChannel.read()。该方法将数据从SocketChannel 读到Buffer中。read()方法返回的int值表示读了多少字节进Buffer里。如果返回的是-1，表示已经读到了流的末尾（连接关闭了）。

##写入 SocketChannel

写数据到SocketChannel用的是SocketChannel.write()方法，该方法以一个Buffer作为参数。示例如下：
```java

String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

注意SocketChannel.write()方法的调用是在一个while循环中的。Write()方法无法保证能写多少字节到SocketChannel。所以，我们重复调用write()直到Buffer没有要写的字节为止。

##非阻塞模式

可以设置 SocketChannel 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用connect(), read() 和write()了。

**connect()**

如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用finishConnect()的方法。像这样：

```java
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...
}
```

**write()**

非阻塞模式下，write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用write()。前面已经有例子了，这里就不赘述了。

**read()**

非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。

非阻塞模式与选择器

非阻塞模式与选择器搭配会工作的更好，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。Selector与SocketChannel的搭配使用会在后面详讲。

#ServerSocketChannel

Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道, 就像标准IO中的ServerSocket一样。ServerSocketChannel类在 java.nio.channels包中。


这里有个例子：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

##打开 ServerSocketChannel

通过调用 ServerSocketChannel.open() 方法来打开ServerSocketChannel.如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

##关闭 ServerSocketChannel

通过调用ServerSocketChannel.close() 方法来关闭ServerSocketChannel. 如：

```java
serverSocketChannel.close();
```

##监听新进来的连接

通过 ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。

通常不会仅仅只监听一个连接,在while循环中调用 accept()方法. 如下面的例子：

```java
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```
当然,也可以在while循环中使用除了true以外的其它退出准则。

##非阻塞模式

ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
        }
}

```

#DatagramChannel


Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。


##打开 DatagramChannel

下面是 DatagramChannel 的打开方式：

```java
DatagramChannel channel = DatagramChannel.open();

channel.socket().bind(new InetSocketAddress(9999));
```

这个例子打开的 DatagramChannel可以在UDP端口9999上接收数据包。

##接收数据

通过receive()方法从DatagramChannel接收数据，如：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);
```

receive()方法会将接收到的数据包内容复制到指定的Buffer. 如果Buffer容不下收到的数据，多出的数据将被丢弃。

##发送数据

通过send()方法从DatagramChannel发送数据，如:

```java
String newData = "New String to write to file..."
                    + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

这个例子发送一串字符到”jenkov.com”服务器的UDP端口80。 因为服务端并没有监控这个端口，所以什么也不会发生。也不会通知你发出的数据包是否已收到，因为UDP在数据传送方面没有任何保证。

##连接到特定的地址

可以将DatagramChannel“连接”到网络中的特定地址的。由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。而是锁住DatagramChannel ，让其只能从特定地址收发数据。

这里有个例子:

```java
channel.connect(new InetSocketAddress("jenkov.com", 80));
```

当连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。这里有几个例子：

查看源代码打印帮助
```java
int bytesRead = channel.read(buf);

int bytesWritten = channel.write(but);
```

#Pipe

Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

这里是Pipe原理的图示：

![Java NIO: Pipe Internals](http://tutorials.jenkov.com/images/java-nio/pipe-internals.png)


##创建管道

通过Pipe.open()方法打开管道。例如：

```java
Pipe pipe = Pipe.open();
```

##向管道写数据

要向管道写数据，需要访问sink通道。像这样：

```java
Pipe.SinkChannel sinkChannel = pipe.sink();
```

通过调用SinkChannel的write()方法，将数据写入SinkChannel,像这样：
```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

##从管道读取数据

从读取管道的数据，需要访问source通道，像这样：

```java
Pipe.SourceChannel sourceChannel = pipe.source();
```
调用source通道的read()方法来读取数据，像这样：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = sourceChannel.read(buf);
```
read()方法返回的int值会告诉我们多少字节被读进了缓冲区。

#Java NIO与IO

我应该何时使用IO，何时使用NIO呢？在本文中，我会尽量清晰地解析Java NIO和IO的差异、它们的使用场景，以及它们如何影响您的代码设计。

##Java NIO和IO的主要区别

下表总结了Java NIO和IO之间的主要差别，我会更详细地描述表中每部分的差异。

IO            | NIO
------------- | -------------
面向流        | 面向缓冲
阻塞IO        | 非阻塞IO
无            | 选择器

##面向流与面向缓冲

Java NIO和IO之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。 Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

##阻塞与非阻塞IO

Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

##选择器（Selectors）

Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。

##NIO和IO如何影响应用程序的设计

无论您选择IO或NIO工具箱，可能会影响您应用程序设计的以下几个方面：

* 对NIO或IO类的API调用。
* 数据处理。
* 用来处理数据的线程数。

##API调用

当然，使用NIO的API调用时看起来与使用IO时有所不同，但这并不意外，因为并不是仅从一个InputStream逐字节读取，而是数据必须先读入缓冲区再处理。

数据处理
使用纯粹的NIO设计相较IO设计，数据处理也受到影响。

在IO设计中，我们从InputStream或 Reader逐字节读取数据。假设你正在处理一基于行的文本数据流，例如：
```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```
该文本行的流可以这样处理：
```java
InputStream input = ... ; // get the InputStream from the client socket

BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```

请注意处理状态由程序执行多久决定。换句话说，一旦reader.readLine()方法返回，你就知道肯定文本行就已读完， readline()阻塞直到整行读完，这就是原因。你也知道此行包含名称；同样，第二个readline()调用返回的时候，你知道这行包含年龄等。 正如你可以看到，该处理程序仅在有新数据读入时运行，并知道每步的数据是什么。一旦正在运行的线程已处理过读入的某些数据，该线程不会再回退数据（大多如此）。下图也说明了这条原则：

![Java IO: Reading data from a blocking stream.](http://tutorials.jenkov.com/images/java-nio/nio-vs-io-1.png)

Java IO: 从一个阻塞的流中读数据） 而一个NIO的实现会有所不同，下面是一个简单的例子：

```java
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);
```

注意第二行，从通道读取字节到ByteBuffer。当这个方法调用返回时，你不知道你所需的所有数据是否在缓冲区内。你所知道的是，该缓冲区包含一些字节，这使得处理有点困难。
假设第一次 read(buffer)调用后，读入缓冲区的数据只有半行，例如，“Name:An”，你能处理数据吗？显然不能，需要等待，直到整行数据读入缓存，在此之前，对数据的任何处理毫无意义。

所以，你怎么知道是否该缓冲区包含足够的数据可以处理呢？好了，你不知道。发现的方法只能查看缓冲区中的数据。其结果是，在你知道所有数据都在缓冲区里之前，你必须检查几次缓冲区的数据。这不仅效率低下，而且可以使程序设计方案杂乱不堪。例如：

```java
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {
    bytesRead = inChannel.read(buffer);
}
```

bufferFull()方法必须跟踪有多少数据读入缓冲区，并返回真或假，这取决于缓冲区是否已满。换句话说，如果缓冲区准备好被处理，那么表示缓冲区满了。

bufferFull()方法扫描缓冲区，但必须保持在bufferFull（）方法被调用之前状态相同。如果没有，下一个读入缓冲区的数据可能无法读到正确的位置。这是不可能的，但却是需要注意的又一问题。

如果缓冲区已满，它可以被处理。如果它不满，并且在你的实际案例中有意义，你或许能处理其中的部分数据。但是许多情况下并非如此。下图展示了“缓冲区数据循环就绪”：

![从一个通道里读数据，直到所有的数据都读到缓冲区里](http://tutorials.jenkov.com/images/java-nio/nio-vs-io-2.png)

##用来处理数据的线程数

NIO可让您只使用一个（或几个）单线程管理多个通道（网络连接或文件），但付出的代价是解析数据可能会比从一个阻塞流中读取数据更复杂。

如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，实现NIO的服务器可能是一个优势。同样，如果你需要维持许多打开的连接到其他计算机上，如P2P网络中，使用一个单独的线程来管理你所有出站连接，可能是一个优势。一个线程多个连接的设计方案如下图所示：

![Java NIO: 单线程管理多个连接](http://tutorials.jenkov.com/images/java-nio/nio-vs-io-3.png)

如果你有少量的连接使用非常高的带宽，一次发送大量的数据，也许典型的IO服务器实现可能非常契合。下图说明了一个典型的IO服务器设计：

![Java IO: 一个典型的IO服务器设计- 一个连接通过一个线程处理.](http://tutorials.jenkov.com/images/java-nio/nio-vs-io-4.png)


<!--Java7 new API-->
#Path

#Files

#AsynchronousFileChannel


#字符集

根据 Sun 的文档，一个 Charset 是“十六位 Unicode 字符序列与字节序列之间的一个命名的映射”。实际上，一个 Charset 允许您以尽可能最具可移植性的方式读写字符序列。
Java 语言被定义为基于 Unicode。然而在实际上，许多人编写代码时都假设一个字符在磁盘上或者在网络流中用一个字节表示。这种假设在许多情况下成立，但是并不是在所有情况下都成立，而且随着计算机变得对 Unicode 越来越友好，这个假设就日益变得不能成立了。
在本节中，我们将看一下如何使用 Charsets 以适合现代文本格式的方式处理文本数据。这里将使用的示例程序相当简单，不过，它触及了使用 Charset 的所有关键方面：为给定的字符编码创建 Charset，以及使用该 Charset 解码和编码文本数据。

##编码/解码

要读和写文本，我们要分别使用 CharsetDecoder 和 CharsetEncoder。将它们称为 编码器 和 解码器 是有道理的。一个 字符 不再表示一个特定的位模式，而是表示字符系统中的一个实体。因此，由某个实际的位模式表示的字符必须以某种特定的 编码 来表示。
CharsetDecoder 用于将逐位表示的一串字符转换为具体的 char 值。同样，一个 CharsetEncoder 用于将字符转换回位。

##处理文本的正确方式

现在我们将分析这个例子程序 UseCharsets.java。这个程序非常简单 ― 它从一个文件中读取一些文本，并将该文本写入另一个文件。但是它把该数据当作文本数据，并使用 CharBuffer 来将该数句读入一个 CharsetDecoder 中。同样，它使用 CharsetEncoder 来写回该数据。
我们将假设字符以 ISO-8859-1(Latin1) 字符集（这是 ASCII 的标准扩展）的形式储存在磁盘上。尽管我们必须为使用 Unicode 做好准备，但是也必须认识到不同的文件是以不同的格式储存的，而 ASCII 无疑是非常普遍的一种格式。事实上，每种 Java 实现都要求对以下字符编码提供完全的支持：

* US-ASCII
* ISO-8859-1
* UTF-8
* UTF-16BE
* UTF-16LE
* UTF-16

##示例程序

在打开相应的文件、将输入数据读入名为 inputData 的 ByteBuffer 之后，我们的程序必须创建 ISO-8859-1 (Latin1) 字符集的一个实例：
```java
Charset latin1 = Charset.forName( "ISO-8859-1" );
```
然后，创建一个解码器（用于读取）和一个编码器 （用于写入）：
```java
CharsetDecoder decoder = latin1.newDecoder();
CharsetEncoder encoder = latin1.newEncoder();
```
为了将字节数据解码为一组字符，我们把 ByteBuffer 传递给 CharsetDecoder，结果得到一个 CharBuffer：
```java
CharBuffer cb = decoder.decode( inputData );
```
如果想要处理字符，我们可以在程序的此处进行。但是我们只想无改变地将它写回，所以没有什么要做的。
要写回数据，我们必须使用 CharsetEncoder 将它转换回字节：
```java
ByteBuffer outputData = encoder.encode( cb );
```
在转换完成之后，我们就可以将数据写到文件中了。

完整实例：

```java
// $Id$

import java.io.*;
import java.nio.*;
import java.nio.channels.*;
import java.nio.charset.*;

public class UseCharsets
{
  static public void main( String args[] ) throws Exception {
    String inputFile = "samplein.txt";
    String outputFile = "sampleout.txt";

    RandomAccessFile inf = new RandomAccessFile( inputFile, "r" );
    RandomAccessFile outf = new RandomAccessFile( outputFile, "rw" );
    long inputLength = new File( inputFile ).length();

    FileChannel inc = inf.getChannel();
    FileChannel outc = outf.getChannel();

    MappedByteBuffer inputData =
      inc.map( FileChannel.MapMode.READ_ONLY, 0, inputLength );

    Charset latin1 = Charset.forName( "ISO-8859-1" );
    CharsetDecoder decoder = latin1.newDecoder();
    CharsetEncoder encoder = latin1.newEncoder();

    CharBuffer cb = decoder.decode( inputData );

    // Process char data here

    ByteBuffer outputData = encoder.encode( cb );

    outc.write( outputData );

    inf.close();
    outf.close();
  }
}

```


#文件锁定

文件锁定初看起来可能让人迷惑。它 似乎 指的是防止程序或者用户访问特定文件。事实上，文件锁就像常规的 Java 对象锁 ― 它们是 劝告式的（advisory） 锁。它们不阻止任何形式的数据访问，相反，它们通过锁的共享和获取赖允许系统的不同部分相互协调。
您可以锁定整个文件或者文件的一部分。如果您获取一个排它锁，那么其他人就不能获得同一个文件或者文件的一部分上的锁。如果您获得一个共享锁，那么其他人可以获得同一个文件或者文件一部分上的共享锁，但是不能获得排它锁。文件锁定并不总是出于保护数据的目的。例如，您可能临时锁定一个文件以保证特定的写操作成为原子的，而不会有其他程序的干扰。
大多数操作系统提供了文件系统锁，但是它们并不都是采用同样的方式。有些实现提供了共享锁，而另一些仅提供了排它锁。事实上，有些实现使得文件的锁定部分不可访问，尽管大多数实现不是这样的。

##锁定文件

要获取文件的一部分上的锁，您要调用一个打开的 FileChannel 上的 lock() 方法。注意，如果要获取一个排它锁，您必须以写方式打开文件。
```java
RandomAccessFile raf = new RandomAccessFile( "usefilelocks.txt", "rw" );
FileChannel fc = raf.getChannel();
FileLock lock = fc.lock( start, end, false );
```
在拥有锁之后，您可以执行需要的任何敏感操作，然后再释放锁：
```java
lock.release();
```
在释放锁后，尝试获得锁的其他任何程序都有机会获得它。

完整实例：

```java
// $Id$

import java.io.*;
import java.nio.*;
import java.nio.channels.*;

public class UseFileLocks
{
  static private final int start = 10;
  static private final int end = 20;

  static public void main( String args[] ) throws Exception {
    // Get file channel
    RandomAccessFile raf = new RandomAccessFile( "usefilelocks.txt", "rw" );
    FileChannel fc = raf.getChannel();

    // Get lock
    System.out.println( "trying to get lock" );
    FileLock lock = fc.lock( start, end, false );
    System.out.println( "got lock!" );

    // Pause
    System.out.println( "pausing" );
    try { Thread.sleep( 3000 ); } catch( InterruptedException ie ) {}

    // Release lock
    System.out.println( "going to release lock" );
    lock.release();
    System.out.println( "released lock" );

    raf.close();
  }
}

```

必须与它自己并行运行。这个程序获取一个文件上的锁，持有三秒钟，然后释放它。如果同时运行这个程序的多个实例，您会看到每个实例依次获得锁。

##文件锁定和可移植性

文件锁定可能是一个复杂的操作，特别是考虑到不同的操作系统是以不同的方式实现锁这一事实。下面的指导原则将帮助您尽可能保持代码的可移植性：

* 只使用排它锁。
* 将所有的锁视为劝告式的（advisory）。





