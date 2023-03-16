---

title: 80386 系统寄存器
date: 2019-02-12 10:09:43

---

80386 的许多结构特性都只能被系统程序员所使用。这一章将对它的系统结构做一个概述。
80386 系统级特性包括以下几点：
+ 内存管理
+ 保护机制
+ 多任务
+ 输入/输出
+ 中断和异常
+ 初始化过程
+ 协处理器和多处理器
+ 调试

<!--more-->
这些特性是通过寄存器和指令实现的，以下几节将对它们进行介绍。这一章的目的只是对以后几章的一个概述，而不是对所有的细节进行描述。这一章所讲到的寄存器、指令将会做一些解释，或者在以后的章节中对其进行详细的说明。

## 系统寄存器 (System Registers)

为系统程序员设计的寄存器可以分为以下几类：

+ EFLAGS (标志寄存器)

+ Memory-Management Registers (内存管理寄存器)

+ Control Registers (控制寄存器)

+ Debug Registers (调试寄存器)

+ Test Registers (测试寄存器)

### 系统标志 (System Flags)

系统标志寄存器 EFLAGS 控制着 I/O、可屏蔽中断 (maskable interupts)、调试 (debuging)、任务切换 (task switching)、保护模式下虚拟 8086 方式的执行、多作务环境 (multitasking environment)。

+ IF (中断许可标志 Interrupt-Enable Flag，比特位 9)

> 设置 IF 使 CPU 可识别外部 (可屏蔽) 中断请求。复位 IF 则禁止中断。IF 对不可屏蔽外部中断和异常的识别没有任何作用。关于中断的详细信息，请参看第 9 章的描述。

+ NT (嵌套任务 Nested Task，比特位 14)

> 处理器用嵌套位来控制被中断或被调用的任务链。NT 对 IRET 指令的操作有影响。更多的信息请参看第 7 章和第 9 章。

+ RF (继续位 Resume Flag，比特位 16)

> RF 位暂时禁止调试异常，以便一条指令可以在一个调试异常结束后立即重看书而且不会引发另一个调试异常。参看第 12 章以获得更多的细节。

+ TF (陷阱位 Trap Flag，比特位 8)

> 设置 TF 可以让处理器工作在单步调试模式。在此模式下，CPU 每执行完一条指令后将自动引发一个异常，这样可以在程序每执行完一条指令后对程序进行查询。单步仅仅是 80386 众多调试特性的一个。参看第 12 章以获得更多的细节。

+ VM (虚拟 8086 模式 Virtual 8086 Mode，比特位 17)

> 当此位设置时，VM 标志说明一个任务正在执行一个 8086 程序。参看第 14 章以得到更多 80386 在保护模式下执行 8086 任务、多任务环境的详细介绍。

### 内存管理寄存器 (Memory–Management Registers)

80386 有 4 个寄存器来寻址特定的数据结构，它们用来实现段式内存管理。

+ GDTR 全局描述符表寄存器 (Global Descriptor Table Register)

+ LDTR 局部描述符表寄存器 (Local Descriptor Table Register)

> 这些寄存器指向段描述符表 GDT 和 LDT。第 5 章对通过描述符表来寻址的机制做了详细的介绍。

+ IDTR 中断描述符表寄存器 (Interrupt Descriptor Table Register)

> 这个寄存器指向一张包含中断处理子程序入口点的表 (IDT)。第 9 章对中断机制进行的详细介绍。

+ TR 任务寄存器 (Task Register)

> 这个寄存器指向当前任务信息存放处，这些信息是处理器所需要的。第 7 章对 80386 的多任务特性做了介绍。

### 控制寄存器 (Control Registers)

图 4-2 显示了 80386 的控制寄存器，CR0、CR2、和 CR3。这些寄存器可以通过 MOV 指令的一些变种形式被系统程序员所访问，这样便可以把它们存入通用寄存器或从通用寄存器中加载，例如：

```
MOV      EAX ,     CR0
MOV      CR3 ,     EBX
```

+ CR0 包含系统控制标志，这些标志控制着整个系统的运行，而不仅仅是针对某一个特定的任务。

+ EM (摸拟位 Emulation，比特位 2)

> EM 指示协处理器功能是否通过摸拟来实现。更多的信息请参看第 11 章。

+ ET (扩展类型 Extension Type，比特位 4)

> ET 指明了系统内协处理器的类型 (80287 或 80387)。详细情况请查看第 11 章和第 10 章。

+ MP (数学部件存在 Math Present，比特位 1)

