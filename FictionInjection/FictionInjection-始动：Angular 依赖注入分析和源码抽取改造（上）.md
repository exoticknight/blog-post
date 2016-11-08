FictionInjection-始动：Angular 依赖注入分析和源码抽取改造（上）
=============================

几年前有一个项目构思，由于技术水平低，当时并没有思考得很清楚，所以一直没怎么着手开始。近来前端很多优秀的库出现了，也让我对那个项目有了新的想法。在写了几个 JavaScript 的项目后我觉得可以尝试开始了。尝试并成功写出一些代码之后，就开（挖）始（坑）了这个系列的博文。

项目是写一个 JavaScript 框架，干什么的在此并不是重点，但是首先需要一个可扩展的模块系统。最简单就是直接用 jQuery 扩展的写法，直接将函数等的挂载在一个对象下，不过如此一来模块之间依赖非常多的话，管理起来会十分困难。也可以使用 AMD / CMD 的模块化的方法，不过考虑到 ES2015 已经加入了 import / export 的语法，最好就直接使用。然而使用了 ES2015 的语法之后，仍然使用 AMD 等的语法就显得很别扭，但是又想要依赖注入功能怎么办？

解决方法是模（fu）仿（zhi）著名的 AngularJS 中关于依赖注入的源代码。

Angular 有两个版本，1.x 和 2.x，但是 2.x 中，淡化了模块的概念，直接采用 component 和 ES2015 的 import / export 的机制，所以依赖注入已经不太算是亮点了。而且 Angular2 采用 TypeScript 编写，从语法编写上也不适合作为参考。最后选定 1.4.5 版本。

> Angular 项目下还有一个不怎么有人知道的 `di.js` 项目，是从 Angular 中独立出来的依赖注入库，但是从文档来看，也是需要 TypeScript 来使用。

文章较长，给个目录

