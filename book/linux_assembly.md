
# 寄存器

用于解决处理器与内存之前的数据存储效率问题存在的。
IA-32平台寄存器核心组有下面几种.

1. 通用寄存器 8个32位寄存器 用于存储正在访问的数据
2. 段寄存器 6个16位寄存器 用于处理内存访问
3. 指令指针 单一的32位寄存器 指向要执行的下一条指令码
4. 浮点数据 8个80位寄存器 用于浮点数学数据
5. 控制 5个32位寄存器 用于确定处理器的操作模式
6. 调试 8个32位寄存器 用于在调试处理器时包含信息

## 通用寄存器

用于临时的存储数据。

* `eax` 用于操作数和结果数据的累加器
* `ebx` 指向数据内存段中数据的指针
* `ecx` 字符串和循环操作的计数器
* `edx` I/O指针
* `edi` 用于字符串的目标的数据指针
* `esi` 用于字符串操作的源的数据指针
* `esp` 堆栈指针
* `ebp` 堆栈数据指针

32位`eax`、ebx、ecx、edx 可以通过16位和8位名称引用

![null][1]

## 段寄存器
用于引用内存位置。ia-32处理器允许3种不同的访问系统内存方法：

* 平坦内存模式 所有指令数据堆栈都包含在相同的地址空间中
* 分段内存模式 把内存划分为独立段的组
* 实地址内存模式 此模式下所有的段寄存器都指向零线性地址，不会被程序改动。

可用的段寄存器
* cs 代码段 
包含指向内存中代码段的指针 代码段是内存中存储指令码的位置 程序不能显示的加载改变cs寄存器
* ss 堆栈段 包含传递给程序中的函数和过程的数据值
* ds 数据段 
* es 附加段指针
* fs 附加段指针
* gs 附加段指针


每个段寄存器都是16位，包含指向内存特定段的起始位置的指针。

## 指令指针寄存器
EIP寄存器有时称为程序计数器，用于跟踪要执行的下一条指令码。应用程序不能直接修改指令指针。

## 控制寄存器

* `CR0` 控制操作模式和处理器状态的系统标志
* `CR1` 没有被使用
* `CR2` 内存页面错误信息
* `CR3` 内存页面目录信息
* `CR4` 支持处理器特性和说明处理器特性能力的标志

## 标志

处理器标志用于确定操作是否成功。
标志被分为3组：
* 状态标志
* 控制标志
* 系统标志

### 状态标志
1. `cf` 进位标志 
2. `pf` 奇偶校验标志 
用于表明数学操作中结果寄存器是否包含错误数据，如果结果中为1的位的总数是偶数，奇偶校验标志设置为1，否则为0
3. `zf` 零标志 如果操作结果为零，零标志就被设置为1，经常用作确定数学操作结果是否为零
4. `af` 辅助进位标志 如果寄存器的第三位发生进位或者借位操作，该标志设置为1
5. `sf` 符号标志 表明结果是正值还是负值
6. `of` 溢出标志 

### 控制标志

Heading

当前只有一个控制标志DF标志，用于控制处理器处理字符串方式。
当DF设置为1，字符串指令自动递减内存地址到下一个字节，0为递增

### 系统标志
略

# 汇编程序

## 程序的组成
汇编语言程序由定义好的段构成，三个最常用的段：
* 数据段 声明带有初始值的数据元素，程序中的变量
* bss段 声明使用零值初始化的数据元素 常用做程序缓冲区
* 文本段 每个程序必须有文本段。用于声明指令码

![enter description here][2]

### 定义段
`.section`命令语句声明段，有一个参数用于声明段的类型。
bss段总是在文本段之后


### 定义起始点

`_start`标签用于表明程序的起始点。
`.globl`命令声明程序入口点。
程序模板：

```assembly
.section .data
#<带有初始值的数据>
.section .bss
#<使用0初始化的数据>
.section .text
.globl _start
_start:
#<Code>

```

例子，用于读取cpu信息

```assembly
# as hello_asm.s -o hello_asm.o
# ld hello_asm.o -e _start -o hello_asm
# ./hello_asm
.section __DATA,__data #for linux  .section .data
output:
.ascii "The processor Vender ID is 'xxxxxxxxxxxx'\n"
.section __TEXT,__text #for linux  .section .text
.globl _start
_start:
movl $0 ,%eax
cpuid
movl $output, %edi
movl %ebx,28(%edi)
movl %edx,32(%edi)
movl %ecx,36(%edi)
movl $4,%eax
movl $1,%ebx
movl $output,%ecx
movl $42 ,%edx
int $0x80
movl $1,%eax
movl $0,%ebx
int $0x80
```


## 汇编中是用C库函数

