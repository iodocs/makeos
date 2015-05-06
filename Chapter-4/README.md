## 章节四 OS核心和C++运行时

#### C++ 内核运行时

内核能够通过C++编程，除了一些陷阱（运行时支持，构造函数，...），你需要注意外，和用C编程基本类似。

编译器会假定所有必要的c++运行时支持在默认情况下是可用的，但是我们不能链接[libsupc++](#jump_libsupc++)到C++内核中，所以需要增加一些基本函数，这些函数在 [cxx.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/runtime/cxx.cc)。


**注意:** 操作符 `new` 和 `delete` 不能在虚拟内存和分页初始化之前使用。 

#### 基本的C++函数

内核代码不能使用标准库的函数，所以我们需要增加一些基本的内存管理和字符串操作函数。

```cpp
void 	itoa(char *buf, unsigned long int n, int base);

void *	memset(char *dst,char src, int n);
void *	memcpy(char *dst, char *src, int n);

int 	strlen(char *s);
int 	strcmp(const char *dst, char *src);
int 	strcpy(char *dst,const char *src);
void 	strcat(void *dest,const void *src);
char *	strncpy(char *destString, const char *sourceString,int maxLength);
int 	strncmp( const char* s1, const char* s2, int c );
```

这些函数定义在 [string.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/runtime/string.cc), [memory.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/runtime/memory.cc), [itoa.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/runtime/itoa.cc)

#### C 类型
接下来，我们要在在代码里使用一些不同的类型，大多数是无符号类型（有符号类型将最高位储存符号，而无符号类型全都储存数字）:

```cpp
typedef unsigned char 	u8;
typedef unsigned short 	u16;
typedef unsigned int 	u32;
typedef unsigned long long 	u64;

typedef signed char 	s8;
typedef signed short 	s16;
typedef signed int 		s32;
typedef signed long long	s64;
```

#### 编译内核

编译内核和编译linux执行文件不同，Linux执行文件的执行依赖于我们的Linux系统，而我们编译出来的内核是需要独立运行的，也就是说编译的内核不能依赖于编译它的Linux(或Windows)系统，所以我们不能使用这些系统提供的标准库。

该 [Makefile](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/Makefile) 
定义了编译和链接我们内核的过程。

对于 X86 体系， `gcc/g++/ld` 需要下面一些参数：

```
# Linker
LD=ld
LDFLAG= -melf_i386 -static  -L ./  -T ./arch/$(ARCH)/linker.ld

# C++ compiler
SC=g++
FLAG= $(INCDIR) -g -O2 -w -trigraphs -fno-builtin  -fno-exceptions -fno-stack-protector -O0 -m32  -fno-rtti -nostdlib -nodefaultlibs 

# Assembly compiler
ASM=nasm
ASMFLAG=-f elf -o
```


译者注：

* <span id="jump_libsupc++">`libsupc++`</span> 是g++的支持库，其包含了解决[RTTI(Run-Time Type Information，通过运行时类型信息)](http://baike.baidu.com/item/RTTI)和异常处理的函数。
* [trigraphs三元符](http://blog.csdn.net/todd911/article/details/8846615)
* [GCC参数指令介绍](http://read.pudn.com/downloads137/doc/comm/586953/GCC%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3.pdf)

下一章: [管理X86架构的基础类](../Chapter-5/README.md/) 
