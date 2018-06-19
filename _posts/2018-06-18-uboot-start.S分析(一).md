---
title: uboot1.3.3_start.S分析（一）
description: 2018-6-18 22:39:34 start.S分析！
categories:
 - uboot1.3.3分析
tags: uboot
---
>前面我通过分析链接脚本我们发现程序的第一个文件是cpu/arm920t/start.S编译得到的，所以我们的程序入口肯定在start.S里面！下面我们进行分析！

# start.S分析：
## 2018-6-18

	.globl _start
	_start:	b       start_code
		ldr	pc, _undefined_instruction
		ldr	pc, _software_interrupt
		ldr	pc, _prefetch_abort
		ldr	pc, _data_abort
		ldr	pc, _not_used
		ldr	pc, _irq
		ldr	pc, _fiq

	_undefined_instruction:	.word undefined_instruction
	_software_interrupt:	.word software_interrupt
	_prefetch_abort:	.word prefetch_abort
	_data_abort:		.word data_abort
	_not_used:		.word not_used
	_irq:			.word irq
	_fiq:			.word fiq

	.balignl 16,0xdeadbeef

&emsp;.globl _start中的_start是标号，在汇编语言中标号就是地址，而.globl是声明标号_start可以被外部访问的全局符号，也就是类细于c语言中的extern，意思就是告诉汇编器_start这个符号要被链接器用到。

&emsp;_start:b start_code该跳转到标号start_code处执行程序，b是跳转指令，是不返回的跳转。

&emsp;ldr pc, _undefined_instruction，ldr指令以读取地址空间中的内容传递到通用寄存器里面，也就是说地址_undefined_instruction处的内容被读取到pc寄存器。地址_undefined_instruction处的内容是.word undefined_instruction，.word是伪指令，在arm中代表的是32位，作用是声明常量。所以就是将地址_undefined_instruction中的内容传给pc，也就是将地址undefined_instruction传给pc，也就是pc指向undefined_instruction，即跳转到标号地址undefined_instruction处执行。

&emsp;.balignl 16,0xdeadbeef中.balignl的含义就是以当前地址为开始开始，找到第一次出现的以第一个参数为整数倍的地址，并将其作为结束地址，在这个结束地址前面存储一个字节长度的数据，存储内容正是第二个参数。如果当前地址正好是第一个参数的倍数，则没有数据被写入到内存。0xdeadbeef就是内存做标记，插在那里，就表示从这个位置往后的一段有特殊作用的内存，而这个位置往前，禁止访问。

	start_code:
		mrs	r0,cpsr
		bic	r0,r0,#0x1f
		orr	r0,r0,#0xd3
		msr	cpsr,r0
&emsp;设置cpu为svc32模式，对于cpsr/spsr寄存器的写入或者读取必须使用指令msr/mrs，对于cpsr的各位含义如下图：
![](https://i.imgur.com/E7UOpsA.png)

N：负数标志位。如果目标寄存器中的有符号数为负数，则N=1，否则N=0。

Z：零标志位。如果目标寄存器中的数为0，则N=1，否则N=1。

C：进位标志位。有以下3种情况

1.无符号加法运算和CMN指令，如果产生进位，则C=1，否则C=0；

2.无符号减法运算和CMP指令，如果产生借位，则C=0，否则C=1；

3.进行移位操作的时候，C中保存最后一位移出的值。

说明：当一条指令中同时含有算术运算指令和移位指令时，影响C的值是算术运算而不是移位操      作。

V：溢出标志位。进行有符号运算时如果发生错误，则V=1，否则V=0。  

一些指令如CMN、TEQ等会无条件的刷新CPSR中的条件标志位，其他指令必须要在指令后面加上S后缀才会改变CPSR中的条件标志位。
 
I：IRQ中断禁止位。I=1代表禁止IRQ中断，I=0代表允许IRQ中断。

F：FIQ中断禁止位。F=1代表禁止FIQ中断，F=0代表允许FIQ中断。
这里和51单片机中的中断使能位有点小差别，51中的是中断使能位，所以为1的时候应该是中断使能，即允许中断。而这里是中断禁止位，为1的时候应该是禁止中断。
 
T：这一位只在ARMv4T指令集版本及以上才有效。因为ARMv4版本及以下都不支持Thumb指令集。在支持Thumb指令集的处理器中，T=0表示处于ARM状态，T=1表示处于Thumb状态。
 
M[4:0]：用于控制7种模式位。

![](https://i.imgur.com/E5jctaM.png)


### 如果发现错误之处请联系email:sj9198@outlook.com
### QQ群:150963666