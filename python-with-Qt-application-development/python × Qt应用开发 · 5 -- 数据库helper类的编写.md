python × Qt应用开发 · 5 -- 数据库helper类的编写
======================================

在[上一篇博文](http://blog.e10t.net/python-with-qt-application-development-4-database-design/)中已经生成了数据库，而代码中的视图和模型也准备好了，但是怎么将两者联系起来呢？显然我们需要在其中做做文章，找一个“中间人”去读取数据库的数据并且转化为适合模型的数据。通常称这个“中间人”为数据库helper类。

##什么是helper类
顾名思义就是类似助手的一个类，数据库的helper类就是一个帮助程序员方便调用数据库的类。此类可以做的事情通常都是包括连接数据库，执行SQL，转换数据类型等。

##操作Python自带的sqlite3库

可以自己纯手写python代码来全程管理数据库，需要操心的地方有点多。

###databaseHelper

在`app`包下新建一个`store`包，新建一个`databaseHelper.py`文件。

为帮助代码的理解，极其推荐先去阅读[Introduction to SQLite in Python](http://www.pythoncentral.io/introduction-to-sqlite-in-python/)和[Advanced SQLite Usage in Python](http://www.pythoncentral.io/advanced-sqlite-usage-in-python/)，SQLite的基本操作和进阶应用都有详细而清晰的介绍，在这就没必要重新再说。

经过以上的阅读，数据库的操作基本能掌握了，然而我们的目标是写出一个比较通用的类。

```language-python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'draco'

import sqlite3

class SqliteHelper(object):
    """
    """
    @property
    def last_error(self):
        return self._last_error

    def __init__(self, db=":memory:"):
        super(SqliteHelper, self).__init__()

        self._last_error = None
        self._db = db
        self._con = None

    def config(self, db):
        self._db = db
        self._con = None

    def _connect(self):
        self.reset_error()
        self._con = sqlite3.connect(self._db, detect_types=sqlite3.PARSE_DECLTYPES|sqlite3.PARSE_COLNAMES, check_same_thread=True)
        # foreign key support
        self._con.execute("pragma foreign_keys = on")
        return self._con.cursor()

    def on_error(self, err):
        self._last_error = err

    def reset_error(self):
        self._last_error = None

    def execute(self, sql, param):
        lid = 0
        cur = self._connect()

        with self._con:
            try:
                if param:
                    cur.execute(sql, param)
                else:
                    cur.execute(sql)

                self._con.commit()
                if cur.lastrowid is not None:
                    lid = cur.lastrowid

                self.reset_error()

            except sqlite3.Error, e:
                if self._con:
                    self._con.rollback()
                self.on_error(e)

            finally:
                return lid

    def get_raw(self, sql, param=None):
        cur = self._connect()
        data = None

        with self._con:
            try:
                if param:
                    cur.execute(sql, param)
                else:
                    cur.execute(sql)

                data = cur.fetchone()[0]

                self.reset_error()

            except sqlite3.Error, e:
                if self._con:
                    self._con.rollback()
                self.on_error(e)

            finally:
                return data

    def query(self, sql, param=None):
        cur = self._connect()
        data = []

        with self._con:
            try:
                if param:
                    cur.execute(sql, param)
                else:
                    cur.execute(sql)

                row = cur.fetchone()

                # format data
                if row:
                    field_names = [f[0] for f in cur.description]
                    data = dict(zip(field_names, row))

                self.reset_error()

            except sqlite3.Error, e:
                if self._con:
                    self._con.rollback()
                self.on_error(e)

            finally:
                return data

    def query_all(self, sql, param=None):
        cur = self._connect()
        data = []

        with self._con:
            try:
                if param:
                    cur.execute(sql, param)
                else:
                    cur.execute(sql)

                rows = cur.fetchall()

                # format data
                if len(rows) > 0:
                    field_names = [f[0] for f in cur.description]
                    data = [dict(zip(field_names, r)) for r in rows]

                self.reset_error()

            except sqlite3.Error, e:
                if self._con:
                    self._con.rollback()
                self.on_error(e)

            finally:
                return data
```

`SqliteHelper`这个类是专门对应SQLite数据库的，如果要采用其他数据库就另外写对应的Helper类就行。

如果你有先阅读推荐的两篇文章，那么这段看似很长的代码其实一点都不难，无非就是`_connect()`连接数据库、`execute()`执行SQL语句、`query_all()`查询所有结果和`query()`查询一个结果这三个基本的功能。只是各种try和except比较多，因为数据库需要在操作失败的时候进行回滚。

`query_all()`和`query()`函数中把原始的数据查询出来之后，将数据打包成了一个数组，使用字典保存每一行的数据，继而作为一个数组元素存在。所以在访问结果（一个表）的时候，通过下标可以访问每一行，通过字段名字访问值。

比较需要注意的是`_connect()`函数的第一行`sqlite3.connect(self._db, detect_types=sqlite3.PARSE_DECLTYPES|sqlite3.PARSE_COLNAMES, check_same_thread=True)`。这个是让sqlite支持日期的设置。

另一个地方是连接数据库之后，必须手动执行一句SQL语句，使sqlite3支持外键。

```language-python
self._con.execute("pragma foreign_keys = on")
```

现在已经封装好了数据库的连接、查询和执行功能了，只需要一句语句就能查询/执行SQL语句了。

###\_\_init\_\_.py

接下来是为设计好的数据库编写特定的API了。

我希望直接使用store这个包来操作了，不再在包里面再另外弄文件。于是可以直接在包的`__init__.py`里面写一下静态的API。

```language-python
# in __init.py__
import time

import databaseHelper

_db = databaseHelper.SqliteHelper("data.db")


def error():
    return _db.last_error


class notebook(object):
    @staticmethod
    def new(name):
        return _db.execute("INSERT INTO notebook VALUES(null,?)", (name,))

    @staticmethod
    def delete(id):
        return _db.execute("DELETE FROM notebook WHERE id=?", (id,))

    @staticmethod
    def update(id, name):
        return _db.execute("UPDATE notebook SET name=? WHERE id=?", (name, id))

    @staticmethod
    def get_all():
        return _db.query_all("SELECT * FROM notebook ORDER BY id ASC")


class chapter(object):
    @staticmethod
    def new(name, notebook_id=1):
        return _db.execute("INSERT INTO chapter VALUES(null,?,?)", (name, notebook_id))

    @staticmethod
    def delete(id):
        return _db.execute("DELETE FROM chapter WHERE id=?", (id,))

    @staticmethod
    def update(id, name):
        return _db.execute("UPDATE chapter SET name=? WHERE id=?", (name, id))

    @staticmethod
    def get_all(notebook_id=None):
        if notebook_id:
            return _db.query_all("SELECT * FROM chapter ORDER BY id ASC WHERE nid=?", (notebook_id,))
        else:
            return _db.query_all("SELECT * FROM chapter ORDER BY id ASC")
```

这样的话，只要执行了`import store`，就可以用形似`store.notebook.new(...)`的API来操作文件夹，语义十分清晰。

在应用中需要得到的数据基本都能定下来，例如取所有的文件夹、取某文件夹下所有的文档等。为这些比较基础的写一下封装有利于避免在应用中写SQL语句，也能避免数据库有什么更改而连带造成应用中的代码也需要更改。

也就是，应用只需要知道调用什么API操作数据就行了，无需考虑应该数据怎么取。

随意插入几个数据，可以看到数据成功存入数据库了。

> 查看SQLite的数据库（一个db文件）可以使用‘SQLite Database Browser’。

![插入数据库1](https://i.imgur.com/bTr0dyQ.png)

##是否可用Pyside提供的QtSql

Pyside本身也提供丰富的数据库支持，如果源数据是比较平面，例如表格和列表，那么使用Pyside自带的QSqlTableModel、QSqlRelationalTableModel、QSqlQueryModel会比自己写更好，无需自己再造轮子。这些Pyside提供的“轮子”已经带有对数据库的操作，而且因为是官方写的，总比自己写出来的放心。

然而树状的阶级型数据，并没有原生model支持，过程都必定要涉及将数据库的表格型结构转化为树状，所以自己写也没差。

形象一点的话，数据的流向如下：

`treeModel <--> 自定义数据结构（数组） <--> 数据库`

这是自己写操作数据库类的情况。

`treeModel <--> QSqlTableModel/QSqlRelationalTableModel/QSqlQueryModel <--> 数据库`

这是用原生model的情况。

需要注意的是上面的流中，treeModel和数据库才是重要的，中间只是一个类似adapter的存在，那么明显直接用自定义的数据结构更方便。

> 类QSqlTableModel、QSqlRelationalTableModel、QSqlQueryModel都在PySide.QtSql下。

##小记

注意真的开发软件如果像这系列的想到一个功能就开发一个，写一段代码就运行一下来测试是不行。尤其是团队开发的时候，不可能每次写一段代码就整个应用运行一次。应用如果很大，编译起来时间长是一个问题，有些必要模块甚至还没开发出来也是一个问题。真正的软件开发还需要架构设计、写文档、画层次图和使用测试驱动开发等流程。此系列的文章只是入门和熟悉python+QT的开发，读者还需要额外学习更多的开发知识。