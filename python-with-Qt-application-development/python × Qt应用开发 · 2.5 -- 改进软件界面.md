python × Qt应用开发 · 2.5 -- 改进软件界面
======================================

[上一篇文章](http://blog.e10t.net/python-with-qt-application-development-2-preliminary-design/)完成基本的界面，但是应用看起来并不算特别好看。

主要是因为软件的界面使用默认的样式和普通的窗口框架。接下来从这两个方面出发，尝试改进软件界面。

##自定义样式

Qt 给控件的样式化提供了一定的自由度，包括背景色、边框和特定控件的自定义，使用类似CSS的语法。这要求编写软件的人了解基础的CSS语法。各种控件和详细例子：[点这里](http://qt-project.org/doc/qt-4.8/stylesheet-examples.html)

要改变一个控件的样式，可以在属性编辑器中找到`styleSheet`属性，将CSS代码写进去。

这里以中间栏的输入框作为例子。

选择输入框，在属性编辑器中找到`styleSheet`，输入以下CSS代码。

```language-css
QLineEdit{
border-radius: 10px;
padding: 5px 10px;
margin: 0 10%;
background-color: white;
border: 1px solid #c9c9c9;
color: #1b1b1e;
}
QLineEdit:hover{
background-color: white;
border: 1px solid #969696;
}
```

结束编辑，立刻可以看到效果。

修改前：

![前](https://i.imgur.com/HyNEzg9.png)

修改后：

![后](https://i.imgur.com/YYoVsyW.png)

给widget加上背景颜色。

```language-css
QWidget{
background-color:#xxx;
}
```

![widget背景色](https://i.imgur.com/MxL75UV.png)

消除控件的自带边框。在属性编辑器中找到`frameShape`，设置值为`NoFrame`；找到`frameShadow`，设置值为`Plain`；找到`lineWidth`，设置值为`0`。

![消除边框](https://i.imgur.com/siT0NF2.png)

自定义按钮样式。

```language-css
QPushButton{
background-color: transparent;
border-style:outset;
border-width: 0px;
padding: 5px 0;
color: #fff;
}
```

![自定义按钮](https://i.imgur.com/S69GG8T.png)

其他细节都是类似的修改。

##使用无框架窗口

必须要在代码中实现。

回忆本系列第一篇文章，使用QtDesigner做出来的知识view而已，controller是需要在`MainWindow.py`中实现的。这里不再累述。

找到`initUI(self)`函数，加入如下代码：

```language-python
# no frame window
self.setWindowFlags(QtCore.Qt.FramelessWindowHint)
```

![无框架窗口](https://i.imgur.com/x3w4utd.png)

没有了系统默认的框架，好看的同时也失去了三个系统提供的便捷功能：功能按钮、窗口拉伸和窗口移动。

下面来解决。

###功能按钮
在最上面加上一个控件，充当窗口的标题栏和关闭按钮的容器。

在控件里面简单地拖进一个标签控件作为标题，一个按钮作为关闭按钮。控件设置为蓝色。

运行（非预览）之后发现背景色没有完全填充，如下图。

![背景色没有完全填充](https://i.imgur.com/YcvA5M8.png)

这个要在代码里面修复。

找到`initUI(self)`函数，加入如下代码：

```language-python
self.ui.widgetHead.setAttribute(QtCore.Qt.WA_StyledBackground, True)
self.ui.widgetLeft.setAttribute(QtCore.Qt.WA_StyledBackground, True)
```

具体来说就是在控件上调用`setAttribute(QtCore.Qt.WA_StyledBackground, True)`。

###窗口拉伸
需要使用到一个名为`QSizeGrip`的类，只是这个类不能在QtDesigner中直接拖入，需要在代码中手动添加。

找到右边栏widget的layout，更名为`verticalLayoutRight`。然后在`initUI(self)`函数中添加如下代码：

```language-python
self.ui.verticalLayoutRight.addWidget(QtGui.QSizeGrip(self), 0, QtCore.Qt.AlignBottom | QtCore.Qt.AlignRight)
```

###窗口移动
窗口移动的功能是设置在窗口的标题栏上的，自实现的标题栏也要实现此功能。

在`MainWindow`类下，直接贴入以下函数即可：

```language-python
def mousePressEvent(self, event):
    """ override mouse press event """
    self._postion = event.globalPos() - self.pos()

def mouseMoveEvent(self, event):
    """ override mouse move event """
    self.move(event.globalPos() - self._postion)

def closeEvent(self, event):
    """ override close event """
    event.accept()
```

原理很简单，自己处理鼠标点击和拖动事件。

最终形态。

![最终](https://i.imgur.com/omAswmv.png)