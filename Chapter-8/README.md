## Chapter 8: Memory management: physical and virtual
## 章节八 内存管理：物理内存与虚拟内存
In the chapter related to the GDT, we saw that using segmentation a physical memory address is calculated using a segment selector and an offset.

在[GDT](https://github.com/iodocs/makeos/tree/master/Chapter-6)一章中，我们看到，采用分段物理内存地址使用的是段选择和计算偏移

In this chapter, we are going to implement paging, paging will translate a linear address from segmentation into a physical address.

在本章中，我们将继续实现分页，将分段的线性地址转换成物理地址。
#### Why do we need paging?

#### 为什么我们需要分页管理内存？

Paging will allow our kernel to:

分页将允许我们的内核：

* use the hard-drive as a memory and not be limited by the machine ram memory limit
* to have a unique memory space for each process
* to allow and unallow memory space in a dynamic way
-------------------------
* 使用硬盘作为存储，且不受RAM内存的限制
* 每个进程使用独立的内存空间
* 控制是否以动态方式使用内存

In a paged system, each process may execute in its own 4gb area of memory, without any chance of effecting any other process's memory, or the kernel's. It simplifies multitasking.

在一个分页系统中，每个进程都可能在一个独有的4GB的空间范围内执行，且不会对其他用户进程或内存进程的内存空间产生影响。这样就简化了系统对多任务的处理。

![Processes memories](./processes.png)

#### How does it work?

#### 分页管理内存是怎样工作的？

The translation of a linear address to a physical address is done in multiple steps:

将线性地址转化为物理地址的几个步骤：


1. The processor use the registry `CR3` to know the physical address of the pages directory.
2. The first 10 bits of the linear address represent an offset (between 0 and 1023), pointing to an entry in the pages directory. This entry contains the physical address of a pages table.
3. the next 10 bits of the linear address represent an offset, pointing to an entry in the pages table. This entry is pointing to a 4ko page.
4. The last 12 bits of the linear address represent an offset (between 0 and 4095), which indicates the position in the 4ko page.
----------------------------
1. 处理器从CR3寄存器中获取页目录的物理地址。（页目录中为页面表）
2. 线性地址的前10位代表一个偏移（0-1023），它指向页目录中的一个索引项。这个索引项包含一个页面表的物理地址。（每个活动的进程都有一个独立的页面表）
3. 线性表接下来的10位偏移量，指向一个4ko大小的页（4ko即4k）。
4. 最后12位（0-4095），标记一个4ko页的位置。

![Address translation](./paging_memory.png)

#### Format for pages table and directory

#### 格式化页表和页目录

The two types of entries (table and directory) look like the same. Only the field in gray will be used in our OS.

两类条目类型（表和目录）看起来一样，在我们的OS中只会使用到灰色区域的字段。

![Page directory entry](./page_directory_entry.png)

![Page table entry](./page_table_entry.png)

* `P`: indicate if the page or table is in physical memory
* `R/W`: indicate if the page or table is accessible in writting (equals 1)
* `U/S`: equals 1 to allow access to non-preferred tasks
* `A`: indicate if the page or table was accessed
* `D`: (only for pages table) indicate if the page was written
* `PS` (only for pages directory) indicate the size of pages:
    * 0 = 4ko
    * 1 = 4mo
-------------------------------
* `p`: 标记一个页或表是否在物理内存中
* `R/W`: 标记一个页或表是否正在写（以书面形式访问） (equals 1：=1则是)
* `U/S`: 标记为1,则允许访问非优选的任务
* `A`: 标记一个页或表是否被访问
* `D`: (只针对页表) 标记也是否被写
* `PS` (只针对页目录) 标记页大小:
    * 0 = 4ko
    * 1 = 4mo
    
**Note:** Physical addresses in the pages diretcory or pages table are written using 20 bits because these addresses are aligned on 4ko, so the last 12bits should be equal to 0.
----------------------------------
**注意:** 页目录与页表的地址，都要被写入20位，因为这些地址都是4k对齐, 所以最后12位应该置0。

* A pages directory or pages table used 1024*4 = 4096 bytes = 4k
* A pages table can address 1024 * 4k = 4 Mo
* A pages directory can address 1024 * (1024 * 4k) = 4 Go
---------------------------------
* 一个页目录或页表使用 1024*4 = 4096 bytes = 4k
* 一个页表可表示地址范围 1024 * 4k = 4 Mo
* 一个页目录可表示地址范围 1024 * (1024 * 4k) = 4 Go

#### How to enable pagination?

#### 怎样启动分页？

To enable pagination, we just need to set bit 31 of the `CR0`registry to 1:
------------------------
启动分页功能，我们只需要将`CR0`寄存器的第31位置之：

```asm
asm("  mov %%cr0, %%eax; \
       or %1, %%eax;     \
       mov %%eax, %%cr0" \
       :: "i"(0x80000000));
```

But before, we need to initialize our pages directory with at least one pages table.
---------------------
但是之前，我们至少需要一个页表来初始化我们的页目录。


**本人新手coder，不知翻译是否有所偏差，希望能持续对此项目贡献绵薄之力**
