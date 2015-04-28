## 章节六 GDT全局描述表


归功于GRUB，你的内核不再是 [real mode(实模式)](http://baike.baidu.com/view/404433.htm)，而是处于[protected mode（保护模式）](http://baike.baidu.com/view/177586.htm)，该模式允许我们使用微处理器的所有能力，比如 虚拟内存管理，分页，安全的多任务。

#### GDT 是什么？

[GDT](http://baike.baidu.com/view/4109224.htm) ("Global Descriptor Table" 全局描述表) 是一个用来定义不同内存区的数据结构，其包括:基地址，大小，访问特权（比如执行和写），这些区域被称为 "segments（段）"。

我们将使用GDT定义不同的内存段：

> 为了保留作者的原义，保留了该段原文

* *"code"*: kernel code, used to stored the executable binary code
* *"data"*: kernel data
* *"stack"*: kernel stack, used to stored the call stack during kernel execution
* *"ucode"*: user code, used to stored the executable binary code for user program
* *"udata"*: user program data
* *"ustack"*: user stack, used to stored the call stack during execution in userland


* *"code"*: 内核代码，用来存储二进制执行代码
* *"data"*: 内核数据
* *"stack"*: 内核栈，用来存储内核执行的调用栈 
* *"ucode"*: 用户代码，用来存储用户程序的二进制代码
* *"udata"*: 用户程序数据
* *"ustack"*: 用户栈，用来存储用户态执行的调用栈


#### 如何加载GDT?

GRUB初始化一个GDT，但这个GDT不属于我们的内核。
这个GDT是使用 LGDT(加载全局描述符) 汇编指令加载的。 它用来获取一个GDT描述结构体的位置。

GRUB initializes a GDT but this GDT is does not correspond to our kernel.
The GDT is loaded using the LGDT assembly instruction. It expects the location of a GDT description structure:

![GDTR](./gdtr.png)

C 结构体:

```cpp
struct gdtr {
	u16 limite;
	u32 base;
} __attribute__ ((packed));
```

**注意:** 指令 ```__attribute__ ((packed))``` 用来告诉gcc，该结构体应该尽可能使用少的内存。没有该指令，gcc将包含一些用来优化执行期间访问内存对齐的字节。

现在我们需要定义我们自己的GDT，然后使用LGDT加载它。GDT 能够存储在内存中的任何位置，它的地址应该通过 GDTR注册来通知给进程。

Now we need to define our GDT table and then load it using LGDT. The GDT table can be stored wherever we want in memory, its address should just be signaled to the process using the GDTR registry.

`GDT table` 是如下结构体段的组成：
The GDT table is composed of segments with the following structure:

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

我们应该在内存中定义我们的GDT，然后使用GDTR注册加载。

We need now to define our GDT in memory and finally load it using the GDTR registry.

我们将存储我们的GDT在如下地址:

```cpp
#define GDTBASE	0x00000800
```

函数 **init_gdt_desc** in [x86.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/x86.cc) 初始化一个GDT段描述符。

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
* http://wiki.osdev.org/Global_Descriptor_Table
* http://www.cnblogs.com/starlitnext/archive/2013/03/07/2948929.html
* http://www.codeproject.com/Articles/43179/Beginning-Operating-System-Development-Part-Three

下一章: [IDT(中断描述符表)和中断](../Chapter-7/README.md/) 
