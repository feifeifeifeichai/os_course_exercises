# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

进程的切换需要硬件支持时钟中断；虚存管理需要地址映射机制，从而需要MMU等硬件；对于文件系统，需要硬件有稳定的存储介质来保证操作系统的持久性。 

至少应当提供中断使能，触发软中断等中断相关的，设置内存寻址模式，设置页表等内存管理相关的，执行I/O操作等文件系统相关的特权指令。


- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？

保护模式和实模式的根本区别是进程内存是否受保护。实模式将整个物理内存看成分段的区域，程序代码和数据位于不同区域，系统程序和用户程序没有区别对待，而且每一个指针都是指向“实在”的物理地址。保护模式：物理内存地址不能直接被程序访问，程序内部的地址（虚拟地址）要由操作系统转化为物理地址去访问。

从实模式切换到保护模式需要注意：初始化准备GDT；加载GDT的物理地址到寄存器gdtr中；关闭中断，因为保护模式和实模式的中断处理机制不相同；接着将寄存器cr0的PE位置1，0表示实模式，1表示保护模式；最后跳转到保护模式。

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？

物理地址：是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。
逻辑地址：在有地址变换功能的计算机中，访问指令给出的地址叫逻辑地址。（一般的定义是段选择子+段内偏移量是逻辑地址。）
线性地址：线性地址是逻辑地址到物理地址变换之间的中间层，是处理器通过段(Segment)机制控制下的形成的地址空间。

联系：程序中使用逻辑地址，逻辑地址通过中间层线性地址转化为最终的物理地址。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

RISC-V架构定义了三种工作模式，又称特权模式（Privileged Mode）：

Machine Mode：机器模式，简称M Mode。

Supervisor Mode：监督模式，简称S Mode。

User Mode：用户模式，简称U Mode。

RISC-V架构定义M Mode为必选模式，另外两种为可选模式。通过不同的模式组合可以实现不同的系统。

RISC-V架构也支持几种不同的存储器地址管理机制，包括对于物理地址和虚拟地址的管理机制，使得RISC-V架构能够支持从简单的嵌入式系统（直接操作物理地址）到复杂的操作系统（直接操作虚拟地址）的各种系统。


- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

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
“:”后的数字表示每一个域在结构体中所占的位数。

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

0x20003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

kern_entry:
    # load pa of boot pgdir 
    movl $REALLOC(__boot_pgdir), %eax
    movl %eax, %cr3

    # enable paging
    movl %cr0, %eax
    orl $(CR0_PE | CR0_PG | CR0_AM | CR0_WP | CR0_NE | CR0_TS | CR0_EM | CR0_MP), %eax
    andl $~(CR0_TS | CR0_EM), %eax
    movl %eax, %cr0

    # update eip
    # now, eip = 0x1.....
    leal next, %eax
    # set eip = KERNBASE + 0x1.....
    jmp *%eax

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

利用宏进行复杂数据结构中的数据访问；

利用宏进行数据类型转换；如 to_struct；

常用功能的代码片段优化；如 ROUNDDOWN, SetPageDirty。


## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
