# 物理内存与虚拟内存管理

归功于 [GRUB](../Chapter-3/README.md) ，在启动后，内核可以知道可用物理内存的大小。

在实现操作系统时，前 8Mb 字节的物理内存将被内核保留使用，这些内存被用来存放：
- The kernel 内核
- GDT, IDT et TSS
- Kernel Stack 内核栈
- Some space reserved to hardware (video memory, ...) 保留硬件所需空间
- Page directory and pages table for the kernel 内核分页目录和分页表

剩下的内存开放给内核和应用程序。

![Physical Memory](physicalmemory.png)


### 虚拟内存映射

内存起始地址到 `0x40000000` 是内核空间， `0x40000000` 到结束留给用户空间。

![Virtual Memory](virtualmemory.png)

虚拟内存中的 1Gb 内核空间，适用于所有任务（内核任务和用户任务）。

The kernel space in virtual memory, which is using 1Gb of virtual memory, is common to all tasks (kernel and user).

这是由指向任务页面目录的第256项到内核页面目录实现代码：

This is implemented by pointing the first 256 entries of the task page directory to the kernel page directory (In [vmm.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/vmm.cc#L204)):

```cpp
/* 
 * Kernel Space. v_addr < USER_OFFSET are addressed by the kernel pages table
 */
for (i=0; i<256; i++) 
    pdir[i] = pd0[i];
```

