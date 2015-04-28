## 章节七 IDT(中断描述符表)和中断 

中断（英语：Interrupt）是指处理器接收到来自硬件或软件的信号，提示发生了某个事件，应该被注意，这种情况就称为中断。

中断分为3种：

- **硬件中断:** 由外部设备发送给处理器，比如 键盘，鼠标，硬盘等。硬件中断是一种在轮询循环，等待外部事件方面避免浪费处理器的宝贵时间的方式。
- **软件中断:** 是一条CPU指令，用以自陷一个中断。由于软中断指令通常要运行一个切换CPU至内核态（Kernel Mode/Ring 0）的子例程，它常被用作实现系统调用（System call）。
- **异常:**  程序运行期间不能自身处理的错误和事件，比如页中断，被零除等。

#### 键盘示例
当用户按下键盘上的一个键时，键盘控制器将产生中断信号至中断控制器。如果中断不被屏蔽，控制器将中断发送给处理器，处理器将执行一个中断 服务子程序来处理中断（键按压或释放键），这个中断处理可能是获取按键并输出到屏幕。一旦字符处理程序完成时，被中断的作业就可以恢复。

#### 什么是PIC?

[PIC](http://en.wikipedia.org/wiki/Programmable_Interrupt_Controller) ([Programmable interrupt controller,可编程中断控制器 ](http://baike.baidu.com/view/2348508.htm)) is a device that is used to combine several sources of interrupt onto one or more CPU lines, while allowing priority levels to be assigned to its interrupt outputs. When the device has multiple interrupt outputs to assert, it asserts them in the order of their relative priority.

The best known PIC is the 8259A, each 8259A can handle 8 devices but most computers have two controllers: one master and one slave, this allows the computer to manage interrupts from 14 devices.

In this chapter, we will need to program this controller to initialize and mask interrupts.

#### What is the IDT?

> The Interrupt Descriptor Table (IDT) is a data structure used by the x86 architecture to implement an interrupt vector table. The IDT is used by the processor to determine the correct response to interrupts and exceptions.

Our kernel is going to use the IDT to define the different functions to be executed when an interrupt occurred.

Like the GDT, the IDT is loaded using the LIDTL assembly instruction. It expects the location of a IDT description structure:

```cpp
struct idtr {
	u16 limite;
	u32 base;
} __attribute__ ((packed));
```

The IDT table is composed of IDT segments with the following structure:

```cpp
struct idtdesc {
	u16 offset0_15;
	u16 select;
	u16 type;
	u16 offset16_31;
} __attribute__ ((packed));
```

**Caution:** the directive ```__attribute__ ((packed))``` signal to gcc that the structure should use as little memory as possible. Without this directive, gcc includes some bytes to optimize the memory alignment and the access during execution.

Now we need to define our IDT table and then load it using LIDTL. The IDT table can be stored wherever we want in memory, its address should just be signaled to the process using the IDTR registry.

Here is a table of common interrupts (Maskable hardware interrupt are called IRQ):


| IRQ   |         Description        |
|:-----:| -------------------------- |
| 0 | Programmable Interrupt Timer Interrupt |
| 1 | Keyboard Interrupt |
| 2 | Cascade (used internally by the two PICs. never raised) |
| 3 | COM2 (if enabled) |
| 4 | COM1 (if enabled) |
| 5 | LPT2 (if enabled) |
| 6 | Floppy Disk |
| 7 | LPT1 |
| 8 | CMOS real-time clock (if enabled) |
| 9 | Free for peripherals / legacy SCSI / NIC |
| 10 | Free for peripherals / SCSI / NIC |
| 11 | Free for peripherals / SCSI / NIC |
| 12 | PS2 Mouse |
| 13 | FPU / Coprocessor / Inter-processor |
| 14 | Primary ATA Hard Disk |
| 15 | Secondary ATA Hard Disk |

#### How to initialize the interrupts?

This is a simple method to define an IDT segment

```cpp
void init_idt_desc(u16 select, u32 offset, u16 type, struct idtdesc *desc)
{
	desc->offset0_15 = (offset & 0xffff);
	desc->select = select;
	desc->type = type;
	desc->offset16_31 = (offset & 0xffff0000) >> 16;
	return;
}
```

And we can now initialize the interupts:

```cpp
#define IDTBASE	0x00000000
#define IDTSIZE 0xFF
idtr kidtr;
```


```cpp
void init_idt(void)
{
	/* Init irq */
	int i;
	for (i = 0; i < IDTSIZE; i++)
		init_idt_desc(0x08, (u32)_asm_schedule, INTGATE, &kidt[i]); //

	/* Vectors  0 -> 31 are for exceptions */
	init_idt_desc(0x08, (u32) _asm_exc_GP, INTGATE, &kidt[13]);		/* #GP */
	init_idt_desc(0x08, (u32) _asm_exc_PF, INTGATE, &kidt[14]);     /* #PF */

	init_idt_desc(0x08, (u32) _asm_schedule, INTGATE, &kidt[32]);
	init_idt_desc(0x08, (u32) _asm_int_1, INTGATE, &kidt[33]);

	init_idt_desc(0x08, (u32) _asm_syscalls, TRAPGATE, &kidt[48]);
	init_idt_desc(0x08, (u32) _asm_syscalls, TRAPGATE, &kidt[128]); //48

	kidtr.limite = IDTSIZE * 8;
	kidtr.base = IDTBASE;


	/* Copy the IDT to the memory */
	memcpy((char *) kidtr.base, (char *) kidt, kidtr.limite);

	/* Load the IDTR registry */
	asm("lidtl (kidtr)");
}
```

After intialization of our IDT, we need to activate interrupts by configuring the PIC. The following function will configure the two PICs by writting in their internal registries using the output ports of the processor ```io.outb```. We configure the PICs using the ports:

* Master PIC: 0x20 and 0x21
* Slave PIC: 0xA0 and 0xA1

For a PIC, there are 2 types of registries:

* ICW (Initialization Command Word): reinit the controller
* OCW (Operation Control Word): configure the controller once initialized (used to mask/unmask the interrupts)

```cpp
void init_pic(void)
{
	/* Initialization of ICW1 */
	io.outb(0x20, 0x11);
	io.outb(0xA0, 0x11);

	/* Initialization of ICW2 */
	io.outb(0x21, 0x20);	/* start vector = 32 */
	io.outb(0xA1, 0x70);	/* start vector = 96 */

	/* Initialization of ICW3 */
	io.outb(0x21, 0x04);
	io.outb(0xA1, 0x02);

	/* Initialization of ICW4 */
	io.outb(0x21, 0x01);
	io.outb(0xA1, 0x01);

	/* mask interrupts */
	io.outb(0x21, 0x0);
	io.outb(0xA1, 0x0);
}
```

#### PIC ICW configurations details

The registries have to be configured in order.

**ICW1 (port 0x20 / port 0xA0)**
```
|0|0|0|1|x|0|x|x|
         |   | +--- with ICW4 (1) or without (0)
         |   +----- one controller (1), or cascade (0)
         +--------- triggering by level (level) (1) or by edge (edge) (0)
```

**ICW2 (port 0x21 / port 0xA1)**
```
|x|x|x|x|x|0|0|0|
 | | | | |
 +----------------- base address for interrupts vectors
```

**ICW2 (port 0x21 / port 0xA1)**

For the master:
```
|x|x|x|x|x|x|x|x|
 | | | | | | | |
 +------------------ slave controller connected to the port yes (1), or no (0)
```

For the slave:
```
|0|0|0|0|0|x|x|x|  pour l'esclave
           | | |
           +-------- Slave ID which is equal to the master port
```

**ICW4 (port 0x21 / port 0xA1)**

It is used to define in which mode the controller should work.

```
|0|0|0|x|x|x|x|1|
       | | | +------ mode "automatic end of interrupt" AEOI (1)
       | | +-------- mode buffered slave (0) or master (1)
       | +---------- mode buffered (1)
       +------------ mode "fully nested" (1)
```

#### Why do idt segments offset our ASM functions?

You should have noticed that when I'm initializing our IDT segments, I'm using offsets to segment the code in Assembly. The different functions are defined in [x86int.asm](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/x86int.asm) and are of the following scheme:

```asm
%macro	SAVE_REGS 0
	pushad
	push ds
	push es
	push fs
	push gs
	push ebx
	mov bx,0x10
	mov ds,bx
	pop ebx
%endmacro

%macro	RESTORE_REGS 0
	pop gs
	pop fs
	pop es
	pop ds
	popad
%endmacro

%macro	INTERRUPT 1
global _asm_int_%1
_asm_int_%1:
	SAVE_REGS
	push %1
	call isr_default_int
	pop eax	;;a enlever sinon
	mov al,0x20
	out 0x20,al
	RESTORE_REGS
	iret
%endmacro
```

These macros will be used to define the interrupt segment that will prevent corruption of the different registries, it will be very useful for multitasking.


译者注：
* [中断](http://zh.wikipedia.org/wiki/%E4%B8%AD%E6%96%B7)
