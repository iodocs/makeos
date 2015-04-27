跟我一起来写操作系统
================================================================

这是一本关于用C/C++写一个操作系统的书籍。

注： 作者一些简单介绍,不作翻译。
**Caution**: This repository is a remake of my old course. It was written several years ago [as one of my first projects when I was in High School](https://github.com/SamyPesse/devos), I'm still refactoring some parts. The original course was in French and I'm not an English native. I'm going to continue and improve this course in my free-time.

**Book**: An online version is available at [http://samypesse.gitbooks.io/how-to-create-an-operating-system/](http://samypesse.gitbooks.io/how-to-create-an-operating-system/) (PDF, Mobi and ePub). It was been generated using [GitBook](https://www.gitbook.com/).

**Source Code**: All the system source code will be stored in the [src](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/tree/master/src) directory. Each step will contain links to the different related files.

**Contributions**: This course is open to contributions, feel free to signal errors with issues or directly correct the errors with pull-requests.

**Questions**: Feel free to ask any questions by adding issues. Please don't email me.

You can follow me on Twitter [@SamyPesse](https://twitter.com/SamyPesse) or support me on [Flattr](https://flattr.com/profile/samy.pesse) or [Gittip](https://www.gittip.com/SamyPesse/).

### 我们将构建一个什么类型的操作系统？
我们的目标是构建一个基于C++开发的类UNIX操作系统，但是我们不仅仅是做一个`POC(概念验证)`。这个操作系统应该能够启动，运行用户态SHELL并能够被扩展。

![Screen](./preview.png)
