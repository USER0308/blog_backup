# 缓冲区溢出攻击

## Linux 下进程地址空间的布局及堆栈帧的结构
要想了解 Linux 下缓冲区溢出攻击的原理，我们必须首先掌握 Linux 下进程地址空间的布局以及堆栈帧的结构。

任何一个程序通常都包括代码段和数据段，这些代码和数据本身都是静态的。程序要想运行，首先要由操作系统负责为其创建进程，并在进程的虚拟地址空间中为其代码段和数据段建立映射。光有代码段和数据段是不够的，进程在运行过程中还要有其动态环境，其中最重要的就是堆栈。图 3 所示为 Linux 下进程的地址空间布局：

图 3 Linux 下进程地址空间的布局

![address](http://ovt2bylq8.bkt.clouddn.com/b3eadeeacb4bcd5d3dad1bea44491b2a.png)

首先，execve(2) 会负责为进程代码段和数据段建立映射，真正将代码段和数据段的内容读入内存是由系统的缺页异常处理程序按需完成的。另外，execve(2) 还会将 bss 段清零，这就是为什么未赋初值的全局变量以及 static 变量其初值为零的原因。进程用户空间的最高位置是用来存放程序运行时的命令行参数及环境变量的，在这段地址空间的下方和 bss 段的上方还留有一个很大的空洞，而作为进程动态运行环境的堆栈和堆就栖身其中，其中堆栈向下伸展，堆向上伸展。

知道了堆栈在进程地址空间中的位置，我们再来看一看堆栈中都存放了什么。相信读者对 C 语言中的函数这样的概念都已经很熟悉了，实际上堆栈中存放的就是与每个函数对应的堆栈帧。当函数调用发生时，新的堆栈帧被压入堆栈；当函数返回时，相应的堆栈帧从堆栈中弹出。典型的堆栈帧结构如图 4 所示。

堆栈帧的顶部为函数的实参，下面是函数的返回地址以及前一个堆栈帧的指针，最下面是分配给函数的局部变量使用的空间。一个堆栈帧通常都有两个指针，其中一个称为堆栈帧指针，另一个称为栈顶指针。前者所指向的位置是固定的，而后者所指向的位置在函数的运行过程中可变。因此，在函数中访问实参和局部变量时都是以堆栈帧指针为基址，再加上一个偏移。对照图 4 可知，实参的偏移为正，局部变量的偏移为负。

图 4 典型的堆栈帧结构

![stack](http://ovt2bylq8.bkt.clouddn.com/74cc6ad52c5df9ead659410aee4aaa28.png)

介绍了堆栈帧的结构，我们再来看一下在 Intel i386 体系结构上堆栈帧是如何实现的。图 5 和图 6 分别是一个简单的 C 程序及其编译后生成的汇编程序。

图 5 一个简单的 C 程序 example1.c
```
int function(int a, int b, int c)
{
        char buffer[14];
        int     sum;
        sum = a + b + c;
        return sum;
}
void main()
{
        int     i;
        i = function(1,2,3);
}
```
图 6 example1.c 编译后生成的汇编程序 example1.s
```
1    .file   "example1.c"
2     .version    "01.01"
3 gcc2_compiled.:
4 .text
5     .align 4
6 .globl function
7     .type    function,@function
8 function:
9     pushl %ebp
10     movl %esp,%ebp
11     subl $20,%esp
12     movl 8(%ebp),%eax
13     addl 12(%ebp),%eax
14     movl 16(%ebp),%edx
15     addl %eax,%edx
16     movl %edx,-20(%ebp)
17     movl -20(%ebp),%eax
18     jmp .L1
19     .align 4
20 .L1:
21     leave
22     ret
23 .Lfe1:
24     .size    function,.Lfe1-function
25     .align 4
26 .globl main
27     .type    main,@function
28 main:
29     pushl %ebp
30  movl %esp,%ebp
31     subl $4,%esp
32     pushl $3
33     pushl $2
34     pushl $1
35     call function
36     addl $12,%esp
37     movl %eax,%eax
38     movl %eax,-4(%ebp)
39 .L2:
40     leave
41     ret
42 .Lfe2:
43     .size    main,.Lfe2-main
44     .ident  "GCC: (GNU) 2.7.2.3"
```
这里我们着重关心一下与函数 function 对应的堆栈帧形成和销毁的过程。从图 5 中可以看到，function 是在 main 中被调用的，三个实参的值分别为 1、2、3。由于 C 语言中函数传参遵循反向压栈顺序，所以在图 6 中 32 至 34 行三个实参从右向左依次被压入堆栈。接下来 35 行的 call 指令除了将控制转移到 function 之外，还要将 call 的下一条指令 addl 的地址，也就是 function 函数的返回地址压入堆栈。下面就进入 function 函数了，首先在第 9 行将 main 函数的堆栈帧指针 ebp 保存在堆栈中并在第 10 行将当前的栈顶指针 esp 保存在堆栈帧指针 ebp 中，最后在第 11 行为 function 函数的局部变量 buffer[14] 和 sum 在堆栈中分配空间。至此，函数 function 的堆栈帧就构建完成了，其结构如图 7 所示。

图 7 函数 function 的堆栈帧

![function](http://ovt2bylq8.bkt.clouddn.com/9bb51ed91e2797967fb5e4673e66ae31.png)

读者不妨回过头去与图 4 对比一下。这里有几点需要说明。首先，在 Intel i386 体系结构下，堆栈帧指针的角色是由 ebp 扮演的，而栈顶指针的角色是由 esp 扮演的。另外，函数 function 的局部变量 buffer[14] 由 14 个字符组成，其大小按说应为 14 字节，但是在堆栈帧中却为其分配了 16 个字节。这是时间效率和空间效率之间的一种折衷，因为 Intel i386 是 32 位的处理器，其每次内存访问都必须是 4 字节对齐的，而高 30 位地址相同的 4 个字节就构成了一个机器字。因此，如果为了填补 buffer[14] 留下的两个字节而将 sum 分配在两个不同的机器字中，那么每次访问 sum 就需要两次内存操作，这显然是无法接受的。还有一点需要说明的是，正如我们在本文前言中所指出的，如果读者使用的是较高版本的 gcc 的话，您所看到的函数 function 对应的堆栈帧可能和图 7 所示有所不同。上面已经讲过，为函数 function 的局部变量 buffer[14] 和 sum 在堆栈中分配空间是通过在图 6 中第 11 行对 esp 进行减法操作完成的，而 sub 指令中的 20 正是这里两个局部变量所需的存储空间大小。但是在较高版本的 gcc 中，sub 指令中出现的数字可能不是 20，而是一个更大的数字。应该说这与优化编译技术有关，在较高版本的 gcc 中为了有效运用目前流行的各种优化编译技术，通常需要在每个函数的堆栈帧中留出一定额外的空间。

下面我们再来看一下在函数 function 中是如何将 a、b、c 的和赋给 sum 的。前面已经提过，在函数中访问实参和局部变量时都是以堆栈帧指针为基址，再加上一个偏移，而 Intel i386 体系结构下的堆栈帧指针就是 ebp，为了清楚起见，我们在图 7 中标出了堆栈帧中所有成分相对于堆栈帧指针 ebp 的偏移。这下图 6 中 12 至 16 的计算就一目了然了，8(%ebp)、12(%ebp)、16(%ebp) 和 - 20(%ebp) 分别是实参 a、b、c 和局部变量 sum 的地址，几个简单的 add 指令和 mov 指令执行后 sum 中便是 a、b、c 三者之和了。另外，在 gcc 编译生成的汇编程序中函数的返回结果是通过 eax 传递的，因此在图 6 中第 17 行将 sum 的值拷贝到 eax 中。

最后，我们再来看一下函数 function 执行完之后与其对应的堆栈帧是如何弹出堆栈的。图 6 中第 21 行的 leave 指令将堆栈帧指针 ebp 拷贝到 esp 中，于是在堆栈帧中为局部变量 buffer[14] 和 sum 分配的空间就被释放了；除此之外，leave 指令还有一个功能，就是从堆栈中弹出一个机器字并将其存放到 ebp 中，这样 ebp 就被恢复为 main 函数的堆栈帧指针了。第 22 行的 ret 指令再次从堆栈中弹出一个机器字并将其存放到指令指针 eip 中，这样控制就返回到了第 36 行 main 函数中的 addl 指令处。addl 指令将栈顶指针 esp 加上 12，于是当初调用函数 function 之前压入堆栈的三个实参所占用的堆栈空间也被释放掉了。至此，函数 function 的堆栈帧就被完全销毁了。前面刚刚提到过，在 gcc 编译生成的汇编程序中通过 eax 传递函数的返回结果，因此图 6 中第 38 行将函数 function 的返回结果保存在了 main 函数的局部变量 i 中。
引用自: https://www.ibm.com/developerworks/cn/linux/l-overflow/index.html

## 函数调用过程探究

### 栈 (stack)

栈，相信大家都十分熟悉，push/pop，只允许在一端进行操作，后进先出 (LIFO)，凡是学过编程的人都能列出一二三点。但就是这个最简单的数据结构，构成了计算机中程序执行的基础，用于内核中程序执行的栈具有以下特点：

每一个进程在用户态对应一个调用栈结构 (call stack)
程序中每一个未完成运行的函数对应一个栈帧 (stack frame)，栈帧中保存函数局部变量、传递给被调函数的参数等信息
栈底对应高地址，栈顶对应低地址，栈由内存高地址向低地址生长
一个进程的调用栈图示如下：

![stack](http://ovt2bylq8.bkt.clouddn.com/ddc3774bfc5f96440cea669e97c82486.png)

### 寄存器 (register)

寄存器位于 CPU 内部，用于存放程序执行中用到的数据和指令，CPU 从寄存器中取数据，相比从内存中取快得多。寄存器又分通用寄存器和特殊寄存器。

通用寄存器有 ax/bx/cx/dx/di/si，尽管这些寄存器在大多数指令中可以任意选用，但也有一些规定某些指令只能用某个特定 “通用” 寄存器，例如函数返回时需将返回值 mov 到 ax 寄存器中；特殊寄存器有 bp/sp/ip 等，特殊寄存器均有特定用途，例如 sp 寄存器用于存放以上提到的栈帧的栈顶地址，除此之外，不用于存放局部变量，或其他用途。

对于有特定用途的几个寄存器，简要介绍如下：

* ax(accumulator): 可用于存放函数返回值
* bp(base pointer): 用于存放执行中的函数对应的栈帧的栈底地址
* sp(stack poinger): 用于存放执行中的函数对应的栈帧的栈顶地址
* ip(instruction pointer): 指向当前执行指令的下一条指令


不同架构的 CPU，寄存器名称被添以不同前缀以指示寄存器的大小。例如对于 x86 架构，字母 “e” 用作名称前缀，指示各寄存器大小为 32 位；对于 x86_64 寄存器，字母 “r” 用作名称前缀，指示各寄存器大小为 64 位。

引用自: https://www.cnblogs.com/bangerlee/archive/2012/05/22/2508772.html

## 缓冲区溢出攻击

### 原理

https://www.cnblogs.com/fanzhidongyzby/archive/2013/08/10/3250405.html

### 解析

考虑以下程序:
```
1 void function(int a, int b)
2{
3
4  char buffer[10];
5
6}
7  void main()
8  {
9
10   func(1,2);
11
12 }
13
```
首先执行 main 函数, 函数中调用了 function 函数, 具体顺序为 7->9->10->1->3->4->5->11->12

执行 7 时, 因为 main 函数也是被调用的, 所以先将传给 main 函数的参数, 以及 main 函数的返回地址和调用 main 函数时 main 函数的栈底地址压入栈, 此时栈空间如下:
```
传入 main 的参数 arg n(此处没有)
 ...
传入 main 的参数 arg 1(此处没有)
---------------
main 函数返回地址, return address
---------------
上一个 ebp
---------------
main 函数内的局部变量 local var(此处没有)
---------------
```
执行到 10 的时候, 调用了 function 函数, 此时的栈空间如下
```
传入的参数 arg n(此处没有)
 ...
传入的参数 arg 1(此处没有)
---------------
main 函数返回地址, return address
---------------
上一个 ebp
---------------
main 函数内的局部变量 local var(此处没有)
---------------
b
---------------
a
---------------
function 的返回地址
---------------
main 函数的 ebp
---------------
```
执行到 5 的时候, 栈空间如下:
```
传入的参数 arg n(此处没有)
 ...
传入的参数 arg 1(此处没有)
---------------
main 函数返回地址, return address
---------------
上一个 ebp
---------------
main 函数内的局部变量 local var(此处没有)
---------------
b
---------------
a
---------------
function 的返回地址
---------------
main 函数的 ebp
---------------
buffer[8~11]
---------------
buffer[4~7]
---------------
buffer[0~3]
---------------
```
注意到:
* 分配了 12 个字节的空间给 buffer
* buffer 溢出的话会向高地址覆盖
* 溢出会依次覆盖 ebp 和返回地址

一旦更改了返回地址, 指向特定的值, 那么缓冲区溢出攻击就算达成了

### 实战:

样例代码:
```
#include<stdio.h>
void function(char b[])
{
	printf("b address is:%p\n",b);
	char buffer[4];
	strcpy(buffer,b);
	printf("buffer address is:%p\n",buffer);
}

void attack()
{
	printf("attack success!!\n");
}

void main()
{
	printf("attack function address is: %p\n",attack);
  char b[]="AA";
  //char b[]="AAAAAAAAAAAAAAAAAAAAAAAA\4\6@"; //todo
	function(b);
}
```
栈空间
```
main 函数返回地址, return address
---------------
上一个 ebp
---------------
b
---------------
function 的返回地址
---------------
main 函数的 ebp
---------------
buffer[0~3]
---------------
```
输出
```
attack function address is: 0x400604
b address is:0x7ffcf7cee710
buffer address is:0x7ffcf7cee6f0
```
### 步骤:

环境:
* Ubuntu 16.04 x86_64
* gcc 5.4.0
* gdb 7.11.1


1. 把样例代码保存为 example1.c, 使用以下命令进行编译
`gcc -g -z execstack -fno-stack-protector -o exe example1.c`
GCC 编译器有一种栈保护机制来阻止缓冲区溢出, 所以我们在编译代码时需要用 –fno-stack-protector 关闭这种机制. 而 -z execstack 用于允许执行栈.

2. 执行 gdb 调试
`gdb exe`
可以看到命令提示符改为 (gdb), 说明进入了 gdb 环境.
3. 启动项目 `start`
在 `printf("attack function address is: %p\n",attack);` 处自动打断停下,
4. `n` 下一步, 执行到 `char b[]="AA"; `,
5. `n` 下一步, 调用 function,
6. `s`step into, 进入 function 函数内部, 在 `printf("b address is:%p\n",b);` 处停下,
7. `n` 下一步, 在 `strcpy(buffer,b);` 处停止,
8. `n` 下一步, 执行完 `strcpy(buffer,b);`,

至此可以停下来了, 仍然在 gdb 环境中

`p buffer` 打印 buffer 内容
`p &buffer` 打印 buffer 虚拟内存地址
`x /16xw &buffer` 打印 buffer 虚拟内存地址及其之后的 16 个单元内容, 每个单元四个字节, 每个字节按十六进制显示,
![buffer_over_flow](http://ovt2bylq8.bkt.clouddn.com/b98d2d1f058c72c4a7cfcb9e0b23e460.png)

在 main 函数中查看 function 的返回地址 (因为是 main 函数调用了 function,function 执行完之后会回到 main 函数, 所以返回地址紧接着 main 函数调用 function 函数之后)

`disassemble main`

![disassemble_main](http://ovt2bylq8.bkt.clouddn.com/25c7e63b883a232e15975181565a80ee.png)

显示的四列中第一列代表虚拟内存地址, 第二列代表偏移量, 第三列汇编语言关键字, 第四列汇编语言参数, 在计算机组成原理中学过精简指令集, 格式为 <操作> < 目的 > < 源 >

可以看到在偏移量为 77 的时候调用了 function, 并且把 function 的起始地址 0x4005b6 给出了, 紧接着偏移量为 78 的虚拟地址就是 function 的返回地址 0x400647

再对比上面 `x /16wx &buffer` 的输出, 可以看到虚拟地址为 0x7fffffffd968(即第二行第三列) 正是 function 的返回地址 0x00400647,

接下来只要构造 b 的值使它溢出填充完整个第一行以及第二行前面的空间 (0x7fffffffd950~0x7fffffffd968)
注意到 buffer 的填充是从每个字节的右往左, 从低地址到高地址, 以第一行 (0x7fffffffd950) 为例
```
偏移地址
   4 3 2 1    8 7 6 5
0x00004141 0x00000000
内容

偏移地址
12 11 10 9 16 15 14 13
0xf7ffe168 0x00007fff
内容
```

所以 b 的内容是 24 个 A 再加上实际跳转地址, 这里我们想要的跳转地址是 attack 函数,

先查看 attack 函数的开始地址 `info line attack`

输出 `Line 11 of "example1.c" starts at address 0x400604 <attack> and ends at 0x400608 <attack+4>.
`

获得首地址为 0x400604, 上面说过, buffer 填充字节是从右到左, 所以 b 的内容为 `AAAAAAAAAAAAAAAAAAAAAAAA\4\6@`

在 [这里](https://baike.baidu.com/item/ASCII/309296?fr=aladdin) 可以查到十六进制的 40 是 @字符

再次编译, 运行, 成功

![success](http://ovt2bylq8.bkt.clouddn.com/12585db8e5f6fa9c4abbc3b8a04b6c5d.png)

参考链接:

buffer_over_flow:

[Linux 及安全实验一：缓冲区溢出漏洞实验](https://www.cnblogs.com/zkzkzkzkzk/p/4450398.html)

[Linux 下缓冲区溢出攻击的原理及对策](https://www.ibm.com/developerworks/cn/linux/l-overflow/index.html)

[课件](https://padlet.com/chunhuachen/sec)

gdb:

[很经典的 GDB 调试命令，包括查看变量，查看内存](https://www.cnblogs.com/rosesmall/archive/2012/04/12/2444431.html)

[Linux 学习 --gdb 调试](https://www.cnblogs.com/hankers/archive/2012/12/07/2806836.html)

[Linux 中用 gdb 查看代码堆栈的信息](https://www.cnblogs.com/chengliangsheng/p/3597010.html)

ASCII:

[ASCII](https://baike.baidu.com/item/ASCII/309296?fr=aladdin)

others:

[Ubuntu 下缓冲区溢出实验注意事项](http://www.aichengxu.com/linux/2502183.htm)
