## 章节八 物理内存和虚拟内存理论

在与 [GDT](../Chapter-7/README.md/) 相关的章节中，我们知道分段物理内存地址使用的是段选择和计算偏移([Linux在X86上的虚拟内存管理](http://home.cuit.edu.cn/Js/KNK/FLK/linux/linux-1.htm))

在本章中，我们将实现内存的分页功能，其原理是将分段的线性地址转换成物理地址（分页表存储了虚拟（线性）地址到物理地址间的映射）。

#### 为什么我们需要分页管理内存？

内存分页将允许我们的内核：

> 为避免歧义，保留部分原文

* use the hard-drive as a memory and not be limited by the machine ram memory limit
* to have a unique memory space for each process
* to allow and unallow memory space in a dynamic way

-------------------------
* 使用硬盘作为存储，且不受RAM内存的限制
* 每个进程使用独立的内存空间
* 控制是否以动态方式使用内存

> 注：每个进程都以为自己拥有全部的内存空间，如果为32位的操作系统，则进程认为自己拥有4GB的执行空间，但实际并没有这么多。



在一个分页系统中，每个进程都可能在一个独有的4GB的空间范围内执行（并不是指4GB的实际内存空间），且不会对其他用户进程或内存进程的内存空间产生影响。这样就简化了系统对多任务的处理。

![Processes memories](./processes.png)


#### 分页管理内存是怎样工作的？



将线性地址转化为物理地址的几个步骤：

> 为避免歧义，保留部分原文

1. The processor use the registry `CR3` to know the physical address of the pages directory.
2. The first 10 bits of the linear address represent an offset (between 0 and 1023), pointing to an entry in the pages directory. This entry contains the physical address of a pages table.
3. the next 10 bits of the linear address represent an offset, pointing to an entry in the pages table. This entry is pointing to a 4kb page.
4. The last 12 bits of the linear address represent an offset (between 0 and 4095), which indicates the position in the 4kb page.

----------------------------
1. 处理器从CR3寄存器中获取页目录的物理地址。（页目录中保存页面表）
2. 线性地址的前10位代表一个偏移（0-1023），它指向页目录中的一个索引项。这个索引项包含一个页面表的物理地址。（每个活动的进程都有一个独立的页面表）
3. 线性表接下来的10位偏移量，指向一个4kb大小的页（4kb即4k）。
4. 最后12位（0-4095），标记一个4kb页的位置。

> 注：[CR3](http://oss.org.cn/kernel-book/ch02/2.1.3.htm) 是页目录基址寄存器，保存页目录表的物理地址

![Address translation](./paging_memory.png)


#### 格式化页表和页目录


下图页目录和页表各字段看起来一样，但在我们的OS中只会使用到灰色区域的字段。

![Page directory entry](./page_directory_entry.png)

![Page table entry](./page_table_entry.png)

> 为避免歧义，保留部分原文

* `P`: indicate if the page or table is in physical memory
* `R/W`: indicate if the page or table is accessible in writting (equals 1)
* `U/S`: equals 1 to allow access to non-preferred tasks
* `A`: indicate if the page or table was accessed
* `D`: (only for pages table) indicate if the page was written
* `PS` (only for pages directory) indicate the size of pages:
    * 0 = 4kb
    * 1 = 4mb

-------------------------------
* `p`: 标记一个页或表是否在物理内存中
* `R/W`: 标记一个页或表是否正在写（以书面形式访问） (equals 1：=1则是)
* `U/S`: 标记为1,则允许访问非优选的任务
* `A`: 标记一个页或表是否被访问过
* `D`: (只针对页表) 标记也是否被写过
* `PS` (只针对页目录) 标记页大小:
    * 0 = 4kb
    * 1 = 4mb

----------------------------------
**注意:** 页目录与页表的地址，都要被写入20位（上图中12-31位），因为这些地址都是遵循4k对齐, 所以最后12位（0-12位）应该置0。

* 一个页目录或页表使用 1024*4 = 4096 bytes = 4k
* 一个页表可表示地址范围 1024 * 4k = 4 Mb
* 一个页目录可表示地址范围 1024 * (1024 * 4k) = 4 Gb

#### 怎样启动内存分页功能？

启动内存分页功能，我们只需要将 `CR0` 寄存器的第31位置1：

```asm
asm("  mov %%cr0, %%eax; \
       or %1, %%eax;     \
       mov %%eax, %%cr0" \
       :: "i"(0x80000000));
```

>注：[CR0](http://oss.org.cn/kernel-book/ch02/2.1.3.htm) 中包含了6个预定义标志，0位是保护允许位PE(Protedted Enable)，用于启动保护模式，如果PE位置1，则保护模式启动（分页模式），如果PE=0，则在实模式下运行。

但是之前，我们至少需要一个页表来初始化我们的页目录，也就是说页目录中至少要有一个页表项。

#### 恒等映射

在恒等映射模式下的分页，头4MB的虚拟内存对应头4MB的物理内存：

![Identity Mapping](identitymapping.png)

这种模式原理很简单：第一个虚拟内存页对应物理内存第一页，第二个虚拟内存页对应物理内存第二页，以此类推。


#### 扩展资料
  * [分页表](http://zh.wikipedia.org/wiki/%E5%88%86%E9%A0%81%E8%A1%A8)
  * [Linux在X86上的虚拟内存管理](http://home.cuit.edu.cn/Js/KNK/FLK/linux/linux-1.htm)
  * [cnblog上网友分享的内存管理笔记](http://www.cnblogs.com/felixfang/p/3420462.html)
  * [kernel-book中寄存器介绍](http://oss.org.cn/kernel-book/ch02/2.1.3.htm)
  * [MMU内存管理单元](http://zh.wikipedia.org/wiki/%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%8D%95%E5%85%83)
  * [SoC嵌入式软件架构设计之二：虚拟内存管理原理、MMU硬件设计及代码分块管理](http://www.cnblogs.com/yueqian-scut/p/4013858.html)
  * [每个程序员都应该了解的“虚拟内存”知识](http://www.oschina.net/translate/what-every-programmer-should-know-about-virtual-memory-part3?print)

下一章: [物理内存和虚拟内存管理](../Chapter-9/README.md/)
