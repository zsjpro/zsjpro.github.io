---
title: uboot1.3.3_start.S分析（二）
description: 2018-6-19 22:39:34 start.S分析！
categories:
 - uboot1.3.3分析
tags: uboot
---
>我们接着uboot1.3.3_start.S分析（一）继续分析start.S

## 2018-6-19
	ldr     r0, =pWTCON
	mov     r1, #0x0
	str     r1, [r0]
关闭看门狗

	mov	r1, #0xffffffff
	ldr	r0, =INTMSK
	str	r1, [r0]
	#if defined(CONFIG_S3C2410)
	ldr	r1, =0x3ff
	ldr	r0, =INTSUBMSK
	str	r1, [r0]
	#endif

关中断

	ldr	r0, =CLKDIVN
	mov	r1, #3
	str	r1, [r0]

***设置时钟分频系数，此处存在问题，因为我们后面需要对SDRAM进行初始化，要修改时序及刷新周期，所以此处要进行完整的时钟初始化。移植的时候要进行修改。***

	#ifndef CONFIG_SKIP_LOWLEVEL_INIT
	bl	cpu_init_crit
	#endif
***跳转到标号cpu_init_crit执行，cpu_init_crit作用是清除I/Dcache，除能MMU，然后执行lowlevel_init，lowlevel_init就是对SDRAM进行初始化。此处在移植的时候也需要根据技术手册实际情况进行修改。***

#### 重定位

	#ifndef CONFIG_SKIP_RELOCATE_UBOOT
	relocate:
		adr	r0, _start	     	/* 代码的存储位置 0  */
		ldr	r1, _TEXT_BASE		/* 链接地址 0x33f80000在u-boot-1.3.3/board/smdk2410/config.mk */
		cmp     r0, r1          /* 判断是否相同是否需要重定位 */
		beq     stack_setup
	
		ldr	r2, _armboot_start  /*_start的地址，也就是地址0*/
		ldr	r3, _bss_start		/*bss段的起始地址*/
		sub	r2, r3, r2		/* uboot程序的大小，程序的大小是不包含bss段的，所以程序的大小就等于bss段的起始地址减去程序的首地址*/
		add	r2, r0, r2		/* 程序的结束地址*/
		/*重定位，也就是拷贝*/
	copy_loop:
		ldmia	r0!, {r3-r10}		/* copy from source address [r0]    */
		stmia	r1!, {r3-r10}		/* copy to   target address [r1]    */
		cmp	r0, r2			/* until source end addreee [r2]    */
		ble	copy_loop
	#endif
***为什么进行重定位？***

***重定位就是在链接地址跟运行地址不同的情况下，执行一段位置无关码，这段位置无关码的作用就是将原来的那份代码全部复制到链接地址那里去，然后自己再长跳转到新的那份代码的刚刚执行的那个位置。这样就实现了链接地址跟运行地址一致的情况了。***

adr指令：
&emsp;简单说就是将地址读出来，adr	r0, _start就是将_start的地址读取到r0，而不是读取地址_start的内容。
cmp指令：
&emsp;比较指令，相当于减法指令，而beq则是跳转指令，例如：

	cmp     r0, r1   
	beq     stack_setup

&emsp;就是说当r0和r1相等的时候就执行beq跳转到stack_setup处执行。相等就表示链接地址和程序的存储地址一致，就不需要进行重定位跳转到stack_setup处执行。

	ldr	r2, _armboot_start
	ldr	r3, _bss_start
	sub	r2, r3, r2/*目前r2存放的是程序的大小*/
	add	r2, r0, r2	
&emsp;r2存储的是程序的存储地址，r3是bss段的起始地址，那么用r3减去r2就是程序的大小，r0是程序的存储地址，存储首地址加上程序的大小就是程序的存储的结束地址。
	
	copy_loop:
	ldmia	r0!, {r3-r10}
	stmia	r1!, {r3-r10}
	cmp	r0, r2
	ble	copy_loop

&emsp;ldmia指令是将寄存器出栈，而stmia是将寄存器压栈，r0存储地址，r1链接地址，然后比较r0和r2也就是程序的结束地址来判断是否拷贝完成从而达到重定位的目的。

### 如果发现错误之处请联系email:sj9198@outlook.com
### QQ群:150963666