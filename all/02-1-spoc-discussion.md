# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  请描述在“计算机组成原理课”上，同学们做的MIPS CPU是从按复位键开始到可以接收按键输入之间的启动过程。

    - 当复位键按下之后，CPU默认从一个写定的初始位置开始读取代码执行。kernel会进行一些内存的初始化，和系统的配置（如果打开了中断支持），如果打开了TLB支持，还会进行页表的填充和TLB的初始化。然后进入到用户态，建立用户栈空间，打印欢迎消息，并开始循环接收输入（串口轮询或等待中断）。

-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

    - BIOS从磁盘读入的第一个扇区内容是磁盘的主引导扇区代码。

    - 因为需要通过磁盘上的主引导记录确定要加载哪一个操作系统；对于不同机器有不同的文件系统，BIOS不知道硬盘中存放的是什么，因此无法直接读入操作系统内核映像。

- 比较UEFI和BIOS的区别。

    - UEFI是一种个人电脑系统规格，用来定义操作系统与系统固件之间的软件界面，作为BIOS的替代方案。

    - UEFI启动对比BIOS启动有几个优势：
        - 1、安全性更强，UEFI启动需要一个独立的分区，这样将系统启动文件和操作系统隔离，可以更好的保护系统的启动；
        - 2、启动配置灵活；
        - 3、支持容量更大，BIOS受到MBR格式限制，无法引导大小超过2TB的硬盘。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

   - 0X55AA

- x86中在UEFI中的可信启动有什么作用？

   - 通过启动前的数字签名检查来保证启动介质的安全性

- RV中BBL的启动过程大致包括哪些内容？

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

   - 中断：外部意外的响应；
   - 异常：指令执行意外的响应；
   - 系统调用：系统调用指令的响应；

-  中断、异常和系统调用的处理流程有什么异同？

    - 相同点：
        - 都会进入异常服务例程，切换为内核态。
    - 不同点：
        - 源头不同，中断源是外部设备，异常和系统调用源是应用程序；
        - 响应方式不同，中断是异步的，异常是同步的，系统调用异步和同步都可以。
        - 处理机制不同，中断对用户程序是透明的，异常会重新执行用户指令或杀死用户进程，系统调用一般是用户程序调用的

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

    - ucore lab8的answer中共有22个系统调用，
    
    > static int (*syscalls[])(uint32_t arg[]) = {
    [SYS_exit]              sys_exit,
    [SYS_fork]              sys_fork,
    [SYS_wait]              sys_wait,
    [SYS_exec]              sys_exec,
    [SYS_yield]             sys_yield,
    [SYS_kill]              sys_kill,
    [SYS_getpid]            sys_getpid,
    [SYS_putc]              sys_putc,
    [SYS_pgdir]             sys_pgdir,
    [SYS_gettime]           sys_gettime,
    [SYS_lab6_set_priority] sys_lab6_set_priority,
    [SYS_sleep]             sys_sleep,
    [SYS_open]              sys_open,
    [SYS_close]             sys_close,
    [SYS_read]              sys_read,
    [SYS_write]             sys_write,
    [SYS_seek]              sys_seek,
    [SYS_fstat]             sys_fstat,
    [SYS_fsync]             sys_fsync,
    [SYS_getcwd]            sys_getcwd,
    [SYS_getdirentry]       sys_getdirentry,
    [SYS_dup]               sys_dup,
};
    
    分为如下几类 
    1. 进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep 
    2. 文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup 
    3. 内存管理：pgdir命令 
    4. 外设输出：putc命令

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

    - 汇编指令的区别
        - 系统调用：使用INT和IRET指令
        - 函数调用：使用CALL和RET指令
    - 安全性的区别
        - 系统调用有堆栈和特权级的转换过程，函数调用没有这样的过程，系统调用相对更为安全
    - 性能的区别
        - 时间角度：系统调用比函数调用要做更多和特权级切换的工作，所以需要更多的时间开销
        - 空间角度：在一些情况下，如果函数调用采用静态编译，往往需要大量的空间开销，此时系统调用更具有优势

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

    - 函数调用的堆栈操作是把返回地址压栈，而系统调用的堆栈操作有硬件实现的部分

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

    - 系统调用的堆栈操作是软件实现的，epc等寄存器的保存。函数调用则是把返回地址写入寄存器。


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
