python × Qt应用开发 · 2.5 -- 补充&界面修改
==================================
14 Apr 04 00:23

这篇博文是作为2的补充。

##菜单
在2中已经大概地展示了基本的界面设计流程和操作，但是有一点遗漏了那就是菜单的设计。

这很简单，在新建一个窗体的时候，QtDesigner就已经为窗体添加上了`QMenuBar`，在窗体的标题栏下面可以看到一个经典的菜单栏，上面有`Type Here`字样。只要双击并填上你希望显示的菜单名字，QtDesigner会自动生成一个菜单，在下拉列表上继续双击`Type Here`将会自动生成`QAction`。`QAction`才是真正代表着菜单里的某个动作。

在下拉菜单里面，还能看到一个`Add Separator`，是添加一个分割线的意思。当生成了一个`QAction`之后，可以看到右边有一个类似加号的图标，是将当前`QAction`转化为`QMenu`的意思，换句话说可以生成子级菜单。子级菜单的操作跟上面描述的菜单操作一模一样。

![菜单设计](http://i.imgur.com/sWHRn5W.jpg)

另外给各个action对象修改好名字，以供日后调用。

![对象名称修改](http://i.imgur.com/4QP853v.jpg)

##调整组件的比例
在上一篇博文的最后，我用layout+属性设置来实现了窗口的拉伸不影响文件夹树的宽度而影响文档编辑组件的长度。之后觉得还是不够完美，因为窗口的可以缩放，但是文件夹树和文档编辑组件的比例却不能调整。于是另外一个用以排版的组件`QSplitter`就要出场了。

成为`QSplitter`的子widget的话，widget间就多了一条调整条，用来调整两边widget的大小比例。

先把原来的自动布局破环掉，找到自动布局的组件，参照下图取消。

![break layout](http://i.imgur.com/t5TLju4.jpg)

同时选中标签页widget和文档编辑widget，点击下图中红框圈住的工具栏上的splitter。

![splitter位置](http://i.imgur.com/HxMlmi8.jpg)

可以看到原来是`QLayout`的位置变成了`QSplitter`来接管。

![QSplitter](http://i.imgur.com/sS9Th84.jpg)

和上一篇博文同样的方法给`centralwidget`设置一个垂直布局。预览一下。

![预览](http://i.imgur.com/V3a4ETT.gif)

ok，两个问题。一、窗口的缩放又影响到了左边文件夹树的大小了。二、调整比例的时候文件夹树会突然变没了。

对于第二个问题，可以直接设置。找到`QSplitter`的属性，如下图。

![QSplitter property](http://i.imgur.com/FLQrnNo.jpg)

+ orientation，就是方向了，分水平还是垂直，跟layout是一样的。
+ opaqueResize，是否实时显示调整。
+ handleWidth，调整条的宽度。
+ childrenCollapsible，组件是否会折叠。就是这个了，取消掉组件就不会不见了。

而第一个问题的解决方法却不在`QSplitter`上，而在其子组件上。

几乎所有的widget，都有一个`sizePolicy`的属性，而在此属性中，除了使用过的`Horizontal Policy`和`Vertical Policy`，还有`Horizontal Stretch`和`Vertical Stretch`。后两个决定了缩放的时候，水平和垂直的缩放比例。

现在`QSplitter`下有两个组件，文件夹树和文档编辑组件。将各自的`Horizontal Stretch`属性设置为0、1，预览一下。啊哈！搞掂。

![预览2](http://i.imgur.com/H3XQBIM.gif)

其中的原理应该是这样的，在窗口缩放的时候，子组件们根据已经设定好的比例0:1，所有的因窗口缩放而引起的大小变化就被分配到文档编辑组件上了。而默认的配置是0:0，也就是变化被平均分配到两个组件上了。

##界面再设计
在修改界面的时候，思考了一下现在的布局，觉得笔记文档应该是需要独立显示的，而现在的设计则是显示在文件夹树中跟文件夹混在一起。如果某一个文件夹下文档比较多，那么文件夹树就会显得比较臃肿，要找出其中一篇文档就会非常困难。况且日后有可能加入搜索功能，将笔记独立显示绝对有利于区分搜索结果。

于是在文件夹树和文档编辑组件之间添加一个`List View`，标识为`listViewDocument`。同样，`Horizontal Stretch`设置为0。这个组件就是日后用来显示文件夹中笔记文档条目的组件。

![最终预览](http://i.imgur.com/8XbhyOL.gif)