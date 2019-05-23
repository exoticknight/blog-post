---
title: 配合 angular 和 angular-material 来开发基于 electron 的应用 · 1-- 分析与配置
categories:
  - [技术, electron]
  - [编程语言, javascript]
tags: [javascript, angularjs, angular-material, electron]
permalink: build-application-with-angular-and-ng-material-based-on-electron-1-analyze-and-configuration
id: 43
updated: '2015-09-05 17:11:13'
date: 2015-06-13 19:32:07
---

![thumbnail](https://i.imgur.com/kvb5CTf.png)

> [应用 github 地址](https://github.com/radioit/radioit-desktop)。github 代码和文章代码并不同步，用作预览和 PR。

## 目标分析

一句话概述：开发的应用是一个抓取网页有用信息并重新统一排布的应用，是 [之前文章][1] 提到的 radioit 计划里脚本的 GUI 版本。

关键词：网页抓取、信息统一、信息排布、脚本的 GUI 版本

功能：

1. 浏览某一个广播站的广播
1. 浏览某一个广播的详细信息
1. 下载某一个广播最新的相关图片
1. 下载某一个广播最新的相关音频
1. 能够通过设置代理来突破某些限制
1. 能够离线浏览（未定）
1. 预定周期下载任务（未定）
1. 整合视频压制工具（未定）

业务流程：

1. 请求特定 url 资源
1. 对取得的 url 资源进行信息提取
1. 信息整合成统一格式
1. 显示信息
1. 某些情况下执行预定命令行（未定）

技术联想：

1. 请求特定 url 资源 -> node
1. 对取得的 url 资源进行信息提取 -> node 的某些库
1. 信息整合成统一格式 -> javascript, json
1. 显示信息 -> html、css、angular、angular material
1. 某些情况下执行预定命令行（未定） -> node

[1]: http://blog.e10t.net/radioit-plan-animate-radio-script-radioitscript/

## 技术分析

技术要点：

1. node.js(io.js)，负责网络连接，网页内容解析提取，非浏览器环境因此能够进行跨域访问
1. angular，MVVM 框架，自动进行数据的渲染
1. angular material，angular 推出的 material design UI 框架，适合作为桌面应用使用
1. stylus，CSS 预处理器，合理直观的 CSS 编写格式

脑内讨论

* Q：为什么使用 Electron？
* A：Electron 有意思地使用了 `main` 进程和 `render` 进程，`render` 进程产生于 `main` 进程中，因此可以简单地产生多个 `render` 进程，也就是多窗口。这是一个优势。

* Q：不用 angular，用 react 是否可以？
* A：可以，然而在假定选用了 react 之后，然后脑内模拟了一下编程的过程，react 似乎并不适合 html 代码经常修改的场合。而自己比较在行的是写 html 和 css，在界面设计上必定经常修改。另外在 material design 的 UI 框架上，使用配合 angular 的 angular material 显然更具操作性。当然 react 下也有 material design 的 UI 框架，但在试用之后感觉不太好用。另外就是自己翻译过一篇很长的有关 angular 的[文章][2]，对 angular 比较熟悉。日后考虑改用 Polymer 重写。

* Q：material design 是必须使用的吗？
* A：作为桌面应用，需要有一点时刻记住的是桌面应用跟网页是不一样的。桌面应用需要稳定的窗口，要有标题栏等清晰的组件，也不需要太花哨的特效。material design 或者受 material design 影响的一些简洁 UI 风格已经在某些桌面软件上应用开来。Electron 作为使用网页作为 GUI 表现，使用 material design 是个稳妥之举。

* Q：为什么不用 SASS / LESS？
* A：SASS 需要 Ruby，对非 Rubyer 是非常无理的要求，逻辑表现能力强大而无用（非常用）；LESS 语法简单，支持混写，但逻辑表现力太弱。stylus 则是既有强有力的特性，也足够简单。有时，工具够用就行。参考：[Why I Choose Stylus (And You Should Too)][3]

* Q：node 和页面中的 angular 如何沟通？
* A：`main` 进程和 `reander` 进程有特定的模块进行通信。`render` 进程能通过页面中的全局变量和 angular 进行通讯。

* Q：为什么要使用 node 的库来处理网页请求和内容提取？angular 自带有 $http 不是更方便？
* A：如此一来，就能各自开发。node 只需要管如何得到数据，angular 只需要管如何显示数据。另外，如果需要更改 GUI，那么只需要去掉 angular，换上其他 UI 框架就可以，数据生成不受任何印象。只是如此开发需要更多的精力。

[2]: http://blog.e10t.net/translation-building-the-2048-game-in-angularjs/

[3]: http://webdesign.tutsplus.com/articles/why-i-choose-stylus-and-you-should-too--webdesign-18412

## 开发配置

### NPM 配置

node.js 的安装是必须的，不多介绍。安装完自带 npm 管理工具。

用的最多的 node 命令：
```bash
npm i xxx -g
npm i xxx --save
npm u xxx --save
npm update
```

第一条是全局安装 node 模块。比如一些常用工具，每一个项目都可以用到的工具等。这些模块可以写在 `package.json` 中的 `devDependencies` 字段中。

第二条是本地安装 node 模块并保存信息到 `package.json` 中。适合项目特定使用的模块。这些模块可以写在 `package.json` 中的 `dependencies` 字段中。

第三条是卸载本地安装的 node 模块。node 模块太多了，尝试多个选最好的。

第四条是升级 node 模块。

以下是 `package.json` 文件的暂时内容。

```javascript
// package.json
{
  "name": "Radioit",
  "description": "radioit desktop edition",
  "version": "0.1.0",
  "main": "main.js",
  "author": "exoticknight",
  "mail": "draco.knight0@gmail.com",
  "devDependencies": {
    "jshint": "latest",
    "rimraf": "latest",
    "electron-packager": "latest",
    "electron-prebuilt": "latest",
    "silence-chromium": "latest",
    "mkdirp": "latest",
    "nib": "latest",
    "stylus": "latest",
    "uglifyjs": "latest",
    "browserify": "latest",
    "watchify": "latest",
    "parallelshell": "latest"
  },
  "scripts": {
    "build:css": "stylus -u nib src/css/app.styl -o static/css/app.css",
    "watch:css": "stylus -u nib src/css/app.styl -o static/css/app.css -w",
    "test": "electron main.js 2>&1 | silence-chromium",
    "start": "npm run build:css && electron main.js 2>&1 | silence-chromium"
  },
  "dependencies": {
    "deepcopy": "^0.5.0",
    "extend": "^2.0.1"
  }
}
```

暂时并没有太多的东西，注意要开发基于 Electron 应用，`electron-packager` 和 `electron-prebuilt` 必不可少，一个是 Electron 的打包工具，一个是 Electron 运行环境。而 `silence-chromium` 则是将 chromium 控制台信息输出到系统终端的工具。其他的工具都是博主开发过程中精选过的工具，还请读者自行 Google 之来学习。

如果你看过本博客之前的一篇文章：[i18n.js 库的编写兼使用 npm 辅助开发][4]，就知道博主是能用 npm 就不用 gulp / grunt 的，因此在 `scripts` 字段中也写上了运行的脚本。

[4]: http://blog.e10t.net/write-i18n-js-with-help-of-npm-as-build-tool/

### angularjs 配置

angular 的版本比较稳定，因此直接用 `bower` 来获取，不推荐其他包管理工具。

> `bower` 需要先使用 `npm install bower -g` 来安装，也需要配置了 git 的环境。如果你使用 github for windows，那么请使用 gitshell 来运行。

angular 的安装在下一节中。

### angular material 配置

对于 `bower` 来说，angular material 跟 angular 是一样的东西，只是后者是前者的依赖。

运行 `bower install angular-material`， bower 会自动将依赖的的 `angular`、`angular-aria` 和 `angular-animate` 一并安装上。

安装完后所有文件会在项目目录下的 `bower_components` 中找到。

### Electron 配置

具体参考：[Quick Start][5]

在 `package.json` 中有一个 `main` 字段，值是 `main.js`。这个就指定了 Electron 启动应用的入口。

准备好文件结构。

```markup
app/
├── package.json
├── main.js
└── index.html
```

[5]: https://github.com/atom/electron/blob/master/docs/tutorial/quick-start.md

### 初始程序

```javascript
var BrowserWindow = require( 'browser-window' );  // Module to create native browser window.
var ipc = require( 'ipc' );
var path = require( 'path' );

// global variable
var APP_NAME = 'Radioit';
var INDEX = 'file://' + path.join( __dirname, 'index.html' );

// Report crashes to our server.
require( 'crash-reporter' ).start();

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the javascript object is GCed.
var mainWindow = null;

// Quit when all windows are closed.
app.on( 'window-all-closed', function () {
  if ( process.platform != 'darwin' )
    app.quit();
});

// This method will be called when Electron has done everything
// initialization and ready for creating browser windows.
app.on( 'ready', appReady );

function appReady () {

    mainWindow = new BrowserWindow({
        'width': 1024,
        'height': 600,
        'resizable': false,
        'accept-first-mouse': true,
        'title': APP_NAME,
        'show': false
    });

    mainWindow.loadUrl( INDEX );
    mainWindow.openDevTools(); // remove this

    mainWindow.webContents.on( 'did-finish-load', function () {
        mainWindow.show();
    });

    mainWindow.on( 'closed', function () {
        mainWindow = null;
    });
}
```

代码好像很多，其实基本就是照抄 quick start，没有任何压力。

博主写的 `main.js` 和 quick start 中的有所不同。在新建 `mainWindow` 的时候，加入了其他参数 `show: false` 和 `resizable: false`，分别是隐藏窗口和窗口不可拉伸。也增加了一个：

```javascript
    mainWindow.webContents.on( 'did-finish-load', function () {
        mainWindow.show();
    });
```

作用是网页内容完全载入后才显示窗口，避免一些内容还没载入完就显示。

最后运行 `npm run test` 看看结果。

> 有什么问题请留言。