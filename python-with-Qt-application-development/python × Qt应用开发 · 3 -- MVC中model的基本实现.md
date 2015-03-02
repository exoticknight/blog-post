python × Qt应用开发 · 3 -- MVC中model的基本实现
==============================
14 Apr 1214:14

上一篇博文中，界面是设计好了，也能运行了，但是这个应用还只是个空壳子，widget里什么都没有。这篇博文就来充实其中一个的widget，左边的文件夹列表。

##MVC大法好
有过一些开发经验的读者肯定会听说过MVC，这里不详细解释MVC了。Qt中也有提供这样的模式，而且既有提供已经整合好的widget，也有提供单单view而自己需要编写相应的model和controller。后一种显然要比前一种麻烦，但是使用上可能后一种反而更多。为什么？因为在实际开发中，涉及到数据的显示很多时候都要附带上数据的操作，那些整合好的widget——比如`List Widget`和`Tree Widget`和`Table Widget`——能做到的基本只是将数据都转化为字符串来输出，一旦涉及到数据的修改的话……也不是不可以做到，但是编写出来的代码既繁琐又不通用。

从Qt的官方文档来看，这些widget的函数大多数都是操作view这个层面的，函数名字里面大多包含`item`，比如插入数据核心步骤是使用`QTableWidgetItem(data)`生成一个包含单个数据——data的item然后用`setItem (row, column, item)`将这个item设置在特定位置，这里的item其实相当于一个单元格了。但注意修改了这个item是并不能保证影响到原先的data的，而且这样单独地设置item的数据数据的操作，稍加思考便能明白，已经丢失了原来数据——也就是data——跟其他可能存在的data的关系了。更进一步，如果修改了这个item的数据，要怎么找回原来的data也是个大问题。

根据之前博文的要求，如果编写文件夹列表这个widget，我们考虑用`Tree View`然后自己编写model会比较好，因为文件夹本身就天然具备树的特征。另外文件夹还允许重名，我们不能只根据文件夹名来标识，而是需要另外的唯一标识，这就暗示了一个树节点所包含的数据是多个的了，最起码包含了节点的名称和节点的标识，单单`QTableWidgetItem`是比较难满足要求的。

##实现一个简单的model
上一篇博文全程在用QtDesigner，这次就转到python代码上了。在app包下新建model包，再在里面新建`treeModel.py`。

接下来……怎么写啊……

还是先来一些前置知识吧。

###Qt中model的基本概念

