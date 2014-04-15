python × Qt应用开发 · 2 -- 界面初步设计
==================================
14 Apr 04 17:05

之前的一篇可以算是前置知识的快速介绍。从这篇开始就是正式地编写应用了。~~其实是因为之前我还没有想好要做什么应用。~~

为了兼顾举举例子和真实性，选了这么一个应用：xx笔记。基本的功能如下：

* 笔记支持富文本粘贴
* 按文件夹分类笔记
* 笔记可以加上标签

##画出大概的样子
先来一个大概的设计图。

![设计图](http://i.imgur.com/tjRrER8.jpg)

就是一个规规矩矩的顶栏功能按键，左栏选目录，右栏浏览的结构。这个只是现阶段的大概构思，最终做出来不一定是这样的，有可能在一些细节上会有所更改，但是整体界面几乎都可以定下来了。

打开`mainwindow.ui`，从`Widget Box`里拉出`Tab Widget`，这个就是标签页控件了。先来修改好属性。

第一个修改的是标签页的标识，修改成`tabFolder`和`tabTag`。接着是标签的显示名字，**注意这里是不能直接修改标签的显示名字的**。要这样修改，先点击某个标签页使其激活，然后在属性的`currentTabText`里面修改。

![修改标签页名字](http://i.imgur.com/3mjTd6W.jpg)

注意在QTabWidget所属的属性下有两个有趣的属性`documentMode`和`movable`，都勾上。设置了`documentMode`表示标签页不会渲染控件的边框，这很有用因为里面的树状列表控件会占据整个标签页，这样不会出现双重边框；`movable`是指标签可以自由移动排序。

![标签widget属性](http://i.imgur.com/fgKX8Ju.jpg)

继续，拉出widget`Tree View`（树状列表视图），直接拖到刚才的标签页控件上，你会看到标签页的内部会很明显变暗了，说明你即将`Tree View`交给标签页来管理（或者说作为其子控件），并修改标识为`treeViewFolder`。

![树状列表在标签页中]()

现在，在标签页的空白出点击右键，依次选`Lay out`->`Lay Out Vertically`，然后可以看到`Tree View`立刻就“听话”地调整好位置了。实际上这里我们给空间应用了一个“自动布局”，不急，在后面会有讲解，这里先这样用。一般来说，`Tree View`周围会出现一些边距。这里要这样修改：先在`Object Inspector`里点选相应的widget，然后在`Property Editor`里面找到`Layout`，将带有“Margin”字样的属性值全改为0就OK了。

![设置布局](http://i.imgur.com/qMv56yL.jpg)

![修改边框](http://i.imgur.com/nJICnLm.jpg)

现在做另外一个叫“标签”的标签页，只是里面放的widget不是`Tree View`而是`List View`，步骤都是一样的不再阐述。

接着是编辑的界面，现阶段暂时先使用`Text Edit`这个widget。这个widget最起码是具备有接收富文本粘贴的功能。

现在把两个widget（`Tab Widget`和`Text Edit`）对齐一下，按`Ctrl+R`快速预览一下。发现在widget都是固定的，调整窗口大小的时候widget没有自动调整。为了能让widget适应窗口大小的调整，于是就要用上面所说的自动布局了。

##布局管理（Layout Management）
自动布局widget可以在`Widget Box`里面看到，提供的有四个布局：

* Vertical Layout
* Horizontal Layout
* Grid Layout
* Form Layout

分别是垂直布局、水平布局、网格布局和表单布局。当然还有另外的自动布局，但是这四个基本能满足普通需要了。

那这些布局怎么用呢？简单来说，你只要将widget放在layout中，layout就能将这些widget按照一定的规则自动管理其大小和位置。举个例子，水平布局，顾名思义就是将widget水平放置。下面结合实践来加深理解。

拖出一个`Horizontal Layout`，分别将之前的`Tab Widget`和`Text Edit`widget放进去，这里最好先放`Text Edit`，免得被`Tab Widget`中的其他布局所影响。放的时候注意，如果布局的边缘出现像下图绿色框中的蓝色边框，才说明放的位置是正确的。

![正确位置出现蓝色条](http://i.imgur.com/lgfq411.jpg)

放好widget之后可以在属性里面调节layout中widget的外边距，注意是widget的外边距而不是layout的外边距。其他的还有`layoutSpacing`，widget之间的距离；`layoutStretch`，widget之间的缩放比例。

然而现在还不能达到期望的效果，为什么呢？我是故意这样做的，目的是为了展示这些布局的另外一个特点：可以嵌套。

现在其实你可以将用layout包围（或者说管理）的widget们看作一个组，但是这个组跟窗口的关联是没有的，也就是说窗口在调整大小的时候并不会引起这个组的变化。然而你是否注意到在右上角`Object Inspector`里面，整个窗口`MainWindow`的子对象是一个叫`centralwidget`的widget，这个widget是包含我们之前摆弄过的`Tab Widget`等widget的。而从文字前面的图标可以看出这个widget并没有应用到任何布局。很自然地想到，为这个widget设定了布局那么就能将窗口的大小关联到里面的widget/widget组上了。

![centralwidget属性](http://i.imgur.com/H828eJk.jpg)

为了能更清楚地看到layout可以嵌套的特性，可以拉出一个`progressBar`，然后在窗体的空白位置点击右键（这里已经暗示了你整个窗体的空白位置实际就是`centralwidget`），像之前选择布局那样选择，这次选择垂直布局。哦，widget们立刻就垂直排好了。

![最终布局图](http://i.imgur.com/qMv56yL.jpg)

再来看看`Object Inspector`，很明显地是一个垂直布局包含了一个水平布局和一个`progressBar`。对于上一级的垂直布局来说，水平布局跟一般的widget没什么分别。

![布局嵌套查看](http://i.imgur.com/3yzaULJ.jpg)

现在预览一下，调整窗口大小的时候，里面的widget也能够跟着放大和缩小了。

还有一个小问题，左边的标签页和右边的编辑区在放大缩小的时候好像都是同样大小的。很多时候调整大小，就是因为编辑区太小了，左边的标签页我们不希望会跟着放大。实现这个有几种方法，最简单的一种是修改标签页`sizePolicy`属性下的`Horizontal Policy`为Preferred，而`Text Edit`的同样属性则设置为Expanding，如下图。这样做，在水平方向的缩放上，同属一个layout中的widge，`Horizontal Policy`设置为Preferred的widget会将放大“让位”给设置为Expanding的widget。

![sizePolicy属性设置](http://i.imgur.com/5RHdNSs.jpg)

最终效果。

![最终效果](http://i.imgur.com/aqR1Vta.gif)

##小结
博文中的流程尽量描述得比较详细就是希望有一些需要的是经验而不是教程的操作能够尽早传达给读者。但是，这篇博文并不能覆盖到所有的界面设计，读者还是需要自己去做各种的实践。在之后的博文里面，我在开发的时候如果遇到比较困难的地方，会研究好之后再详细解释的。