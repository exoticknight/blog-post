---
title: python中有趣的zip函数
categories:
  - [技术, 杂谈]
  - [编程语言, python]
tags: [python]
permalink: python-interesting-function-zip
id: 21
updated: '2014-08-03 22:00:38'
date: 2014-08-03 21:50:14
---

懂python的都知道有一个内置函数zip，作用是将多个序列里面的对应组合起来。

直接上图。

![zip作用图示](http://i.imgur.com/lamRGpg.png)

图中a1、a2、result都是序列。zip在执行的时候将序列a1中的一个元素取出，再取出序列a2对应位置中的元素，组合成一个tuple，最后将这些tuple组合成一个数组。

所以，最后我们得到一个数组result=[(1,a),(2,b),(3,c),...]

所以这个zip的意思理解成拉链比较形象。

两个序列是两边的链，zip函数将它们拉上组合起来。

当然zip函数可接收多个序列。

然后很有趣的事情就来了，如果在这之后再执行`zip(*result)`，就可以取回序列a1和序列a2！

代码上看更直观。

```python
>>> a1 = [1, 2, 3]
>>> a2 = ['a', 'b', 'c']
>>> result = zip(a, b)
>>> result
[(1, 'a'), (2, 'b'), (3, 'c')]
>>> zip(*result)
[(1, 2, 3), ('a', 'b', 'c')]
```

注意到`result`前有一个`*`号，这其实是告诉zip函数传入的是一个序列，于是`result`从一个参数变成了多个参数了。

从`zip([(1, 'a'), (2, 'b'), (3, 'c')])`变成了`zip((1, 'a'), (2, 'b'), (3, 'c'))`了。

于是也就不难理解为什么能拆回来。

图示就是如下。

![拆开result](http://i.imgur.com/YWDddpl.png)

实际上起到了unzip的作用了～

python还真是可愛い～

![python拟人](http://next.rikunabi.com/tech/contents/ts_report/img/201312/002412/part3_img.jpg)