<a name="catalogue"></a>
* [实现依赖注入](#di)
* [从 API 入手](#api)
* [开始分析](#analyze)
* [统一包装](#wrapup)
* [小结](#brief)
* [注入器的奥秘](#injector)
* [读源码的意外发现](#unexpected)

<a name="di"></a>
##实现依赖注入[↑](#catalogue)
JavaScript 如何实现依赖注入呢？AngularJS 给出了三个解决方法。

```language
// 直接在参数里面声明
module.service( function ( $http ) {} )

// 使用显示注释
function a () {}
a.$inject = ['$http']
module.service( a )

// 数组内联
module.service( ['$http', function ( http ) {}] )
```

实际上三个方式都是一样的，只是使用方式不一样，最后都是使用了 JavaScript 的闭包来实现依赖的注入，原理如下：

```language
function method ( $http ) {
    return function ( args ) {
        // $http.get( args )
        // .....
    }
}
```

将依赖作为参数传入 `menthod` 得到的返回值就是是一个可以调用 `$http` 服务的函数了。

像这样 `['$http', function ( http ) {}]` 最后一个元素是函数的结构可以称为一个**'可注入结构'**。

<a name="api"></a>
##从 API 入手[↑](#catalogue)
Angular 库的源代码文件非常大，一般基本不会从头开始看。而 API 作为库对外的窗口，从 API 的使用顺藤摸瓜地查找代码是比较好的做法。Angular 模块的使用一般如下：

```language
// declare module
const a = angular.module( 'a', [] );
const b = angular.module( 'b', [] );
const c = angular.module( 'c', ['a'] );

// use the module
a
.value( 'a', 123 )
.factory( 'a', function() { return 123; } )
.service( 'serviceName', ... )
.directive( 'directiveName', ... )
.filter( 'filterName', ... );
```

先创建一个模块，用数组说明它的依赖模块，然后模块就可以调用 `value`、`service`、`directive` 等 API，API 的第一个参数是名字，第二个参数则是值或者函数或者数组，了解 AngularJS 的读者应该知道其实是值或者返回值的构造函数或者包含依赖和构造函数的数组。

打开 Angular.js ，查找出 `module(name, requires, configFn)` 函数的定义，位于一个更大的函数 `setupModuleLoader` 内。`setupModuleLoader` 为 `module` 函数编写了一些检测函数和变量。最重要的是 `modules` 变量，用来保存所有的模块信息。

下面来分析 `module` 函数。

```language
      if (requires && modules.hasOwnProperty(name)) {
        modules[name] = null;
      }
```

可以看出如果模块重复创建是会覆盖之前的。

```language
        /** @type {angular.Module} */
        var moduleInstance = {
```

`moduleInstance` 就是将会返回出去的模块对象，可以看到里面有 `name` 和 `requires` 等属性和 `provider` 和 `factory` 等函数。

里面所有的方法，都是通过调用 `invokeLater` 和 `invokeLaterAndSetModuleName` 生成的新函数。新函数的上下文中带有 `provider` 和 `method` 信息。比如 `service` 函数：provider='$provide'，method='service'，暂时还看不出来信息有什么用，可以先跳过。新函数在调用的时候会将信息连同调用的参数一起 push 进模块的 `_invokeQueue` 属性中。

绕了一大圈，就是知道了：在调用 `value`、`service`、`provider` 这些基本的模块功能函数的时候，其实只是将构造函数和相关信息先保存了下来，根本就没有做初始化模块等工作。

但是作为一个库必定需要跟 `window` 或者 `document` 产生点关系不然无法操作 DOM，根据编写过不少库的经验来看，通常将这样的代码放在最后。于是拉到最后一看，gotcha。

```language
  jqLite(document).ready(function() {
    angularInit(document, bootstrap);
  });
```

明显意思就是在文档准备完毕的时候调用 `angularInit`，转到 `angularInit` 的定义发现调用了 `bootstrap(appElement, module ? [module] : [], config);`，再转到 `bootstrap` 的定义，在函数内部又会调用 `doBootstrap` 函数，一系列的检查之后，调用了 `createInjector` 函数就结束了，转到 `createInjector` 的定义一看，有 `$provide` `factory` 等字样，说明找对地方了。

<a name="analyze"></a>
##开始分析[↑](#catalogue)

重点来分析 `createInjector` 函数。

函数体大概可以分成四段。

第一段是定义了 `providerCache`、`instanceCache`、`providerInjector` 和 `instanceInjector`。最后返回 `instanceInjector` 对象。`providerInjector` 和 `instanceInjector` 各为将 `providerCache` 和 `instanceCache` 传入 `createInternalInjector` 函数的返回值。

第二段是 provider 函数的定义，用以供初始化时候的调用。

第三段是 `loadModules` 函数的定义，作用是，显然，初始化模块。

第四段是 `createInternalInjector` 函数的定义，函数返回的是真正的注入器。

从第一段的代码来看，真正的工作是在 `loadModules` 函数中，因为 `createInternalInjector` 函数只返回一个对象，没有 'side effect' 的代码。

`loadModules` 函数上来就是一个对模块数组的遍历，然后在遍历内取模块的属性 `_invokeQueue` 来调用 `runInvokeQueue` 函数对已经缓存下来的对象或方法的构造函数进行处理。

值得注意的是在做调用 `runInvokeQueue` 前，有一个递归的调用 `loadModules(moduleFn.requires)`，表明了在初始化本模块之前，会先初始化依赖的模块。

到目前为止，可以判明模块初始化的分两个阶段，第一个是：声明模块及其依赖模块 -> 缓存模块变量的构造函数；第二个是：选取一个根模块（对应 AngularJS 中的 'app' 模块） -> 找出其依赖的模块，对于每一个依赖，递归地先初始化其依赖的模块，再初始化自身 -> 处理模块中缓存的变量的构造函数。

如此采取先缓存所有模块再通过依赖树来初始化的做法虽然看起来繁琐，但是得到一个重要的特性就是声明模块的时候不用关注依赖的顺序，只需要表明依赖就可以了。如果声明的时候就立刻初始化，则必须小心检查所依赖的模块初始化是否已经完成了，然而如此一来就退化成了普通的模块化方法了。**延迟初始化是实现依赖注入的重要过程**。

模块依赖已经明了，现在来看看作为处理函数的 `runInvokeQueue` 函数。

```language
      function runInvokeQueue(queue) {
        var i, ii;
        for (i = 0, ii = queue.length; i < ii; i++) {
          var invokeArgs = queue[i],
              provider = providerInjector.get(invokeArgs[0]);

          provider[invokeArgs[1]].apply(provider, invokeArgs[2]);
        }
      }
```

重要的代码只有两行，`provider = providerInjector.get(invokeArgs[0]);` 和 `provider[invokeArgs[1]].apply(provider, invokeArgs[2]);`。

往上看一下，调用 `providerInjector.get` 相当于是调用 `getService`。

```language
    function getService(serviceName, caller) {
      if (cache.hasOwnProperty(serviceName)) {
        if (cache[serviceName] === INSTANTIATING) {
          throw $injectorMinErr('cdep', 'Circular dependency found: {0}',
                    serviceName + ' <- ' + path.join(' <- '));
        }
        return cache[serviceName];
      } else {
        try {
          path.unshift(serviceName);
          cache[serviceName] = INSTANTIATING;
          return cache[serviceName] = factory(serviceName, caller);
        } catch (err) {
          if (cache[serviceName] === INSTANTIATING) {
            delete cache[serviceName];
          }
          throw err;
        } finally {
          path.shift();
        }
      }
    }
```

代码虽多，但基本就是干一件事，返回 `cache` 中的对象，如果没有，就用 `factory` 创建一个再返回。而调用的对象 `providerInjector` 的定义来看，`cache` 就等于：

```language
      providerCache = {
        $provide: {
            provider: supportObject(provider),
            factory: supportObject(factory),
            service: supportObject(service),
            value: supportObject(value),
            constant: supportObject(constant),
            decorator: decorator
          }
      },
```

OK，现在可以知道了那些被延迟初始化的模块元素会在这里被处理了。

从上文可以知道，`invokeArgs[0]` 的值为 `$provider`，`invokeArgs[1]` 的值为 service / factory 等，`invokeArgs[2]` 则为参数数组。

看看以下的示例：

```language
// 如此使用
m.service( 'b', ['a', function ( a ) { this.a = a }] );

// 初始化的时候实际上调用
$provider.service( 'b', ['a', function ( a ) { this.a = a }] );
```

<a name="wrapup"></a>
##统一包装[↑](#catalogue)

接下来就是分析模块元素（对外表现为 API）的代码了。

函数有点多，但是还是能看得出来。`supportObject` 不用管，只是负责转换一下参数，基本的函数是 `provider`，`factory` 会调用它，然后 `value` 和 `service` 会调用 `factory`。

```language
  function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      provider_ = providerInjector.instantiate(provider_);
    }
    if (!provider_.$get) {
      throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
  }
```

第一个判断和第二个判断在 `factory` 调用的时候是无效的，因为 `factory` 调用 `provider` 的时候第二个参数是 Object，而且带有 `$get` 属性。实际上在本阶段做的是，就是将调用 API 传入的第二个参数（第一个参数是名字）再包装一层对象，再**存储在 `providerCache` 中**，对象统一拥有 `$get` 属性，或者说，接口。

其中，`$get` 属性是一个可供调用的函数，功能是即使模块元素混杂存储，也能被统一的接口成功调用。

对于 `value`，调用 API 的时候传入的是值，因此需要包装成返回这个值的函数才赋值给 `$get`。

对于 `constant`，值是不变的，所以可以看到就直接存储了。

对于 `decorator`，同样会定义 `$get` 属性。

对于 `service`，设计上应该生成一个单例并存储下来。不过在这里，仍然是继续包装起来。

源代码：

```language
  function enforceReturnValue(name, factory) {
    return function enforcedReturnValue() {
      var result = instanceInjector.invoke(factory, this);
      if (isUndefined(result)) {
        throw $injectorMinErr('undef', "Provider '{0}' must return a value from $get factory method.", name);
      }
      return result;
    };
  }

  function factory(name, factoryFn, enforce) {
    return provider(name, {
      $get: enforce !== false ? enforceReturnValue(name, factoryFn) : factoryFn
    });
  }

  function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
      return $injector.instantiate(constructor);
    }]);
  }
```

实际干了如下的事情：

```language
  (function (name) {
    return provider(name, {  // 跟其他 API 一样调用 provider 函数
      $get: function () {
        return instanceInjector.invoke(
            ['$injector', function($injector) {
                return $injector.instantiate(constructor);  // 注意 constructor 是用户自定义的‘可注入结构’
            }]  // 这又是一个‘可注入结构’，注入的是 '$injector'，实际上就等于 instanceInjector
            , this);
      }
    });
  })( 'serviceName' )
```

最后依然将包装好的函数存入 `$provider`。

只是为什么还是存储在 `$provider`，而不是直接调用函数进行初始化？比如 `service`，为什么还要再包装上一层‘可注入结构’？

<a name="brief"></a>
##小结[↑](#catalogue)

前文提到，使用延迟初始化实现了模块的依赖注入，使依赖的模块不需要提前定义。

实际上模块内的元素（factory / service 等）也是可以使用依赖注入的。使用过 AngularJS 的肯定知道定义某一个 controller 的时候可以注入某个 service，然而 controller 和 service 的定义顺序应该不能对代码运行造成影响。

因此，在此时，模块元素的“构造函数”（注意是用户自定义的那个函数而并非供
 new 调用的那个函数）还并不具备运行的条件，因为还是需要等依赖的元素初始化。

于是某种意义上，模块元素就需要**第二重注入**。把‘可注入结构’缓存在 `$provider` 中实际上就是对应了前文叙述的‘把模块先全部缓存’，包装上一个函数再统一放在 `$get` 属性下明显是方便供下一阶段的调用。

万事俱备，只欠注入了。

> 分析到现阶段，大家应该对平常使用频繁的 `service`、`factory` 等函数有了更深的认识了。

<a name="injector"></a>
##注入器的奥秘[↑](#catalogue)
现在把精力放在 `createInternalInjector` 函数。

此函数在[开始分析](#analyze)一节中已经提到了，作用只是返回一个对象。这个对象就是真正的注入器。

此函数被调用了两次，分别是得到 `providerInjector` 和 `instanceInjector`。

注入器中重要的函数有三个，分别是 `getService`、`invoke` 和 `instantiate`。

在[开始分析](#analyze)中已经大致介绍了，`getService` 函数干一件事，返回 `cache` 中的对象，如果没有，就用 `factory` 创建一个再返回。

对于 `providerInjector`，`factory` 函数是：

```language
function(serviceName, caller) {
  if (angular.isString(caller)) {
    path.push(caller);
  }
  throw $injectorMinErr('unpr', "Unknown provider: {0}", path.join(' <- '));
}
```

不难理解，因为在调用 provider 的时候，`providerCache` 中的函数应该已经在上一个初始化模块阶段中被定义好，如果没找到，那么肯定是调用了未定义的 provider。

对于 `instanceInjector`，`factory` 函数是：

```language
function(serviceName, caller) {
  var provider = providerInjector.get(serviceName + providerSuffix, caller);
  return instanceInjector.invoke(provider.$get, provider, undefined, serviceName);
}
```

`instanceCache` 本身就是空的，因此在找不到的时候，就去 `providerInjector` 里找 provider，然后得到其调用的结果，就是真正需要的实例（instance）了。`$get` 在这里就凸显出统一调用的用处了。

`invoke` 函数则是处理**'可注入结构'**和调用函数。从源码中也可以看到组装参数和调用函数，其中也会调用 `getService` 去得到实参的值来实现注入。从这里的 `getService` 出发，又有可能调用 `factory` 继而继续调用 `invoke` 来得到所依赖的实例，直到没有任何依赖需要实例化，从而完美的实现了自洽。

`instantiate` 函数是用来处理 'service' 的，是用来模拟 `new` 的，从代码来看也是如此：复制一个函数的 prototype，绑定为函数的 `this`，然后调用函数。因此调用 `service` API 的时候，可以完全使用构造函数的写法，同时也能得到注入特性。

<a name="unexpected"></a>
##读源码的意外发现[↑](#catalogue)
读源码一般都会有一定的收获，或是技巧上的，或是思想上的。

当读完了 Angular 的依赖注入的代码后，才发现 Angular 虽然表明支持模块化，但是实际上所谓的模块化只是徒有其名，模块的定义只是方便框架自己做延迟初始化的工作，没有模块之实。模块只是依赖树上的节点，最终生成出来的命名空间跟模块没有一丁点的关系，所有模块里的东西，不论是 'value'、'service' 和 'provider' 等，都是平铺在 `instanceCache` 里面的。这样的做法明显的一个结果就是命名冲突，两个不同模块里面的同名对象，后实例化的会覆盖掉先实例化的。这一点非常的不好，因为完全不符合模块化的预期结果。

在下一篇编（fu）写（zhi）注入功能的时候，我会修改这部分使其能满足模块化的实际预期。