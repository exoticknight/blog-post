python × Qt应用开发 · 1 -- 基本界面
==================================
14 Mar 31 13:39

##至少出来一个窗口
既然是GUI，那么最起码能运行并显示一个窗口出来。

首先在python的工程里面建好工程结构。

![工程结构](http://i.imgur.com/97PENH0.jpg)

结构并非必要，只是个人习惯这样建而已。下面来解说一下。

* `main.py`，整个应用的入口
* `app`包，用来放应用的文件
* `app.view`包，用来放应用ui相关的文件

打开QtDesigner，会有一个自动弹出框，直接选择其中的“Main Window”，然后点create，一个窗口就出来啦。QtDesigner的主界面暂时没什么好说的，有经历过GUI设计的读者估计也很熟悉。

![新建ui](http://i.imgur.com/LpPm5Sz.png)

现在我们先保存，文件名为`mainwindow.ui`，保存到`app.view`下。

接下来是比较重要的一步，也是之后经常用到的步骤：将.ui文件编译成.py文件。

执行以下命令。
```bash
pyside-uic mainwindow.ui -o mainwindow.py
```

非常好理解，使用`pyside-uic`将`mainwindow.ui`编译，输出为`mianwindow.py`。

以后每一次更改了.ui文件，都要这样执行一下取得.py文件。我自己为了方便，写了一个批处理文件，要编译.ui文件的时候就可以直接拖到这个批处理文件上自动编译了。代码如下，保存为`ui2py.bat`。
```batchfile
pyside-uic %1 -o %~n1.py
```

好的，现在我们已经拥有这个窗口的类的基本代码了。好奇的你可能想知道生成了什么。

![基本界面代码](http://i.imgur.com/hbBWmnm.jpg)

哇，一堆代码。都是编译生成的，无需做修改，只是要留意类名`Ui_MainWindow`，之后要使用这个类。

OK下面来调用这个`Ui_MainWindow`类。在app包下新建`GUI.py`，写入以下代码。

```python
from PySide import QtCore, QtGui
from view.mainwindow import Ui_MainWindow

class MainWindow(QtGui.QMainWindow):
    def __init__(self, parent=None):
        super(MainWindow, self).__init__(parent)
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)
```

这个就是最基本的对窗口类的使用。从代码上可以看出，自定义的`MainWindow`类继承了`QtGui.QMainWindow`并且初始化，接着调用`Ui_MainWindow`来生成ui。一个窗口其实就创建好了。这里超前说一下，以后的功能实现代码基本都写在这个类里面了，所以这个类并不是ui类，而是类似于MVC中的Controller，ui类是`Ui_MainWindow`。

窗口创建之后还要显示出来，注意应用程序跟窗口是两个概念。现在，在`main.py`中写入如下代码。

```python
import sys
from PySide import QtCore, QtGui

from app import GUI

if __name__ == "__main__":
    app = QtGui.QApplication(sys.argv)
    window = GUI.MainWindow()
    window.show()
    sys.exit(app.exec_())
```

写过python代码的读者应该很熟悉这个判断的代码了，就是入口嘛。代码生成了一个变量名为`app`的`QApplication`实例，生成了`GUI.MainWindow`实例（自己写的），然后最后一句是运行app实例，等待其返回的状态码来关闭程序。实际上在程序运行的时候，代码是会卡在`app.exec_()`这里的，一旦执行了什么关闭程序的操作之后才会执行`sys.exit()`，所以这里并不是显示窗口之后立刻结束。

啊，终于可以运行了。

![基本界面运行图](http://i.imgur.com/wpfSWcF.jpg)

啥都没有，正常，我们还没有加入控件呢。

##加入一些控件
回到QtDesigner，从左边的`Widget Box`里面找到到`Label`、`Line Edit`、`Push Button`这三个widget，点击拖动到中间的窗口设计上。

![放上了widget的设计](http://i.imgur.com/qUthBdW.jpg)

从右上角的`Object Inspector`中可以看到刚刚添加的widget，每一个都有自己唯一的标识，现在来把这些标识改成符合自己风格或者标准的新标识。双击标识或者点选widget之后在右下角的`Property Editor`里面的`objectName`进行修改。

![Object Inspector修改](http://i.imgur.com/XDrxpTZ.jpg)
![Property Editor修改](http://i.imgur.com/uO1DbF1.jpg)

进行以下修改：

* label -> labelTest
* lineEdit -> lineEditTest
* pushButton -> pushButtonTest

可以看出是偏向“类型+自定义标识”的命名，这样修改的好处是使用的IDE有自动补全功能的话，只要输入例如`pushBu`，IDE就会自动列出所有的`pushButton`widget供选择。至于自定义标识单词首字母大写，则是个人习惯而已。

保存&编译一下，回IDE运行查看。（以后不会再提醒保存&编译了）

![带widget运行](http://i.imgur.com/NZ9ptnS.jpg)

现在除了输入框可以输入、按钮可以按之外，没什么可以做的事，因为我们还没有指定这些操作会触发哪些事件。接下来就是比较难的部分：使用Qt的信号&槽机制。

##Signals & Slots(信号&槽)
接下来的步骤可能有点难以理解，我尽量解释。先贴一段从IBM上找到的文字。[原文](https://www.ibm.com/developerworks/cn/linux/guitoolkit/qt/signal-slot/)

> 所有从 QObject 或其子类 ( 例如 Qwidget) 派生的类都能够包含信号和槽。当对象改变其状态时，信号就由该对象发射 (emit) 出去，这就是对象所要做的全部事情，它不知道另一端是谁在接收这个信号。这就是真正的信息封装，它确保对象被当作一个真正的软件组件来使用。槽用于接收信号，但它们是普通的对象成员函数。一个槽并不知道是否有任何信号与自己相连接。而且，对象并不了解具体的通信机制。

简单来说，只要对象继承于QObject或其子类，就可以发射（emit）信号和使用槽来接收信号。对象可以通过emit signal（发射信号）来告诉外界自己正在做什么或者某种状态，而能够接收到信号的slot（槽）则会被信号激活而调用（因为其本身是一个对象成员函数）。那么顺理成章地，widget（显然是从QObject继承下来的）可以通过emit signal来报告状态，然后我们只要用slot来接收就行了。

具体一点，在我们在上面写的简单界面中，输入框的内容改变了，应该能够发射信号告诉外界这件事，而我们只要编写一个slot来接收这个信号就可以做爱做的事了。*如果有GTK编程经历的读者可能发现跟GTK的信号和回调函数有点像。*

直接Google关键字`QLineEdit`（widget前都会加上一个Q），从官方文档中可以查出在输入框文字变化的时候会emit一个`textEdited`的signal。

在`GUI.py`里简单实现一下。

```python
class MainWindow(QMainWindow):
    def __init__(self, parent=None):
        super(MainWindow, self).__init__(parent)
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)

        # initialize ui
        self.initUI()

    def initUI(self):
        self.ui.lineEditTest.textEdited[str].connect(self.updateLabelText)
        self.ui.pushButtonTest.clicked.connect(self.buttonClick)

    def buttonClick(self):
        QMessageBox.about(self, 'title', 'content')

    def updateLabelText(self, text):
        self.ui.labelTest.setText(text)
```

添加了一个`initUI(self)`作为自己对窗口初始化的函数和一个`buttonClick(self)`和`updateLabelText(self, text)`作为slot。

慢慢来解释。首先是两个slot，`buttonClick(self)`的作用是弹出一个消息框，标题是“title”，内容是“content”；`updateLabelText(self, text)`的作用是将接收到的文字更新到label上。

注意这里调用窗口中的某个叫xxx的widget是使用`self.ui.xxx`来调用的，简单易懂。

`initUI(self)`函数是专门用来自定义的ui的（注意只是自己写的并不是什么必要的函数）。函数内第一行和第二行的代码的意思是将widget上的一些信号“connect”（连接）到我们自己定义的slot。通过将signal和slot连接起来，emit signal的时候这些slot就会被触发，也就是相当于执行了。但是！第一行代码到`textEdited[str]`可能有读者就开始不明白了，为什么比下面的信号`clicked`多了`[str]`？在上面就已经说明过`textEdited`是代表文本变化的signal，于是signal里面携带上变化后的文本也是理所当然的事情，但是问题就在这里了：普遍来说，signal里面是可以包括不同数据/多个数据的（或者说，你可以理解成不同类型的参数/多个参数），在C++上有类似重载的机制来分辨，但是在python里面是没有提供重载（[为什么不提供](http://www.zhihu.com/question/20053359)，来自知乎)，如何分辨？我猜测PySide是使用了map来解决的，实际上在系列之后的文章中也会使用到一个叫`QSignalMapper`的类来实现从多个无参数的signal转换成有参数的signal。

那说了那么多，那这里的`[str]`究竟是怎么回事？我认为，这里可以理解为假定这个signal有可能携带一个数据，但这个数据可以有几种类型，用`[str]`就是选定了其中带文本类型的数据的signal来进行跟slot的连接。就像取数组里面的一个元素，不是吗:)

而`clicked`就容易理解了，就是点击了按钮呗，无需要传参，直接连接到`buttonClick`。

下面来运行一下，记得运行的文件是`main.py`。gif动态图展示。

![连接了槽的运行](http://i.imgur.com/wqsveLg.gif)

##小记
这篇博文实际涉及的东西不多，但是就是后面开发的基础，特别是signal&slot的概念，有不清楚的，尽量去问Google。本系列的重点也不在这些概念上，以后的博文就不再作解释了。

最后来总结一下重点：

* 工程结构
* QtDesigner的基础使用
* ui文件编译
* 应用运行的最小代码和widget（包括窗体）的调用
* Signals & Slots