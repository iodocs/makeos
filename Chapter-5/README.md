## 章节五 管理X86架构的基础类

我们已经知道怎样编译C++内核并通过GRUB启动该二进制文件，现在我们能够用C/C++做一些很酷的事了。

#### 输出文本到屏幕控制台

我们继续使用 `VGA` 默认模式`(03h)`来对用户显示一些文本。屏幕可以通过起始地址为0xB8000的Video Memory(显存)直接访问，屏幕分辨率是80*25，每个字符在屏幕上被定义为2个字节：一个是字符码，另一个是属性字节(描述了字符的表现形式，包括了字符颜色等属性)。这些意味着显存总大小为 4000B (80B * 25B * 2B)。

在`IO`类([io.cc](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/io.cc))中：
* **x,y**: 定义光标的位置 
* **real_screen**: 定义显存指针
* **putc(char c)**: 输出一个单独的字符到屏幕上并管理光标的位置。
* **printf(char* s, ...)**: 输出字符串

增加一个方法  **putc** 到 [IO Class](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/io.cc)，其作用是输出一个字符到屏幕并更新(x,y)光标的位置。

```cpp
/* put a byte on screen */
void Io::putc(char c){
	kattr = 0x07;
	unsigned char *video;
	video = (unsigned char *) (real_screen+ 2 * x + 160 * y);
	// newline
	if (c == '\n') {
		x = 0;
		y++;
	// back space
	} else if (c == '\b') {
		if (x) {
			*(video + 1) = 0x0;
			x--;
		}
	// horizontal tab
	} else if (c == '\t') {
		x = x + 8 - (x % 8);
	// carriage return
	} else if (c == '\r') {
		x = 0;
	} else {
		*video = c;
		*(video + 1) = kattr;

		x++;
		if (x > 79) {
			x = 0;
			y++;
		}
	}
	if (y > 24)
		scrollup(y - 24);
}
```

增加一个非常有用且被熟知的方法：[printf](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/kernel/arch/x86/io.cc#L155)

```cpp
/* put a string in screen */
void Io::print(const char *s, ...){
	va_list ap;

	char buf[16];
	int i, j, size, buflen, neg;

	unsigned char c;
	int ival;
	unsigned int uival;

	va_start(ap, s);

	while ((c = *s++)) {
		size = 0;
		neg = 0;

		if (c == 0)
			break;
		else if (c == '%') {
			c = *s++;
			if (c >= '0' && c <= '9') {
				size = c - '0';
				c = *s++;
			}

			if (c == 'd') {
				ival = va_arg(ap, int);
				if (ival < 0) {
					uival = 0 - ival;
					neg++;
				} else
					uival = ival;
				itoa(buf, uival, 10);

				buflen = strlen(buf);
				if (buflen < size)
					for (i = size, j = buflen; i >= 0;
					     i--, j--)
						buf[i] =
						    (j >=
						     0) ? buf[j] : '0';

				if (neg)
					print("-%s", buf);
				else
					print(buf);
			}
			 else if (c == 'u') {
				uival = va_arg(ap, int);
				itoa(buf, uival, 10);

				buflen = strlen(buf);
				if (buflen < size)
					for (i = size, j = buflen; i >= 0;
					     i--, j--)
						buf[i] =
						    (j >=
						     0) ? buf[j] : '0';

				print(buf);
			} else if (c == 'x' || c == 'X') {
				uival = va_arg(ap, int);
				itoa(buf, uival, 16);

				buflen = strlen(buf);
				if (buflen < size)
					for (i = size, j = buflen; i >= 0;
					     i--, j--)
						buf[i] =
						    (j >=
						     0) ? buf[j] : '0';

				print("0x%s", buf);
			} else if (c == 'p') {
				uival = va_arg(ap, int);
				itoa(buf, uival, 16);
				size = 8;

				buflen = strlen(buf);
				if (buflen < size)
					for (i = size, j = buflen; i >= 0;
					     i--, j--)
						buf[i] =
						    (j >=
						     0) ? buf[j] : '0';

				print("0x%s", buf);
			} else if (c == 's') {
				print((char *) va_arg(ap, int));
			}
		} else
			putc(c);
	}

	return;
}
```

#### 汇编接口
大量的指令在汇编里是可用的，但和用C的感觉是不同的，所以我们对这些指令封装成接口方便使用。

在C代码里，我们能够直接通过 "asm()" 指令调用汇编代码， gcc会使用 `gas`（GNU Assembler）编译汇编代码。

**注意:** gas 使用 AT&T 语法.

```cpp
/* output byte */
void Io::outb(u32 ad, u8 v){
	asmv("outb %%al, %%dx" :: "d" (ad), "a" (v));;
}
/* output word */
void Io::outw(u32 ad, u16 v){
	asmv("outw %%ax, %%dx" :: "d" (ad), "a" (v));
}
/* output word */
void Io::outl(u32 ad, u32 v){
	asmv("outl %%eax, %%dx" : : "d" (ad), "a" (v));
}
/* input byte */
u8 Io::inb(u32 ad){
	u8 _v;       \
	asmv("inb %%dx, %%al" : "=a" (_v) : "d" (ad)); \
	return _v;
}
/* input word */
u16	Io::inw(u32 ad){
	u16 _v;			\
	asmv("inw %%dx, %%ax" : "=a" (_v) : "d" (ad));	\
	return _v;
}
/* input word */
u32	Io::inl(u32 ad){
	u32 _v;			\
	asmv("inl %%dx, %%eax" : "=a" (_v) : "d" (ad));	\
	return _v;
}
```

译者注：
* 地址[0xb8000]是保护模式下显存的起始地址
* [VGA ：03h](http://netclass.csu.edu.cn/NCourse/hep094/homepage/Appendix/App34-11.htm)

下一章: [GDT全局描述表](../Chapter-6/README.md/) 
