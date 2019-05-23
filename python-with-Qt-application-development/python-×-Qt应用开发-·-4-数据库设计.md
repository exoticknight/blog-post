---
title: python × Qt 应用开发 · 4 -- 数据库设计
categories:
  - [技术, 软件]
  - [编程语言, python]
tags: [python, Qt, software]
permalink: python-with-qt-application-development-4-database-design
id: 16
updated: '2015-02-27 19:08:22'
date: 2014-03-29 08:32:08
---

在上一篇探讨 MVC 模式使用的博文中，已经成功实现了 `Tree View`widget 的数据显示。而实际上，其中的数据是我们自己硬编码的，显然不符合要求。节点中的 `_data` 域也只是简单的字符串，没有体现出 MVC 的优势。对于本应用中要求的比较有结构化的数据，很自然地考虑使用数据库来组织。

## 数据库的考虑
作为本地应用，考虑使用 python 自带的嵌入式数据库 `sqlite3`。SQLite 跟普通数据库管理如 MYSQL 等大同小异，SQL 语句也是相差不大，一般有学习过数据库的读者找找资料大概就能轻松理解，而没有使用过数据库或者不懂数据库的读者则应该先补充好数据库的知识再来阅读本文。

在 python，只需要加上 `import sqlite3` 语句就能使用 SQLite，方便至极。

## 笔记本
笔记本的数据结构是一棵棵树，叶子节点是章节。考虑使用一个名为 `notebook` 的表来保存笔记本数据，一个名为 `chapter` 的表来保存章节。

打开数据库设计工具，这里选用了一个在线的工具 [WWW SQL Designer](http://ondras.zarovi.cz/sql/demo/)。画出以下设计图。

![数据库设计图 1](https://i.imgur.com/9HDCvP7.png)

表 `notebook`

<table>
<thead>
<tr>
<th > 字段 </th>
<th > 内容 </th>
</tr>
</thead>
<tbody>
<tr>
<td>id</td>
<td > 主键 </td>
</tr>
<tr>
<td>name</td>
<td > 笔记本名称 </td>
</tr>
</tbody>
</table>

表 `chapter`

<table>
<thead>
<tr>
<th > 字段 </th>
<th > 内容 </th>
</tr>
</thead>
<tbody>
<tr>
<td>id</td>
<td > 主键 </td>
</tr>
<tr>
<td>name</td>
<td > 章节名称 </td>
</tr>
<tr>
<td>nid</td>
<td > 所属笔记本 id，外键 </td>
</tr>
</tbody>
</table>

表 `document`

<table>
<thead>
<tr>
<th > 字段 </th>
<th > 内容 </th>
</tr>
</thead>
<tbody>
<tr>
<td>id</td>
<td > 主键 </td>
</tr>
<tr>
<td>cid</td>
<td > 章节 id，外键 </td>
</tr>
<tr>
<td>title</td>
<td > 文档标题 </td>
</tr>
<tr>
<td>content</td>
<td > 文档内容 </td>
</tr>
<tr>
<td>last_update</td>
<td > 文档最后更新时间 </td>
</tr>
</tbody>
</table>

然后使用输出功能输出 sql 脚本文件，注意这里的代码在我的电脑上是能够运行的，但不一定能够在你的电脑上运行：

```language
CREATE TABLE 'chapter' (
'id' INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,
'name' TEXT NOT NULL  DEFAULT 'new group',
'nid' INTEGER NOT NULL  DEFAULT 0 REFERENCES 'notebook' ('id') REFERENCES 'notebook' ('id')
);

CREATE TABLE 'document' (
'id' INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,
'cid' INTEGER NOT NULL  DEFAULT 0 REFERENCES 'chapter' ('id') REFERENCES 'chapter' ('id'),
'title' TEXT NOT NULL  DEFAULT 'New document',
'content' TEXT DEFAULT NULL,
'last_update' NUMERIC NOT NULL
);

CREATE TABLE 'notebook' (
'id' INTEGER NOT NULL  DEFAULT NULL PRIMARY KEY AUTOINCREMENT,
'name' TEXT NOT NULL  DEFAULT 'new notebook'
);
```

## 生成数据库
保存 sql 脚本到工程根目录下的 `data.sql` 中，到 SQLite 官网下载 windows 版本的 CLI 文件 `sqlite3.exe`，同样放在工程根目录下。执行以下命令：

```bash
sqlite3.exe data.db < data.sql
```

如无报错则生成了一个 `data.db` 的数据库文件了。

## 小结
文章只说明了各个表和字段的关系，字段的详细属性请查看 sql 语句。