---
title: simpleTemplate.js 的三版本分化和新功能编写
categories:
  - [技术, 前端]
  - [编程语言, javascript]
tags: [javascript, string-template]
permalink: simpletemplatejs-three-versions-and-new-features
id: 33
updated: '2015-01-03 22:29:19'
date: 2015-01-03 22:29:19
---

考研之后终于有时间将拖了很久的simpleTemplate.js给写完了。

## 三版本和新功能

写这篇文章的时间离同主题的[上一篇文章](http://blog.e10t.net/optimize-simpletemplatejs-by-myself/)已经太远了。距离上次大概实现了基本功能之后，又发生了很多事情，也对这个js库思考了不少，最后决定将这个库拆成三个版本，分别是bare版、regular版和advanced版。三个版本的功能是逐渐增加的，而各版本又在各自功能的要求上尽量做到完善和小型化。

例如bare版只有数据输出的功能，被我用来在之前[某个chrome插件](http://blog.e10t.net/chrome-extension-test-page-and-report/)中使用。

回看以前自己写下的文字和代码，总有种陌生的熟悉感。然而新旧代码看似已经不同，实际要表达的思想却是一脉相承的，新的代码并不是重写，而是继承、改进和发展旧的代码。没有以前写下的代码就没有现在写出的代码，没有以前思考的基础就没有现在思考的凭依。因此这添加的新功能其实在旧功能写成之时，便拥有了出现的需求和生长的土壤。

然，新功能要出来，代码是会膨胀的，在某些用不上新功能的情况下，代码就成了冗余。故，分化出三个版本分别应付不同的情况。新功能就是出现三版本的原因。

## 新功能一：过滤器

此功能的想法是结合了对输出数据的二次处理和判断的拓展。

在regular版中，数据输出只有纯输出功能，明显是不够的；判断只能判断数据域是否存在数据，或者数据是否为真，明显也是不够的。（代码中体现为简单的`if (!data)`）。

那么数据域允许形如javascript语句的语法如何？判断使用形如`a > b`、`a == b`的语法如何？

当然是可以的，但是这样实在没必要，那还不如直接eval罢？

我思来想去，发现其本质其实都是对数据进行一连串的操作，每一个操作通过将上一个操作的输出作为输入而串起来，即使判断，例如大于判断，也可以理解成将上一个操作的结果作为左值传入进一个"大于"函数，输出了布尔值而已。

于是这样一来所有存在数据域的地方都可以统一配套上过滤器们就解决问题了。

ok，梳理一下。原来整个流程是这样的：

解析模板 -> 生成模板对象 -> 填充数据 -> 渲染

现在需要增加一个执行过程：

解析模板 -> 生成模板对象 -> 填充数据 -> 执行语句 -> 渲染

另外解析模板也要改进以适合对新语法的解析。

过滤器语法形如：`{=data | filter1 | filter2 1 | filter3 string "string | " }`

### 解析

语法变得太复杂，不考虑使用正则匹配了，尤其是需要分别识别引号内外的`|`和空格，太鬼畜了。

于是很感谢在大学的时候选修了编译原理，不然真不知道如何解决了。凭着半桶水的记忆和知识，写了一个简单的词法分析函数，把整串字符串扔进去就切出数据域和过滤器们以及参数了。具体代码有点长，也写得烂，就不贴出来了，请移步github查看，在`parseStatement`函数中。

但是仍然需要将需要解析的语句从最原始的模板文本中分离出来，这里就可以用正则了。离远一点看那个语法的形式，其实就是分成了有引号括住的字符和没有引号括住的字符，以及最开头有特定的符号。

于是正则就成这样：`/\{\s*([>|<|!|@|=|#])(?:"[^"]*"|[^\{\}"]+)*\s*\}/g`

### 执行

执行写起来比解析容易多了，因为已经有既定的数据结构了。需要在这里稍微记录一下的就是串联执行过滤器的过程。

以下是只为了展示思路的精简代码。

```javascript
/*
data structure：
[
    ['filter1'],
    ['filter2', 'arg21'],
    ['filter3', 'arg31', 'arg32']
]
*/
for ( i = 0; i < filterCount; i++ ) {
    result = [result].concat( filters[i].slice( 1 ) );
    result = func.apply( scope, result );
}
```

## 新功能二：错误提示

写这么一个函数：

```javascript
var error = function ( level, type, location, message ) {
    /*
     * @param level int, {0:statement, 1:field, 2:runtime}
     * @param type string
     * @param location {string, int}
     * @param message string
    */
    throw ( 'Error\n'
        + '[Level]\n'
        + ['statement', 'field', 'runtime'][level] + '\n\n'
        + '[Type]\n'
        + type + '\n\n'
        + '[Location]\n'
        + location + '\n\n'
        + '[Message]\n'
        + message
    );
}
```

凡是在生成模板，渲染模板的时候遇到什么问题，直接调用这个函数。

所以其实没什么好说的，只是约定好错误提示的一些格式就可以了。

End.