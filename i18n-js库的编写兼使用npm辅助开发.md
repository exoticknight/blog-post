---
title: i18n.js库的编写兼使用npm辅助开发
categories:
  - [技术, 前端]
  - [编程语言, javascript]
tags: [javascript, i18n]
permalink: write-i18n-js-with-help-of-npm-as-build-tool
id: 40
updated: '2015-04-25 14:00:30'
date: 2015-04-23 19:06:26
---

近来由于开发某页面需要支持多语言切换，遂写一个切换页面语言的JS库。

## 编写库的既定事项

写JS库也不是一两次了，当然只是小型或者微型的。不过思想和方法和大型库都是通用的。一般是直接在sublime text里打开一个JS文件，然后写下一个Self-Executing Anonymous Functions（自执行匿名函数？），接着在函数里面创造库的对象，最后将对象挂在`window`对象下。

Show you the code的话就是以下所示：

```javascript
(function( window, undefined ){

var i18n = {};

i18n.bar = function () {
    return;
};

window.i18n = i18n;

})( window );
```

学得这样的写法是来源于对jQuery源代码的阅读。

通过将代码都包在一个匿名函数中，实现了一个闭包。如此一来在闭包内随便折腾，也不会污染到外部全局环境（当然是在编写可靠的代码的情况下）。

不过，随着AMD和CommonJS标准的流行开来，越来越多JS库都将自己模块化。过程也不复杂，只要遵循一定的规则就可以了。

