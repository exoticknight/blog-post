python × Qt应用开发 · 4 -- 数据库设计
=========================
14 Apr 23 23:22

在上一篇探讨MVC模式使用的博文中，已经成功实现了`Tree View`widget的数据显示。而实际上，其中的数据是我们自己硬编码的，显然不符合要求。节点中的`_data`域也只是简单的字符串，没有体现出MVC的优势。对于本应用中要求的比较有结构化的数据，很自然地考虑使用数据库来组织。

##数据库的考虑
作为本地应用，考虑使用python自带的嵌入式数据库`sqlite3`。SQLite跟普通数据库管理如MYSQL等大同小异，SQL语句也是相差不大，一般有学习过数据库的读者找找资料大概就能轻松理解，而没有使用过数据库或者不懂数据库的读者则应该先补充好数据库的知识再来阅读本文。

在python，只需要加上`import sqlite3`语句就能使用SQLite，方便至极。

##文件夹
文件夹的数据结构是一棵树，叶子节点是一篇篇的笔记文档。考虑使用一个名为`folder`的表来保存文件夹数据，一个名为`document`的表来保存笔记文档。

打开数据库设计工具，这里选用了一个在线的工具[WWW SQL Designer](http://ondras.zarovi.cz/sql/demo/)。画出以下设计图。

![数据库设计图1](http://i.imgur.com/FFHtR5P.jpg)

其中表`folder`中，字段`id`是文件夹的id，字段`fid`是父文件夹的id，字段`name`是文件夹名，字段`remark`则是备注。表`document`中，字段`fid`是父文件夹id，字段`title`是笔记文档的标题，字段`content`是笔记内容，字段`last_update`是最后更新时间，字段`tags`则是标签。而由于在所有表中的`id`字段都是主键，因此这样就能确定好树的结构了。

##标签
仔细思考了一下，标签和笔记文档是多对多的关系，使用关系型的数据库要完整地表达的确比较困难。暂时的解决方法如下：在表`document`中增加一个字段`tags`，其中的数据是以`,`分隔的标签id；表`tag`保存标签数据，表`t2d`保存标签到笔记文档的单向关系。

表`t2d`中，字段`tid`为标签id，字段`did`为文档id，同时加上index关系。

如此一来，在读取某篇笔记的时候，可以通过解释`tags`字段取出标签。而在使用标签搜索笔记的时候就使用表`t2d`，利用上数据库的查找优势。考虑到标签的作用多数是用来作为查找笔记的关键字，这样的解决方法应该是比较合理的。

加上标签后的设计图如下：

![数据库设计图2](http://i.imgur.com/qui7MU3.jpg)

然后使用输出功能输出sql脚本文件，注意这里的代码在我的电脑上是能够运行的，但不一定能够在你的电脑上运行：

```sql
CREATE TABLE folder (
id INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,
name TEXT NOT NULL  DEFAULT 'New Folder',
remark TEXT DEFAULT NULL,
fid INTEGER NOT NULL  DEFAULT 0
);

CREATE TABLE document (
id INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,
fid INTEGER NOT NULL  DEFAULT 0 REFERENCES folder (id),
title TEXT NOT NULL  DEFAULT 'New document',
content TEXT DEFAULT NULL,
last_update NUMERIC NOT NULL ,
tags TEXT DEFAULT NULL
);

CREATE TABLE t2d (
id INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,
tid INTEGER NOT NULL  REFERENCES tag (id),
did INTEGER NOT NULL  REFERENCES document (id)
);

CREATE TABLE tag (
id INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,
name TEXT NOT NULL  DEFAULT 'tag'
);

CREATE INDEX t2dindex ON t2d (tid, did);

```

##生成数据库
保存sql脚本到工程根目录下的`data.sql`中，到SQLite官网下载windows版本的CLI文件`sqlite3.exe`，同样放在工程根目录下。执行以下命令：

```bash
sqlite3.exe data.db < data.sql
```

如无报错则生成了一个`data.db`的数据库文件了。

##小结
文章只说明了各个表和字段的关系，字段的详细属性请查看sql语句。