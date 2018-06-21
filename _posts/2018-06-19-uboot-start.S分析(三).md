---
title: uboot1.3.3_start.S分析（三）
description: 2018-6-20 23:09:57 start.S分析！
categories:
 - uboot1.3.3分析
tags: uboot
---
>我们接着uboot1.3.3_start.S分析（二）继续分析start.S

## 2018-6-20
	stack_setup:
	ldr	r0, _TEXT_BASE		/* upper 128 KiB: relocated uboot   */
	sub	r0, r0, #CFG_MALLOC_LEN	/* malloc area                      */
	sub	r0, r0, #CFG_GBL_DATA_SIZE /* bdinfo                        */
	#ifdef CONFIG_USE_IRQ
	sub	r0, r0, #(CONFIG_STACKSIZE_IRQ+CONFIG_STACKSIZE_FIQ)
	#endif
	sub	sp, r0, #12		/* leave 3 words for abort-stack    */

&emsp;设置栈，为什么设置栈？

***首先我们了解栈的作用：***

***1.保存现场/上下文:***

***&emsp;当CPU执行的时候需要用到r0、r1等寄存器，那么原本寄存器中是存在值的，当我们用到的时候很可能会赋予新的数值，那么原来的数值就需要进行压栈来保存，因为原来的数值可能会被其他的函数调用，如果不进行保存就会出现错误数值被应用。但是当我们调用完成以后就要出栈，将原来的数值在恢复到r0、r1中以供别的函数使用。这就是保存现场。***

***2.传递参数，在汇编调用c函数的时候需要传递参数。***

***&emsp;汇编调用c函数的时候，如果c函数定义有形参，那么第一个形参的值就需要存在r0寄存器中，第二个形参的值存在r1中，以此类推，如果参数小于4个，那么就直接通过寄存器传输，但是如果参数大于4个，这个时候就需要用到栈来保存了。***

***3.保存临时变量。***

***&emsp;包括函数的非静态局部变量以及编译器自动生成的其他临时变量。***



### 如果发现错误之处请联系email:sj9198@outlook.com
### QQ群:150963666