### 以printf函数为例

```assembly
.section .data
output:
.asciz "The processor  vender id is '%s'\n"

.section .bss
.lcomm buffer ,12
.section .text
.globl _start
_start:
movl $0,%eax
movl %ebx, (%edi)
movl %edx, 4(%edi)
movl %ecx, 8(%edi)
pushl $buffer
pushl $output
call printf
addl $8 ,%esp 
pushl $0
call exit
```

**需注意的是，在64位机上编译上面程序需要强制32位编译，用gcc -m32 、as --32选项**


**使用as编译**

```shell
as --32 -o printcpu.o printcpu.s
ld -dynamic-linker /lib/ld-linux.so.2 -o cpu2 -lc printcpu.o
./cpu2
```

**使用gcc编译**
使用gcc汇编程序时需要注意gcc连接器查找的是`main`标签作为入口，所以需要将程序的`.globl`命令修改成下面这样
```assembly
.section .text
.globl main
main:
```

```shell
gcc -m32 -o cpu printcpu.s
./cpu
```

------------
# 第五章 传送数据
## 5.1 定义数据元素
### 5.1.1 数据段
程序的数据段是最常见的定义数据元素的位置。 
使用`.data`指令来声明数据段。在这个段中声明的任何数据元素都保留在内存中并可以被汇编语言程序中的指令读取和写入。

有另一种类型的数据段`.rodata`，该数据段中定义的数据元素只能只读

数据段中定义数据需要两个语句:一个标签、一个命令。

标签相当于c语言的变量名。

命令相当于c语言的类型声明关键字

| 命令    | 数据类型                 |
| ------- | ------------------------ |
| .ascii  | 文本字符串               |
| .asciz  | 以空字符结尾的文本字符串 |
| .byte   | 字节                     |
| .double | 双精度浮点数             |
| .float  | 单精度浮点数             |
| .int    | 32位整数                 |
| .long   | 32位整数(和int相同)      |
| .octa   | 16字节整数               |
| .quad   | 8字节整数                |
| .short  | 16位整数                 |
| .single | 单精度浮点数(和.float相同)|


举个栗子:
```x86asm
output:
.ascii "the string example\n"
pi:
.float 3.1415926
;可以在一行里定义多个值
sizes:
.long 100,150,200,250,300
;把长整型(4字节)值100放在sizes引用开始的内存位置中，类似数组，每个长整型占用4个字节 32位，可以通过访问内存位置sizes+8访问200这个值
```

```x86asm
.section .data
msg:
.ascii "This is a test message"
factors:
.double 37.5,45.33,12.30
height:
.int 54
length:
.int 62,35,47
```
下面的图描述了数据在内存中存放的情况

73页图

### 5.1.2 定义静态符号(常量值)
`.equ`命令用于定义常量值

```x86asm
.equ factor ,3
.equ LINUX_SYS_CALL, 0x80
;引用静态数据元素
movl $LINUX_SYS_CALL, %eax
```
### 5.1.3 bss段
bss段数据元素和数据段中的定义有些不同，无需声明特定的数据类型。

| 命令   | 描述                             |
| ------ | -------------------------------- |
| .comm  | 声明未初始化的数据的通用内存区域 |
| .lcomm | 声明未初始化的数据的本地通用内存区域|

本地通用内存区域是为不会从本地汇编代码之外进行访问的数据保留的。
命令格式： `.comm symbol, length`
例子：待定，未明白
```x86asm
.section .bss
```

## 5.2 传递数据元素
注：在csapp里对此指令的讲述更好
`MOV`指令用作通用数据传送指令。用于在内存和寄存器之间传送数据。
### 5.2.1 mov指令
`MOV` 基本格式：
`movx source, dest`

根据数据元素的长度不同movx有以下变种

* movl 用于32位长字值
* movw 用于16位字值
* movb 用于8位字节值

把32位eax寄存器值传送给32位ebx寄存器：`movl %eax, %ebx`
对于16位寄存器：`movw %ax,%bx`
对于8位寄存器：`movb %al,%bl`

mov的操作指令可用于以下类型的传送：

* 立即数传送给通用寄存器
* 立即数传送给内存
* 通用寄存器传送给另一个通用寄存器
* 通用寄存器传送给段寄存器
* 段寄存器传送给通用寄存器
* 通用寄存器传送给控制寄存器
* 控制寄存器传送给通用寄存器
* 通用寄存器传送给调试寄存器
* 调试寄存器传送给通用寄存器
* 内存位置传送给通用寄存器
* 内存位置传送给段寄存器
* 通用寄存器传送给内存位置
* 段寄存器传送给内存位置

### 5.2.2 立即数传送给寄存器和内存