![基本概念图](http://qt-project.org/doc/qt-4.8/images/modelview-models.png)

上图来自Qt官方文档，简单易懂。可以看到，三个基本的model——List Model、Table Model和Tree Model——都有着类似的结构：有一个根`Root item`，有row（行）的概念，有列column（列）的概念。

相似的结构暗示了view和model应该是可以自由配对的，因为可以基于某些规定的接口来结合在一起，而且这些接口估计跟根节点`Root item`和行列的属性（长度是最显著的特征了）有关。`List Model`就是一个一维数组，以单下标确定位置；`Table Model`是一个而为数组，以双下标确定位置；`Tree Model`就是一棵树，以父节点和行数确定位置。或许你会觉得这跟普通的数据解结构没什么分别，但是注意了，途中的一个节点（就是一个正方格）是没有规定成什么数据类型，也就是不局限于是数字或者字符这样的基本类型，也可以是更复杂的对象甚至对象的数据结构形式。这些一个个的节点（正方格）只是在模型的层面上表现一致而已。

另外似乎除了`Table Model`以外并不能看出column的作用，实际上`List Model`和`Tree Model`也有column的概念，只不过基本上在模型层面上是只有一列。这个是什么意思呢？也就是说，对于`List Model`而言，最关心的特征是row也就是行数，但读取数据时非要加上column来定位的话应该在节点（正方格）内的数据上体现出数组来，对比`Table Model`则是在节点（正方格）上就体现出需要row和column同时定位。对于`Tree Model`而言也是同样。

文字太多不要觉得麻烦，当你先阅读了大概的描述，潜意识有了一些模糊的概念之后，立刻去写代码，理解起来比一边写一边看容易得多。

###上代码
我们来先用代码描述这些“节点”（正方格），以`Tree Model`为例。

```python
class GenericNode(object):
    def __init__(self, data, parent=None):
        self._data = data
        self._parent = parent
        self._children = []

        if parent is not None:
            parent.appendChild(self)
```

先写一个一般性的节点类`GenericNode`，一个节点包含了三个必要的域：父节点、子节点列表和数据。一个节点在被创造出来的时候就可以决定其父节点和数据，而父节点是可以为空的（根节点或者孤立节点），若指定的父节点不为空则需要要求父节点将新创造的节点加入到它自身的子节点列表中。注意此时appendNode函数我们还没有实现。

然后立刻就是appendNode函数。

```python
    def appendChild(self, child):
        self._children.append(child)
        child._parent = self
```

非常简单，将节点加入自身的子节点列表中。这里考虑到加入的子节点有可能并没有指定过父节点，所以追加了一句`child._parent = self`。

接下来是数据的访问。

```python
    def data(self):
        return self._data

    def setData(self, value):
        self._data = value
```

getter、setter，没什么值得解释。

再来是对查询节点关系的回应。

```python
    def parent(self):
        return self._parent

    def child(self, row):
        return self._children[row]

    def childCount(self):
        return len(self._children)

    def row(self):
        if self._parent:
            return self._parent._children.index(self)
```

都是非常简单的函数。`parent(self)`和`child(self, row)`分别是返回父节点和和指定的子节点（对应着上面的图可以明显看出应该通过数组方式访问子节点）。`childCount(self)`是返回子节点长度以便能遍历子节点，`row(self)`则是查询本节点在兄弟节点中的位置。

最后加入一个属性来跟其他类型的节点区别一下，因为其他节点将会继承自这个类。

```python
    @property
    def type(self):
        """ custom function """
        return "generic"
```

接着是特殊节点。

```python
class NotebookNode(GenericNode):
    @property
    def type(self):
        """ OVERRIDE """
        return "notebook"

class ChapterNode(GenericNode):
    @property
    def type(self):
        """ OVERRIDE """
        return "chapter"
```

继承的同时重写了`type`属性来区分不同的节点，暂时不去写更详细的函数。

节点部分就基本写好了，接下来是模型结构部分。基本框架的代码如下。

```python
class CatalogTreeModel(QtCore.QAbstractItemModel):
    def __init__(self, root=None, parent=None):
        super(CatalogTreeModel, self).__init__(parent)

        self._rootNode = GenericNode(None) if root is None else root

    def rowCount(self, parent):
        """ IMPLEMENT """
        if not parent.isValid():
            parentNode = self._rootNode
        else:
            parentNode = parent.internalPointer()

        return parentNode.childCount()

    def columnCount(self, parent):
        """ IMPLEMENT """
        return 1

    def index(self, row, column, parent):
        """ IMPLEMENT """
        if not parent.isValid():
            parentNode = self._rootNode
        else:
            parentNode = parent.internalPointer()

        childNode = parentNode.child(row)

        if childNode:
            return self.createIndex(row, column, childNode)
        else:
            return QtCore.QModelIndex()

    def flags(self, index):
        """ IMPLEMENT """
        if not index.isValid():
            return QtCore.Qt.NoItemFlags
        
        return QtCore.Qt.ItemIsSelectable | QtCore.Qt.ItemIsEnabled | QtCore.Qt.ItemIsEditable

    def headerData(self, section, orientation, role=None):
        """ IMPLEMENT """
        pass

    def parent(self, index):
        """ IMPLEMENT """
        if not index.isValid():
            return QtCore.QModelIndex()

        node = index.internalPointer()
        parentNode = node.parent()

        if parentNode == self._rootNode:
            return QtCore.QModelIndex()

        return self.createIndex(parentNode.row(), 0, parentNode)
        
    def data(self, index, role):
        """ IMPLEMENT """
        pass
```

Don't panic，待我慢慢解释。
根据官方文档，继承`QtCore.QAbstractItemModel`和init函数没什么好说的，注意到其他函数里面，都注释有`IMPLEMENT`字眼，已经明显地说明这些函数需要重写实现的了。

> rowCount(self, parent)

返回本层节点的个数。

> columnCount(self, parent)

返回节点中数据的个数，直接返回1是因为数据显然只有1个。

> index(self, row, column, parent)

这个函数实现view对model的访问，`parent`是一个从view传过来的`QModelIndex`对象，通过参数`row`和`column`来确定访问其某一个子节点，`isValid()`是其用以检测此对象是否有效的函数，取得了子节点之后需要将其包装成`QModelIndex`对象返回，也就是需要语句`self.createIndex(row, column, childNode)`的原因。由于包装的关系`parent`也需要使用`internalPointer()`得到真正的节点对象（在这里就是GenericNode或其子类）才能继续操作。

个人理解就这就相当于在view和model直接加入了一层数据访问实现层，官方文档称之为*index-based system*。使用的时候不用管那么多，记得中间需要这样转换就是了。当然你要研究的话可以去看源码，但是显然文章的重点并不在这里。

> flags(self, index)

概括来说，这个函数可以设定节点在view中的表现方式。从它需要返回的常量的名字可以看得出来，节点是有多种组合方式的，这里的代码表示了节点是“可选择的”（Selectable）、“可交互的”（Enabled）和“可编辑的”（Editable）。

[常量列表](http://qt-project.org/doc/qt-4.8/qt.html#ItemFlag-enum)：

* QtCore.Qt.NoItemFlags
* QtCore.Qt.ItemIsSelectable
* QtCore.Qt.ItemIsEditable
* QtCore.Qt.ItemIsDragEnabled
* QtCore.Qt.ItemIsDropEnabled
* QtCore.Qt.ItemIsUserCheckable
* QtCore.Qt.ItemIsEnabled
* QtCore.Qt.ItemIsTristate

多种属性的组合可以通过或运算`|`将其组合在一起，比如代码中的`QtCore.Qt.ItemIsSelectable | QtCore.Qt.ItemIsEnabled`。

> headerData(self, section, orientation, role)

是对于数据段的标题显示的设置，我们这里只有一个数据，又不需要显示标题，暂时不管。

> parent(self, index)

根据官网所说，这个函数直接用官网的代码保证不会在view查询节点的时候得到根节点就可以了，所以这里的代码原封不动使用官网提供的例子。

> data(self, index, role)

需要重点讲解的函数。顾名思义就是对数据的访问，参数`index`毫无疑问就是一个`QModelIndex`，而`role`是什么呢？`role`可以理解为“角色”，它的值表明了view对于数据的要求，比如`QtCore.Qt.DisplayRole`说明view要求model提供一个可以供显示的字符串，会作为view中节点的名字；再比如`QtCore.Qt.DecorationRole`说明view要求提供一个图标作为节点中的装饰，等。我认为这个是model的精华所在，通过这样不同的角色的区分，同一个节点可以为view提供不同的数据类型，分别用作操作和显示等。比起`Tree Widget`，使用这种方式无疑更具灵活性。更厉害的是，如果你觉得常量提供的角色不够，可以使用`QtCore.Qt.UserRole`和`QtCore.Qt.UserRole + 1`、`QtCore.Qt.UserRole + 2`这样来扩充。因为这些常量本质上只是数字而已。

一部分常量列表：

* QtCore.Qt.DisplayRole
* QtCore.Qt.DecorationRole
* QtCore.Qt.EditRole
* QtCore.Qt.ToolTipRole
* QtCore.Qt.StatusTipRole
* QtCore.Qt.WhatsThisRole
* QtCore.Qt.SizeHintRole
* ....

[更详细的列表](http://qt-project.org/doc/qt-4.8/qt.html#ItemDataRole-enum)

现在来实现这个函数。

```python
    def data(self, index, role):
        """ IMPLEMENT """
        if not index.isValid():
            return None

        node = index.internalPointer()

        if role == QtCore.Qt.DisplayRole:
            return node.data()
        elif role == QtCore.Qt.DecorationRole:
            pass
        elif role == QtCore.Qt.ToolTipRole:
            return node.type
```

要做的事情就是先判断一下`index`是否有效，然后就判断`role`的值，返回不同的数据。这里是名字显示数据data，而悬浮提示则是节点的类型。

至此model基本实现了，最后是使用。

返回`MainWindow.py`，在`MainWindow`类中加入：

```python
    def buildCatalog(self):
        root = treeModel.GenericNode("root")

        notebook1 = treeModel.NotebookNode("NotebookNode1", root)
        chapter2 = treeModel.ChapterNode("ChapterNode2", folder1)

        notebook3 = treeModel.NotebookNode("NotebookNode3", root)
        chapter4 = treeModel.ChapterNode("ChapterNode4", notebook3)

        self._folderModel = treeModel.CatalogTreeModel(root)

        self.ui.treeViewCatalog.setModel(self._folderModel)
        self.ui.treeViewCatalog.expandAll()
```

做的事情很简单，使用之前编写的代码建一棵树，结构是：

notebook1
┗chapter2<br>
notebook3
┗chapter4

把根节点交给model，使用`setModel`函数将model绑定到view上。为了好看把树全部展开。

最后在`__init__`函数中调用这个函数，运行。

![最终运行图](https://i.imgur.com/PICCd6F.png)

##小结
终于结束了本博文，使用MVC模式的代码也能工作了。回想文章开头的“MVC大法好”，这句话的可是有前提的，就是能理解好概念和驾驭到代码。
