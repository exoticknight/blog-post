---
title: 配合 angular 和 angular-material 来开发基于 electron 的应用 · 0-- 前言
categories:
  - [技术, electron]
  - [编程语言, javascript]
tags: [javascript, angularjs, angular-material, electron]
permalink: build-application-with-angular-and-ng-material-based-on-electron-0-preface
id: 42
updated: '2015-10-20 08:28:04'
date: 2015-06-04 23:02:37
---

## Electron

Electron 是什么？它之前的名字是 Atom Shell，是 Github 开发的结合了 io.js 和 chromium 的跨平台桌面应用框架。Github 自己出的编辑器 Atom 以及微软出的编辑器 VSCode 都是基于这个框架。

众所周知，Google chrome 就是基于 chromium 而发展出来的一款优秀的浏览器。因其出色的体验和网页解析性能，所有国内出产的 < del > 山寨 </del > 浏览器 / 双核浏览器，无不选用了 chrome 作为内核。所以在网页解析渲染方面，使用 chromium 是极其正确的选择。

那跟平常的桌面应用构建，使用 Electron 又有什么优势呢？

普通的桌面应用构建，比较成熟的语言不外乎 C/C++、Java、C#、Python 等。然而 C/C++ 易学难精，即使其 GUI 框架有 MFC、Qt、KDE 等众，也是极难快速开发；Java 的 GUI 烂得不提也罢；C# 极有可能成为以后霸主，然而还在跨平台表现上有所欠缺；Python 则是个人喜好关系顺带一提，其实很少作为 GUI 主力语言被使用。（当然你可以阅读本人的另一个 [有关 python 和 Qt 构建桌面应用的系列][1]）

Electron 则是使用了 Javascript 作为主力语言，并且为其加上了原生支持 html5 和 CSS3 的浏览器。从 GUI 构建来说，使用 html 和 css 的网页构建显然更加简单，成熟的工具和技术数不胜数；而作为桌面应用着重依赖的 IO、进程和网络通信模块等则由支持 ES6 的 io.js 提供，这样前端后端均采用 Javascript 语言，大大降低技术复杂性。

[1]: http://blog.e10t.net/python-with-qt-application-development-catalogue/

## 与 NW.js（旧名 node-webkit）的异同

如果你有经常关注前端的消息，那么一定听说过一个国人开发的 GUI 框架：node-webkit。然后一看到 Electron，就会皱皱眉头：这不就是 node-webkit 嘛！

然而，Electron 和 node-webkit 并不一样，其 github 项目上有详细的对比，[链接][2]。

就个人理解来说，NW.js 偏向网页主导，是一个加上了 node.js 的浏览器；Electron 则是 javascript 主导，是 io.js 加上了一个 chromium。

> 准确来说，Electron 只是选择了网页作为 GUI，并非为 GUI 绑定了 javascript。在 Electron 文档的 [Quick start][2] 中很明确地指出「It doesn't mean Electron is a JavaScript binding to GUI libraries. Instead, Electron uses web pages as its GUI, so you could also see it as a minimal Chromium browser, controlled by JavaScript.」

在听说了 node-webkit 之后，我曾经上手把玩了一下，当时也是惊讶于其结合了浏览器内核而得到的强大表现力。因为自己在前端方面有一点技术，所以在编写界面的过程中感觉非常舒服。不过我也留意到其在软件方面的能力明显有比较大的欠缺，除了能读写文件外似乎没有什么亮点。（当然不排除在改名为 NW.js 后会加入了更多功能的可能性）

总之，NW.js 更像是将网页打包成应用，而 Electron 则是实际开发的应用。

[2]: https://github.com/atom/electron/blob/master/docs/development/atom-shell-vs-node-webkit.md

[3]: https://github.com/atom/electron/blob/master/docs/tutorial/quick-start.md

## angular 和 angular material

如果要将网页设计应用到软件界面开发上，那么一些 MVC 框架或 UI 框架就比较适合。MVC 框架中比较有名的是 knockout 和 Backbone，而 UI 框架，则是 reactjs、angularjs 和 polymer 最为著名。国产的还有 avalon。

那么为什么选 angular 呢？因为 angular 的理念比较符合开发网页应用，更重要的是有 angular material 这样一个比较能使用的 UI 主题。相比之下，knockout 和 Backbone 功能太弱，reactjs 则是太激进（一开始我是选 reactjs 的，但是一番尝试之后还是放弃了），polymer 则未作深入了解。

不过，就像 Electron 只是选用了网页作为呈现 GUI 的方式，那么在编写基于 Electron 的应用的时候，GUI 框架的选择其实并非固定死的，如有必要或者个人喜好，转而使用 polymer 或者 reactjs 也未尝不可。

## 本系列的目的

如果有看过鄙人写的 [python × Qt 应用开发系列][1]，那么一定知道本人的教程都偏向实践，喜欢实际解释代码和一定程度地搞清楚技术的细枝末节，而非跟着网上一搜一大把的英文教程或者官方文档演示一篇后以近乎翻译一般地写出所谓的 “教程”。官方文档就摆在那，谁不会 RTFM？

在本系列中，鄙人同样会以记录一个应用的开发流程的形式来呈现成功（或者说，可行）的开发方式。有时会有大量的代码，有时又会有长篇的理论讨论，有时又会有大段的思维解释，希望读者能耐心读下去。