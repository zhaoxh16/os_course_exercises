# lec2：lab0 SPOC思考题

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
  **Answer**: 
  - ucore属于分时操作系统，切换进程的方式是根据时钟中断来切换，因此硬件设计上需要支持时钟外设；虚存与物理内存的映射需要MMU硬件来控制；文件系统需要由存储介质（如SATA，SSD，FLASH等）硬件来支持。
  - 为了提供这些功能，需要提供以下特权指令：
    - 允许和禁止中断，控制中断禁止屏蔽位的特权指令
    - 在进程间切换处理的指令
    - 执行I/O操作的特权指令
    - 设置时钟的特权指令



- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？
  **Answer**: 

  - 实模式是为了兼容早期的8086机器而做的，寻址空间不超过1M，且无法发挥80386以上级别的32位CPU的4GB内存管理能力。
    保护模式有32位（4G）寻址空间；支持内存分页机制，提供了对虚拟内存的良好支持；支持多任务和优先级机制，不同程序可以运行在不同的优先级上，操作系统运行在最高的优先级0上；配合良好的检查机制后，可以在任务间实现数据的安全共享和隔离。

  - 物理地址内存空间是处理器提交到总线上用于访问计算机系统中内存和外设的最终地址。一个计算机系统中只有一个物理地址空间。
    线性地址空间是在操作系统的虚存管理下每个运行的应用程序能访问的地址空间。每个运行的应用程序都认为自己独享整个计算机系统的地址空间，这样可以让多个运行的应用程序之间相互隔离。

    逻辑地址空间是应用程序直接使用的地址空间。



- 你理解的RV的特权模式有什么区别？不同 模式在地址访问方面有何特征？

  **Answer**:

  - RISC-V包含可选的三种模式：用户模式，机器模式和监督模式，其中用户模式为一般用户程序所在的模式，而机器模式和监督模式都有比用户模式更高的权限。
  - 机器模式（M模式）是RISC-V中可以执行的最高权限模式，可以自由地访问硬件平台。它的最重要的特性是拦截和处理异常和中断的能力。在机器模式下直接操作物理地址。
  - 用户模式（U模式）（可选）禁止执行特权指令和访问特权控制状态寄存器，同时每个程序被限制只能访问自己的那部分内存，这个内存的位置由机器模式通过物理内存保护功能指定。
  - 监管模式（S模式）（可选）是为了使处理器可以提供更好的通用计算功能而设立的。默认情况下，M模式执行所有的异常处理程序，但RISC-V提供了一种异常委托模式，可以选择性地将中断和同步异常交给S模式处理，从而完全绕过M模式。同时，S模式提供了一种传统的虚拟内存系统，将内存划分为固定大小的页来进行地址转换和对内存内容的保护。

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

### 课堂实践练习

#### 练习一

请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

  - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)

  - ##### [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)

  - ##### [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)

  - ##### [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)

  - ##### [[IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)]


请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

 > 利用宏进行复杂数据结构中的数据访问；
 > 利用宏进行数据类型转换；如 to_struct, 
 > 常用功能的代码片段优化；如  ROUNDDOWN, SetPageDirty