而对于编写一个简单的JS库，将github上[UMD](https://github.com/umdjs/umd)项目给出的模板修改一下就OK。

修改后代码：

```javascript
;(function( root, name, definition ) {
    if ( typeof define === 'function' && define.amd ) {
      define( [], definition );
    } else if ( typeof module === 'object' && module.exports ) {
      module.exports = definition();
    } else {
      root[name] = definition();
    }
})( this, 'i18n', function() {

var i18n = {};

i18n.bar = function () {
    return;
};

// Return this library
return i18n;

});
```

注意最后不再需要手动将库挂载在`window`对象下，而是只是返回对象。挂载方式已经转交给外部函数判断。

## 思考多一点，代码少一点

很久的以前，我曾经写过一个jQuery插件，功能是为表格添加分页和异步载入。然而在写之前并没有清晰地定下整个插件的功能和限制，导致最后写出来的插件身兼数职，连表格美化与自定义CSS等也做了进去。加进去的功能有可能只是随手实现的，也许并不适合此插件管辖，造成了“做得不好非要做”的尴尬。

另外，功能的繁琐与代码段的反复抽象提取导致了代码的凌乱不堪，进而导致测试出bug的时候完全搞不清楚问题所在。

最后代码膨胀到完全不能控制，自己写出来的代码连自己都不敢修改。

> 在写jQuery插件的时候，十分容易变成了写“使用jQuery的代码集合”，缺少性能和架构的考虑。这跟jQuery本身十分强大和灵活的特性有很大的关系。

写库或插件，目的应该是将通用或者复杂的逻辑实现封装起来，通过提供简洁的API来实现功能的调用。

先将手从键盘上收回，拿出纸笔，好好列出对JS库的描述。

* Q1：i18n.js要做什么？
* A1：对页面上的文本进行语言切换。
* Q2：如何定位文本？
* A2：为DOM元素增加'data-i18n'属性进行标记。
* Q3：如何找到标注DOM元素？
* A3：从给出的DOM元素作为根进行深度优先/广度优先遍历。
* Q4：译文的来源？
* A4：用户遵循某一标准自定义每一套语言的字典。
* Q5：如何将译文和元素对应？
* A5：对每一条文本，以唯一ID标识。凡是'data-i18n'属性的值为此ID的元素，即使用此ID对应文本。

思路是不是清晰了很多呢？可以看到核心逻辑就是一个有访问函数的DFS或BFS算法。

## 工欲善其事，必先利其器？

近年的前端大发展，也催生了很多自动化工具。node的流行更是让很多软件管理和后端开发的思想能应用到前端开发上。

经典的前端开发不外乎就是写HTML、写CSS、写Javascript，然而在前端代码量越来越大的现在，一个自动化的构建工具则能大大提高工作效率。

如果Google一下前端构建工具，那么基本就是Grunt和Gulp。

本质上，Grunt和Gulp都是任务运行器，尝试将前端的代码生成甚至发布统合到几个甚至一个命令行中。它们本身作为npm的一个模块，并没有什么作用，真正做事的是以其为平台的大量插件。通过将各种各样的插件整合起来，Grunt和Gulp就能实现自动化的任务执行。

但是慢着，以前不是很流行什么网页三剑客的吗？甚至用DreamWrear就能做网页啊。任务运行器、插件什么的是个什么鬼？！

是这样的，现在的前端开发，虽然最终结果还是写HTML、写CSS、写Javascript，但是过程却已经变化多端，内容也逐渐丰富。

HTML的话：

切图输出其实也已经算一种自动化。然而现在还能使用jade、HAML或者各种模板引擎生成，也就是有可能不是直接手写HTML代码了。这个就需要依赖编译了。

CSS的话：

SASS、LESS和Stylus都已经存在了很久了，源代码产出CSS也是需要编译的。CSS文件也能够进行合并和版本控制，如此一来又需要额外的工具。

Javascript的话：

本身就是一个编程语言，有工具能对其语法进行排错，不能不用吧？流行又高效的模块化开发，需要工具合并吧？压缩源代码，又需要操作了吧？注释呢？文档呢？统统需要工具啊。

总结起来，HTML要编译，CSS要编译、合并、压缩和，Javascript要编译/合并、压缩甚至生成文档。最后发布还要顾及CDN或者缓存或者bug跟踪进行版本管理如果以上每一步都要自己操作，那么即使只是打命令行也是够呛。

而使用上自动化构建，则在设定好以上多种工具的使用流程之后（几乎）一劳永逸，只需要专心写好流程最开始的源代码就OK，构建工具会完全自动地生成最终结果。能少干活就少干活，那个程序员愿意做重复性工作？

这也就是为什么自动化构建工具在一日发展千里、需求一日多改的前端如此受欢迎的原因了。

### Grunt VS Gulp

是个程序员总会遇到圣战的时候，或是Emacas VS Vim，或是C# VS Java，或是Python VS Ruby，或是AngularJS VS ReactJS，或是IOS VS Android……

<del>当然，PHP是最好的语言所以不用战争。</del>

也有人只是选择困难症后期患者，一旦选项多于一就会头痛欲裂、浑身不自在。

那么，究竟Grunt or Gulp？

为此很多人写过分析的文章，有[中文的][1]、[英文的][2]和[另一篇英文的][3]，总的来说就是，

Grunt：插件比较多，社区成熟，风格偏配置，插件比较混乱，代码较长，过程有临时目录

Gulp：插件不够Grunt多，风格偏代码，插件功能单一专注，代码较短，流式工作无需临时目录

个人选择是Gulp，那个插件数量不够多是个伪缺点，只是不过Grunt多，其实也有上千个，还不够用？！从其他优点来看都是完胜Grunt了。

[1]: http://www.w3ctech.com/topic/114 "谈谈Grunt,NPM,Gulp"

[2]: http://www.hongkiat.com/blog/gulp-vs-grunt/ "The Battle Of Build Scripts: Gulp Vs Grunt"

[3]: http://jaysoo.ca/2014/01/27/gruntjs-vs-gulpjs/ "Grunt vs Gulp - Beyond the Numbers"

### 逆袭的npm

那是不是选择Gulp来构建i18n.js呢？

并不是。

如果有仔细看给出的分析文章，可以看到还有一个构建工具：npm。

众所周知npm实际上是nodejs的包管理工具，然而在其配置文件package.json里面却也可以设置一些可运行项，然后通过`npm run xxx`来运行。从文章来看，也是能够胜任构建的任务。

那么问题来了，从网上基本千篇一律的教程来看，Grunt和Gulp的使用都是装上了自带npm的node，然后通过npm来安装的。既然npm本身就能作为构建工具，那为啥要用Grunt和Gulp？

注意到那篇中文的分析文章还提到“npm一般用在个人项目里,对于团队项目则不适用”，然而果真如此吗？

使用英文搜索一下，不难发现国外也有人提出[停止使用Grunt和Gulp的主张][4]，在文中列出类似或同类构建工具的问题：

1. Bloat
1. Relying on plugins
1. Separate pain in updating
1. False Promises
1. Bad behaviours

接着提出了使用npm的主张，并且[还给出了详细方法][5]，可以看到使用npm更易懂更简洁。

我使用Grunt和Gulp的经验并不多（实际也不是什么复杂的东西），对于文中提出的第一个问题已经深有感触。明明只是简单的工作，却要写一大堆罗嗦的配置。另外Grunt/Gulp插件使用都是local安装，于是明明只是写几个KB大小的库，却要将项目的文件夹弄成几十MB大。插件作用都很专一，更新频率很低，全局安装就好，每开一个项目就独立往项目塞一样的工具简直是闲得蛋疼，尤其npm下载插件经常由于网络原因而失败。

> 当然独立安装项目依赖也有其存在的意义。当将项目发布给其他人使用或者开发的时候，独立安装项目依赖可以保证环境是一样的。

所以结论是，__不要为使用Grunt/Gulp而使用Grunt/Gulp，很多情况下并不需要将事情弄复杂。__

[4]: http://blog.keithcirkel.co.uk/why-we-should-stop-using-grunt "Why we should stop using Grunt & Gulp"

[5]: http://blog.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/ "How to Use npm as a Build Tool"

### package.json

参考国外配置npm的文章，写好package.json。

```javascript
{
  "name": "i18n.js",
  "devDependencies": {
    "concat-cli": "latest",
    "jade": "latest",
    "jshint": "latest",
    "rimraf": "latest",
    "nodemon": "latest",
    "parallelshell": "latest"
  },
  "scripts": {
    "clean:test": "rimraf test/*",
    "clean:dist": "rimraf dist/*",

    "lint": "jshint src/js/main.js",

    "test:html": "jade -P src/test.jade --out test",
    "watch:html": "jade -w -P src/test.jade --out test",

    "prebuild:js": "npm run lint",
    "build:js": "concat-cli -f src/js/wrap/prefix.js src/js/main.js src/js/wrap/suffix.js -o dist/i18n.js",
    "postbuild:js": "uglifyjs dist/i18n.js -o dist/i18n.min.js -m -c",

    "pretest:js": "npm run lint",
    "test:js": "concat-cli -f src/js/wrap/prefix.js src/js/main.js src/js/wrap/suffix.js -o test/i18n.js",
    "watch:js": "nodemon --watch src/js --exec \"npm run test:js\"",

    "pretest": "npm run clean:test",
    "test": "npm run test:js && test:html",
    "test:watch": "parallelshell \"npm run watch:js\" \"npm run watch:html\"",

    "prebuild": "npm run clean:dist",
    "build": "npm run build:js"
  }
}
```

清晰明了。

测试环境清理：rimraf

HTML构建：jade

Javascript排错：jshint

Javascript合并：concat-cli（多个文件复制合并）

全部都是一句话配置，直指命令行。多个任务最终又可以汇集在`test`/`test:watch`中。

> 使用concat-cli构建Javascript比较少见，更多的是使用browserify配合require语法。然而i18n.js库实在太小了，真的不需要复杂的模块化管理。

## 编写i18n.js

### 拆分

先将原js文件拆分成三个。

```javascript
// prefix.js
;(function( root, name, definition ) {
    if ( typeof define === 'function' && define.amd ) {
      define( [], definition );
    } else if ( typeof module === 'object' && module.exports ) {
      module.exports = definition();
    } else {
      root[name] = definition();
    }
})( this, 'i18n', function() {

```

```javascript
// suffix.js

// Return this library
return i18n;

});
```

```javascript
// main.js
var i18n = {};

i18n.bar = function () {
    return;
};
```

接下来可以专心在'main.js'中写代码了。

> 在敲入代码之前记得使用`npm run watch:js`，不然配置毫无意义。

### 内部变量

```javascript
// Save the global object, which is window in browser / global in Node.js.
var root = this;

// This library and internal object
var i18n = {},
    _ = {};

// Current version.
i18n.version = '0.0.1';

// Internel store
var TRANSLATION_TABLE = {};

// Current language
var CURRENT_LANGUAGE = '';

// Save the previous value of the `i18n` variable, can be restored later
// if 'noConflict' is called.
var previousi18n = root.i18n;
```

`TRANSLATION_TABLE`保存翻译文本，`CURRENT_LANGUAGE`保存当前使用的语言，`_`是内部使用的命名空间。另外使用`root`保存全局对象，`previousi18n`保存之前已存在的'i18n'对象。

### 库函数（API）

```javascript
// Restore the previous value of 'i18n' and return our own i18n object.
i18n.noConflict = function () {
    root.i18n = previousi18n;
    return i18n;
};
```

noConflict函数，学jQuery的。

```javascript
// Load the translation table
i18n.load = function ( table ) {
    TRANSLATION_TABLE = _.deepCopy( TRANSLATION_TABLE, table );
    return i18n;
};
```

载入翻译文本，使用深复制（应对多层对象）。

```javascript
// Return the current set language
i18n.current = function () {
    return CURRENT_LANGUAGE;
};
```

返回当前使用的语言。

```javascript
// Change the language, apply to all cached nodes or document.body
i18n.use = function ( language ) {
    var langTable = TRANSLATION_TABLE[language],
        nodes;

    if ( langTable ) {
        nodes = _.filterNodes( root.document.body );
        _.translate( nodes, langTable );
        CURRENT_LANGUAGE = language;
    }

    return i18n;
};
```

切换语言。流程是匹配出语言配置，再从body开始抓取出需要翻译的DOM元素（\_.filterNodes函数），然后翻译（\_.translate函数），最后设置当前语言。

### 内部函数

API函数的内容写得简单，主要是需要基于不少的内部函数。

首先是深复制。

> Javascript中的赋值都是复制，因此对于基本类型（primitive value）：Undefined、Null、Boolean、Number、String来说，直接赋值就是复制。其他的复杂类型，直接赋值同样是复制——然而，复制的是引用，并不是引用的对象。

```javascript
// can handle array and nested objects, not perfect
_.deepCopy = function ( des, src ) {
    var beCopiedIsArray = false,
        target,
        name,
        clone,
        beCopied;

    target = des;

    for ( name in src ) {
        beCopied = src[name];

        if ( beCopied === src ) {
            continue;
        }

        if ( _.isObject( beCopied ) || ( beCopiedIsArray = _.isArray( beCopied ) ) ) {

            if ( beCopiedIsArray ) {
                beCopiedIsArray = false;
                clone = [];
            } else {
                clone = {};
            }

            target[name] = _.deepCopy( clone, beCopied );

        } else if ( beCopied !== undefined ) {
            target[name] = beCopied;
        }

    }

    return target;
};
```

改写自jQuery1.7内部实现的对象深复制函数，只保留了识别数组和对象的功能。因为译文文本就是JSON格式的普通对象（plain object），无需要实现太复杂的复制。核心代码的思想就是检测在当前对象的每一个属性（省略了hasOwnProperty的检测），如果是数组（\_.isArray）或者普通对象（\_.isObject），则实实在在创建一个数组 / 对象以供复制。

而数组 / 对象检测则是用以下代码：

```javascript
// figure out array
_.isArray = Array.isArray || function( obj ) {
    return Object.prototype.toString.call( obj ) === '[object Array]';
};

// figure out object
_.isObject = function( obj ) {
    return Object.prototype.toString.call( obj ) === '[object Object]';
};
```

而库的核心，一个带访问函数的DFS。DOM操作自带取子元素和兄弟元素，写起来很简单。

```javascript
// Walk the DOM, call the visit
_.walkDOM = function ( dom, visit ) {
    var node;

    // nodeType === 1 means element
    // nodeType === 11 means DocumentFragment
    if ( dom && 1 === dom.nodeType || 11 === dom.nodeType ) {
        visit( dom ); // 访问当前DOM元素

        node = dom.firstChild; // 取当前DOM元素的第一个子元素
        while ( node ) {
            _.walkDOM( node, visit ); // 对此子元素递归调用
            node = node.nextSibling; // 从此子元素返回，处理下一个兄弟元素
        }
    }
};
```

通过查看元素的属性来筛选出将要翻译的元素。

```javascript
// Returns array of elements that have attribute 'data-i18n'
_.filterNodes = function ( root ) {
    var nodes = [];

    // traverse DOM tree and collect elements with 'data-i18n' attribute
    _.walkDOM( root, function ( ele ) {
        if ( _.hasAttr( ele, 'data-i18n' ) ) {
            nodes.push( ele );
        }
    });

    return nodes;
};
```

上一个函数中用到的'_.hasAttr'，特别实现是因为IE的取属性方式跟其他浏览器不一样。

```javascript
// Return true if ele has attribute otherwise false
_.hasAttr = function ( ele, attr ) {
    return ele.hasAttribute ? ele.hasAttribute( attr ) : ele[attr] !== undefined;
};
```

接下来是改变元素的文本。代码很简单，做的事情就是遍历DOM元素数组，取属性'data-i18n'的值作为key值，在译文表格中查询value值（\_.getTranslatedText），最后改变元素的文本（\_.setText）。

```javascript
// Translate each node in array with given language table
_.translate = function ( nodes, table ) {
    var key, text, i, length;

    for ( i = 0, length = nodes.length; i < length; i++ ) {
        key = nodes[i].getAttribute( 'data-i18n' );

        if ( key ) {
            text = _.getTranslation( key, table );

            if ( typeof text === 'string' ) {
                _.setText( nodes[i], text );
            }
        }
    }
};
```

\_.getTranslatedText 支持使用点记法，代码直接用以前写过的。[参考](http://blog.e10t.net/implements-list-and-flag-in-simpletemplatejs/)

```javascript
// get translation via path, support dot
_.getTranslatedText = function ( path, json ) {
    var fieldPath = path.split( '.' ),
        data = json,
        index,
        indexLength;

    for ( index = 0, indexLength = fieldPath.length; index < indexLength; index++ ) {
        data = data[fieldPath[index]];
        if ( !data ) {
            return '';
        }
    }

    return data;
};
```

\_.setText 函数就是用'innerText'或'textContent'来设置元素文本。

```javascript
// cross-browser set text
_.setText = function ( ele, text ) {
    var nodeType = ele.nodeType,
        textAttr;

    if ( nodeType && 1 === nodeType ) {
        textAttr = ( 'innerText' in ele ) ? 'innerText' : 'textContent';
        ele[textAttr] = text;
    }
};
```

### 测试

看起来大概写完了，来写一些测试。

> 实际上应该先写测试，再写代码。但是一来库很小，二来我不太懂，所以……不过之后写比较大型的库的时候要好好地用mocha等的测试框架。

用jade语法写一个HTML文件。

```markup
doctype html
html
  head
    meta(charset="UTF-8")
    title test
    script(src="i18n.js")
  body
    select#language(name="language",onchange="toggle()")
      option(value="en") English
      option(value="zh") 中文
      option(value="jp") 日本語

    h1(data-i18n="TITLE") Title
    p(data-i18n="p.text") This is test text.
    button(data-i18n="BUTTON_TEXT") change
    button(data-i18n="BUTTON_ADD",onclick="add()") add

  script.
    i18n.load({
      'en': {
        'TITLE': 'Title',
        'BUTTON_TEXT': 'change',
        'BUTTON_ADD': 'add',
        'p': {
          'text': 'This is test text.'
        }
      },
      'zh': {
        'TITLE': '标题',
        'BUTTON_TEXT': '变',
        'BUTTON_ADD': '添加',
        'p': {
          'text': '这是测试文本。'
        }
      },
      'jp': {
        'TITLE': 'タイトル',
        'BUTTON_TEXT': '変更',
        'BUTTON_ADD': '追加する',
        'p': {
          'text': 'これはテストテキストです'
        }
      }
    });

    function toggle () {
      var ele = document.getElementById( 'language' ),
        value = ele.value;

      i18n.use( value );
    }
```

控制台运行`npm run test:html`生成HTML文件，用浏览器打开，切换一下语言，没问题。

## 继续开发

应用i18n.js的多语言页面，是有可能动态添加DOM元素的（AJAX拉取数据之类的操作），所以i18n.js库也需要将添加的DOM元素翻译一下。于是再添加一个名为'translate'的API好了。

> 由于需要同时修改jade文件和js文件，所以使用`npm run test:watch`，同时监视jade文件和js文件的变化。

```javascript
// Translate nodes
i18n.translate = function ( eles ) {
    var langTable, nodeList, i, index, nodes;

    langTable = TRANSLATION_TABLE[CURRENT_LANGUAGE];

    if ( langTable ) {
        nodeList = Object.prototype.toString.call( eles ) === '[object NodeList]' ||
            'length' in eles ?
            eles :
            [eles];

        for ( i = 0, index = nodeList.length; i < index; i++ ) {
            nodes = _.filterNodes( nodeList[i] );
            _.translate( nodes, langTable );
        }
    }

    return nodes;
};
```

做的事其实和`use`大同小异，只是目标DOM元素不一样。

修改一下测试文件，增加一点代码。

在 body 中添加两个按钮。

```markup
    button(data-i18n="BUTTON_ADD_1",onclick="add(1)") add one
    button(data-i18n="BUTTON_ADD_2",onclick="add(2)") add two
```

在数据中增加按钮的文本。

```javascript
'BUTTON_ADD_1': 'add one line',
'BUTTON_ADD_2': 'add two line',
/* ... */
'BUTTON_ADD_1': '添加一行',
'BUTTON_ADD_2': '添加两行',
/* ... */
'BUTTON_ADD_1': '1行を追加する',
'BUTTON_ADD_2': '2行を追加する',
```

在脚本中增加一个函数，用作模拟动态添加DOM元素。可以添加一个或多个DOM元素。

```javascript
function add ( num ) {
  var p = document.getElementsByTagName( 'p' )
    , newP = document.createElement( 'div' )
    , i
    , node
    , fragment;

  fragment = document.createDocumentFragment();
  newP.innerHTML = '<p data-i18n="p.text">This is test text.</p>';

  for ( i = 0; i < num; i++ ) {
    node = newP.firstChild.cloneNode( true );
    fragment.appendChild( node );
  }
  i18n.translate( fragment );

  p = p[p.length-1];
  p.parentNode.insertBefore( fragment, p.nextSibling );
}
```

再使用浏览器测试一下，同样没问题了。

### 再拆分一下

现在`main.js`文件看起来比较复杂，可以再分别拆分成`var.js`，存放顶层变量；`util.js`，包含内部的函数；`api.js`，包含库的API。

稍微修改一下`package.json`文件，相关位置改成拆分后的文件。

```javascript
    "lint": "jshint src/js/var.js && jshint src/js/util.js && jshint src/js/api.js",
/* ... */
    "build:js": "concat-cli -f src/js/wrap/prefix.js src/js/var.js src/js/util.js src/js/api.js src/js/wrap/suffix.js -o dist/i18n.js",
/* ... */
    "test:js": "concat-cli -f src/js/wrap/prefix.js src/js/var.js src/js/util.js src/js/api.js src/js/wrap/suffix.js -o test/i18n.js",
```

最后运行`npm run build`将js库编译出来并压缩。