立即数的每个值前面需要加上`$`符号
例子：

```x86asm
movl $0, %eax ;move the value 0 to he eax register
movl $0x80, %ebx ;move the hexadecimal value 80 to the ebx register
movl $100, height ;move the value 100 to the height memory location

```

### 5.2.3 在寄存器之间传递数据
8个通用寄存器eax,ebx,ecx,edx,edi,esi,ebp,esp是用于保存数据的最常用寄存器，这些寄存器内容可以传送给可用的任何其他类型寄存器，和通用寄存器不同，专用寄存器只能传送给通用寄存器，或者接收通用寄存器传送过来的内容。

```x86asm
movl %eax, %ecx # move 32-bits
movw %ax, %cx #move 16-bits
```


### 5.2.4 在内存和寄存器之间传送数据

1 把数值从内存传送到寄存器

`movl value, %eax`

指令把value标签指定的内存位置的数值传送给eax,传送value标签引用的内存位置开始的4字节数据，`movb`用于1字节，`movw`用于2字节

```x86asm
.section .data
value:
.int 1
.section .text
.globl _start
_start:
nop
movl value, %ecx
movl $1, %eax
movl $0, %ebx
int $0x80

```


2 把数值从寄存器传送给内存

`movl %ecx, value`

指令把ecx寄存器里的4字节数据传送给value标签指定内存位置

3 使用变址的内存位置

```x86asm
values:
.int 10,15,20,25,30,35,40,45,50,55,60

```
变址内存模式，内存位置由以下因素确定

* 基址
* 添加到基址上的偏移地址
* 数据元素的长度
* 确定选择哪个数据元素的变址

表达式的格式：`base_address (offset_address,index,size)`
获取的数值位于：`base_address+offset_address_index * size`

例子，引用values数组中20：

```x86asm
movl $2, %edi
movl values(, %edi,4), %eax
```

例子，输出values数组中的数值

```x86asm
.section .data
output:
.asciz "The value is %d\n"
values:
.int 10,15,20,25,30,35,40,45,50,55,60
.section .text
.globl _start
_start:
nop
movl $0,%edi
loop:
movl values(,%edi,4),%eax
pushl %eax
pushl $output
call printf
addl $8,%esp
inc %edi
cmpl $11,%edi
jne loop
movl $0, %ebx
movl $1, %eax
int $0x80
```

```x86asm
#mac下
.section __DATA,__data
output:
.asciz "The value is %d\n"
values:
.int 10,15,20,25,30,35,40,45,50,55,60
.section __TEXT,__text
.globl _main
_main:
nop
movl $0,%edi
loop:
movl values(,%edi,4),%eax
pushl %eax
pushl $output
call _printf
addl $8,%esp
inc %edi
cmpl $11,%edi
jne loop
movl $0, %ebx
movl $1, %eax
int $0x80

```

![enter description here][3] 

4 使用寄存器间接寻址

使用指针访问存储在内存位置中的数据称为间接寻址(indirect addressing)

当使用标签引用内存位置中包含的数值时，可以通过在指令中的标签加上`$`符号获取数值的内存位置的地址

`movl $values, %edi`

用于把values标签引用的内存地址传送给edi寄存器
**在平坦内存模型中，所有内存地址都是使用32位数字表示的**

`movl %ebx, (%edi)`

如果edi寄存器外边没有括号，指令只是把ebx寄存器内容加载到edi寄存器中，如果edi寄存器外边有括号，那么指令就把ebx寄存器的内容传送给edi寄存器中包含的内存位置

`movl %edx, 4(%edi)`

这条指令把edx值放在edi寄存器指向位置之后的4字节内存位置，也可以放到相反方向：

`movl %edx, -4(%edi)`**此处有疑问书中写的是`-4(&edi)`**

演示间接寻址模式：

```x86asm
;➜  as -gstabs -o movtest.o mvotest4.s 
;➜  ld movtest.o -e _start -o movtest
.section .data
values:
.int 10,15,20,25,30,35,40,45,50,55,60
.section .text
.globl _start
_start:
nop
movl values,%eax
movl $values, %edi
movl $100, 4(%edi)
movl $1, %edi
movl values(,%edi,4), %ebx
movl $1,%eax
int $0x80

```
可以通过echo命令查看程序退出码

```bash
➜  asm  ./movtest  
➜  asm  echo $?
100
```

## 5.3 条件传送指令

条件传送指令功能是mov指令发生在特定的条件下。

旧式的汇编语言中会看到下面代码：

```x86asm
    dec %ecx ;ecx递增1 如果ecx寄存器没有溢出 进位标志没有设置位1
    jz continue ;jnc指令跳转continue
    movl $0, %ecx ;如果溢出了 ecx设置为0
continue:
```

