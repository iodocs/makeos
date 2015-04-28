## 章节二 准备开发环境

首先准备一个可用的开发环境。通过使用 `Vagrant` 和 `Virtualbox` ，你可以在Linux，Windows，Mac编译和测试你编写的操作系统。

### 安装 Vagrant
> `Vagrant` 是一个免费并开源的软件，其能够创建和配置虚拟的开发环境。
>  你可以认为它是VirtualBox的封装,可以让VirtualBox用起来更便捷。

Vagrant 将帮助我们在你所使用的系统上创建一个干净的虚拟开发环境，现在开发下载并安装适合你当前系统的 Vagrant。

[Vagrant官网] http://www.vagrantup.com

### 安装 Virtualbox

> Oracle VM VirtualBox 是一款开源虚拟机软件， 其可以虚拟基于X86 和 AMD64/Inter64 的电脑。

Vagrant 需要依赖VirtualBox来工作， 从 [VirtualBox 官网]("https://www.virtualbox.org/wiki/Downloads" "https://www.virtualbox.org/wiki/Downloads") 下载和安装适合你当前系统的安装包。

[VirtualBox官网] https://www.virtualbox.org/wiki/Downloads

### 启动和尝试开发环境

当 Vagrant 和 Virtualbox 安装成功后，你需要下载适合Vagrant的ubuntu lucid32 镜像：
```
vagrant box add lucid32 http://files.vagrantup.com/lucid32.box
```

当镜像准备好后，我们需要定义 *Vagrantfile*  [创建文件 *Vagrantfile*](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/Vagrantfile)。 这个文件定义了开发环境依赖的软件：nasm，make，build-essential，grub，qemu。

启动Box (开发环境)：

```
vagrant up
```

通过SSH进入开发环境：

```
vagrant ssh
```
在Vargrantfile所在的目录放置的文件将自动映射到开发环境(Ubuntu Lucid32)的 */vagrant* 目录下：
```
cd /vagrant
```

#### 构建和测试我们的操作系统
该文件 [**Makefile**](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/Makefile) 定义了构建内核，用户libc，和用户态程序的的一些基本规则。


构建:

```
make all
```

通过qemu测试我们的操作系统:

```
make run
```

qemu的使用请参阅  [QEMU Emulator Documentation](http://wiki.qemu.org/download/qemu-doc.html) 

你可以通过 `Ctrl-a` 退出模拟器。

### 译者注：
-----
#### 自动动手准备操作系统和依赖软件

本文中作者实际是准备了一个在Virtualbox内运行的Ubuntu 10.04.4 LTS操作系统，然后安装了实验所需的软件 ```sudo apt-get install nasm make build-essential grub qemu zip```，所以熟悉Linux的同学完全可以自己准备这套环境。


#### 基于Vagrant使用本地Box
当然掌握Vagrant后，会让你的开发之路变得更为顺畅，鉴于国内网络情况，我们无法很顺畅的使用这些工具，很多在线安装都无法直接进行。

我们提供在国内网盘上提供了该 [lucid32 Box](http://pan.baidu.com/s/1c0cuNDI)，用户可以下载后在本地导入:
```
$ vagrant box add lucid32 ./lucid32.box
==> box: Adding box 'lucid32' (v0) for provider:
    box: Downloading: file://e:/Vagrant/lucid32.box
    box: Progress: 100% (Rate: 1065M/s, Estimated time remaining: --:--:--)
==> box: Successfully added box 'lucid32' (v0) for 'virtualbox'!

$ mkdir devos
$ cd devos
```
将 [Vagrantfile](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/blob/master/src/Vagrantfile) 拷贝到该目录下
```
$ vagrant up

$ vagrant ssh

```

#### 基于Vagrant使用完整版本地Box

我们还准备了一个安装了以上所述软件的 [makeos Box](http://pan.baidu.com/s/1nt7k7Ct)，同学可以按上面的方法导入本地的Box使用，启动时也可以免去长时间的网络安装软件包的时间。


参考：
* [使用 Vagrant 打造跨平台开发环境](http://segmentfault.com/a/1190000000264347) 
* [使用Vagrant在Windows下部署开发环境](http://blog.smdcn.net/article/1308.html)

下一章: [基于GRUB启动](../Chapter-3/README.md/) 