## 章节一 X86体系结构及操作系统介绍

### X86 体系结构是什么？

> `X86` 是一个intel通用计算机系列的标准编号缩写,也标识一套通用的计算机指令集合。 该指令集最早出现在 `Intel 8086` CPU上，后续的CPU兼容该指令集。

1981年，IBM的PC首先使用了 X86指令集 ，至此后，其已经成为最通用的指令集。 大量的软件，操作系统（比如 DOS，Windows，Linux，BSD，Solaris，Mac OS X）都支持X86体系的硬件。
在本教学中，我们将设计一个基于 `X86-32` 体系的操作系统，得益于向后兼容的特性，我们设计的操作系统将兼容新的PC(请谨慎在物理机上试验)。

### 关于我们的操作系统

我们的目标是构建一个基于C++开发的类UNIX操作系统，但是我们不仅仅是做一个`POC(概念验证)`。这个操作系统应该能够启动，运行用户态SHELL并能够被扩展。
该操作系统基于`X86`体系构建，运行在32位下，并兼容IBM的PC。

**要点:**
* C++编码
* X86-32体系
* 基于Grub启动
* 基本的模块化驱动
* 基本的UNIX风格
* 多任务
* 用户态ELF执行
* 模块
	* IDE 磁盘
	* DOS 分区
	* 时钟
	* EXT2 (只读)
	* Bochs VBE (图像显示接口)
* 用户态
	* Posix API
	* Libc
	* 可运行shell或一些执行文件（比如 lua）

## 参考资料
* [X86架构](http://baike.baidu.com/link?url=S6VXX4KQpo9U56AQe7BM8Ku-tKobnOQh47I_MfmwD3vf4ahK0XpeB9BCsyHSZLV1S7Ct1PfQGjO09agAYVkcaq)


下一章: [准备开发环境](../Chapter-2/README.md/) 