条件传送指令可以避免处理jmp指令，提高速度


### 5.3.1 cmov 指令

指令格式如下： `cmovx source, destination`

x是一个或者两个字母的代码，表示将触发传送操作的条件，条件取决于eflags寄存器当前值：

| EFLAGS | 名称               | 描述                             |
| ------ | ------------------ | -------------------------------- |
| CF     | 进位(carry)标志    | 进位或者借位                     |
| OF     | 溢出overflow标志   | 整数值过大或者过小               |
| PF     | 奇偶校验parity标志 | 寄存器包含数学操作造成的错误数据 |
| SF     | 符号标志           | 指出结果为正还是负               |
| ZF     | 零zero标志         |                                  |

![enter description here][4]

![enter description here][5]



### 5.3.2 使用comv指令

下面是选择整数中最大一个的程序

```x86asm
.section .data
output:
.asciz "The largest value is %d\n"
values:
.int 105,222,23,56,52,445,25,53,117,5
.section .text
.globl main
main:
nop
movl values, %ebx
movl $1, %edi
loop:
movl values(,%edi,4), %eax
cmp %ebx,%eax
cmova %eax,%ebx
inc %edi
cmp $10, %edi
jne loop
pushl %ebx
pushl $output
call printf
addl $8, %esp
pushl $0
call exit
```


## 5.4交换数据

![enter description here][6]

如果需要交换两个寄存器的数据，指令码就像下面这样：

```x86asm
movl %eax, %ecx
movl %ebx, %eax
movl %ecx, %ebx
```
需要增加一个临时保存数据的寄存器来交换数据，数据交换指令就解决了这个问题



### 5.4.1


| 指令      | 描述                                               |
| --------- | -------------------------------------------------- |
| XCHG      | 在两个寄存器之间或者寄存器和内存位置之间交换       |
| BSWAP     | 反转一个32位寄存器中的字节顺序                     |
| XADD      | 交换两个值并且把总和存储在目标操作数中             |
| CMPXCHG   | 把一个值和一个外部值进行比较，并且交换它和另一个值 |
| CMPXCHG8B | 比较两个64位值并且交换他们                         |


#### 1. XCHG
在两个通用寄存器之间或者寄存器和内存位置之间交换数据值。
格式如下：
`xchg operand1, operand2`
operand1或者operand2可以是通用寄存器，也可以是内存位置**(但是二者不能都是内存位置)**，可以对任何通用的8，16，32位寄存器使用这个命令，但是两个操作数的长度*必须相同*
当一个操作数是*内存*位置时，处理器LOCK信号被自动标明，防止在交换过程中任何其他处理器访问这个内存位置。
*XCHG指令可能对程序性能有不良影响*

#### 2. BSWAP

![BSWAP][7]
第0-7位和第24-31位进行交换，第8-15位和第6-23位交换

例子：
```x86asm
.section .text
.globl _start
_start:
nop
movl $0x12345678, %ebx
bswap %ebx
movl $1, %eax
int $0x80

```
运行结果：

```bash
➜  asm  gcc -o swaptest -gstabs swaptest.s
➜  asm  ./swaptest 
➜  asm  gdb -q swaptest 
Reading symbols from swaptest...done.
(gdb) break *main+1
Breakpoint 1 at 0x80483bc: file swaptest.s, line 5.
(gdb) run
Starting program: /home/dodola/asm/swaptest 

Breakpoint 1, main () at swaptest.s:5
5	movl $0x12345678, %ebx
(gdb) step
6	bswap %ebx
(gdb) print/x $ebx
$1 = 0x12345678
(gdb) step
7	movl $1, %eax
(gdb) print/x $ebx
$2 = 0x78563412
(gdb) 


```

#### 3. XADD

XADD指令用于交换两个寄存器或者内存位置和寄存器的值，把两个值相加，然后把结果存储在目标位置（寄存器或者内存位置），指令格式：

`xadd source, destination`

其中source*必须*是寄存器，destination可以是寄存器也可以是内存位置，并且destination包含相加的结果。


#### 4. CMPXCHG
//待
#### 5. CMPXCHG8B
//待








  [1]: ./images/QQ20160201-0.png "QQ20160201-0.png"
  [2]: ./images/QQ20160202-0.png "QQ20160202-0.png"
  [3]: ./images/QQ20160210-0.png "QQ20160210-0.png"
  [4]: ./images/QQ20160210-1.png "QQ20160210-1.png"
  [5]: ./images/QQ20160210-2.png "QQ20160210-2.png"
  [6]: ./images/QQ20160210-3.png "QQ20160210-3.png"
  [7]: ./images/5-3.png "5-3.png"