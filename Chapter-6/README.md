## 章节六 GDT全局描述表

归功于GRUB，你的内核不再是 [real mode(实模式)](http://baike.baidu.com/view/404433.htm)，而是处于[protected mode（保护模式）](http://baike.baidu.com/view/177586.htm)，该模式允许我们使用微处理器的所有能力，比如 虚拟内存管理，分页，安全的多任务。

#### GDT 是什么？

[GDT ("Global Descriptor Table" 全局描述表)](http://baike.baidu.com/view/4109224.htm) 是一个用来定义不同内存区的数据结构，其包括:基地址，大小，访问特权（比如执行和写），这些区域被称为 "segments（段）"。

我们将使用GDT定义不同的内存段：

> 为了保留作者的原义，保留了该段原文

* *"code"*: kernel code, used to stored the executable binary code
* *"data"*: kernel data
* *"stack"*: kernel stack, used to stored the call stack during kernel execution
* *"ucode"*: user code, used to stored the executable binary code for user program
* *"udata"*: user program data
* *"ustack"*: user stack, used to stored the call stack during execution in userland

> 中文解释

* *"code"*: 内核代码，用来存储可执行二进制代码
* *"data"*: 内核数据
* *"stack"*: 内核栈，用来存储内核执行的调用栈 
* *"ucode"*: 用户代码，用来存储用户可执行程序的二进制代码
* *"udata"*: 用户程序数据
* *"ustack"*: 用户栈，用来存储用户态执行的调用栈


#### 如何加载GDT?

GRUB初始化一个GDT，但这个GDT不属于我们的内核。
GDT是使用 [LGDT(加载全局描述符) 汇编指令](http://www.0x01f.com/static/IntelCode/instruct32_hh/vc155.htm) 加载的。 LGDT需要一个GDT描述结构的位置。

GDT描述符：

![GDTR](./gdtr.png)

C 结构体:

```cpp
struct gdtr {
	u16 limite;
	u32 base;
} __attribute__ ((packed));
```

**注意:** 指令 ```__attribute__ ((packed))``` 用来告诉gcc，该结构体应该在内存中紧凑排布。没有该指令，gcc将包含一些用来优化执行期间访问内存对齐的字节。

现在我们需要定义我们自己的GDT，然后使用LGDT加载它。GDT可以被放在内存的任何位置，那么当程序员通过段寄存器来引用一个段描述符时，CPU必须知道GDT的入口，也就是基地址放在哪里，所以Intel的设计者门提供了一个寄存器GDTR用来存放GDT的入口地址，程序员将GDT设定在内存中某个位置之后，可以通过LGDT指令将GDT的入口地址装入此寄存器，从此以后，CPU就根据此寄存器中的内容作为GDT的入口来访问GDT了。

>全局描述符表寄存器GDTR
>
>GDTR寄存器中用于存放全局描述符表GDT的32位的线性基地址和16位的表限长值。基地址指定GDT表中字节0在线性地址空间中的地址，表长度指明GDT表的字节长度值。指令LGDT和SGDT分别用于加载和保存GDTR寄存器的内容。在机器刚加电或处理器复位后，基地址被默认地设置为0，而长度值被设置成0xFFFF。在保护模式初始化过程中必须给GDTR加载一个新值。

`GDT table` 是如下段描述符结构体组成的数组：

![GDTR](./gdtentry.png)

C 结构体:

```cpp
struct gdtdesc {
	u16 lim0_15;
	u16 base0_15;
	u8 base16_23;
	u8 acces;
	u8 lim16_19:4;
	u8 other:4;
	u8 base24_31;
} __attribute__ ((packed));
```

#### 如何定义我们的GDT table?

我们应该在内存中定义我们的GDT，然后使用 `LGDT指令`加载到`GDTR寄存器`。

我们将GDT存储在内存如下地址:

```cpp
#define GDTBASE	0x00000800
```

[x86.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/x86.cc) 中的函数 **init_gdt_desc** 初始化一个GDT段描述符：

```cpp
void init_gdt_desc(u32 base, u32 limite, u8 acces, u8 other, struct gdtdesc *desc)
{
	desc->lim0_15 = (limite & 0xffff);
	desc->base0_15 = (base & 0xffff);
	desc->base16_23 = (base & 0xff0000) >> 16;
	desc->acces = acces;
	desc->lim16_19 = (limite & 0xf0000) >> 16;
	desc->other = (other & 0xf);
	desc->base24_31 = (base & 0xff000000) >> 24;
	return;
}
```

函数 **init_gdt** 初始化GDT，下面涉及的其它函数后面将做解释，主要是用来多任务。

```cpp
void init_gdt(void)
{
	default_tss.debug_flag = 0x00;
	default_tss.io_map = 0x00;
	default_tss.esp0 = 0x1FFF0;
	default_tss.ss0 = 0x18;

	/* initialize gdt segments */
	init_gdt_desc(0x0, 0x0, 0x0, 0x0, &kgdt[0]);
	init_gdt_desc(0x0, 0xFFFFF, 0x9B, 0x0D, &kgdt[1]);	/* code */
	init_gdt_desc(0x0, 0xFFFFF, 0x93, 0x0D, &kgdt[2]);	/* data */
	init_gdt_desc(0x0, 0x0, 0x97, 0x0D, &kgdt[3]);		/* stack */

	init_gdt_desc(0x0, 0xFFFFF, 0xFF, 0x0D, &kgdt[4]);	/* ucode */
	init_gdt_desc(0x0, 0xFFFFF, 0xF3, 0x0D, &kgdt[5]);	/* udata */
	init_gdt_desc(0x0, 0x0, 0xF7, 0x0D, &kgdt[6]);		/* ustack */

	init_gdt_desc((u32) & default_tss, 0x67, 0xE9, 0x00, &kgdt[7]);	/* descripteur de tss */

	/* initialize the gdtr structure */
	kgdtr.limite = GDTSIZE * 8;
	kgdtr.base = GDTBASE;

	/* copy the gdtr to its memory area */
	memcpy((char *) kgdtr.base, (char *) kgdt, kgdtr.limite);

	/* load the gdtr registry */
	asm("lgdtl (kgdtr)");

	/* initiliaz the segments */
	asm("   movw $0x10, %ax	\n \
            movw %ax, %ds	\n \
            movw %ax, %es	\n \
            movw %ax, %fs	\n \
            movw %ax, %gs	\n \
            ljmp $0x08, $next	\n \
            next:		\n");
}
```


译者注：
* [Global_Descriptor_Table](http://wiki.osdev.org/Global_Descriptor_Table)
* [Bran的内核开发指南——全局描述字符表GDT](http://article.yeeyan.org/view/197439/169982)
* [Beginning Operating System Development, Part Three](http://www.codeproject.com/Articles/43179/Beginning-Operating-System-Development-Part-Three)
* [用GRUB引导内核--GDT的设置](http://www.cnblogs.com/bamboo-talking/archive/2012/04/06/2435501.html)
* [GDT(Global Descriptor Table)全局描述符表](http://www.cnblogs.com/starlitnext/archive/2013/03/07/2948929.html)

下一章: [IDT(中断描述符表)和中断](../Chapter-7/README.md/) 
