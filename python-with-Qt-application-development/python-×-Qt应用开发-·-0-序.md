---
title: python × Qt 应用开发 · 0 -- 序
categories:
  - [技术, 软件]
  - [编程语言, python]
tags: [python, Qt, software]
permalink: python-with-qt-application-development-0-prologue
id: 10
updated: '2014-04-15 15:44:24'
date: 2014-03-13 23:47:44
---

## python？
python 是个很好用的语言，即使学校没有教，很多同学也会自己学。虽然比较少用 python 来做桌面程序，但是某些情况下需要用到 python 的数学或者统计功能的时候，又需要用有 GUI 的程序来交差。个人比较推荐用 C# 来做桌面应用程序（Windows 平台的话），只是 VS 这庞大的 IDE 和各种库还是会让某些有独特癖好的人侧目。这个时候用大家都比较喜欢的 python 就最好啦，搭配 Qt 来做 GUI 实在是方便。

于是就有了本系列。对比起其他教程 / 指南里只有单独的控件编写实例，在系列中我会使用 python 和 Qt 真实地编写出一个应用来，一边写应用一边写本系列的博文。途中遇到的问题也会记录下来并且尽量还原解决过程，希望能让读者有开发的真实感。实际上我在自主学习的时候在找资料和控件的测试使用上已经疲于奔命，也希望本系列能够总结出一个比较有效和固定的流程。

## Qt？
其实要在 python 实现 GUI 并不一定要使用 Qt，python 原生自带的 Tkinter 和下载一个 wxPython 库也可以。只是 Tkinter 嘛，做出来的界面实在难看，你看 python 自带的 IDLE 就知道了；wxPython 嘛，似乎没有什么比较成型的 GUI 设计工具。Qt 的话有一个 QtDesigner，比较好用。于是这里就是用 Qt 了。（实际上是以前写 C++ 的时候 GUI 用 Qt，不想转了 = =）

OK，现在选定了 Qt 之后还有一件事，就是用 PyQt 还是 PySide。为什么有两个呢？嗯，我没有仔细去研究过，Qt 本身的历史就比较复杂。这里我选用 PySide 来，没别的，因为之前的一个科创项目 [Micro XenServer Manager](https://github.com/exoticknight/Micro-XenServer-Manager) 中已经用了 PyQt，这里就尝试使用另外一个。在网上搜寻过相关资料，PySide 和 pyqt 的差别可以说在普通情况下影响不大，所以打算使用 pyqt 的读者也可以看本系列的博文来共同学习。

至于安装 python 和 PySide，不在这里详述，在 Windows 下 python 和 PySide 的安装就是两个 exe 文件的事情而已，Linux 下的话会用的人比我还专业。注意 QtDesigner 也要安装上，方法自己 Google 去。（最新版的 PySide 似乎会附带上）

系列博文使用的是 python2.7.3，PySide1.2.1。