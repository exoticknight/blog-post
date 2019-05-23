---
title: 一段简单的js模板渲染代码
categories:
  - [技术, 前端]
  - [编程语言, javascript]
tags: [javascript, string-template]
permalink: a-piece-of-simple-javascript-for-template-render
id: 26
updated: '2014-11-06 22:17:16'
date: 2014-10-25 14:30:29
---

![thumbnail](https://i.imgur.com/aWBIWv4.jpg)
## 前言
在为一些页面写 javascript 的时候，发现经常出现如下的一种使用情况：通过AJAX请求数据，然后将数据填入 HTML 的模板中，最后将这段 HTML 插入或者替代到原网页中。

看到这样的使用情况，有所了解的人可能说直接上 jQuery 啦，用 `$.get` 之类的函数取到数据后再各种 DOM 操作，easy 啦～或者也有人说“嗯，这样的情况用 AngrularJS 或者 React 吧，国产的 AvalonJS 也不错哦”。嗯都说的没错，其实自己喜欢用哪个就用那个，顺手就好。

不过嘛，jQuery虽然厉害，但是在一些简单页面中带上一个压缩了也近 100k 的大库总感觉得不偿失（当然本身整个网站需要的话论外），而且实际用到的功能很少。

> 加上一些插件，jQuery 也有模板渲染功能啦。jQuery 本身也有 `tmpl()` 和 `template()`。

然后 AngularJS 之流嘛，好用是好用，只是有大材小用之感。

于是，<del>勇敢的少年快起床找**</del>勇敢的少年来写原生js吧！

## 真实用例

### 描述
在做某个博客的一个页面的时候，需要拉取 github event 来展示。然而因为博客本身搭建在github上，只依靠github提供的Jekyll引擎渲染，毫无后端可言，因此不能直接生成带有数据的页面。于是就只能捎上个 AJAX 库（墙裂推荐[jx](http://www.openjs.com/scripts/jx/)，谁用谁知道），在页面载入后再请求 github 的数据。

看了看 github api 返回的内容，一个 event 的基本结构如下：

```javascript
{
"id": "2338198221",
"type": "PushEvent",
"actor": {
  "id": 1171407,
  "login": "exoticknight",
  "gravatar_id": "",
  "url": "https://api.github.com/users/exoticknight",
  "avatar_url": "https://avatars.githubusercontent.com/u/1171407?"
},
"repo": {
  "id": 24838315,
  "name": "scau-sidc/scau-sidc.github.io",
  "url": "https://api.github.com/repos/scau-sidc/scau-sidc.github.io"
},
"payload": {
  "push_id": 472403458,
  "size": 1,
  "distinct_size": 1,
  "ref": "refs/heads/master",
  "head": "5a20542ff0ccd9b3b71dc393eeb45bb74b5f40dc",
  "before": "fb9144e0910813529f234cd00b24c4b0b21b67a2",
  "commits": [
    {
      "sha": "5a20542ff0ccd9b3b71dc393eeb45bb74b5f40dc",
      "author": {
        "email": "draco.knight0@gmail.com",
        "name": "exoticknight"
      },
      "message": "037",
      "distinct": true,
      "url": "https://api.github.com/repos/scau-sidc/scau-sidc.github.io/commits/5a20542ff0ccd9b3b71dc393eeb45bb74b5f40dc"
    }
  ]
},
```

数据存在于各种路径中……

然后是HTML模板：

```markup
<div class="board-card mb4 p-responsive" data-id="{cid}">
    <a alt="{uname}" title="{uname}" class="card-avatar" target="_blank" href="{uurl}"><img class="animated" src="{aurl}" alt="avatar" /></a>
    <span class="card-type">{ctype}</span>
    <small class="card-commits"><a target="_blank" href="{curl}">{chash}</a></small>
    <p>{csummary}</p>
</div>
```

其实 HTML 模板代码是怎样没所谓，留意其中用 `{` 和 `}` 括住的东西就是了。那是 field name，`{field}` 最终会被替换成数据。

### 当我只想先快速解决的时候……
直接一个replace了事……

```javascript
var fillTemplate = function( template, field, data ) {
    re = new RegExp( '\\{\\s*' + field + '\\s*\\}', 'g' );
    return template.replace( re, data || '' );
}
```

缺点也是简单易见的，因为只是替换了字符串再返回，所以如果替换了后的字符串（通常是数据）也包含 `{field}`，而下一次替换的 field name 刚好符合，那么最后生成出来的东西显然是错误的。

<del>但是写起来快( ～'ω')～</del>

### 当我有时间折腾的时候……
来写一个**容易让别人使用**的js库吧。

先来想想自己究竟想要怎么用这个库开始。

我设想我将像如下那样的使用这个库：

```javascript
// 生成一个template对象
var template = simpleTemplate( '<div class="board-card mb4 p-responsive" data-id="{cid}"><a alt="{uname}" title="{uname}" class="card-avatar" target="_blank" href="{uurl}"><img class="animated" src="{aurl}" alt="avatar" /></a><span class="card-type">{ctype}</span><small class="card-commits"><a target="_blank" href="{curl}">{chash}</a></small><p>{csummary}</p></div>' );

// 填充数据
template.fill({
    'cid': '1234567',
    'uname': 'exo',
    'uurl': 'http://test.com/test',
    'aurl': 'http://test.com/test'
}).fill({
    'csummary': '测试咯'
});

// 渲染
var html = template.render();
```

于是大概框架出来了。

```javascript
(function ( window, document, undefined ) {

var simpleTemplate = function( templateStr , prefix, suffix ) {

    var template = function( str ) {
    }

    template.prototype.fill = function( jsonObj ) {
        return this;
    }

    template.prototype.render = function() {
    }

    return new template( templateStr );
}

window.simpleTemplate = simpleTemplate;

})( window, document )
```

模板的处理思路，我思考了这么一个方法。

假设模板为 `<p>{field1}</p><p>{field1}</p><p>{filed2}</p>`。

先将模板字符串使用 field 作为分隔符，使用 `split()` 来切分。<del>（`split()` 支持正则真是太好了）</del>因为 split 函数在IE中有比较严重的问题，IE8 中还没有改过来，所以在新版中只能换一个思路了，详情见下面的[更新](#update-2014-11-06)。

则得到一个字符串数组 `text = ["<p>","</p><p>","</p><p>","</p>"]`，这个作为最后合成所需的数组之一。

```javascript
var splitRe = new RegExp( fieldPrefix + '\\s*\\w[\\w\\d]*\\s*' + fieldSuffix, 'g' )
var templateText = str.split( splitRe )
```

再对原模板字符串逐一匹配出 field，也组合成一个数组 `indexs = ["field1","field1","filed2"]`。这个数组的作用是标记 field 的位置。

在匹配中也将 field 编成一个字典，用来存储数据。这里就是`{'filed1':'','field2':''}`。

```javascript
while ( result = indexRe.exec( str ) ) {
    this.data[result[1]] = '';
    fieldIndexs.push( result[1] );
}
```

这样准备工作就完成了。

填充数据的工作就是简单的将数据存储进那个 field 作为 key 的字典。

```javascript
for ( var name in jsonObj ) {
    if ( this.data.hasOwnProperty( name ) ) {
        this.data[name] = jsonObj[name];
    }
}
```

最后渲染，简单易懂就直接上代码吧。

```javascript
for ( i = 0; i < text.length; i++ ) {
    temp.push( text[i] );
    temp.push( this.data[indexs[i]] );
}
```

接着 `temp.join('')` 就可以生成最终的内容了。

这个方法因为将模板分割成小段字符串，所以不会存在重复渲染的问题。

应用的实例可以到[这里](http://scau-sidc.github.io/trend/)查看。

代码还没有 push 上 github，所以以下贴完整代码，满足只看代码星人。

```javascript
(function ( window, document, undefined ) {
var simpleTemplate = function( templateStr , prefix, suffix ) {
    var fieldPrefix = prefix ? '\\' + prefix : '\\{',
        fieldSuffix = suffix ? '\\' + suffix : '\\}',
        splitRe = new RegExp( fieldPrefix + '\\s*\\w[\\w\\d]*\\s*' + fieldSuffix, 'g' ),
        indexRe = new RegExp( fieldPrefix + '(\\s*\\w[\\w\\d]*\\s*)' + fieldSuffix, 'g' );

    var template = function( str ) {
        var templateText = str.split( splitRe ),
            fieldIndexs = [];

        // initial data and index
        this.data = {};

        var result;
        while ( result = indexRe.exec( str ) ) {
            this.data[result[1]] = '';
            fieldIndexs.push( result[1] );
        }

        // getters
        this.getTemplateText = function() {
            return templateText;
        }

        this.getFieldIndexs = function() {
            return fieldIndexs;
        }
    }

    template.prototype.fill = function( jsonObj ) {
        for ( var name in jsonObj ) {
            if ( this.data.hasOwnProperty( name ) ) {
                this.data[name] = jsonObj[name];
            }
        }

        return this;
    }

    template.prototype.resetData = function() {
        for ( var i in this.data ) {
            this.data[i] = '';
        }
    }

    template.prototype.render = function() {
        var temp = [],
            text = this.getTemplateText(),
            indexs = this.getFieldIndexs(),
            i;

        // merge text array & data array
        for ( i = 0; i < text.length; i++ ) {
            temp.push( text[i] );
            temp.push( this.data[indexs[i]] );
        }

        // clean
        this.resetData();

        return temp.join('');
    }

    return new template( templateStr );
}

window.simpleTemplate = simpleTemplate;

})( window, document )
```

<a name="update-2014-11-06"></a>
## 2014.11.06更新

在IE678中，split 函数如果使用正则匹配作为参数，那么结果中的空字符会被“吞”掉。

看代码

```javascript
','.split(/,/g).length === 2;  // true in non-IE

','.split(/,/g).length === 0;  // true in IE678
```

那么很明显如果我使用 split 函数的话，如果模板中出现 `{field}{field}` 的话，`}{` 中的空字符就会没了，模板文本和数据域的位置会打乱。

于是就只能避免使用 split 函数，另辟蹊径。在解决这个问题的同时，我也在加上新的功能，简要来说就是支持列表循环和标志位了。最新的一个版本可以上[github项目](https://github.com/exoticknight/jswarehouse/tree/master/simpleTemplate)查看，算是本文中 js 的进化版。改进的思路我之后再写一篇博文来记录吧。