> MP 控制 WAIT 指令的执行，WAIT 用于系统与协处理器的同步。第 11 章对其进行详细介绍。

+ PE (保护模式允许 Protection Enable，比特位 0)

> 设置 PE 将让处理器工作在保护模式下。复位 PE 将返回到实模式工作。关于模式切换请参看第 14 章和第 10 章。

+ PG (分页允许 Paging，比特位 31)

> PG 指明处理器是否通过页表来转换线性地址到物理地址。关于分页地址转换请查看第 5 章。关于如何设置 PG 位，请查看第 10 章。

+ TS (任务已切换 Task Switched，比特位 3)

> 处理器第次做任务切换时将设置 TS 位，当执行协处理器指令时将会测试 TS 位。详细信息请查看第 11 章。

+ CR2 被用来当 PG 位置位时，处理缺页异常。当发生缺页异常时，处理器自动将引起缺页异常的线性地址存放到 CR2。关于缺页中断请查看第 9 章。

+ CR3 只有当 PG 位设置时才有用。通过 CR3，CPU 可以定位当前任务的页目录表。关于页表和页地址转换机制请查看第 5 章。

### 调试寄存器 (Debug Register)

调试寄存器使 80386 有很好的调试功能，包括断点、不改变代码段情况下设置指令断点。关于它们的格式和用途请参看第 12 章。

### 测试寄存器 (Test Registers)

测试寄存器并不是 80386 体系结构的标准部件。它们仅仅是用来测试 TLB 地址转换信息，这些存贮的是来自页表中的。查看第 12 章，关于怎样使用这些寄存器。

## 系统指令 (System Instructions)

系统指令能完成以下功能：

1. 检测指针参数 (Verification of pointer parameters)(参看第 6 章)：
   + ARPL —— 调整 RPL (Adjust RPL)
   + LAR —— 加载访问权限 (Load Access Rights)
   + LSL —— 加载段界限 (Load Segment Limit)
   + VERR —— 读检验 (Verify for Reading)
   + VERW —— 写检验 (Verify for Writing)
2. 寻址描述符表 (Addressing descriptor tables)(参看第 5 章)：
  + LLDT —— 加载局部描述符表寄存器 (Load LDT Register)
  + SLDT —— 存储局部描述符表寄存器 (Store LDT Register)
  + LGDT —— 加载全局描述符表寄存器 (Load GDT Register)
  + SGDT —— 存储全局描述符表寄存器 (Store GDT Register)

3. 多任务 (Multitasking) (参看第 7 章)：

   + LTR —— 加载任务寄存器 (Load Task Register)

   + STR —— 存储任务寄存器 (Store Task Register)

4. 协处理器和多处理器 (Coprocessing and Multiprocessing) (参看第 11 章)：

   + CLTS —— 清除任务已切换标志 (Clear Task-Switched Flag)

   + ESC —— 转译指令 (Escape instructions)

   + WAIT —— 等待直到协处理器空闲 (Wait until Coprocessor not Busy)

   + LOCK —— 引发总线锁信号 (Assert Bus-Lock Signal)

5. 输入和输出 (Input and Output) (参看第 8 章)：
   + IN —— 输入
   + OUT —— 输出
   + INS —— 输入串
   + OUTS —— 输出串

6. 中断控制 (Interrupt control) (参看第 9 章)：
   + CLI —— 清除中断允许标志位 (Clear Interrupt-Enable Flag)
   + STI —— 设置不断允许标志位 (Set Interrupt-Enable Flag)
   + LIDT —— 加载中断描述符表寄存器 (Load IDT Register)
   + SIDT —— 存储中断描述符表寄存器 (Store IDT Register)

7. 调试 (Debugging) (参看第 12 章)：
   + MOV —— 向调试寄存器输入或输出 (Move to and from debug registers)

8. TLB 测试 (TLB testing)(参看第 10 章)：
   + MOV —— 向测试寄存器输入或输出 (Move to and from test registers)

9. 系统控制 (System Control)：
   + SMSW —— 保存机器状态字 (Set MSW)
   + LMSW —— 加载机器状态字 (Load MSW)
   + HLT —— 处理器挂起 (HALT Processor)
   + MOV —— 向控制寄存器输入或输出 (Move to and from control registers)

> SMSW 和 LMSW 指令主要用于兼容 80286 处理器。80386 程序可以通过变形的 MOV 指令访问 CR0，来访问 MSW。HLT 指令使处理器停止工作，直到收到了个 INTR 或者 RESET 信号。

除了在上面提到的章节外，每条指令还可以在第 17 章，中找到相关介绍。
