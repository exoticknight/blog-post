---
title: python × Qt 应用开发 · 2 -- 界面初步设计
categories:
  - [技术, 软件]
  - [编程语言, python]
tags: [python, Qt, software]
permalink: python-with-qt-application-development-2-preliminary-design
id: 12
updated: '2015-02-27 17:59:33'
date: 2014-03-18 11:12:57
---

之前的一篇可以算是前置知识的快速介绍。从这篇开始就是正式地编写应用了。~~ 其实是因为之前我还没有想好要做什么应用。~~

为了兼顾举举例子和真实性，选了这么一个应用：PQ 笔记。基本的功能如下：

* 笔记支持富文本粘贴
* 按文件夹分类笔记

## 画出大概的样子
先来一个大概的设计图。

![设计图](https://i.imgur.com/uAJ6icH.png)

就是一个规规矩矩的三栏布局，左边是笔记本的目录树，中间是文档列表，右边是文档内容。这个只是现阶段的大概构思，最终做出来不一定是这样的，有可能在一些细节上会有所更改，但是整体界面几乎都可以定下来了。

打开 `ui_mainwindow.ui`，从 `Containers` 里拉出三个 `Widget`，分别命名为 `widgetLeft`、`widgetMiddle` 和 `widgetRight`。这就是左中右三栏的容器。

在左边栏中拖入一个 `pushButton` 和一个 `treeView`，分别对应设计图上的两个控件。注意，如果拖放位置正确（也就是 QtDesigner 知道你要将控件放进左边栏里面），你会看到左边栏 `widgetLeft` 会变暗了。

![拖放正确](https://i.imgur.com/Izn7dD6.png)

对着左边栏空白处点击右键，依次选择 ` 布局 ` -> ` 垂直布局 `。

![选择垂直布局](https://i.imgur.com/rvWMn63.png)

可以看到控件非常听话地从上到下排列好了。这里为 widget 中的控件快速设定了一个布局，相当于告诉 widget 中的控件该如何显示自己。

注意，这个 "布局" 并不是 widget 中的属性，而是独立的另一个类 `QLayout` 及其子类的实例。在对象查看器中点选 `widgetLeft` 后在下面的属性编辑器中可以看到有一栏 `Layout`，这个才是控件们服服帖帖的原因。只不过，当为一个 `widget` 选择了布局之后，QtDesigner 自动给这个 widget 增加了一个布局，然后将 widget 里面的子控件加入到布局中，于是子控件们都知道应该如何显示了。

![widget 中的 layout](https://i.imgur.com/PpYcKIZ.png)

同理，在中间控件 `widgetMiddle` 中放入一个 `LineEdit` 和一个 `ListView`，在右边控件 `widgetRight` 中放入 `TextEdit`，并且设置好布局。

现在使用快捷键 `Ctrl + r` 预览，发现拉伸窗口的时候，里面的三栏控件没有任何反应，这可不是想要的效果。

![预览 1](https://i.imgur.com/NTwV9wH.png)

注意整个窗口其实也是一个 widget，同样需要为其设置布局。

![窗口 layout](https://i.imgur.com/XB2vSJm.png)

再预览，出现一个新问题：三个栏不能各自调整大小。

![预览 2](https://i.imgur.com/XsXU5ed.png)

要实现这个功能需要另外一种布局管理，分裂器（QSplitter）。分裂器允许元素调整各自的大小。

先打破布局。

![打破布局](https://i.imgur.com/24ng0C0.png)

按着 ctrl 选择三个分栏 widget，注意是分栏 widget，再在其中一个 widget 的空白处点击右键，在布局中可以看到有 ` 使用分裂器水平布局 `。

![应用分裂器](https://i.imgur.com/jFmH6nh.png)

最后为窗口应用垂直布局就可以了。预览的时候当鼠标移动到分栏控件之间会发现可以调整大小了，同时调整窗口也能影响到三个分栏的大小。

> 从实际来说，调整窗口的大小的时候，更多是希望调整右边栏即文档显示栏的大小。

QSplitter 还能设置一些细节。

找到 `QSplitter` 的属性：

+ orientation，控件排列方向，水平还是垂直
+ opaqueResize，是否实时显示调整
+ handleWidth，调整条的宽度
+ childrenCollapsible，控件调整成过小时是否会隐藏

似乎没有什么可以用的。

然而，问题的解决方法却不在 `QSplitter` 上，而在其子组件上。

实际上，几乎所有的 widget，都有一个 `sizePolicy` 的属性，而在此属性中，有子属性 `Horizontal Stretch` 和 `Vertical Stretch`，对应中文 ` 水平伸展 ` 和 ` 垂直伸展 `，决定水平和垂直的缩放比例。

![sizePolicy](https://i.imgur.com/47JsPlH.png)

在属性编辑器中，可以看到 ` 水平伸展 ` 的值默认为 0，也就是左栏：中栏：右栏 = 0：0：0，现在将右栏的 ` 水平伸展 ` 值设为 1，也就是左栏：中栏：右栏 = 0：0：1。

预览一下，效果就出来了。

![预览 3](https://i.imgur.com/S21DeGJ.gif)

原理应该是这样的，在窗口缩放的时候，默认的配置是 0：0：0，表示变化被平均分配到两个组件上了。而修改后子组件们根据已经设定好的比例 0：0：1，所有的因窗口缩放而引起的大小变化 ** 全部 ** 被分配到文档编辑组件上了。

## 布局管理（Layout Management）
布局可以在 `Widget Box` 里面看到，提供的有四个布局：

* Vertical Layout
* Horizontal Layout
* Grid Layout
* Form Layout

分别是垂直布局、水平布局、网格布局和表单布局。当然还有另外的自动布局，但是这四个基本能满足普通需要。

垂直 / 水平布局不用解释了。网格布局是类似表格，一个控件占据一个单元格位置；表单布局是类似平常表单，从上到下排成多行，每行分两栏，左边放标签控件，右边放输入框控件。

## 菜单
在新建一个窗体的时候，QtDesigner 就已经为窗体添加上了 `QMenuBar`，在窗体的标题栏下面可以看到一个经典的菜单栏，上面有 ` 在这里输入 ` 字样。只要双击并填上你希望显示的菜单名字，QtDesigner 会自动生成一个菜单，在下拉列表上继续双击 ` 在这里输入 ` 将会自动生成 `QAction`。`QAction` 才是真正代表着菜单里的某个动作。

在下拉菜单里面，还能看到一个 ` 添加分隔符 `，是添加一个分割线的意思。当生成了一个 `QAction` 之后，可以看到右边有一个类似加号的图标，是将当前 `QAction` 转化为 `QMenu` 的意思，换句话说可以生成子级菜单。子级菜单的操作跟上面描述的菜单操作一模一样。

![菜单设计](https://i.imgur.com/9MbfQzg.png)

另外给各个 action 对象修改好名字，以供日后调用。

![对象名称修改](https://i.imgur.com/cRj9IL6.png)
