---
title: '[译]使用 AngularJS 编写 2048 游戏'
categories:
  - [翻译, 技术]
tags: [translation, angularjs]
permalink: translation-building-the-2048-game-in-angularjs
id: 18
updated: '2014-06-11 14:13:25'
date: 2014-05-29 12:58:14
---

> 原文[http://www.ng-newsletter.com/posts/building-2048-in-angularjs.html](http://www.ng-newsletter.com/posts/building-2048-in-angularjs.html)，所有版权属于原文。考虑到排版和图片，内容稍有修改，对文章表达的意思并无太大影响。第一次渣翻长文，求翻译大大们拍砖和调教。

我们最近经常被问到的其中一个问题是作为一个框架，Angular 在什么情况下使用并不太适合。我们的标准答案通常是编写游戏的时候，因为 Angular 有它自己的事件处理循环（$digest 循环）而游戏通常要求非常多的底层 DOM 操作。其实这个答案并不准确因为 Angular 其实是能支持多数游戏的编写的。即使要求大量的 DOM 操作的游戏，Angular 也能胜任其静态部分，例如高分记录和游戏菜单。

如果你是像我那样的人（同时是个技术人），你有可能会喜欢玩那个流行的 [2048](http://gabrielecirulli.github.io/2048) 游戏。游戏的目标是通过合并相同数值的方块来得到数值是 2048 的方块。

![Injection](http://www.ng-newsletter.com/images/2048/game.gif)

[在 HackerNews 上讨论](https://news.ycombinator.com/item?id=7554348)

在今天的博文中，我们准备使用 AngularJS 来仿制这个游戏，而且是从头到尾完整地解释整个 app 的编写流程。这个 app 是一个相当复杂的应用，我们也希望利用这篇博文来展示如何编写复杂的 AngularJS 应用。

此 Angular 版应用的[demo](http://d.pr/SnWD)。

来让我们开始吧！

> TL;DR: 本应用的完整源代码都能在文章底部的 github 链接里面找到。

## 目录

1. [计划](#planning)
1. [模块结构](#modular)
1. [GameController](#game-controller)
1. [测试，测试，测试](#tdd)
1. [建造游戏网格](#build-grid)
1. [SCSS 来救援](#scss)
1. [Tile 指令](#the-tile-directive)
1. [游戏棋盘](#starting-the-game)
1. [网格理论](#grid-theory)
1. [玩法（键盘）](#keyboard)
1. [按下开始按钮之时](#start-button)
1. [游戏循环](#game-loop)
1. [计算得分](#keeping-score)
1. [游戏结束和获胜界面](#game-over)
1. [动画](#running-the-animation)
1. [自定义](#customizing-size)
1. [演示](#demo)

<a name="planning"></a>
## 第一步：计划

![Minification](http://www.ng-newsletter.com/images/2048/3d-board.png)

我们首先想做的是对将要编写的应用进行高层次设计。如果是仿制一个应用或是从零开始，我们都会这样做，不论应用有多大。

审视一下，我们可以看到游戏是有一块游戏棋盘，上面有一些方块。每一个方块的位置就是数值方块的位置。我们可以利用这一个事实，使用 CSS3 而不是 javascript 来摆放方块，后者需要知道方块摆放的位置。当摆放方块的时候，我们只需要保证方块覆盖在合适的位置上就可以了。

CSS3 的使用让我们不但能够免于在 CSS 上搞动画的工作，而且能使用标准的 AngularJS 行为（译者注：AngularJS behavior）来跟踪游戏棋盘、方块和游戏逻辑的状态。

因为我们只有一个页面，所以只需要一个 controller 来管理页面。

既然在应用的运行期间只有一个游戏棋盘，我们就另外创建单一一个 `GridService` 的 service 实例来保存所有的网格逻辑。service 都是单例对象，适合用来存储网格。我们会使用 `GridService` 来放置和移动方块、寻找可供移动的位置和管理网格。

我们将游戏的逻辑和运行存储在另外一个叫 `GameManager` 的 service 中。`GameManager` 负责管理游戏状态、处理移动和维护得分（包括当前得分和最高得分）。

最后，还需要一个组件来控制键盘。我们将使用一个名为 `KeyboardService` 的 service（只需要一个键盘动作的处理）。我们会在这篇文章中实现桌面版的处理，然而我们也可以重用同一个 service 来处理触屏动作使其能在移动设备上使用。

### 开始编写应用

要开始编写，我们先创建一个基本的应用（我们使用[yeoman](http://yeoman.io) angular generator 来生成应用的结构，但这是非必要的。我们只是将其作为一个起始点，但很快就会做出分支）。我们新建一个包含整个应用的目录，然后再在 `app/` 目录旁边建一个 `test/` 目录。

![Minification](http://www.ng-newsletter.com/images/2048/directory_structure.png)

> 以下使用 yeamon tool 来构建项目的指南。如果你更喜欢自己动手，可以跳过依赖安装直接进入下一章节。

我们要先保证安装了 `yeamon` 才能在项目中使用。Yeamon 依赖 NodeJS 和 npm。NodeJS 的安装并不在本文叙述的范围内但是在 [NodeJS.org](http://nodejs.org) 上有一个很好的指南。

在 `npm` 安装完后，我们就能安装 yeamon tool，`yo`，和 angular generator（`yo` 会使用这个生成器来生成我们的 Angular 应用）：

```bash
$ npm install -g yo
$ npm install -g generator-angular
```

安装完之后，就可以使用 yeamon tool 来创建应用了，按照下面的来：

```bash
$ cd ~/Development && mkdir 2048
$ yo angular twentyfourtyeight
```

工具会问你一些问题，一律答 yes，除了只选 `angular-cookies` 作为依赖，因为我们不需要除了缺省以外的依赖。

> 注意使用 Angular generator 会要求你安装 ruby 环境、gem 和 compass。文章下面给出的完整代码中会介绍如何避免使用 ruby 和 compass。

#### 我们的 angular 模块

新建 `scripts/app.js` 文件来控制我们的应用。来，开始编写吧：

```javascript
angular.module('twentyfourtyeightApp', [])
```

<a name="modular"></a>
## 模块结构

现在比较推荐的 Angular 应用结构是根据功能来构建而不是类型。也就是说，不是以 controllers（译者注：控制器）、services、directives 等来分离我们的组件而是以功能来定义模块结构。例如在我们的应用中，定义了一个 `Game` 模块和 `Keyboard` 模块。

![Minification](http://www.ng-newsletter.com/images/2048/scripts_dir.png)

这样的模块结构让我们能够清晰分离出跟文件结构相匹配的职责。这样做既能帮助我们构建大型的复杂的 angular 应用，也能让功能在不同的应用间共享。

之后我们将会建立起匹配文件和目录结构的测试环境。

#### 视图

在我们的项目中，从视图开始编写是最容易的。审视一下，要做的视图 / 模板只有一个。我们不需要多个视图，所以只需要一个 `<div>` 元素来包含应用中的所有内容。

在我们的的 `app/index.html` 文件中，我们需要包含所有的依赖（包括 `angular.js` 自身和自己编写的 javascript 文件——现在就只有 `scripts/app.js`），就像下面的：

```markup
<!-- index.html -->
<doctype html>
<html>
  <head>
    <title>2048</title>
    <link rel="stylesheet" href="styles/main.css">
  </head>
  <body ng-app="twentyfourtyeightApp"
    <!-- header -->
    <div class="container" ng-include="'views/main.html'"></div>
    <!-- script tags -->
    <script src="bower_components/angular/angular.js"></script>
    <script src="scripts/app.js"></script>
  </body>
</html>
```

> 你完全可以编写更复杂的多视图游戏——如果你这样打算的话请在下面留言，我们非常期待你的表现。

现在 `app/index.html` 文件做好了，我们只需要在 `app/views/main.html` 中继续细节化应用层面的视图就可以了。当我们需要在应用中引入新资源的时候就只需要修改 `index.html` 了。

赶快打开 `app/views/main.html`，所有的游戏相关的视图都放在此。通过使用 `controllerAs` 语法，控制器就可以显式暴露在任何需要在 `$scope` 中找数据和查询控制器对应组件的地方。

```markup
<!-- app/views/main.html -->
<div id="content" ng-controller='GameController as ctrl'>
  <!-- Now the variable: ctrl refers to the GameController -->
</div>
```

> `controllerAs` 语法是 1.2 版本提供的比较新的语法。当要在页面处理多个控制器的时候非常有用，因为这样就能指定包含我们需要的功能和数据的控制器。

在视图中，我们想至少要显示如下几个东西：

1. 游戏的静态标题
2. 当前的游戏得分和本地用户最高的得分
3. 游戏棋盘

游戏的静态标题可以像下面那么简单：

```markup
<!-- heading inside app/views/main.html -->
<div id="content" ng-controller='GameController as ctrl'>
  <div id="heading" class="row">
    <h1 class="title">ng-2048</h1>
    <div class="scores-container">
      <div class="score-container">{{ ctrl.game.currentScore }}</div>
      <div class="best-container">{{ ctrl.game.highScore }}</div>
    </div>
  </div>
  <!-- ... -->
</div>
```

注意在引用 `currentScore` 和 `highScore` 的时候我们也在视图中引用了 `GameController`。`controllerAs` 语法让我们能显式地引用自己感兴趣的控制器。

<a name="game-controller"></a>
## 控制器 GameController

现在既然已经有了一个合理的项目结构，我们赶快来创建一个 `GmaeController` 来控制会在视图上显示的数据。在 `app/scripts/app.js` 中，我们可以在主要模块 `twentyfourtyeightApp` 里创建这个控制器。

```javascript
angular
.module('twentyfourtyeightApp', [])
.controller('GameController', function() {
});
```

在视图中，我们已经引用了一个 `game` 对象，此对象会在 `GameController` 中进行设置。`game` 对象引用的是主 * 游戏对象 *。我们会在另外一个新的模块中创建这个主游戏对象，新的模块也会保存游戏中的所有引用。

现在还没有创建这个模块，应用不会在浏览器中载入。而在控制器里面，我们可以加上对 `GameManager` 的依赖：

```javascript
.controller('GameController', function(GameManager) {
  this.game = GameManager;
});
```

记住，我们正在做的是为应用中不同的部分创建模块级别的依赖，所以为了能在我们的应用中加载这些模块，需要在我们 Angular 模块中作为依赖来列出。将 `Game` 作为 `twentyfourtyeightApp` 的依赖，要在我们定义模块的地方的数组中列出。

完整的 `app/scripts/app.js` 文件看起来应该像下面那样：

```javascript
angular
.module('twentyfourtyeightApp', ['Game'])
.controller('GameController', function(GameManager) {
  this.game = GameManager;
});
```

### The Game

现在已经将部分数据绑定到视图上（译者注：原文 Now that we have the view partially hooked up to the view，或有误），我们可以开始编写游戏的逻辑了。在 `app/scripts/` 目录下新建 `app/scripts/game/game.js` 中创建游戏模块：

```javascript
angular.module('Game', []);
```

> 当创建模块的时候，我们通常将其放在以模块命名的目录内，而以模块命名的文件来完成初始化工作。比如，我们正在写一个游戏 (译者注：game) 模块，于是我们在 `app/scripts/game` 目录下的 `game.js` 中编写。这个方法在生产环境下被认为是可扩展的和合理的。

`Game` 模块会提供唯一的核心组件：`GameManager`。

我们编写的 `GameManager` 模块要做到：维持游戏的状态和玩家能做出的移动，维护得分、判断游戏结束和搞清楚是玩家赢了还是输了。

当在编写应用的时候，我们通常将已知需要的方法写成桩方法，为这些方法写测试然后再填内容。

> 为了文章起见，我们在这个模块里会走一遍这个流程。当继续写剩下的模块的时候，我们则只会涉及到应该测试的核心组件。

我们知道到现在为止 `GameManager` 中会提供的几个 * 已知的 * 功能：

1. 创建一个新的游戏
2. 处理游戏循环 / 移动操作
3. 更新得分
4. 跟踪游戏的进行情况

记住这几个功能，我们就能勾勒出 `GameManager` 服务的基本轮廓以供测试：

```javascript
angular.module('Game', [])
.service('GameManager', function() {
  // Create a new game
  this.newGame = function() {};
  // Handle the move action
  this.move = function() {};
  // Update the score
  this.updateScore = function(newScore) {};
  // Are there moves left?
  this.movesAvailable = function() {};
});
```

完成了基本的功能性函数之后，先挪一下，去写测试来决定在 `GameManager` 中 * 已知的 * 需要支持的函数中空白部分的内容。

<a name="tdd"></a>
## 测试驱动开发（TDD）

在开始实施测试前，我们需要配置好 karma 来驱动我们的测试。如果你对 karma 并不熟悉，就只需要了解到它是一个测试运行器，能让我们舒服而高效地在控制台和代码中自动化操作前端测试。

![Running karma](http://www.ng-newsletter.com/images/2048/running_karma.png)

Karma 作为一个 npm 包，依赖于 NodeJS。运行命令行来安装：

```bash
$ npm install -g karma
```

> 参数 `-g` 告诉 npm 这个包作为全局模块来安装。没有这个参数，包将只会安装到本地的工作目录上。

如果你是通过 yeoman angular 生成器来构建应用的话可以跳过以下的部分。

要使用 karma，需要一个配置文件。虽然我们这里不会深入叙述如何配置 Karma（在 [ng-book](https://www.ng-book.com) 中查看详细的 karma 配置选项），但是过程中决定性的部分就是让 Karma 载入所有我们想要测试的文件。

我们可以使用 `karma init` 命令来生成一个基本的配置文件：

```bash
$ karma init karma.conf.js
```

命令会问几个问题然后生成 `karma.conf.js`。这里我们修改一下其中两个选项：`files` 数组和打开 `autoWatch`：

```javascript
  // ...
  files: [
    'app/bower_components/angular/angular.js',
    'app/bower_components/angular-mocks/angular-mocks.js',
    'app/bower_components/angular-cookies/angular-cookies.js',
    'app/scripts/**/*.js',
    'test/unit/**/*.js'
  ],
  autoWatch: true,
  // ...
```

一旦写好了配置文件，任何时候我们保存文件都可以运行测试了（测试文件在 `test/unit/` 目录内）。

我们像如下那样执行命令 `karma start` 来运行测试：

```bash
$ karma start karma.conf.js
```

### 编写第一个测试

karma 已经配置好了，可以写对 `GameManager` 的基本测试了。然而我们还并不清楚应用的整个功能，所以暂时只能写有限的测试。

> 在编写应用的时候我们经常发现 API 需要修改，所以与其在变化前投入大量时间，不如建立好对基本功能的测试然后在深入测试中找到最终的 API。

用是否有可能的移动来作为第一个写的测试是个好选择。简单地编写几个我们已知需要的返回真 / 假的调用，来测试我们应用的逻辑行为。

创建 `test/unit/game/game_spec.js` 文件然后开始填入内容：

```javascript
describe('Game module', function() {
  describe('GameManager', function() {
    // Inject the Game module into this test
    beforeEach(module('Game'));

    // Our tests will go below here
  });
});
```

> 在这个测试中我们使用 [Jasmine](http://jasmine.github.io/2.0/introduction.html) 语法。

跟其他单元测试一样，我们需要创建一个 `GameManager` 对象的实例。我们可以使用普通的语法（测试服务的时候）将它注入到测试中。

```javascript
  // ...
  // Inject the Game module into this test
  beforeEach(module('Game'));

  var gameManager; // instance of the GameManager
  beforeEach(inject(function(GameManager) {
    gameManager = GameManager;
  });

  // ...
```

有了这个 `gameManager` 实例，就可以建立对函数 `movesAvailable()` 的期望值。

我们定义的 `movesAvailable()` 函数是用来检测是否有空格剩余和是否有方块可以合并。另外这个结果跟游戏是否结束是有关联的，我们会将这个方法放进 `GameManager` 中，但是在之后创建的 `GridService` 中才实现大多数的复杂细节。

棋盘上要有剩余可走的地方，必须满足以下两个条件：

1. 棋盘上有空余空格
2. 方块可以合并

弄清楚了这两个条件，我们就可以写出测试来看看是否符合。

基本的思路就是我们写出的单元测试对于设定的条件要能作可观察到的反应。然后因为要依赖 `GridService` 来反映游戏的状态，所以需要模拟出这个条件来保证在 `GameManager` 中的逻辑是正确的。

#### 模拟 `GridService`

要模拟 `GridService`，我们只需要简单地 * 重写 * 缺省的 Angular 行为，替换 * 真正的 * 服务为我们模拟出来的服务，然后就可以在模拟的服务中建立可控制条件。

详细一点说就是，我们简单地创建一个拥有模拟方法的假对象然后通过在 `$provide` 中换上来骗 Angular 说这个假对象是 * 真 * 对象。

```javascript
  // ...
  var _gridService;
  beforeEach(module(function($provide) {
    _gridService = {
      anyCellsAvailable: angular.noop,
      tileMatchesAvailable: angular.noop
    };

    // Switch out the real GridService for our
    // fake version
    $provide.value('GridService', _gridService);
  }));
  // ...
```

现在我们就可以用 `_gridService` 这个假对象实例来建立条件了。

我们希望当有单元格剩余的时候函数 `movesAvailable()` 返回 true。在 `GridService` 中模拟一个 `anyCellsAvailable()` 函数（其实还没写）。我们期望这个在 `GridService` 的函数能告诉我们还有剩余的单元格。

```javascript
// ...
describe('.movesAvailable', function() {
  it('should report true if there are cells available', function() {
    spyOn(_gridService, 'anyCellsAvailable').andReturn(true);
    expect(gameManager.movesAvailable()).toBeTruthy();
  });
  // ...
```

现在基础工作已经做好了，我们可以接着建立第二个条件了。如果方块可以合并，那么我们希望 `movesAvailable()` 保证会返回 true。相反的情况也是返回 true 因为既没有单元格空余也没有可合并的方块才是没有步数可走。

另外两个保证这个结果的测试是：

```javascript
// ...
it('should report true if there are matches available', function() {
  spyOn(_gridService, 'anyCellsAvailable').andReturn(false);
  spyOn(_gridService, 'tileMatchesAvailable').andReturn(true);
  expect(gameManager.movesAvailable()).toBeTruthy();
});
it('should report false if there are no cells nor matches available', function() {
  spyOn(_gridService, 'anyCellsAvailable').andReturn(false);
  spyOn(_gridService, 'tileMatchesAvailable').andReturn(false);
  expect(gameManager.movesAvailable()).toBeFalsy();
});
// ...
```

将基础工作搞好，我们也好在实现真正函数前写好测试。

> 虽然考虑到整个文章的整体性我们不会再在文章中使用 TDD，但是我们建议你应该始终使用 TDD。可以在下面的完整代码中查看更多的测试代码。

## 回到 GameManager

现在我们的任务就是实现函数 `movesAvailable()`。然而我们已经确认了代码可行性__和__要求的条件，实现起来实在简单。

```javascript
  // ...
  this.movesAvailable = function() {
    return GridService.anyCellsAvailable() ||
            GridService.tileMatchesAvailable();
  };
  // ...
```

<a name="build-grid"></a>
## 建造游戏网格

到现在为止我们已经让 `GameManager` 运行起来了，然后就是要创建 `GridService` 来处理在棋盘中的所有状况。

回忆一下我们的想法：在 `GridService` 中使用两个本地数组变量，基本数组 `grid` 和基本数组 `tiles`。在 `app/scripts/grid/grid.js` 文件中写服务：

```javascript
angular.module('Grid', [])
.service('GridService', function() {
  this.grid   = [];
  this.tiles  = [];
  // Size of the board
  this.size   = 4;
  // ...
});
```

当开始一个新的游戏的时候，我们需要清空这些数组。而因为 `grid` 数组只是用来放方块的 DOM 元素组成的。

然而数组 `tiles` 则是动态的，它会跟踪游戏过程中的当前的方块。使用游戏中不同的状态之前，先在页面上建造好网格先吧，这样我们也好看看大概样子是怎么样。

回到 `app/views/main.html`，我们开始设计网格。因为网格是动态而又带有我们给它写的逻辑，所以只有就其放在其指令（译者注：directive）中才合乎逻辑。使用指令可以让主要的模板保持简洁，同样也能将功能封装在指令中而让主要的控制器保持简洁。

在 `app/index.html` 中我们将网格指令添加上然后在控制器中传递给 `GameManager` 实例。

```markup
  <!-- instructions -->
  <div id="game-container">
    <div grid ng-model='ctrl.game' class="row"></div>
    <!-- ... -->
```

我们是在 `Grid` 模块里写这个指令的，所以在 `app/scripts/grid/` 目录下，新建一个 `grid_directive.js` 文件来安放我们的 `grid` 指令。

在 `grid` 指令里面，我们只需要少量变量因为它需要封装视图，能做的事情不多。

指令会需要持有 `GameManager` 的实例（或者至少是一个有 `grid` 和 `tiles` 数组的模型），所以将其设置为指令的依赖。另外，不希望指令由于页面上的其它内容或者 GameManager 自身的原因瘫痪，所以我们创建了隔离作用域。

> 查看我们写的 [自定义指令](http://www.ng-newsletter.com/posts/directives.html) 来更加深入指令的编写，或者查看 [ng-book](https://www.ng-book.com) 中有关指令的细节。

```javascript
angular.module('Grid')
.directive('grid', function() {
  return {
    restrict: 'A',
    require: 'ngModel',
    scope: {
      ngModel: '='
    },
    templateUrl: 'scripts/grid/grid.html'
  };
});
```

这个指令的主要功能是构建网格视图，所以我们不需要写任何自定义逻辑。

### grid.html

在指令的模板里面，我们会运行两个 `ngRepeat` 来显示网格和方块数组，还会（暂时）在循环中使用 `$index` 来跟踪。

```markup
<div id="game">
  <div class="grid-container">
    <div class="grid-cell"
      ng-repeat="cell in ngModel.grid track by $index">
      </div>
  </div>
  <div class="tile-container">
    <div tile
      ng-model='tile'
      ng-repeat='tile in ngModel.tiles track by $index'>
    </div>
</div>
</div>
```

第一个 `ng-repeat` 简单易懂，就是遍历了 grid 数组然后生成了 class 属性是 `grid-cell` 的单个空 div 元素。

在第二个 `ng-repeat` 中，我们会为每一个显示的元素生成一个名为 `tile` 的指令。这个 `tile` 指令会负责生成每一个方格元素的样子。我们很快就会去编写 `tile` 指令……

精明的读者可能会发现我们只适用一维数组来显示二维网格。当我们渲染视图的时候，我们只会得到一列“方格”，而不是一个网格。

要将它弄成网格，我们来深入 CSS 的编写。

<a name="scss"></a>
## 开始 SCSS

在这个项目中，我们会使用 SASS 的一个变种：scss。scss 除了是一个更强大的 CSS 外，还能动态地生成 CSS。

应用所有显示的元素的主要部分会使用 CSS 来完成，包括动画、布局和可视元素（方格的颜色等）。

要创建二维的棋盘，我们会用到 CSS3 关键字：`transform` 来处理每一个特定的方格的位置。

### CSS3 transform 属性

CSS3 transform 属性是一个可以让我们对元素进行 2D 或者 3D 上的移动、扭曲、旋转、缩放等操作（支持动画）的属性。用上了此属性，就可以直接将方块放在棋盘上然后剩下就只是应用合适的 `transform` 属性的事了。

例如，在下面的演示中，我们有一个宽 40px 高 40px 的盒子。

<div style="margin:40px;padding:40px 0;border-bottom:1px solid #333;">
  <div style="width:40px;height:40px;background-color:blue;"></div>
</div>

```css
.box {
  width:40px;
  height:40px;
  background-color: blue;
}
```

如果我们应用一个 `translateX(300px)` 的 `transform` 属性，就可以将盒子向右移动了 300px，就像下面所展示的：

<div style="margin:40px;padding:40px 0;border-bottom:1px solid #333;">
  <div style="width:40px;height:40px;background-color:blue;-webkit-transform:translateX(300px);transform:translateX(300px);"></div>
</div>

```css
.box.transformed {
  -webkit-transform: translateX(300px);
  transform: translateX(300px);
}
```

使用 translate 属性，我们只需应用 CSS 类就可以在棋盘上随便移动方块了。现在，精妙之处在于页面是多变的，我们如何能将类写得足够动态可以对应到网格上的正确位置。

这里就是 SCSS 大显身手的地方了。我们会创建几个变量（例如一行有多少个方格）然后在 SCSS 中结合数学来帮助我们计算。

来看一下计算棋盘上正确位置需要的变量：

```css
$width: 400px;          // The width of the whole board
$tile-count: 4;         // The number of tiles per row/column
$tile-padding: 15px;    // The padding between tiles
```

让 SCSS 用这些变量帮我们动态计算位置。首先算出每一个方格的大小。在 SCSS 中非常简单：

```css
$tile-size: ($width - $tile-padding * ($tile-count + 1)) / $tile-count;
```

现在我们就可以使用适当的宽和高来建立那个 `#game` 容器了。同时 `#game` 容器也会被设置成位置参照，它的子元素将会使用绝对定位。我们将 `.grid-container` 和 `tile-container` 放在 `#game` 容器内。

我们这里只展示跟 scss 有关的部分。剩下的代码可以在文章末尾的 github 地址上找到。

```css
#game {
  position: relative;
  width: $width;
  height: $width; // The gameboard is a square

  .grid-container {
    position: absolute;   // the grid is absolutely positioned
    z-index: 1;           // IMPORTANT to set the z-index for layering
    margin: 0 auto;       // center

    .grid-cell {
      width: $tile-size;              // set the cell width
      height: $tile-size;             // set the cell height
      margin-bottom: $tile-padding;   // the padding between lower cells
      margin-right: $tile-padding;    // the padding between the right cell
      // ...
    }
  }
  .tile-container {
    position: absolute;
    z-index: 2;

    .tile {
      width: $tile-size;        // tile width
      height: $tile-size;       // tile height
      // ...
    }
  }
}
```

注意为了让 `.tile-container` 放在 `.grid-container` 前面，我们__必须__要为 `.tile-container` 更高的 `z-index` 值。如果没有设置 `z-index` 值，浏览器会将两个元素放在同等高度，就不好看了。

做好这一步之后，现在我们来动态生成方块的位置。我们需要是一个 `.position-{x}-{y}` 类，用来应用到方块上，这样浏览器就会知道方块的位置然后将它放置好。既然我们是计算相对于网格容器的的 transformation 属性值，那就使用 `0,0` 作为第一个方块的初始位置。

我们对队列进行迭代，结合基于计算出来的期望偏移，动态生成每一个类。

```css
.tile {
  // ...
  // Dynamically create .position-#{x}-#{y} classes to mark
  // where each tile will be placed
  @for $x from 1 through $tile-count {
    @for $y from 1 through $tile-count {
      $zeroOffsetX: $x - 1;
      $zeroOFfsetY: $y - 1;
      $newX: ($tile-size) * ($zeroOffsetX) + ($tile-padding * $zeroOffsetX);
      $newY: ($tile-size) * ($zeroOffsetY) + ($tile-padding * $zeroOffsetY);

      &.position-#{$zeroOffsetX}-#{$zeroOffsetY} {
        -webkit-transform: translate($newX, $newY);
        transform: translate($newX, $newY);
      }
    }
  }
  // ...
}
```

> 注意我们不得不使用从 1 开始的偏移量来计算位置，而不是传统的从 0 开始。这是受 SASS 自身的限制所迫。不过我们可以使用将索引减 1 来解决。

现在我们写好了动态的 `.position-#{x}-#{y}`CSS 类，方块能够显示在页面上了。

![2-d grid
](http://www.ng-newsletter.com/images/2048/screen.png)

### 为不同的方块上色

注意到当有不同的方块出现的时候，各自都是不同颜色的。不同的颜色标识着不同方块所代表的值。如此一来玩家能看得出方格所处的状态。使用和我们迭代方格数目的时候同样的技巧来创建方格颜色方案。

要创建出颜色方案，我们首先要创建一个 SCSS 数组，包含有每一种需要用到的背景颜色。每一种颜色：

```css
$colors:  #EEE4DA, // 2
          #EAE0C8, // 4
          #F59563, // 8
          #3399ff, // 16
          #ffa333, // 32
          #cef030, // 64
          #E8D8CE, // 128
          #990303, // 256
          #6BA5DE, // 512
          #DCAD60, // 1024
          #B60022; // 2048
```

使用了 `$colors` 数组，我们只要迭代每一个颜色就能基于方块的值来动态创建一个类。也就是说，当一个方块的值是 2，我们会给它加上指定背景颜色是 `#EEE4DA` 的 `.tile-2` 类。与其给每个方块用硬编码，我们不如用 SCSS 的魔法来完成：

```css
@for $i from 1 through length($colors) {
  &.tile-#{power(2, $i)} .tile-inner {
    background: nth($colors, $i)
  }
}
```

当然，我们需要自己定义 `power()` 混合（译者注：mixin）。定义如下：

```css
@function power ($x, $n) {
  $ret: 1;

  @if $n >= 0 {
    @for $i from 1 through $n {
      $ret: $ret * $x;
    }
  } @else {
    @for $i from $n to 0 {
      $ret: $ret / $x;
    }
  }

  @return $ret;
}
```

<a name="the-tile-directive"></a>
## Tile 指令

SCSS 的繁琐工作完成了，我们可以回到 tile 指令的编写中了。通过动态的位置布局，让 CSS 按我们所设计的那样将方块摆放到位。

然而 `tile` 指令是一个自定义视图的容器，并不需要做很多事。我们需要的是它负责显示的单元格的访问权。除此以外，并不需要在指令内放任何功能。代码简单到足以自我描述：

```javascript
angular.module('Grid')
.directive('tile', function() {
  return {
    restrict: 'A',
    scope: {
      ngModel: '='
    },
    templateUrl: 'scripts/grid/tile.html'
  };
});
```

现在，`tile` 指令中有趣的地方就是我们如何动态的为网格布局。而模板会需要用到在隔离作用域（译者注：isolate scope）中的 `ngModel` 变量来处理好一切。

```markup
<div ng-if='ngModel' class="tile position-{{ ngModel.x }}-{{ ngModel.y }} tile-{{ ngModel.value }}">
  <div class="tile-inner">
    {{ ngModel.value }}
  </div>
</div>
```

我们几乎已经可以将这个基础的指令直接显示了。对于每一个有 `x` 和 `y` 坐标的方块而言，它们都会 * 自动 * 被赋予一个 `.position-#{x}-#{y}` 的类。浏览器会 * 自动 * 地将它们放到我们期待的位置。

这意味着我们的方块对象会需要一个 `x` 和 `y` 以及 `value` 让指令来使用。为此，对于每一个显示的方块，我们都需要创建一个新的对象。

### TileModel

与其创建一个 * 哑 * 对象，我们还不如创建一个比较智能的对象，既存储数据也能提供功能。

我们希望能使用 Angular 的依赖注入，因此创建一个服务来安置数据模型。我们在 `Grid` 模块中创建一个 `TileModel` 服务，因为跟游戏棋盘有关的操作时，它只需要使用底层的 `TileModel`。

使用 `.factory` 方法，我们能够简单地创建一个工厂函数。跟使用 `service()` 函数时传递的用以定义服务的函数会被默认为服务的构造函数不同的是，使用 `factory()` 函数会认为传递函数返回的对象才是服务。所以，只用 `factory()` 函数，我们可以将服务赋给任何对象以便在我们 Angular 应用中随时 * 注入 *。

在 `app/scripts/grid/grid.js` 文件中，我们可以创建 `TileModel` 工厂：

```javascript
angular.module('Grid')
.factory('TileModel', function() {
  var Tile = function(pos, val) {
    this.x = pos.x;
    this.y = pos.y;
    this.value = val || 2;
  };

  return Tile;
})
// ...
```

现在在我们 Angular 应用中的任何地方，我们都可以 * 注入 * 这个 `TileModel` 并想全局对象一样使用。非常方便不是吗？

> 不要忘了要为我们在 `TileModel` 中实现的任何功能写测试。

### 我们第一个网格

现在我们已经写好了 `TileModel` 了，我们可以开始在 `tiles` 数组中放入 `TileModel` 的实例了，然后发现它们 * 神奇地 * 出现在网格中正确的位置上。

让我们来试试在 `GridService` 中的 `tiles` 数组中加入一些方块：

```javascript
angular.module('Grid', [])
.factory('TileModel', function() {
  // ...
})
.service('GridService', function(TileModel) {
  this.tiles  = [];
  this.tiles.push(new TileModel({x: 1, y: 1}, 2));
  this.tiles.push(new TileModel({x: 1, y: 2}, 2));
  // ...
});
```

<a name="starting-the-game"></a>
## 棋盘已经准备好了

现在我们具备显示方块的能力了，还需要在 `GridService` 中实现准备棋盘的功能。当第一次载入页面的时候我们想创建一个空的棋盘。而同样的动作也应该发生在当用户在进行游戏的时候点击了 `New Game` 或者 `Try again` 的时候。

要清理棋盘，我们会在 `GridService` 中创建一个叫 `buildEmptyGameBoard()` 的函数。这个方法会负责将 `GridService` 中的 `grid` 数组和 `tiles` 数组填充 null。

在开始编写代码之前，我们先写出测试以保证 `buildEmptyGameBoard()` 函数的行为没问题。然而这个写的过程在上面已经讲过一遍了，所以不再讨论直接给出结果。写出来的测试大概就像下面那样：

```javascript
// In test/unit/grid/grid_spec.js
// ...
describe('.buildEmptyGameBoard', function() {
  var nullArr;

  beforeEach(function() {
    nullArr = [];
    for (var x = 0; x < 16; x++) {
      nullArr.push(null);
    }
  })
  it('should clear out the grid array with nulls', function() {
    var grid = [];
    for (var x = 0; x < 16; x++) {
      grid.push(x);
    }
    gridService.grid = grid;
    gridService.buildEmptyGameBoard();
    expect(gridService.grid).toEqual(nullArr);
  });
  it('should clear out the tiles array with nulls', function() {
    var tiles = [];
    for (var x = 0; x < 16; x++) {
      tiles.push(x);
    }
    gridService.tiles = tiles;
    gridService.buildEmptyGameBoard();
    expect(gridService.tiles).toEqual(nullArr);
  });
});
```

既然测试写好了，就可以实现 `buildEmptyGameBoard()` 函数的函数体了。

函数并不大，代码也足以自我说明。在 `app/scripts/grid/grid.js` 中

```javascript
.service('GridService', function(TileModel) {
  // ...
  this.buildEmptyGameBoard = function() {
    var self = this;
    // Initialize our grid
    for (var x = 0; x < service.size * service.size; x++) {
      this.grid[x] = null;
    }

    // Initialize our tile array
    // with a bunch of null objects
    this.forEach(function(x,y) {
      self.setCellAt({x:x,y:y}, null);
    });
  };
  // ...
```

上面的代码使用了一些足以自我描述出会做什么的辅助方法。部分我们会在整个项目中用到辅助函数如下列出，都是自我描述的：

```javascript
// Run a method for each element in the tiles array
this.forEach = function(cb) {
  var totalSize = this.size * this.size;
  for (var i = 0; i < totalSize; i++) {
    var pos = this._positionToCoordinates(i);
    cb(pos.x, pos.y, this.tiles[i]);
  }
};

// Set a cell at position
this.setCellAt = function(pos, tile) {
  if (this.withinGrid(pos)) {
    var xPos = this._coordinatesToPosition(pos);
    this.tiles[xPos] = tile;
  }
};

// Fetch a cell at a given position
this.getCellAt = function(pos) {
  if (this.withinGrid(pos)) {
    var x = this._coordinatesToPosition(pos);
    return this.tiles[x];
  } else {
    return null;
  }
};

// A small helper function to determine if a position is
// within the boundaries of our grid
this.withinGrid = function(cell) {
  return cell.x >= 0 && cell.x < this.size &&
          cell.y >= 0 && cell.y < this.size;
};
```

##### 究竟是什么？！？？

`this._positionToCoordinates()` 和 `this._coordinatesToPosition()` 这两个函数是什么？

回忆起之前我们已经讨论过了我们会使用一个一维数组来存储网格。在考虑到性能和复杂动画的处理，这是较为可取的。关于动画我们会稍后研究。我们暂且只能从使用单维数组表示多维数组的复杂性得到一点好处。

<a name="grid-theory"></a>
## 一维数组中的多维数组

如何使用单维数组表示多维数组？先来看看在棋盘上用每一个单元格的值来标出网格位置，不需要有颜色。在代码中，这个多维数组被分解成数组的数组。

![2-d grid](http://www.ng-newsletter.com/images/2048/grid-1.png) ![2-d grid](http://www.ng-newsletter.com/images/2048/grid-2.png)

看看每个单元格的位置，如果单维数组来看，可以看出一个关系来：

![2-d grid](http://www.ng-newsletter.com/images/2048/grid-3.png)

我们可以看到，在第一个单元格，`(0,0)` 单元格对应的数组下标是 `0`。第二个数组元素下标是 1 而单元格是 `(1.0)`。移动到下一行，单元格是 `(0,1)` 对应第四个数组元素而下标是 5 的数组元素是单元格 `(1,1)`。

据此可以推断出两个位置之间的等式关系。

####i = <span style="color:red">x</span> + <span style="color:blue">n</span>y

`i` 代表数组元素的下标，`x` 和 `y` 是多维数组中的位置坐标，`n` 是一行 / 列的单元格数。

我们在上面定义的两个辅助函数就是将数组下标转换为 x-y 坐标的过程和相反的转换过程。从理论上来说，使用 x-y 坐标处理单元格会比较简单，但是从功能上考虑我们却会在单维数组里存放方块。

```javascript
// Helper to convert x to x,y
this._positionToCoordinates = function(i) {
  var x = i % service.size,
      y = (i - x) / service.size;
  return {
    x: x,
    y: y
  };
};

// Helper to convert coordinates to position
this._coordinatesToPosition = function(pos) {
  return (pos.y * service.size) + pos.x;
};
```

### 初始化玩家位置

在游戏的一开始，我们想预先放几块。我们会为玩家随机在棋盘上挑选放方块的地方。

```javascript
.service('GridService', function(TileModel) {
  this.startingTileNumber = 2;
  // ...
  this.buildStartingPosition = function() {
    for (var x = 0; x < this.startingTileNumber; x++) {
      this.randomlyInsertNewTile();
    }
  };
  // ...
```

构建一开始的位置非常简单因为它只根据我们想放多少块方块来调用 `randomlyInsertNewTile()` 函数。`randomlyInsertNewTile()` 函数需要我们知道所有可以随机放置方块的位置。这个功能非常容易实现因为需要做的只是遍历单维数组的同时记录下还没有方块放置的位置。

```javascript
.service('GridService', function(TileModel) {
  // ...
  // Get all the available tiles
  this.availableCells = function() {
    var cells = [],
        self = this;

    this.forEach(function(x,y) {
      var foundTile = self.getCellAt({x:x, y:y});
      if (!foundTile) {
        cells.push({x:x,y:y});
      }
    });

    return cells;
  };
  // ...
```

有了一个棋盘上所有可用的坐标的列表，我们就可以简单地在数组中取随机位置。`randomAvailableCell()` 函数为我们处理。要实现函数的方法非常多。以下是我们在 2048 中实现的方法：

```javascript
.service('GridService', function(TileModel) {
  // ...
  this.randomAvailableCell = function() {
    var cells = this.availableCells();
    if (cells.length > 0) {
      return cells[Math.floor(Math.random() * cells.length)];
    }
  };
  // ...
```

从这里开始，我们可以简单地创建一个新的 TileModel 实例然后插入到我们的 `this.tiles` 数组中了。

```javascript
.service('GridService', function(TileModel) {
  // ...
  this.randomlyInsertNewTile = function() {
    var cell = this.randomAvailableCell(),
        tile = new TileModel(cell, 2);
    this.insertTile(tile);
  };

  // Add a tile to the tiles array
  this.insertTile = function(tile) {
    var pos = this._coordinatesToPosition(tile);
    this.tiles[pos] = tile;
  };

  // Remove a tile from the tiles array
  this.removeTile = function(pos) {
    var pos = this._coordinatesToPosition(tile);
    delete this.tiles[pos];
  }
  // ...
});
```

现在，得益于我们使用的 Angular，视图中的棋盘上，网格块会神奇地显示出方块来。

记住，明智的做法是接下来写测试来测试我们对于功能的假设实现。我们已经在为项目写测试的过程中发现了不少 bug，同样的事情你也会遇到的。

<a name="keyboard"></a>
## 键盘交互

很好，现在我们已经将方块放到棋盘上了。但一个不能玩的游戏有啥意思呢？是时候将注意力转移到加入交互上面去了。

> 为文章起见，我们只准备着眼在键盘的交互而没有考虑触控的交互。然而，加上触控支持并不应该太难，特别是我们只关注滑动动作，这个在 `ngTouch` 里有提供。我们将其留给你自己实现。

游戏本身使用方向键（或者 a,w,s,d 键）来玩。在游戏中，我们希望让玩家在页面上跟游戏简单地交互。而不是要求玩家将焦点移到在游戏棋盘元素上（或者同样问题下的其他元素）。玩家只需要让页面获得焦点就可以进行游戏了。

要做到这种交互，就要将事件监听绑定在 document 上。在 Angular 中，我们会 ` 绑定 ` 自己的事件监听在 Angular 提供的 `$ducoment` 服务上。要处理用户交互的创建，我们会将键盘事件绑定包裹在一个服务中。记住在页面中我们只需要一个键盘处理器，所以只要一个服务就可以了。

此外，我们也希望为用户的任何输入动作作出自定义的反应。使用了服务能自然地注入到应用中然后根据用户的输入来决定应用的反应。

首先，在 `app/scripts/Keyboard/keyboard.js` 文件中创建一个新的模块（因为我们正在做基于模块的开发的）`KeyBoard`（文件不存在就要先创建）。

```javascript
// app/scripts/keyboard/keyboard.js
angular.module('Keyboard', []);
```

正如创建任何新的 JavaScript 一样，我们需要在 `index.html` 中引用。现在 `<script>` 标签列表看起来是这样的：

```markup
  <!-- body -->
  <script src="scripts/app.js"></script>
  <script src="scripts/grid/grid.js"></script>
  <script src="scripts/grid/grid_directive.js"></script>
  <script src="scripts/grid/tile_directive.js"></script>
  <script src="scripts/keyboard/keyboard.js"></script>
  <script src="scripts/game/game.js"></script>
</body>
</html>
```

然后，因为新建一个模块，我们同样需要告诉自己的 Angular 模块在应用在需要依赖这个新模块：

```javascript
.module('twentyfourtyeightApp', ['Game', 'Grid', 'Keyboard'])
```

`Keyboard` 服务的实现思路，就是在 `$document` 上 ` 绑定 ` 了 `Keydown` 事件来捕获用户的键盘操作。而另一端，在我们的 angular 对象中，我们会注册一有用户操作就触发的处理函数。

来写代码。

```javascript
// app/scripts/keyboard/keyboard.js
angular.module('Keyboard', [])
.service('KeyboardService', function($document) {

  // Initialize the keyboard event binding
  this.init = function() {
  };

  // Bind event handlers to get called
  // when an event is fired
  this.keyEventHandlers = [];
  this.on = function(cb) {
  };
});
```

`init()` 函数会作为 `KeyboardService` 的开始，然后开始监听键盘事件。我们会过滤掉不感兴趣的键盘事件。

对于感兴趣的事件，我们会阻止它的默认行为然后将它交给我们的 `KeyEventHandlers`。

![2-d grid](http://www.ng-newsletter.com/images/2048/keyboard.png)

如何知道那些是我们感兴趣的呢？既然 * 感兴趣的 * 键盘操作是固定的，那么我们就去检查事件是否有其中一种键盘事件所激发。

一旦方向键被按下，document 会接收到一个包含被按下的按键的键码的事件。

我们可以为这些事件建立一个映射，然后查询捕获到的键盘动作是否在这个 * 感兴趣的 * 映射中。

```javascript
// app/scripts/keyboard/keyboard.js
angular.module('Keyboard', [])
.service('KeyboardService', function($document) {

  var UP    = 'up',
      RIGHT = 'right',
      DOWN  = 'down',
      LEFT  = 'left';

  var keyboardMap = {
    37: LEFT,
    38: UP,
    39: RIGHT,
    40: DOWN
  };

  // Initialize the keyboard event binding
  this.init = function() {
    var self = this;
    this.keyEventHandlers = [];
    $document.bind('keydown', function(evt) {
      var key = keyboardMap[evt.which];

      if (key) {
        // An interesting key was pressed
        evt.preventDefault();
        self._handleKeyEvent(key, evt);
      }
    });
  };
  // ...
});
```

每当一个存在于我们映射中的按键触发了 `keydown` 事件，`KeyboardService` 就会执行 `this._handleKeyEvent` 函数。

这个函数的整个职责就是调用每一个为按键注册了的处理函数。它就是简单地对处理函数数组进行迭代，使用按键和原事件组为参数来调用处理函数。

```javascript
// ...
this._handleKeyEvent = function(key, evt) {
  var callbacks = this.keyEventHandlers;
  if (!callbacks) {
    return;
  }

  evt.preventDefault();
  if (callbacks) {
    for (var x = 0; x < callbacks.length; x++) {
      var cb = callbacks[x];
      cb(key, evt);
    }
  }
};
// ...
```

另一方面，我们只需要将处理函数压入处理函数数组就可以了。

```javascript
// ...
this.on = function(cb) {
  this.keyEventHandlers.push(cb);
};
// ...
```

### 使用 Keyboard Service

现在我们已经有能力来监控用户的键盘事件，我们需要在应用开始运行的时候就监控。因为我们将它做成了一个服务，所以可以很简单地在主要的控制器中做这件事。

![2-d grid](http://www.ng-newsletter.com/images/2048/keyboard-sequence.png)

首先，我们需要调用 `init()` 函数来开始监听键盘。接着，我们会注册函数来告诉 `GameManager` 来调用 `move()` 函数。

回到 `GameController`，我们添加上 `newGame()` 函数和 `startGame()` 函数。`newGame()` 函数会告诉游戏服务创建一个新的游戏然后开始键盘事件处理。

来开始编码吧！我们需要将 `Keyboard` 模块作为一个新的模块依赖 * 注入 * 到应用中：

```javascript
angular.module('twentyfourtyeightApp', ['Game', 'Keyboard'])
// ...
```

然后就可以将 `KeyboardService` 注入到 `GameController` 中来开始跟用户交互了。首先，`newGame()` 方法：

```javascript
// ... (from above)
.controller('GameController', function(GameManager, KeyboardService) {
  this.game = GameManager;

  // Create a new game
  this.newGame = function() {
    KeyboardService.init();
    this.game.newGame();
    this.startGame();
  };

  // ...
```

我们还没有在 `GameManager` 中定义 `newGame()` 方法，但很快就会去填好内容。

一旦我们开始了新游戏，我们会调用 `startGame()`。`startGame()` 函数会准备好键盘服务的事件处理函数。

```javascript
.controller('GameController', function(GameManager, KeyboardService) {
  // ...
  this.startGame = function() {
    var self = this;
    KeyboardService.on(function(key) {
      self.game.move(key);
    });
  };

  // Create a new game on boot
  this.newGame();
});
```

<a name="start-button"></a>
## 按下那开始按钮

我们做了许多工作来达到开始游戏这么个目的。最后要实现的方法就是 `GameManager` 中的 `newGame()` 了，函数会：

1. 创建一个空的棋盘
1. 准备好开始的位置
1. 初始化游戏

其实我们已经在 `GridService` 中实现了这些逻辑，所以现在就差把它们连起来了！

在我们的 `app/scripts/game/game.js` 文件中，加入 `newGame()` 函数吧。此函数会重置游戏状态成应有的初始条件：

```javascript
angular.module('Game', [])
.service('GameManager', function(GridService) {
  // Create a new game
  this.newGame = function() {
    GridService.buildEmptyGameBoard();
    GridService.buildStartingPosition();
    this.reinit();
  };

  // Reset game state
  this.reinit = function() {
    this.gameOver = false;
    this.win = false;
    this.currentScore = 0;
    this.highScore = 0; // we'll come back to this
  };
});
```

在浏览器中载入页面，包含功能的网格就出来了…… 然而这个阶段还是非常无聊因为我们还没有定义任何移动的功能。

<a name="game-loop"></a>
## 动起来（游戏循环）

现在我们来深入游戏功能的实现。当用户按下任何一个方向键，我们会调用 `GridService` 中的 `move()` 函数（在 `GameController` 中写的）。

![non-playable version](http://www.ng-newsletter.com/images/2048/game-1.png)

要编写 `move()` 函数，我们需要定义游戏的约束。那就是说，我们需要定义游戏对于每一个给出的移动的操作。

对于每一步移动，我们要：

1. 确定用户按下的方向键指示的方向
1. 为棋盘上每一个方块找到所有最远的可能移动的位置。同时抓取下一个方块看是否能 * 合并 * 起来。
1. 对于每一个方块，我们想确定下一个位置是否存在一个等值的方块。
  - 如果下一个方块不存在，那么只将方块移动到可能的最远位置即可。（意味着这个最远位置就是棋盘的边缘。）
  - 如果下一个方块存在：
    + 且方块值不同的话，那么将方块放在最远位置（下一个方块就是当前方块的移动边界）。
    + 且方块值和当前方块相同的话，我们就找到一个可能的合并了。
      * 如果该方块已经是合并的结果了，则跳过并认定为已使用。
      * 如果方块还没合并过，那么则认为需要合并。

既然定义了功能，就可以制定出写 `move()` 函数的策略了。

```javascript
angular.module('Game', [])
.service('GameManager', function(GridService) {
  // ...
  this.move = function(key) {
    var self = this; // Hold a reference to the GameManager, for later
    // define move here
    if (self.win) { return false; }
  };
  // ...
});
```

移动是有限制条件的：如果游戏已经结束或者游戏循环因为某种原因而终止了，那么就只需要返回并继续。

接下来我们需要在网格上走一下来找出所有可供移动的地方。而因为掌握空方格的位置其实是网格的职责，因此我们会在 `GridService` 中写一个新的函数来帮助我们找出这些可能会经过的方格。

![non-playable version](http://www.ng-newsletter.com/images/2048/grid-vectors.gif)

我们通过提取玩家按键指示的 * 向量 * 来决定方向。例如，如果玩家按下了右键头键，那么就是想移动到 `x` 值 * 更大的 * 方格上。

如果玩家按了上箭头，那么玩家就是想将方块移动到 `y` 值 * 更小的 * 方格上。我们可以使用一个 JavaScript 对象将向量和玩家按键映射起来（从 `KeyboardService` 中得到的按键），就像这样：

```javascript
// In our `GridService` app/scripts/grid/grid.js
var vectors = {
  'left': { x: -1, y: 0 },
  'right': { x: 1, y: 0 },
  'up': { x: 0, y: -1 },
  'down': { x: 0, y: 1 }
};
```

现在我们就可以简单地迭代每一个可能的位置，并使用向量来控制迭代的方向：

```javascript
.service('GridService', function(TileModel) {
  // ...
  this.traversalDirections = function(key) {
    var vector = vectors[key];
    var positions = {x: [], y: []};
    for (var x = 0; x < this.size; x++) {
      positions.x.push(x);
      positions.y.push(x);
    }
    // Reorder if we're going right
    if (vector.x > 0) {
      positions.x = positions.x.reverse();
    }
    // Reorder the y positions if we're going down
    if (vector.y > 0) {
      positions.y = positions.y.reverse();
    }
    return positions;
  };
  // ...
```

现在新的函数 `traversalDirections()` 定义好了，在 `move()` 函数中就可以在可能的移动上进行迭代了。回到 `GameMabager`，我们会根据这些可能的位置在网格上走动。

```javascript
// ...
this.move = function(key) {
  var self = this;
  // define move here
  if (self.win) { return false; }
  var positions = GridService.traversalDirections(key);

  positions.x.forEach(function(x) {
    positions.y.forEach(function(y) {
      // For every position
    });
  });
};
// ...
```

在位置的循环中，我们会对可供移动的位置进行迭代同时查找存在的方块。从这里开始，我们将编写函数的第二部分，找出从该方块出发能到达的所有方格。

```javascript
// ...
// For every position
// save the tile's original position
var originalPosition = {x:x,y:y};
var tile = GridService.getCellAt(originalPosition);

if (tile) {
  // if we have a tile here
  var cell = GridService.calculateNextPosition(tile, key);
  // ...
}
```

![non-playable version](http://www.ng-newsletter.com/images/2048/next-process.gif)

如果我们确实在该方格内找到了方块，就会开始查看该方格最远能到哪里。先在网格上找到下一个位置，检查这个方格是否在棋盘内和方格是否为空。

如果该方格是空的 ** 而且 ** 在棋盘内，那么继续取得下一个方格然后执行一样的检查。

如果两个条件中任意一个不满足，那么要不我们到达了棋盘的边界，要不我们找到了下一个方块。我们会保存前一个位置（译者注：原文为 the next position，翻译为下一个位置。但根据描述和下文的代码此处应该为前一个位置。）同时抓取下一个方格（不管是否存在下一个方格）。

而这个过程是对网格进行操作，于是就这个函数放在 `GridService`：

```javascript
// in GridService
// ...
this.calculateNextPosition = function(cell, key) {
  var vector = vectors[key];
  var previous;

  do {
    previous = cell;
    cell = {
      x: previous.x + vector.x,
      y: previous.y + vector.y
    };
  } while (this.withinGrid(cell) && this.cellAvailable(cell));

  return {
    newPosition: previous,
    next: this.getCellAt(cell)
  };
};
```

现在我们可以计算下一个有可能放得下我们的方块的地方，接着就是检查是否有合并的可能。

一个 * 合并 * 的定义是两个相同值的方块碰撞在一起。我们会检查 `next` 的位置上是否有相同值的方块并且还之前没有被 * 合并 * 过。

```javascript
// ...
// For every position
// save the tile's original position
var originalPosition = {x:x,y:y};
var tile = GridService.getCellAt(originalPosition);

if (tile) {
  // if we have a tile here
  var cell = GridService.calculateNextPosition(tile, key),
      next = cell.next;

  if (next &&
      next.value === tile.value &&
      !next.merged) {
    // Handle merged
  } else {
    // Handle moving tile
  }
  // ...
}
```

如果这个所谓的下一个位置并 * 不 * 符合上面的条件，那么我们就会将方块从当前的位置移动到这个下一个位置（else 语句）。

这是更相比之下更容易处理的条件，所需要做的就是将方块移动到 newPosition 位置。

```javascript
// ...
if (next &&
    next.value === tile.value &&
    !next.merged) {
  // Handle merged
} else {
  GridService.moveTile(tile, cell.newPosition);
}
```

### 移动方块

就像你大概猜测那样，`moveTile()` 函数最好就是定义在 `GridService` 中。

移动一个方块就是简单地更新一下方块在数组中的位置和更新 `TileModel` 而已。

就像我们定义的那样，函数里面有两个目的不同的操作。当我们：

##### 在数组中移动方块的时候

数组 `GridService.tiles`（译者注：原文为 GridService）为后端映射了方块的位置。数组中方块的位置 * 没有 * 和网格中方块的位置绑定。

##### 更新 TileModel 中的位置的时候

我们要为前端的 CSS 更新坐标来放置方块。

简而言之：为了在后端能跟踪方块们，我们需要更新 `GridService` 中的 `this.tiles` 数组 * 同时 * 更新方块对象的位置。

于是 `moveTile()` 就变成了一个简单的两步操作：

```javascript
// GridService
// ...
this.moveTile = function(tile, newPosition) {
  var oldPos = {
    x: tile.x,
    y: tile.y
  };

  // Update array location
  this.setCellAt(oldPos, null);
  this.setCellAt(newPosition, tile);
  // Update tile model
  tile.updatePosition(newPosition);
};
```

现在我们需要定义我们的 `tile.updatePosition()` 方法。这个方法所做的就像它字面上的那样，就是简单地更新模型自己的 `x` 和 `y` 坐标。

```javascript
.factory('TileModel', function() {
  // ...

  Tile.prototype.updatePosition = function(newPos) {
    this.x = newPos.x;
    this.y = newPos.y;
  };
  // ...
});
```

回到 `GridService` 中，我们已经可以只是调用 `moveTile()` 来同时更新 `GridService.tiles` 数组和方块自己的位置了。

### 合并一个方块

既然我们已经处理了 * 比较简单 * 的情况了，那么合并一个方块就是我们下一个需要攻克的问题。合并定义如下：

* 当一个方块在下一个可移动的方格上遇到相同值的方块的时候就需要合并。*

当一个方块被合并出来，棋盘就算需要改变，同样当前得分和最高得分也需要更新（如果需要的话）。

合并需要几个步骤：

1. 在最后的位置上添加一个新的带合并值的方块
1. 移除旧方块
1. 更新游戏得分
1. 检查游戏是否结束

拆解后，合并操作很简单。

```javascript
// ...
var hasWon = false;
// ...
if (next &&
    next.value === tile.value &&
    !next.merged) {
  // Handle merged
  var newValue = tile.value * 2;
  // Create a new tile
  var mergedTile = GridService.newTile(tile, newValue);
  mergedTile.merged = [tile, cell.next];

  // Insert the new tile
  GridService.insertTile(mergedTile);
  // Remove the old tile
  GridService.removeTile(tile);
  // Move the location of the mergedTile into the next position
  GridService.moveTile(merged, next);
  // Update the score of the game
  self.updateScore(self.currentScore + newValue);
  // Check for the winning value
  if (merged.value >= self.winningValue) {
    hasWon = true;
  }
} else {
// ...
```

我们只想支持一行只有一个方块移动的效果（就是说如果一行里面有两个可以合并的情况，则只会合并一个），因此不得不跟踪 ` 合并了的 ` 方块。通过将 `.merged` 标志设置成随便什么东西而不是 `undefined` 就可以做到。

在结束这个函数的编写之前，还需要解释一下这里用到的我们还没有定义的函数。

`GridService.newTile()` 函数就是简单地创建 `TileModel` 对象。合并操作就放在包含创建新方块函数的｀GridService｀中：

```javscript
// GridService
this.newTile = function(pos, value) {
  return new TileModel(pos, value);
};
// ...
```

我们一会再回来叙述 `self.updateScore()`。现在暂时只需要知道它更新游戏得分就可以了（就像函数名所表明的那样）。

### 移动了方块之后

我们只希望在一次有效的方块移动之后才增加新的方块，因此需要检查一下是否真的有任何一个方块移动了。

```javascript
var hasMoved = false;
// ...
  hasMoved = true; // we moved with a merge
} else {
  GridService.moveTile(tile, cell.newPosition);
}

if (!GridService.samePositions(originalPos, cell.newPosition)) {
  hasMoved = true;
}
// ...
```

当所有的方块都已经移动过了（或尝试移动过），我们就继续检查玩家是否赢了。如果是，那么实际上我们就要设置 `self.win` 这个标志了。

> 当有方块碰撞的时候我们会移动方块，所以在合并的条件下，我们只简单地设置 `hasMoved` 为 true。

最后，我们要检查一下棋盘上是否有任何的方块移动。如果有，则：

1. 给棋盘添加一个新的方块
1. 检查一下有没有必要展示游戏结束界面

```javascript
if (!GridService.samePositions(originalPos, cell.newPosition)) {
  hasMoved = true;
}

if (hasMoved) {
  GridService.randomlyInsertNewTile();

  if (self.win || !self.movesAvailable()) {
    self.gameOver = true;
  }
}
// ...
```

### 重置方块

在运行任何主游戏程序前，我们要重置每一个方块以便不再跟踪其合并的状态。详细来说，就是每一次移动之后，都要清理所有记录以便让所有方块能再次被移动。因此在执行移动的循环体开头，我们会调用：

```javascript
GridService.prepareTiles();
```

`GridService` 中的 `prepareTiles()` 函数只是简单地迭代每一个方块然后重置其状态而已：

```javascript
this.prepareTiles = function() {
  this.forEach(function(x,y,tile) {
    if (tile) {
      tile.reset();
    }
  });
};
```

<a name="keeping-score"></a>
## 计算得分

回头来看看 `updateScore()` 方法，游戏本身需要记录两个得分：

1. 当前游戏的得分
1. 玩家的最高得分

`currentScore` 只是一个在每一次游戏的时候保存在内存中的变量，因此无需特殊对待。

然而 `highScore` 则是一个贯穿每一次的游戏的变量。我们有几个方法来保存，比如 localstorage，cookies，或者两者结合。

因为 cookies 是两个方法中最简单而且跨浏览器安全，我们就继续使用 cookies 来存储这个 highScore。

Angular 中使用 `angular-cookies` 模块是管理 cookies 的最简单的方法了。

要使用这个模块，可以到 [angularjs.org](http://angularjs.org) 上下载或者使用包管理器例如 bower 来安装。

```bash
$ bower install --save angular-cookies
```

照旧，我们要在 `index.html` 中引用这个脚本然后在应用中将 `ngCookies` 设置成模块级别的依赖。

像这样更新一下 `app/index.html`：

```markup
<script src="bower_components/angular-cookies/angular-cookies.js"></script>
```

然后添加 `ngCookies` 作为模块依赖（在 `Game` 模块中，我们引用 cookies 的地方）：

```javascript
angular.module('Game', ['Grid', 'ngCookies'])
// ...
```

有了 `ngCookies` 作为依赖，我们就可以将 `$cookieStore` 服务 * 注入 * 到 `GameManagere` 服务中。现在可以在浏览器中对 cookies 进行读写了。

例如，要读取玩家的最高得分，我们会写一个函数从用户的 cookie 中取来：

```javascript
this.getHighScore = function() {
  return parseInt($cookieStore.get('highScore')) || 0;
}
```

回到 `GameManager` 类中的 `updateScore()` 函数，我们开始编写更新当前得分的代码。如果得分比之前记录的最高得分高，那么就更新 cookie 中的最高得分。

```javascript
this.updateScore = function(newScore) {
  this.currentScore = newScore;
  if (this.currentScore > this.getHighScore()) {
    this.highScore = newScore;
    // Set on the cookie
    $cookieStore.put('highScore', newScopre);
  }
};
```

### track by 之怒

既然我们已经将方块显示出来了，一个 bug 也同样出现了，那就是一些有奇怪行为的方块复制品冒出来。进一步来说，就是方块可能会在不应该出现的地方出现。

原因是 Angular 通过基于一个唯一的标识来获知 `titles` 数组里面的有什么方块。而我们把这个唯一的标识在视图中设定为方块在数组中的 `$index`（也就是数组中的位置）。然而我们在数组中将方块移来移去，`$index` 不再起到唯一标识的作用。我们需要另外的监测方法。

```markup
<div id="game">
  <!-- grid-container -->
  <div class="tile-container">
    <div tile
      ng-model='tile'
      ng-repeat='tile in ngModel.tiles track by $index'></div>
  </div>
</div>
```

与其依靠数组来标识方块的位置，我们不如使用方块自己唯一的 uuid 来跟踪。自己创建唯一标识能保证 angular 将方块数组中的每一个方块看成是唯一的对象。只要唯一的 uuid 没有变，那么 angular 就会根据这个标识来将方块识别为独立的对象。

创建新实例的时候使用 `TileModel`，我们能非常轻松地为方块实现出唯一标识。我们还能以自己的方式来创建唯一标识。

> 只要对于每一个创建的 `TileModel` 实例是唯一的，那么怎么创建这个唯一 id 的方法并无影响。

要生成这个唯一的 id，我们跳到 [StackOverflow](http://stackoverflow.com/questions/105034/how-to-create-a-guid-uuid-in-javascript) 上找一个 [遵循 rfc4122](http://www.ietf.org/rfc/rfc4122.txt) 的全球唯一标识生成器，然后将其打包成一个工厂，提供一个函数：`next()`：

```javascript
.factory('GenerateUniqueId', function() {
  var generateUid = function() {
    // http://www.ietf.org/rfc/rfc4122.txt
    var d = new Date().getTime();
    var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
      var r = (d + Math.random()*16)%16 | 0;
      d = Math.floor(d/16);
      return (c === 'x' ? r : (r&0x7|0x8)).toString(16);
    });
    return uuid;
  };
  return {
    next: function() { return generateUid(); }
  };
})
```

要 * 使用 * 工厂 `GenerateUniqueId`，就要将它注入然后调用 `GenerateUniqueId.next()` 来产生一个新的 uuid。回到 `TileModel` 中，我们已经可以为实例生成一个唯一的 id 了（在构造函数中）。

```javascript
// In app/scripts/grid/grid.js
// ...
.factory('TileModel', function(GenerateUniqueId) {
  var Tile = function(pos, val) {
    this.x      = pos.x;
    this.y      = pos.y;
    this.value  = val || 2;
    // Generate a unique id for this tile
    this.id = GenerateUniqueId.next();
    this.merged = null;
  };
  // ...
});
```

现在每一个方块都已经有了唯一的标识了，于是就可以告诉 Angular 使用 id 而不是 `$index` 来追踪方块了。

```markup
<!-- ... -->
<div tile
      ng-model='tile'
      ng-repeat='tile in ngModel.tiles track by $id(tile.id)'></div>
<!-- ... -->
```

这样做会出现一个问题。我们使用 nulls 来初始化数组（显式地）和 nulls 来重置数组（而不是对数组排序或者调整长度），angular 会不顾一切地试图将 nulls 看作对象。但是 null 值并不包含有唯一标识，于是就会引起浏览器抛异常，并且不知道怎么处理复制出来的对象。

所以，我们要使用内置的 angular 工具来追踪唯一标识或者数组中的 `$index` 位置（因为 null 对象在方格内只有一个所以可以通过数组的位置来追踪）。我们可以向如下那样修改一下 grid_directive 中的视图来解决 null 对象：

```javascript
<!-- ... -->
<div tile
      ng-model='tile'
      ng-repeat='tile in ngModel.tiles track by $id(tile.id || $index)'></div>
<!-- ... -->
```

> 这个问题也能通过实现不同的底层数据结构来解决，例如使用迭代器来查看每一个 `TileModel` 的位置而不是依靠数组下标或者每次改变数组后重新调整（或在 `$digest()` 中调整）。为了保持简单和清晰，我们使用数组来实现因为这种实现方法只需要处理上文的副作用就可以了。

<a name="game-over"></a>
## 我们赢了?!?? 游戏结束

当我们在原来 2048 游戏中输了的时候，一个 * 游戏结束 * 界面会滑入，让我们重新开始游戏或者在 twitter 上 follow 游戏作者。这不仅是游戏中一个酷酷的效果，也是一个中断游戏的好方法。

使用基本的 angular 技术也能做到这个。我们已经在 `GameManager` 使用了变量 `gameOver` 来确定游戏什么时候结束。可以直接就用一个 `<div>` 元素来装着游戏结束界面，然后使用绝对定位覆盖在棋盘上。这种技术（和 Angular）的神奇在于实现起来没有任何的花招：

就是简单地创建一个包含游戏结束或者玩家获胜的信息的 `<div>` 元素，根据游戏的状态来选择显示。比如，游戏结束界面看起来可以像这样：

```markup
<!-- ... -->
<div id="game-container">
  <div grid ng-model='ctrl.game' class="row"></div>
    <div id="game-over"
        ng-if="ctrl.game.gameOver"
        class="row game-overlay">
      Game over
      <div class="lower">
        <a class="retry-button" ng-click='ctrl.newGame()'>Try again</a>
      </div>
    </div>
  <!-- ... -->
```

困难的部分是处理样式 / CSS。为效率起见，我们只是将元素设置成绝对定位在网格之上，让浏览器来决定真正的位置。这里附上 * 相关的 * 一部分 css（提醒一下，完整 CSS 在下面的 gtihub 地址中有）：

```css
.game-overlay {
  width: $width;
  height: $width;
  background-color: rgba(255, 255, 255, 0.47);
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  text-align: center;
  padding-top: 35%;
  overflow: hidden;
  box-sizing: border-box;

  .lower {
    display: block;
    margin-top: 29px;
    font-size: 16px;
  }
}
```

> 我们可以使用完全相同的技术来做获胜界面，同样创建一个代表获胜的 `.game-overlay` 元素即可。

<a name="running-the-animation"></a>
## 动画

原 2048 游戏中其中一个令人印象深刻的地方是方块似乎会魔术般地从一个网格滑到下一个网格，另外游戏结束 / 获胜界面的显示显得很自然。因为使用 Angular，我们能做到 * 几乎一样的效果 *（感谢 CSS）。

实际上，我们做出来的游戏能够容易地实现诸如滑动，出现，显现等的动画效果。我们几乎不会碰到 JavaScript（只需一点点）就可以实现这些效果。

### CSS 位置动画（也就是添加滑动的方块）

因为我们通过 CSS 设置类 `position-[x]-[y]` 来定位方块，当为方块设置新位置的时候，DOM 元素会加上类 `position-[newX]-[newY]` 并移除类 `position-[oldX]-[oldY]`。在这种情况下，我们在 `.tile` 类上定义一个 CSS 变换来实现 CSS 类上自带滑动效果。

相关的 SCSS：

```css
.tile {
  @include border-radius($tile-radius);
  @include transition($transition-time ease-in-out);
  -webkit-transition-property: -webkit-transform;
  -moz-transition-property: -moz-transform;
  transition-property: transform;
  z-index: 2;
}
```

CSS 变换定义好后，现在方块就会在网格之间滑动了（对，就是 * 那么简单 *）。

### 游戏结束界面动画

如果想在动画上取得更多的效果，可以使用 `ngAnimate` 模块来做。此模块本身配合 angular 一起就是开箱即用了。

在使用前，同样需要安装 `ngAnimate` 模块。在 [angularjs.org](http://angularjs.org) 上下载或者使用包管理器（例如 bower）来安装。

```bash
$ bower install --save angular-animate
```

同样，我们接着就需要在 HTML 中引用以便浏览器加载。修改 `index.html` 来引用 `angular-animate.js` 文件。

```markup
<script src="bower_components/angular-animate/angular-animate.js"></script>
```

最后，就像其他 angular 模块一样，我们要告诉 angular 我们的应用依赖什么模块来运行。在应用的依赖数组中加入：

```javascript
angular
.module('twentyfourtyeightApp', ['Game', 'Grid', 'Keyboard', 'ngAnimate', 'ngCookies'])
// ...
```

### ngAnimate

虽然对 ngAnimate 的深度探讨超出本文范围（看 [ng-book](https://www.ng-book.com) 来深入了解其机制），但是我们还是粗浅了解一下其工作机制以便在应用里实现动画。

引入了 `ngAnimate` 作为模块级别依赖之后，任何时候 angular 为相关的（对于我们的应用而言）指令添加一个新对象的时候，它也会增添上一个 CSS 类（免费）。我们可以利用这些类来给游戏中的不同组件赋予 CSS 动画：

<table>
  <tr>
    <th>Directive</th>
    <th>Added class</th>
    <th>Leaving class</th>
  </tr>
  <tr>
    <td>ng-repeat</td>
    <td>ng-enter</td>
    <td>ng-leave</td>
  </tr>
  <tr>
    <td>ng-if</td>
    <td>ng-enter</td>
    <td>ng-leave</td>
  </tr>
  <tr>
    <td>ng-class</td>
    <td>[className]-add</td>
    <td>[className]-remove</td>
  </tr>
</table>

当一个元素被添加进 `ng-repeat` 的作用域，新的 DOM 元素会被自动添加上 CSS 类 `ng-enter`。然后，当它真正地添加到视图上后，就会被添加上 CSS 类 `ng-enter-active`。这个机制很重要因为它让我们能够在 CSS 类 `ng-enter` 里设定动画的样子和在 CSS 类 `ng-enter-active` 里设定动画的样式。当元素在 `ng-repeat` 迭代器中被移除的时候类 `ng-leave` 也是如此的工作机制。

当 DOM 元素上一个新的 CSS 类被添加（或被移除），相应的 `[classname]-add` 和 `[classname]-add-active` 也会添加到 DOM 元素上。同理，也可以在相应的类里设定 CSS 动画。

### 游戏结束界面动画

我们能使用类 `ng-enter` 来让游戏结束界面和获胜界面动起来了。记住，类 `.game-overlay` 是使用 `ng-if` 指令来实现隐藏和显示的。当 `ng-if` 的条件变化了，`ngAnimate` 会在等式值为真的时候添加上 `.ng-enter` 和 `.ng-enter-active`（或者移除元素时添加 `.ng-leave` 和 `.ng-leave-active`）。

我们会在类 `.ng-enter` 中设定好动画，然后在类 `.ng-enter-active` 中激活。相关的 SCSS：

```css
.game-overlay {
  // ...
  &.ng-enter {
    @include transition(all 1000ms ease-in);
    @include transform(translate(0, 100%));
    opacity: 0;
  }
  &.ng-enter-active {
    @include transform(translate(0, 0));
    opacity: 1;
  }
  // ...
}
```

所有的 SCSS 在文章底部的 github 连接中可以看到。

<a name="customizing-size"></a>
## 定制位置

假设我们想使用不同的棋盘大小。例如，原 2048 是 4x4 的。如果我们想要 3x3 或者 6x6 呢？不用变动太多的代码我们就能轻松实现。

棋盘本身是通过 SCSS 来创建和定位的，而网格又是通过 `GridService` 来管理的。所以我们在这两个地方修改一下以便能自定义棋盘。

### 动态 CSS

好吧实际上我们并不是打算弄动态 CSS，但是我们可以创建更加多实际会用得上的 CSS。与其使用单个 `#game` 标签，我们可以实时创建可以动态设置网格的 DOM 元素标签。也就是说，我们将 3x3 的棋盘版本嵌套在 ID 是 `#game-3` 的 DOM 元素下，将 6x6 的棋盘版本嵌套在 id 标签是 `#game-6` 的元素下。

可以在原本已经是动态的 SCSS 中编写出一个[混合](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#mixins)。就是很简单地找到 css ID 标签 `#game` 然后将其包裹进一个混合。例如：

```css
@mixin game-board($tile-count: 4) {
  $tile-size: ($width - $tile-padding * ($tile-count + 1)) / $tile-count;
  #game-#{$tile-count} {
    position: relative;
    padding: $tile-padding;
    cursor: default;
    background: #bbaaa0;
    // ...
}
```

现在我们可以引用 `game-board` 混合来动态创建一个包含有不同棋盘版本的样式表了，棋盘的版本都各自独立在其 `#game-[n]` 标签下。

要做出这样不同的版本，我们只需要遍历所有的棋盘大小然后调用上面的混合就可以了。

```css
$min-tile-count: 3;       // lowest tile count
$max-tile-count: 6;       // highest tile count
@for $i from $min-tile-count through $max-tile-count {
  @include game-board($i);
}
```

### 动态的 GridService

现在已经编写好了应付不同大小棋盘的 CSS 了，我们还需要修改 `GridService` 好让启动应用的时候能设置网格的大小。

Angular 让这变得十分简单。首先，我们需要将 `GridService` 变成 `provider`，而不是一个直接的 `service`。如果你不清楚服务（译者注：service）和提供者（译者注：provider）之间的不同，看 [ng-book](https://www.ng-book.com) 作深入了解。简单来说，一个提供者能够让我们在运行之前对其进行配置。

此外，我们也需要将提供者中的构造函数修改为 `$get` 方法：

```javascript
.provider('GridService', function() {
  this.size = 4; // Default size
  this.setSize = function(sz) {
    this.size = sz ? sz : 0;
  };

  var service = this;

  this.$get = function(TileModel) {
    // ...
```

提供者中任何不在 `$get` 方法中的方法都能在应用的 `.config()` 函数中访问得到。`$get()` 中的所有东西不能被 `.config()` 方法访问，而能在运行的时候被应用访问。

实现动态棋盘大小的工作就这么多。现在我们试着做一个 6x6 的棋盘而不是 4x4 的棋盘。在 app 模块的 `.config()` 函数中，我们叫来 `GridServiceProvider` 来设置大小：

```javascript
angular
.module('twentyfourtyeightApp', ['Game', 'Grid', 'Keyboard', 'ngAnimate', 'ngCookies'])
.config(function(GridServiceProvider) {
  GridServiceProvider.setSize(4);
})
```

> Angular 在创建一个提供者的时候，会自动生成一个仅供配置时使用的模块，我们使用名字：[serviceName]Provider 来实现注入。

<a name="demo"></a>
## 演示 demo

完整的 demo 在这里：[http://ng2048.github.io/](http://ng2048.github.io/)。

## 总结

唷！我们希望你已经在愉快地使用 Angular 来编写这个 2048 游戏了。博文中应该已经覆盖了大部分的过程了。如果你觉得不错，可以在下面留下评论。如果你对继续学习 Angular 有兴趣，务必去看看我们的书[Complete Book on AngularJS](https://www.ng-book.com/)。这是唯一一本会不断更新 AngularJS 知识的书，并且包括了在 AngularJS 中所有你需要了解的东西。

[在 HackerNews 上讨论](https://news.ycombinator.com/item?id=7554348)

## Thanks

非常感谢 [Gabriele Cirulli](http://gabrielecirulli.com/) 编写出了妙极的（和让人上瘾的）2048，同样感谢他对此文的启发。文中的很多主意都是从原游戏中搜集、提炼，用以阐明如何使用 Angular 来编写。

## 完整代码

游戏的完整代码在 Github 上，地址是 http://d.pr/pNtX。要在本地编译游戏，只需要复制代码然后运行：

```bash
$ npm install
$ bower install
$ grunt serve
```

## 问题与解决方法

如果你使用不了 npm install，保证你安装了最新的 node.js 和 npm。

这个版本库在 node v0.10.26 和 npm 1.4.3 上测试。

以下是一个安装最新版本的 node 和 node 版本管理器 `n` 的方法：

```bash
$ sudo npm cache clean -f
$ sudo npm install -g n
$ sudo n stable
```
