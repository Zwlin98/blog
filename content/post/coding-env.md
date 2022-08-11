---
title: "我的编程环境"
date: 2019-12-21 12:00:00
---

### 2022-04-08 更新

之前的方案在我用了MacOS以及对于docker的更加熟悉之后，有待更新，之后再写一篇我现在的编程方案的文章。

---

以下为原文

## 引言
从大一接触Linux以来，就喜欢上了在Linux上敲击命令行的感觉，也坚持用linux当主力机了有个一年，但是也逐渐发现了日常使用Linux的问题，毕竟还是得用QQ，得用office交作业，虽然也很喜欢`latex`，但大部分学校作业交的还是`doc`，而不是`pdf`，这些虽然在Linux上也有解决办法，但这些解决办法都不能让人满意。我对现在的使用方法还是挺满意的，也是最新的解决办法。
<!--more-->
同时很多便利的软件并不支持Linux，这也使得我变得爱折腾，例如想下百度网盘的东西，哪怕本身就是超级会员，也难以在Linux上完成，这使我尝试Aira下载，提取直链下载，虽然最后能下成，但还是麻烦。其他还有很多诸如此类的问题，解决办法都得绕圈子。这使我渐渐放弃了单纯Linux机器的想法。

回顾一下我使用Linux的发行版
+ Ubuntu 用了最久的，真心习惯了，文档也丰富
+ deepin 颜值高和移植的国产软件不少，但出bug很难找到解决办法
+ Manjaro 颜值在线，Arch系列，用过一阵子
+ CentOS,虽然很多教程都是基于这个发行版，但是我真的喜欢不起来
+ 服务器常年用Ubuntu LTS

习惯shell的我，再用回Windows，可以说是相当难受，虽然Win10的Power Shell有很大的进步，但还是找不回Linux的感觉。所以我选择了虚拟机的方案。

## 虚拟机解决方案

接下来说说我尝试过的虚拟机解决方案，目前用的也是基于虚拟机的环境。包括这篇文章的编写，也是在虚拟机中完成的。有兴趣的也可以尝试一下

### Linux+Xserver

这个方法很简单，就是使用X-server软件，将Linux图形界面转发到windows上来，我之前用这个办法写Python，Pycharm在Linux上的界面是可以转发过来的，但这个方法也有不少问题，例如无法输入中文等。推荐的X-server软件是`Vcxsrv`同时，Pycharm有自带的远程调试功能，可以调用远程的解释器来执行代码，[相关文档在这里](https://www.jetbrains.com/help/pycharm/configuring-remote-interpreters-via-ssh.html)我现在也是一直用这个办法来写Python的。

### Linux+VSCode

在这里再次吹一波VSCODE，不亏是微软出品的编辑器，功能齐全，在非大型工程的表现极佳，我平时小型的代码，和配置文件的编辑，都会使用VSCODE。同时Markdown也是在VSCode中编写的。VScode新推出不久的[remote-development](https://code.visualstudio.com/docs/remote/remote-overview)，真的是相当完美一个功能。

Visual Studio Code远程开发允许使用容器，远程计算机或Linux的Windows子系统（WSL）作为功能齐全的开发环境。 您可以：
+ 在您部署到的同一操作系统上进行开发，或使用更大或更专业的硬件。
+ 将开发环境沙盒化，以避免影响本地计算机配置。
+ 使新贡献者易于上手，并使每个人都处于一致的环境中。
+ 使用本地操作系统上不可用的工具或运行时，或管理它们的多个版本。
+ 使用Windows Linux子系统开发Linux部署的应用程序。
+ **从多台机器或位置访问现有的开发环境**
+ 调试在其他位置（例如客户站点或云中）运行的应用程序。
> 我觉得对于有多台电脑的人来时，第五点相当有吸引力。而其实我这么折腾也就是是为了第2、3、5、6点。

[官方文档](https://code.visualstudio.com/docs/remote/ssh)写的相当好，我这里搬运也没有什么意思。但鉴于文档的`remote-ssh`是基于AWS的服务器来做示范，我这里提一下用虚拟机的不同点：
+ 本地虚拟机固定IP比较容易
+ 使用SSH密匙对登录的配置方法参考[这里](https://code.visualstudio.com/docs/remote/troubleshooting)
+ 虚拟机配置和其他云服务器没什么区别，我的腾讯云服务器也配置成功，阿里云也应该差不多，同时强烈建议大家使用密匙对登录方式(可以查看自己云服务商的文档)。
