Advanced Programmable Interrupt Controller (APIC)用于CPU管理和路由中断。分为两个部分

- local APIC，整合进每个逻辑核中。
- external I/O APIC（in the Intel’s System Chip Set）

本地APIC管理处理器间中断（IPIs）并接收来自处理器中断引脚的中断信号，然后将这些信号转发到核心进行处理，而I/O APIC接收外部中断事件并将其转发到目标本地APICs 。

- Local APIC：一个本地APIC可以接收、生成和转发中断，既可以将中断发送给它的本地核心（通过处理器间中断（IPIs）或本地中断源，例如定时器中断、性能监控计数器中断、热传感器中断），也可以发送给其他核心（通过IPIs），还可以接收来自外部设备的中断（通过I/O APIC）。每个本地APIC由一组APIC寄存器组成，用于控制其功能或暴露系统中断的状态。处理器可以通过其本地APIC的APIC寄存器生成IPIs或设置本地中断。

- Local APIC Registers：默认情况下，现代APIC以xAPIC模式运行，将本地APIC寄存器作为内存映射的4 kB区域暴露在物理地址空间中。该区域的地址通过IA32_APIC_BASE MSR设置，并且每个逻辑核心都是独立的[28]。在启动时，该区域的默认物理地址为0xFEE00000，但可以通过修改IA32_APIC_BASE MSR的值来按核心移动此区域。本地APIC寄存器的宽度可能为32、64或256位，但它们在内存映射区域中作为32位值进行映射，并始终对齐到128位边界。因此，宽于32位的寄存器会被分割并映射到多个128位对齐的内存映射区域。这意味着每个16字节（128位）区域中的第4到第15字节在架构上未定义。Intel表示，任何涉及访问APIC寄存器的第4到第15字节的操作都可能导致未定义行为，且不得执行[28]。如果支持，APIC可以设置为x2APIC模式，该模式在xAPIC的基础上进行了多项改进，例如提高中断传递性能并提供基于MSR的APIC寄存器访问，从而禁用内存映射接口。操作系统可以通过设置IA32_APIC_BASE MSR的第10位来启用或禁用x2APIC模式。

### Software and Hardware Vulnerabilities

分析漏洞的时候从高层次，将漏洞分为architectural and transient vulnerabilities.

仅依赖于架构定义的接口和功能就可以利用架构漏洞。瞬态漏洞没有架构可见的影响，因为它们仅在微架构级别可见，因此需要侧通道来观察它们。

作者认为，硬件漏洞和软件漏洞根本原因上没有太大不同，使用现有的软件漏洞进行分类。

![image-20241012100921863](C:\Users\王佳顺\Desktop\周五学术日\ÆPIC_Leak\picture1.png)

#### Out-of-bounds Operation (CWE-119)

Improper Restriction of Operations within the Bounds of a Memory Buffer（内存缓冲区操作范围不当）

软件对memory buffer进行操作，但它可以在缓冲区的预期边界之外读取或写入内存位置

#### Use after Free（CWE-416）

即使这些内部缓冲区（如行填充缓冲区和存储缓冲区）的条目已经被之前完成的加载（或存储）操作释放，旧内容仍然在故障加载中被瞬态使用。（MDS）

#### Confused Deputy（CWE-411）

混淆代理漏洞是指中介在转发请求到目标资源时，没有保留关于源头访问权限的信息。

#### Type Confusion（CWE-843）

在Foreshadow-VMM[83]变种的Foreshadow攻击[72]中，CPU在处理客户机页表的页表项时出现了类型混淆。在虚拟机内发生缺页错误时，CPU将页表项误当作主机页表，错误地将存储的页框号解释为主机物理地址，而不是客户机物理地址。

#### Incorrect Calculation（CWE-682）

#### Race Condition（CWE-362）

条件竞争，例如数据在CPU意识到虚拟地址指向架构上不可访问的数据之前，已经被访问并转发给依赖的操作。

#### Improper Neutralization (Injection) (CWE-74)

软件使用来自上游组件的外部影响输入来构造命令、数据结构或记录的一部分或全部，但没有中和，或错误地中和了可能修改其解析或解释方式的特殊元素，当这些元素被发送到下游组件时会改变其行为。

#### Improper Initialization (CWE-665)

初始化不当

