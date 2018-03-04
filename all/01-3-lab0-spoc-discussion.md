# lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中是如何具体体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？
  > 硬件支持：中断，异常处理，段页式内存管理，虚实地址映射等
  > 特权指令：异常/中断管理指令，TLB/MMU管理，特权等级控制，段页式内存管理指令

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？
  > 实模式：不使用虚拟地址，直接使用物理内存地址，此时可访问的物理内存空间不超过1MB；保护模式：支持内存分页，支持虚拟内存，支持32位地址空间，虚地址经过MMU转换为物理地址，支持多任务，支持优先级，OS运行在最高优先级0上，应用程序运行在较低优先级上
  > 物理地址：处理器提交到总线上、用于访问计算机系统中内存和外设的最终地址；线性地址：OS在虚存管理下每个运行的应用程序能访问的地址空间，它们都认为自己独享整个计算机系统地址空间；逻辑地址：应用程序直接使用的地址空间

- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

  > 表示“位域”，即把字节中的二进制位划分为不同区域，并说明每个区域的位数，每个域有一个域名。如gd_off_15_0有16位，gd_ss有16位，gd_args有5位，等。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

  > gatedesc占用8Byte，即2个32位字。x86为小端序，执行指令后其对应内存单元从低地址起为：
  > 00000011 00000000 00000010 00000000 00000000 11110001 00000000 00000000.
  > 作为unsigned，intr为32位，其值为0x00020003

### 课堂实践练习

#### 练习一

请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

  - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)

  - ##### [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)

  - ##### [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)

  - ##### [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)

  - ##### [[IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)]

  > 如下代码：
  ```asm
      .include "defines.h"
      .data
      hello:
        .string "hello world\n"

      .globl	main
      main:
        movl	$SYS_write,%eax
        movl	$STDOUT,%ebx
        movl	$hello,%ecx
        movl	$12,%edx
        int	$0x80

        ret
  ```
  > 首先包含了头文件defines.h，内含各种常量定义；数据段定义了hello字符串值为"hello world\n"；代码段主函数将系统调用$SYS_write放置于%eax寄存器，将输出目标$STDOUT放置在%ebx寄存器，将输出对象$hello字符串放置在%ecx寄存器，将$hello字符串长度12放置在%edx寄存器，接着执行系统调用$0x80，将hello字符串内容输出到stdout，最后返回。


#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore中宏定义的用途，并举例描述其含义。

 > 利用宏进行复杂数据结构中的数据访问；
 > 利用宏进行数据类型转换；如 to_struct;
 > 常用功能的代码片段优化；如  ROUNDDOWN, SetPageDirty