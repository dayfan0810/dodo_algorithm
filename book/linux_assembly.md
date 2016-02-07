
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

* CR0 控制操作模式和处理器状态的系统标志
* CR1 没有被使用
* CR2 内存页面错误信息
* CR3 内存页面目录信息
* CR4 支持处理器特性和说明处理器特性能力的标志

## 标志

处理器标志用于确定操作是否成功。
标志被分为3组：
* 状态标志
* 控制标志
* 系统标志

### 状态标志
1. cf 进位标志 
2. pf 奇偶校验标志 
用于表明数学操作中结果寄存器是否包含错误数据，如果结果中为1的位的总数是偶数，奇偶校验标志设置为1，否则为0
3. zf 零标志 如果操作结果为零，零标志就被设置为1，经常用作确定数学操作结果是否为零
4. af 辅助进位标志 如果寄存器的第三位发生进位或者借位操作，该标志设置为1
5. sf 符号标志 表明结果是正值还是负值
6. of 溢出标志 

### 控制标志
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

### 定义段
`.section`命令语句声明段，有一个参数用于声明段的类型。
bss段总是在文本段之后

![QQ20160202-0.png](quiver-image-url/0E788C8A99A3F5AF08CD9BA2124881F7.png)


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


使用as编译
```shell
as --32 -o printcpu.o printcpu.s
ld -dynamic-linker /lib/ld-linux.so.2 -o cpu2 -lc printcpu.o
./cpu2
```
使用gcc编译
```shell
gcc -m32 -o cpu printcpu.s
./cpu
```
