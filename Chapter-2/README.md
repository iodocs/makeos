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

译者注：

一些Vagrant中文参考资料：

http://segmentfault.com/a/1190000000264347

http://blog.smdcn.net/article/1308.html

下一章: [基于GRUB启动](../Chapter-3/README.md/) 