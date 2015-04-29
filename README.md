跟我一起来写操作系统
================================================================

这是一本关于用C/C++写一个操作系统的书籍。

* 原作者：https://github.com/SamyPesse
* 原项目：https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System


译者注：下文是作者对该项目的简单介绍，包括写本书的起因，GitBook，源码，开放程度等。 

**Caution**: This repository is a remake of my old course. It was written several years ago [as one of my first projects when I was in High School](https://github.com/SamyPesse/devos), I'm still refactoring some parts. The original course was in French and I'm not an English native. I'm going to continue and improve this course in my free-time.

作者在高中时期创建了[第一个项目 devos](https://github.com/SamyPesse/devos), 本项目就是devos的重构版，作者承诺业余时间会持续改进本项目。

**Book**: An online version is available at [http://samypesse.gitbooks.io/how-to-create-an-operating-system/](http://samypesse.gitbooks.io/how-to-create-an-operating-system/) (PDF, Mobi and ePub). It was been generated using [GitBook](https://www.gitbook.com/).

`How-to-create-an-operating-system` 英文版已经发布在gitbook [https://www.gitbook.com/book/samypesse/how-to-create-an-operating-system/](https://www.gitbook.com/book/samypesse/how-to-create-an-operating-system/) 上  。

**Source Code**: All the system source code will be stored in the [src](https://github.com/SamyPesse/How-to-Make-a-Computer-Operating-System/tree/master/src) directory. Each step will contain links to the different related files.

源码在本项目的src目录内，书中涉及源码的地方会给予链接。

**Contributions**: This course is open to contributions, feel free to signal errors with issues or directly correct the errors with pull-requests.

本项目是完全开放性的，欢迎大家提 issues 和 pull-requests。

**Questions**: Feel free to ask any questions by adding issues. Please don't email me.

You can follow me on Twitter [@SamyPesse](https://twitter.com/SamyPesse) or support me on [Flattr](https://flattr.com/profile/samy.pesse) or [Gittip](https://www.gittip.com/SamyPesse/).

你能够 Twitter 作者 [@SamyPesse](https://twitter.com/SamyPesse) 或者在 [Flattr](https://flattr.com/profile/samy.pesse)，[Gittip](https://www.gittip.com/SamyPesse/) 去支持作者。

### 我们将构建一个怎样的操作系统？
我们的目标是构建一个基于C++开发的类UNIX操作系统，但是我们不仅仅是做一个`POC(概念验证)`。这个操作系统应该能够启动，运行用户态SHELL并能够被扩展。

![Screen](./preview.png)


### [本书目录](./SUMMARY.md)


### 译者注：
本书并不是一本正式的书籍，我们也没有把这个项目当作单纯的翻译项目，而是希望完全的中文化，适合当前国人的需求。

* 概念：原作者对一些概念会一带而过，在中文化时，我们会尽量提供更详尽的注解和额外的参考文章
* 工具：文中涉及的开发环境所需的工具套件，我们将重新整合和提供，方便国内用户使用
* 同步：原文中解决的问题和合并的PR，我们会保持跟踪并尽快同步

本中文文档中任何的问题 欢迎 issue 和 pull-requests， 也欢迎大家协同翻译。

分享一些相关的书籍和话题：
* [Bran的内核开发指南_中文版](http://www.cnblogs.com/liloke/archive/2011/12/21/2296004.html)
* [写一个操作系统内核有多难？](http://www.zhihu.com/question/22463820)
