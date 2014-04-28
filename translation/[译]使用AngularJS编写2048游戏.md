[译]使用AngularJS编写2048游戏
================================
原文[http://www.ng-newsletter.com/posts/building-2048-in-angularjs.html](http://www.ng-newsletter.com/posts/building-2048-in-angularjs.html)，考虑到排版和图片，内容稍有修改，对文章表达的意思并无影响。极其希望得到翻译方面的意见和建议。

我们最近经常被问到的其中一个问题是作为一个框架，Angular在什么情况下使用并不太适合。我们的标准答案通常是编写游戏的时候，因为Angular有它自己的事件处理循环（$digest循环）而游戏通常要求非常多的底层DOM操作。其实这个答案并不准确因为Angular其实是能支持多数游戏的编写的。即使要求大量的DOM操作的游戏，Angular也能胜任其静态部分，例如高分记录和游戏菜单。

If you are anything like me (and the rest of the tech industry), you may be addicted to the popular 2048 game. The objective of the game is to get to the 2048’s tile by matching like-value tiles.
如果你是像我那样的人（），你有可能会喜欢玩那个流行的[2048](http://gabrielecirulli.github.io/2048)游戏。游戏的目标是通过合并相同数值的方块来得到数值是2048的方块。

![Injection](http://www.ng-newsletter.com/images/2048/game.gif)

[在HackerNews上讨论](https://news.ycombinator.com/item?id=7554348)

在今天的博文中，我们准备使用AngularJS来仿制这个游戏，而且是从头到尾完整地解释整个app的编写流程。这个app是一个相当复杂的应用，我们也希望利用这篇博文来展示如何编写复杂的AngularJS应用。

此Angular版应用的[demo](http://d.pr/SnWD)。

来让我们开始吧！

> TL;DR: the entire source for this application is available at on github in the link at the end of the article.

##目录

1. [计划](#planning)
1. [模块结构](#modular)
1. [GameController](#game-controller)
1. [测试，测试，测试](#tdd)
1. [建造游戏网格](#build-grid)
1. [SCSS to the rescue](#scss)
1. [The tile directive](#the-tile-directive)
1. [The Boardgame](#starting-the-game)
1. [Grid theory](#grid-theory)
1. [Gameplay (keyboard)](#keyboard)
1. [Pressing the start button](#start-button)
1. [The game loop](#game-loop)
1. [Keeping score](#keeping-score)
1. [Game over and win screens](#game-over)
1. [Animation](#running-the-animation)
1. [Customization](#customizing-size)
1. [Demo](#demo)

<a name="planning"></a>
##第一步：计划

![Minification](http://www.ng-newsletter.com/images/2048/3d-board.png)

我们首先想做的是对将要编写的应用进行高层次设计。如果是仿制一个应用或是从零开始，我们都会这样做，不论应用有多大。

审视一下，我们可以看到游戏是有一块游戏棋盘，上面有一些方块。每一个方块的位置就是数值方块的位置。我们可以利用这一个事实，使用CSS3而不是javascript来摆放方块，后者需要知道方块摆放的位置。当摆放方块的时候，我们只需要保证方块覆盖在合适的位置上就可以了。

CSS3的使用让我们不但能够免于在CSS上搞动画的工作，而且能使用标准的AngularJS行为（译者注：AngularJS behavior）来跟踪游戏棋盘、方块和游戏逻辑的状态。

因为我们只有一个页面，所以只需要一个controller来管理页面。

既然在应用的运行期间只有一个游戏棋盘，我们就另外创建单一一个`GridService`的service实例来保存所有的网格逻辑。service都是单例对象，适合用来存储网格。我们会使用`GridService`来放置和移动方块、寻找可供移动的位置和管理网格。

我们将游戏的逻辑和运行存储在另外一个叫`GameManager`的service中。`GameManager`负责管理游戏状态、处理移动和维护得分（包括当前得分和最高得分）。

最后，还需要一个组件来控制键盘。我们将使用一个名为`KeyboardService`的service（只需要一个键盘动作的处理）。我们会在这篇文章中实现桌面版的处理，然而我们也可以重用同一个service来处理触屏动作使其能在移动设备上使用。

###开始编写应用

要开始编写，我们先创建一个基本的应用（我们使用[yeoman](http://yeoman.io) angular generator来生成应用的结构，但这是非必要的。我们只是将其作为一个起始点，但很快就会做出分支）。我们新建一个包含整个应用的目录，然后再在`app/`目录旁边建一个`test/`目录。

![Minification](http://www.ng-newsletter.com/images/2048/directory_structure.png)

> 以下使用yeamon tool来构建项目的指南。如果你更新换自己动手，可以跳过依赖安装直接进入下一章节。

我们要先保证安装了`yeamon`才能在项目中使用。Yeamon依赖NodeJS和npm。NodeJS的安装并不在本文叙述的范围内但是在[NodeJS.org](http://nodejs.org)上有一个很好的指南。

在`npm`安装完后，我们就能安装yeamon tool，`yo`，和angular generator（`yo`会使用这个生成器来生成我们的Angular应用）：

```bash
$ npm install -g yo
$ npm install -g generator-angular
```

安装完之后，就可以使用yeamon tool来创建应用了，按照下面的来：

```bash
$ cd ~/Development && mkdir 2048
$ yo angular twentyfourtyeight
```

工具会问你一些问题，一律答yes，除了只选`angular-cookies`作为依赖，因为我们不需要除了缺省以外的依赖。

> 注意使用Angular generator会要求你安装ruby环境、gem和compass。文章下面给出的完整代码中会介绍如何避免使用ruby和compass。

####我们的angular模块

新建`scripts/app.js`文件来控制我们的应用。来，开始编写吧：

```javascript
angular.module('twentyfourtyeightApp', [])
```

<a name="modular"></a>
##模块结构

现在比较推荐的Angular应用结构是根据功能来构建而不是类型。也就是说，不是以controllers（译者注：控制器）、services、directives等来分离我们的组件而是以功能来定义模块结构。例如在我们的应用中，定义了一个`Game`模块和`Keyboard`模块。

![Minification](http://www.ng-newsletter.com/images/2048/scripts_dir.png)

这样的模块结构让我们能够清晰分离出跟文件结构相匹配的职责。这样做既能帮助我们构建大型的复杂的angular应用，也能让功能在不同的应用间共享。

之后我们将会建立起匹配文件和目录结构的测试环境。

####视图

在我们的项目中，从视图开始编写是最容易的。审视一下，要做的视图/模板只有一个。我们不需要多个视图，所以只需要一个`<div>`元素来包含应用中的所有内容。

在我们的的`app/index.html`文件中，我们需要包含所有的依赖（包括`angular.js`自身和自己编写的javascript文件——现在就只有`scripts/app.js`），就像下面的：

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

现在`app/index.html`文件做好了，我们只需要在`app/views/main.html`中继续细节化应用层面的视图就可以了。当我们需要在应用中引入新资源的时候就只需要修改`index.html`了。

赶快打开`app/views/main.html`，所有的游戏相关的视图都放在此。通过使用`controllerAs`语法，控制器就可以显式暴露在任何需要在`$scope`中找数据和查询控制器对应组件的地方。

```markup
<!-- app/views/main.html -->
<div id="content" ng-controller='GameController as ctrl'>
  <!-- Now the variable: ctrl refers to the GameController -->
</div>
```

> `controllerAs`语法是1.2版本提供的比较新的语法。当要在页面处理多个控制器的时候非常有用，因为这样就能指定包含我们需要的功能和数据的控制器。

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

注意在引用`currentScore`和`highScore`的时候我们也在视图中引用了`GameController`。`controllerAs`语法让我们能显式地引用自己感兴趣的控制器。

<a name="game-controller"></a>
##控制器GameController

现在既然已经有了一个合理的项目结构，我们赶快来创建一个`GmaeController`来控制会在视图上显示的数据。在`app/scripts/app.js`中，我们可以在主要模块`twentyfourtyeightApp`里创建这个控制器。

```javascript
angular
.module('twentyfourtyeightApp', [])
.controller('GameController', function() {
});
```

在视图中，我们已经引用了一个`game`对象，此对象会在`GameController`中进行设置。`game`对象引用的是主*游戏对象*。我们会在另外一个新的模块中创建这个主游戏对象，新的模块也会保存游戏中的所有引用。

现在还没有创建这个模块，应用不会在浏览器中载入。而在控制器里面，我们可以加上对`GameManager`的依赖：

```javascript
.controller('GameController', function(GameManager) {
  this.game = GameManager;
});
```

记住，我们正在做的是为应用中不同的部分创建模块级别的依赖，所以为了能在我们的应用中加载这些模块，需要在我们Angular模块中作为依赖来列出。将`Game`作为`twentyfourtyeightApp`的依赖，要在我们定义模块的地方的数组中列出。

完整的`app/scripts/app.js`文件看起来应该像下面那样：

```javascript
angular
.module('twentyfourtyeightApp', ['Game'])
.controller('GameController', function(GameManager) {
  this.game = GameManager;
});
```

###The Game

现在已经将部分数据绑定到视图上（译者注：原文Now that we have the view partially hooked up to the view，或有误），我们可以开始编写游戏的逻辑了。在`app/scripts/`目录下新建`app/scripts/game/game.js`中创建游戏模块：

```javascript
angular.module('Game', []);
```

> 当创建模块的时候，我们通常将其放在以模块命名的目录内，而以模块命名的文件来完成初始化工作。比如，我们正在写一个游戏(译者注：game)模块，于是我们在`app/scripts/game`目录下的`game.js`中编写。这个方法在生产环境下被认为是可扩展的和合理的。

`Game`模块会提供唯一的核心组件：`GameManager`。

我们编写的`GameManager`模块要做到：维持游戏的状态和玩家能做出的移动，维护得分、判断游戏结束和搞清楚是玩家赢了还是输了。

当在编写应用的时候，我们通常将已知需要的方法写成桩方法，为这些方法写测试然后再填内容。

> 为了文章起见，我们在这个模块里会走一遍这个流程。当继续写剩下的模块的时候，我们则只会涉及到应该测试的核心组件。

我们知道到现在为止`GameManager`中会提供的几个*已知的*功能：

1. 创建一个新的游戏
2. 处理游戏循环/移动操作
3. 更新得分
4. 跟踪游戏的进行情况

记住这几个功能，我们就能勾勒出`GameManager`服务的基本轮廓以供测试：

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

完成了基本的功能性函数之后，先挪一下，去写测试来决定在`GameManager`中*已知的*需要支持的函数中空白部分的内容。

<a name="tdd"></a>
##测试驱动开发（TDD）

在开始实施测试前，我们需要配置好karma来驱动我们的测试。如果你对karma并不熟悉，就只需要了解到它是一个测试运行器，能让我们舒服而高效地在控制台和代码中自动化操作前端测试。

![Running karma](http://www.ng-newsletter.com/images/2048/running_karma.png)

Karma作为一个npm包，依赖于NodeJS。运行命令行来安装：

```bash
$ npm install -g karma
```

> 参数`-g`告诉npm这个包作为全局模块来安装。没有这个参数，包将只会安装到本地的工作目录上。

如果你是通过yeoman angular生成器来构建应用的话可以跳过以下的部分。

要使用karma，需要一个配置文件。虽然我们这里不会深入叙述如何配置Karma（在[ng-book](https://www.ng-book.com)中查看详细的karma配置选项），但是过程中决定性的部分就是让Karma载入所有我们想要测试的文件。

我们可以使用`karma init`命令来生成一个基本的配置文件：

```bash
$ karma init karma.conf.js
```

命令会问几个问题然后生成`karma.conf.js`。这里我们修改一下其中两个选项：`files`数组和打开`autoWatch`：

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

一旦写好了配置文件，任何时候我们保存文件都可以运行测试了（测试文件在`test/unit/`目录内）。

我们像如下那样执行命令`karma start`来运行测试：

```bash
$ karma start karma.conf.js
```

###编写第一个测试

karma已经配置好了，可以写对`GameManager`的基本测试了。然而我们还并不清楚应用的整个功能，所以暂时只能写有限的测试。

> 在编写应用的时候我们经常发现API需要修改，所以与其在变化前投入大量时间，不如建立好对基本功能的测试然后在深入测试中找到最终的API。

用是否有可能的移动来作为第一个写的测试是个好选择。简单地编写几个我们已知需要的返回真/假的调用，来测试我们应用的逻辑行为。

创建`test/unit/game/game_spec.js`文件然后开始填入内容：

```javascript
describe('Game module', function() {
  describe('GameManager', function() {
    // Inject the Game module into this test
    beforeEach(module('Game'));

    // Our tests will go below here
  });
});
```

> 在这个测试中我们使用[Jasmine](http://jasmine.github.io/2.0/introduction.html)语法。

跟其他单元测试一样，我们需要创建一个`GameManager`对象的实例。我们可以使用普通的语法（测试服务的时候）将它注入到测试中。

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

有了这个`gameManager`实例，就可以建立对函数`movesAvailable()`的期望值。

我们定义的`movesAvailable()`函数是用来检测是否有空格剩余和是否有方块可以合并。另外这个结果跟游戏是否结束是有关联的，我们会将这个方法放进`GameManager`中，但是在之后创建的`GridService`中才实现大多数的复杂细节。

棋盘上要有剩余可走的地方，必须满足以下两个条件：

1. 棋盘上有空余空格
2. 方块可以合并

弄清楚了这两个条件，我们就可以写出测试来看看是否符合。

基本的思路就是我们写出的单元测试对于设定的条件要能作可观察到的反应。然后因为要依赖`GridService`来反映游戏的状态，所以需要模拟出这个条件来保证在`GameManager`中的逻辑是正确的。

####模拟`GridService`

要模拟`GridService`，我们只需要简单地*重写*缺省的Angular行为，替换*真正的*服务为我们模拟出来的服务，然后就可以在模拟的服务中建立可控制条件。

详细一点说就是，我们简单地创建一个拥有模拟方法的假对象然后通过在`$provide`中换上来骗Angular说这个假对象是*真*对象。

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

现在我们就可以用`_gridService`这个假对象实例来建立条件了。

我们希望当有空格剩余的时候函数`movesAvailable()`返回true。在`GridService`中模拟一个`anyCellsAvailable()`函数（其实还没写）。我们期望这个在`GridService`的函数能告诉我们有剩余的空格。

```javascript
// ...
describe('.movesAvailable', function() {
  it('should report true if there are cells available', function() {
    spyOn(_gridService, 'anyCellsAvailable').andReturn(true);
    expect(gameManager.movesAvailable()).toBeTruthy();
  });
  // ...
```

现在基础工作已经做好了，我们可以接着建立第二个条件了。如果方格可以合并，那么我们希望`movesAvailable()`保证会返回true。相反的情况也是返回true因为既没有方格空余也没有可合并的空格才是再没有可走的地方。

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

> 虽然考虑到整个文章的整体性我们不会再在文章中使用TDD，但是我们建议你应该始终使用TDD。可以在下面的完整代码中查看更多的测试代码。

##回到GameManager

现在我们的任务就是实现函数`movesAvailable()`。然而我们已经确认了代码可行性__和__要求的条件，实现起来实在简单。

```javascript
  // ...
  this.movesAvailable = function() {
    return GridService.anyCellsAvailable() || 
            GridService.tileMatchesAvailable();
  };
  // ...
```

<a name="build-grid"></a>
##建造游戏网格

到现在为止我们已经让`GameManager`运行起来了，然后就是要创建`GridService`来处理在棋盘中的所有状况。

回忆一下我们的想法：在`GridService`中使用两个本地数组变量，基本数组`grid`和基本数组`tiles`。在`app/scripts/grid/grid.js`文件中写服务：

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

当开始一个新的游戏的时候，我们需要清空这些数组。而因为`grid`数组只是用来放方块的DOM元素组成的。

然而数组`tiles`则是动态的，它会跟踪游戏过程中的当前的方块。使用游戏中不同的状态之前，先在页面上建造好网格先吧，这样我们也好看看大概样子是怎么样。

回到`app/views/main.html`，我们开始设计网格。因为网格是动态而又带有我们给它写的逻辑，所以只有就其放在其指令（译者注：directive）中才合乎逻辑。使用指令可以让主要的模板保持简洁，同样也能将功能封装在指令中而让主要的控制器保持简洁。

在`app/index.html`中我们将网格指令添加上然后在控制器中传递给`GameManager`实例。

```markup
  <!-- instructions -->
  <div id="game-container">
    <div grid ng-model='ctrl.game' class="row"></div>
    <!-- ... -->
```

我们是在`Grid`模块里写这个指令的，所以在`app/scripts/grid/`目录下，新建一个`grid_directive.js`文件来安放我们的`grid`指令。

在`grid`指令里面，我们只需要少量变量因为它需要封装视图，能做的事情不多。

指令会需要持有`GameManager`的实例（或者至少是一个有`grid`和`tiles`数组的模型），所以将其设置为指令的依赖。另外，不希望指令由于页面上的其它内容或者GameManager自身的原因瘫痪，所以我们创建了隔离作用域。

> 查看我们写的[自定义指令](http://www.ng-newsletter.com/posts/directives.html)来更加深入指令的编写，或者查看[ng-book](https://www.ng-book.com)中有关指令的细节。

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

###grid.html

在指令的模板里面，我们会运行两个`ngRepeat`来显示网格和方块数组，还会（暂时）在循环中使用`$index`来跟踪。

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

第一个`ng-repeat`简单易懂，就是遍历了grid数组然后生成了class属性是`grid-cell`的单个空div元素。

在第二个`ng-repeat`中，我们会为每一个显示的元素生成一个名为`tile`的指令。这个`tile`指令会负责生成每一个方格元素的样子。我们很快就会去编写`tile`指令……

精明的读者可能会发现我们只适用一维数组来显示二维网格。当我们渲染视图的时候，我们只会得到一列“方格”，而不是一个网格。

要将它弄成网格，我们来深入CSS的编写。

<a name="scss"></a>
##开始SCSS

在这个项目中，我们会使用SASS的一个变种：scss。scss除了是一个更强大的CSS外，还能动态地生成CSS。

应用所有显示的元素的主要部分会使用CSS来完成，包括动画、布局和可视元素（方格的颜色等）。

要创建二维的棋盘，我们会用到CSS3关键字：`transform`来处理每一个特定的方格的位置。

###CSS3 transform属性

CSS3 transform属性是一个可以让我们对元素进行2D或者3D上的移动、扭曲、旋转、缩放等操作（支持动画）的属性。用上了此属性，就可以直接将方块放在棋盘上然后剩下就只是应用合适的`transform`属性的事了。

例如，在下面的演示中，我们有一个宽40px高40px的盒子。

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

如果我们应用一个`translateX(300px)`的`transform`属性，就可以将盒子向右移动了300px，就像下面所展示的：

<div style="margin:40px;padding:40px 0;border-bottom:1px solid #333;">
  <div style="width:40px;height:40px;background-color:blue;-webkit-transform:translateX(300px);transform:translateX(300px);"></div>
</div>

```css
.box.transformed {
  -webkit-transform: translateX(300px);
  transform: translateX(300px);
}
```

使用translate属性，我们只需应用CSS类就可以在棋盘上随便移动方块了。现在，精妙之处在于页面是多变的，我们如何能将类写得足够动态可以对应到网格上的正确位置。

这里就是SCSS大显身手的地方了。我们会创建几个变量（例如一行有多少个方格）然后在SCSS中结合数学来帮助我们计算。

来看一下计算棋盘上正确位置需要的变量：

```css
$width: 400px;          // The width of the whole board
$tile-count: 4;         // The number of tiles per row/column
$tile-padding: 15px;    // The padding between tiles
```

让SCSS用这些变量帮我们动态计算位置。首先算出每一个方格的大小。在SCSS中非常简单：

```css
$tile-size: ($width - $tile-padding * ($tile-count + 1)) / $tile-count;
```

现在我们就可以使用适当的宽和高来建立那个`#game`容器了。同时`#game`容器也会被设置成位置参照，它的子元素将会使用绝对定位。我们将`.grid-container`和`tile-container`放在`#game`容器内。

我们这里只展示跟scss有关的部分。剩下的代码可以在文章末尾的github地址上找到。

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

注意为了让`.tile-container`放在`.grid-container`前面，我们__必须__要为`.tile-container`更高的`z-index`值。如果没有设置`z-index`值，浏览器会将两个元素放在同等高度，就不好看了。

做好这一步之后，现在我们来动态生成方块的位置。我们需要是一个`.position-{x}-{y}`类，用来应用到方块上，这样浏览器就会知道方块的位置然后将它放置好。既然我们是计算相对于网格容器的的transformation属性值，那就使用`0,0`作为第一个方块的初始位置。

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

> 注意我们不得不使用从1开始的偏移量来计算位置，而不是传统的从0开始。这是受SASS自身的限制所迫。不过我们可以使用将索引减1来解决。

现在我们写好了动态的`.position-#{x}-#{y}`CSS类，方块能够显示在页面上了。

![2-d grid
](http://www.ng-newsletter.com/images/2048/screen.png)

###为不同的方块上色

注意到当有不同的方块出现的时候，各自都是不同颜色的。不同的颜色标识着不同方块所代表的值。如此一来玩家能看得出方格所处的状态。使用和我们迭代方格数目的时候同样的技巧来创建方格颜色方案。

要创建出颜色方案，我们首先要创建一个SCSS数组，包含有每一种需要用到的背景颜色。每一种颜色：

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

使用了`$colors`数组，我们只要迭代每一个颜色就能基于方块的值来动态创建一个类。也就是说，当一个方块的值是2，我们会给它加上指定背景颜色是`#EEE4DA`的`.tile-2`类。与其给每个方块用硬编码，我们不如用SCSS的魔法来完成：

```css
@for $i from 1 through length($colors) {
  &.tile-#{power(2, $i)} .tile-inner {
    background: nth($colors, $i)
  }
}
```

当然，我们需要自己定义`power()`混合（译者注：mixin）。定义如下：

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
##Tile指令

SCSS的繁琐工作完成了，我们可以回到tile指令的编写中了。通过动态的位置布局，让CSS按我们所设计的那样将方块摆放到位。

然而`tile`指令是一个自定义视图的容器，并不需要做很多事。我们需要的是它负责显示的方格的访问权。除此以外，并不需要在指令内放任何功能。代码是自我描述的：

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

现在，`tile`指令中有趣的地方就是我们如何动态的为网格布局。而模板会需要用到在隔离作用域（译者注：isolate scope）中的`ngModel`变量来处理好一切。

```markup
<div ng-if='ngModel' class="tile position-{{ ngModel.x }}-{{ ngModel.y }} tile-{{ ngModel.value }}">
  <div class="tile-inner">
    {{ ngModel.value }}
  </div>
</div>
```

我们几乎已经可以将这个基础的指令直接显示了。对于每一个有`x`和`y`坐标的方块而言，它们都会*自动*被赋予一个`.position-#{x}-#{y}`的类。浏览器会*自动*地将它们放到我们期待的位置。

这意味着我们的方块对象会需要一个`x`和`y`以及`value`让指令来使用。为此，对于每一个显示的方块，我们都需要创建一个新的对象。

###TileModel

与其创建一个*哑*对象，我们还不如创建一个比较智能的对象，既存储数据也能提供功能。

我们希望能使用Angular的依赖注入，因此创建一个服务来安置数据模型。我们在`Grid`模块中创建一个`TileModel`服务，因为跟游戏棋盘有关的操作时，它只需要使用底层的`TileModel`。

使用`.factory`方法，我们能够简单地创建一个工厂函数。跟使用`service()`函数时传递的用以定义服务的函数会被默认为服务的构造函数不同的是，使用`factory()`函数会认为传递函数返回的对象才是服务。所以，只用`factory()`函数，我们可以将服务赋给任何对象以便在我们Angular应用中随时*注入*。

在`app/scripts/grid/grid.js`文件中，我们可以创建`TileModel`工厂：

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

现在在我们Angular应用中的任何地方，我们都可以*注入*这个`TileModel`并想全局对象一样使用。非常方便不是吗？

> 不要忘了要为我们在`TileModel`中实现的任何功能写测试。

###我们第一个网格

现在我们已经写好了`TileModel`了，我们可以开始在`tiles`数组中放入`TileModel`的实例了，然后发现它们*神奇地*出现在网格中正确的位置上。

让我们来试试在`GridService`中的`tiles`数组中加入一些方块：

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
##The Board’s ready for the game



<a name="grid-theory"></a>
##Multi-dimensional array in one dimension

<a name="keyboard"></a>
##Keyboard interaction

<a name="start-button"></a>
##Press the start button

<a name="game-loop"></a>
##Get your move on (the game loop)

<a name="keeping-score"></a>
##Keeping the score

<a name="game-over"></a>
##We won?!?? Game over

<a name="running-the-animation"></a>
##Animation

<a name="customizing-size"></a>
##Location customizations

<a name="demo"></a>
##演示demo

完整的demo在这里：[http://ng2048.github.io/](http://ng2048.github.io/)。

##总结

唷！我们希望你已经在愉快地使用Angular来编写这个2048游戏了。博文中应该已经覆盖了大部分的过程了。如果你觉得不错，可以在下面留下评论。如果你对继续学习Angular有兴趣，务必去看看我们的书[Complete Book on AngularJS](https://www.ng-book.com/)。这是唯一一本会不断更新AngularJS知识的书，并且包括了在AngularJS中所有你需要了解的东西。

[在HackerNews上讨论](https://news.ycombinator.com/item?id=7554348)

##Thanks

非常感谢[Gabriele Cirulli](http://gabrielecirulli.com/)编写出了妙极的（和让人上瘾的）2048，同样感谢他对此文的启发。文中的很多主意都是从原游戏中搜集、提炼，用以阐明如何使用Angular来编写。

##完整代码

游戏的完整代码在Github上，地址是http://d.pr/pNtX。要在本地编译游戏，只需要复制代码然后运行：

```bash
$ npm install 
$ bower install
$ grunt serve
```

##问题与解决方法

如果你使用不了npm install，保证你安装了最新的node.js和npm。

这个版本库在node v0.10.26和npm 1.4.3上测试。

以下是一个安装最新版本的node和node版本管理器`n`的方法：

```bash
$ sudo npm cache clean -f
$ sudo npm install -g n
$ sudo n stable
```
