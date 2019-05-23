---
title: 自顾自地对 simpleTemplate.js 进行优化
categories:
  - [技术, 前端]
  - [编程语言, javascript]
tags: [javascript, string-template]
permalink: optimize-simpletemplatejs-by-myself
id: 30
updated: '2014-11-12 20:05:34'
date: 2014-11-12 20:05:08
---

[simpleTemplate.js](https://github.com/exoticknight/jswarehouse/tree/master/simpleTemplate) 的功能已经实现得差不多了，看能不能给它做一点优化。

## 测试框架

随手写一个记录运行前后时间的。

```markup
<script id="t_1" type="x-tmpl-simpleTemplate">
{@list}<p>{*}</p>{-list}
</script>
<script>
var template = simpleTemplate( document.getElementById( 't_1' ).innerHTML );

var startTime,
    endTime,
    text,
    loop,
    loopLength,
    times,
    totalCount;

totalCount = 0;

for ( times = 0; times < 10; times++ ) {
    startTime = new Date().getTime();

    // test
for ( loop = 0, loopLength = 10/*100/1000*/; loop < loopLength; loop++ ) {
    text = template.fill({
        // data
    }).render();
}

    // end test

    endTime = new Date().getTime();
    totalCount += ( endTime - startTime );
} // end tests for

console.log( totalCount / 10/*100/1000*/ );
</script>
```

## 缓存数组长度

说的是将以下代码：

```javascript
for ( var i = 0; i < arr.length; i++ )
```

改成：

```javascript
for ( var i = 0, length = arr.length; i < length; i++ )
```

从查阅网上的资料来看，这样改是有效的。[这里](http://jsperf.com/array-length-in-loop)

我用上面自己写的代码来测试。用一个随便生成的 1000 个元素的随机数数组来填充，然后渲染 100 次。以此作为一次测试，进行十次，再取平均数。

从测试来看，效果都不怎么样，没有多大提升的感觉。

## 字符串拼接

老生常谈了。在写代码的时候我觉得使用数组的 `push` 最后 `join` 会比较快，但是网上说现代浏览器下用 `+=` 更快，真是神奇了，通常感觉 `+=` 都会比较慢的。

于是我去试了试，结果让人惊讶。在 chrome/IE78910 下都有性能提高，尤其是 chrome。

> IE6789 的测试我是用 IE10 里面的开发人员工具切换模式来测试的，文档模式都是用 "标准"。

看图。1000 个数组数据，渲染 100 次，测试 10 次取平均。

![对比](https://i.imgur.com/VJTv9d0.png)

## IE 下的字符串

之前为了绕开 IE8 以下的 split 函数 bug，我已经将代码重写了一次，利用了 lastIndex 属性。没想到这个 lastIndex 属性在 IE 下还是跟其他浏览器不一样，测试了一下似乎是在处理换行上有点问题，简直神烦。

于是我就将模板中所有的回车换行统统搞掉算了。

```javascript
return new Template( templateStr.replace( /[\r\n]/gm, '' ) );
```