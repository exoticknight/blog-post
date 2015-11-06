《Understanding ECMAScript 6》笔记
===============================

> 在线免费阅读：https://leanpub.com/understandinges6/read/

> 部分代码使用原书，代码版权归原书所有

<a name="catalogue"></a>

1. [块级绑定（Block Bindings）](#block-bindings)
1. [字符串](#string)
1. [正则](#regex)
1. [字符串模板（template strings）](#template-strings)
1. [标签模板（tagged templates）](#tagged-templates)
1. [函数](#function)
1. [对象](#object)
1. [解构（Destructuring）](#destructuring)
1. [Symbols](#symbols)
1. [生成器（Generators）](#generators)
1. [迭代器（Iterators）](#iterators)
1. [类](#class)
1. [Promises](#promises)
1. [模块（Modules）](#modules)
1. [杂七杂八](#miscellaneous)

<a name="block-bindings"></a>
##块级绑定（Block Bindings）[↑](#catalogue)

###let

块级{}中有效

同块级不可重复声明

没有变量提升

> 块级会形成暂时性死区（TDZ，Temporal Dead Zone）

###const

基本和 `let` 相同，值不可修改

> `let` 和 `const` 最好不要在全局下使用

<a name="string"></a>
##字符串[↑](#catalogue)

###unicode 支持更好

###新增部分函数，支持双字节

`codePointAt`，双字节版的 `charCodeAt`，得到字符 unicode

`fromCodePoint`，双字节版的 `fromCharCode`，从 unicode 得出字符

`includes`，包含某字符串

`startsWith`，以某字符串开始

`endsWith`，以某字符串结束

`repeat`，重复字符串

`normalize`，unicode 正规化，举个例子：两个 unicode 字符合成一个

<a name="regex"></a>
##正则[↑](#catalogue)

###新增标志 `u`

正则识别 unicode 字符

###新增标志 `y`

sticky，部分浏览器早就实现了

<a name="template-strings"></a>
##字符串模板（template strings）[↑](#catalogue)

```javascript
let a = 1
let b = 2
let s = `${a} ${a + b}`  // '1 3'
```

<a name="tagged-templates"></a>
##标签模板（tagged templates）[↑](#catalogue)

```javascript
let a = 1
function tag ( strings, ...values ) {
  console.log( strings )
  console.log( values )
  return values[0]
}
let s = tag`a ${a}`  // 'a 1'
// ["a ", "", raw: Array[2]]
// [1]
```

<a name="function"></a>
##函数[↑](#catalogue)

###默认参数

```javascript
function foo ( bar = 1 ) {
  console.log( bar )
}
```

###剩余参数

```javascript
function foo ( bar, ...rest ) {  // ✓
  ;
}

function foo ( bar, ...rest, last ) {  // ×
  ;
}
```

###函数属性 name

各种例子

```javascript
function doSomething() {
    // ...
}
console.log( doSomething.name );          // "doSomething"

var doAnotherThing = function () {
    // ...
};
console.log( doAnotherThing.name );       // "doAnotherThing"

var doSomethingAgain = function doSomethingElse () {
    // ...
};
console.log( doSomethingAgain.name );      // "doSomethingElse"

var person = {
    get firstName () {
        return "Nicholas"
    },
    sayName: function () {
        console.log( this.name );
    }
}
console.log( person.sayName.name );   // "sayName"
console.log( person.firstName.name ); // "get firstName"

console.log( doSomething.bind().name );   // "bound doSomething"

console.log( ( new Function() ).name );     // "anonymous"
```

###new.target

避免了很多使用 `new` 的坑

```javascript
function Foo () {
  if ( typeof new.target !== "undefined" ) {
    console.log( 'good' );  // using new
  } else {
    throw new Error( 'You must use new with Person.' )  // not using new
  }
}

var foo = new Foo();  // good
foo = Foo.call( foo );  // error!
```

###块级函数

块级中可定义函数

###箭头函数

`this`, `super`, `arguments` 和 `new.target` 的值都在定义函数时绑定而非运行时绑定

不可 `new`

不可改变 `this` 的值

没有 `arguments`

> 跟普通函数一样拥有 name 属性

```javascript
var foo = value => value;  // input value, output value
var foo = () => {};
var foo = ( x, y ) => x + y;
var foo = id => ({ x: 'x' });
```

```javascript
// this 的绑定
var foo = {
  init: function () {
    document.addEventListener( 'click', (function ( e ) {
      console.log( e.type );
    }).bind( this ), false);
  }
};


// ------------------------

var foo = {
  init: function () {
    document.addEventListener( 'click', e => {console.log( e.type )}, false);
  }
};
```

###立即调用函数表达式（Immediately-Invoked Function Expressions (IIFEs)）

```javascript
let foo = function ( s ) {
  console.log( s );
}( 'text' )  // text

// -------------------------

let foo = ( s => {
  console.log( s );
})( 'text' )  // text
```

###新增尾递归优化

<a name="object"></a>
##对象[↑](#catalogue)

###对象字面属性值简写（Property Initializer Shorthand）

```javascript
function foo ( text ) {
  return {
    name  // name: name
  }
}
```

###对象方法简写（Method Initializer Shorthand）

```javascript
var foo = {
  bar () {}
}
```

###计算属性名语法

对象的属性可以使用中括号 `[]` 表示需要「被计算」，结果转换为字符串作为属性名使用。

```javascript
let a = function () {}
let foo = {
  a: 'text a',
  [a]: 'function a'
}
console.log( foo['a'] )  // text a
console.log( foo[a] )  // function a
```

###Object.is()

和经典的 `===` 几乎一样，区别在于：

```javascript
console.log( +0 === -0);             // true
console.log( Object.is( +0, -0 ) );     // false

console.log( NaN === NaN );           // false
console.log( Object.is( NaN, NaN ) );   // true
```

###Object.assign()

```javascript
Object.assign( target, ...source )
```

读取源对象可列举的、自身的属性，将其赋值到目标对象上，覆盖旧属性，并非通常意义的复制。

###复制存取器属性

> 此小节查询 MDN 后补充上

使用 `Object.getOwnPropertyDescriptor(source, key)` 读取，使用 `Object.defineProperties` 定义。

###属性允许重复定义

属性以最后一个定义的值为准

###修改原型

`Object.getPrototypeOf`，得到原型

`Object.setPrototypeOf`，设置原型

###super

用以访问对象的 prototype

<a name="destructuring"></a>
##解构（Destructuring）[↑](#catalogue)

```javascript
var {a, b: { c, d }} = c
( {a, b: { c, d }} = c )

var [a, [b, c]] = d
var [a, , [b, c]] = d  // 跳过一个

function foo ( { bar1, bar2 } = {} ) {
  ;
}
```

解构可以有默认值，但只会在需要的时候求值。

[2ality](http://www.2ality.com/2015/01/es6-destructuring.html) 有更详细清晰的解释：

```javascript
let {prop: y=someFunc()} = someValue;

let x, y;
[x=3, y=x] = [];     // x=3; y=3
[x=3, y=x] = [7];    // x=7; y=7
[x=3, y=x] = [7, 2]; // x=7; y=2
```

<a name="symbols"></a>
##Symbols（不知道如何翻译，是第七种原始类型）[↑](#catalogue)

```javascript
var foo = Symbol()
var foo = Symbol( 'bar' )
```

`Symbol( 'description' )` 生成局部 Symbol，即使 `description` 相同生成的 Symbol 也不一样

`Symbol.for( 'description' )` 生成全局 Symbol，`description` 相同则 Symbol 相同

###获取对象的 Symbol 数组

`Object.getOwnPropertySymbols( object )`

###强制转换 Symbol 为 String

> 原书本节未完成

###有名的预定义 Symbol

> 原书本节大部分未完成

<a name="generators"></a>
##生成器（Generators）[↑](#catalogue)

生成迭代器的函数

```javascript
function *createIterator() {
  yield 1;
  yield 2;
  yield 3;
}

var o = {
  *createIterator ( items ) {
    for ( let i=0; i < items.length; i++ ) {
      yield items[i];
    }
  }
};

let iterator = o.createIterator( [1, 2, 3] );
```

<a name="iterators"></a>
##迭代器（Iterators）[↑](#catalogue)

###for-of 语法

数组、字符串、映射（Map）、集合（Set）和元素数组（NodeList）都可迭代（iterable），可使用 for-of 语法

###得到内置迭代器

Symbol.iterator 指向得到迭代器的函数

```javascript
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();
iterator.next();  // 1
```

###自定义迭代器

```javascript
let collection = {
  items: [],
  *[Symbol.iterator]() {
    yield *this.items.values();  // `yield *` 语法，委托了数组 `items` 的内置迭代器
  }
};
```

###对象、数组、映射、集合都具有的默认迭代器

`ertries()`，返回键值对迭代器

`keys()`，返回键迭代器

`values()`，返回值迭代器

###字符串迭代器

通过 `[]` 的访问是 code unit 方式

通过迭代器则是字符方式（几乎是，某些 unicode 支持不足）

###元素数组（NodeList）迭代器

返回的是数组中的单个元素

###向迭代器传参数

```javascript
function *foo () {
  let bar = yield 1;
}

let it = foo()
console.log( it.next() )  // Object {value: 1, done: false}，执行语句 `yield 1` 然后暂停
console.log( it.next( 2 ) )  // Object {value: undefined, done: true}，将 2 作为 `yield 1` 的返回值，
                             // 迭代器内部继续执行语句 `let bar = 2`，
                             // 之后执行完毕，无返回值，`value` 为 `undefined`，`done` 为 `true`
console.log( it.next() )  // Object {value: undefined, done: true}
```

###生成器使用 return 提前返回

```javascript
function *createIterator() {
  yield 1;
  return 42;
  yield 2;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 42, done: true }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

###委托生成器

使用 `yield *`

```javascript
function *createNumberIterator() {
  yield 1;
  yield 2;
  return 3;
}

function *createRepeatingIterator(count) {
  for (let i=0; i < count; i++) {
    yield "repeat";
  }
}

function *createCombinedIterator() {
  let result = yield *createNumberIterator();
  yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

可以 `yield *"string"`，会调用字符串的默认迭代器

```javascript
function *foo () {
  yield * "hello"
}
let it = foo()
console.log( it.next() )  // Object {value: "h", done: false}
console.log( it.next() )  // Object {value: "e", done: false}
```

###异步任务调度

以下是书中的例子，写得并不好，变量 `task` 的管理容易出问题：

```javascript
var fs = require("fs");

var task;

function readConfigFile() {
    fs.readFile("config.json", function(err, contents) {
        if (err) {
            task.throw(err);
        } else {
            task.next(contents);
        }
    });
}

function *init() {
    var contents = yield readConfigFile();
    doSomethingWith(contents);
    console.log("Done");
}

task = init();
task.next();
```

<a name="class"></a>
##类[↑](#catalogue)

###类声明

```javascript
class foo {
  // 相当于构造函数
  constructor ( name ) {
    this.name = name;
  }
  // 相当于 foo.prototype.bar
  bar () {
    console.log( this.name );
  }
}
```

类的属性最好都在构造函数里面创建。

类声明本质上就是以前的函数声明，除了以下有所不同：

1. 类声明不会像函数声明那样被提升
1. 类内部的代码全部以 `strict mode` 运行
1. 所有方法都是不可列举的，相当于使用了 `Object.defineProperty()`
1. 不使用 `new` 会抛异常
1. 以类名命名方法来覆盖类名会抛异常（类名对于类内部来说是以 `const` 定义的，对于外部则不是）

###类表达式

```javascript
let foo = class {}
```

```javascript
let foo = class foo2 {}  // foo === foo2
```

匿名类作为参数

```javascript
function createFoo ( c ) {
  return new c()
}

createFoo( class {
  constructor () {
    ;
  }
})
```

立即调用类表达式（有点像立即调用函数表达式）

```javascript
let foo = new class {
  constructor ( name ) {
    this.name = name
  }
}( 'foo' )
```

###存取器属性

```javascript
class foo {
  constructor ( name ) {
    this.name = name
  }

  get className () {
    return 'class ' + this.name
  }

  set className ( value ) {
    this.name = 'class' + value
  }
}
```

###静态成员

```javascript
class foo {
  constructor ( name ) {
    this.name = name
  }

  // 相当于 foo.prototype.bar
  bar () {
    console.log( this.name )
  }

  // 相当于 foo.staticBar
  static staticBar () {
    console.log( this.name )
  }

  // get / set 也可以用
  static get barName () {
    return 'bar'
  }
}
```

> 静态成员同样不可列举

###派生类

比起 ECMAScript5，ECMAScript6 的派生方便了很多

```javascript
class Rectangle {
  constructor ( length, width ) {
    this.length = length;
    this.width = width;
  }

  getArea () {
    return this.length * this.width;
  }
}

class Square extends Rectangle {
  constructor ( length ) {
    super( length, length );
  }
}
```

在派生类的构造函数中，调用 `super` 是必须的。如果连构造函数都没有，则：

```javascript
class Square extends Rectangle {
  // 无构造函数
}

// 相当于
class Square extends Rectangle {
  constructor ( ...args ) {
    super( ...args )
  }
}
```

> 1. 只能在派生类中用 `super()`
> 1. 使用 `this` 前必先调用 `super()` 来**初始化** `this`
> 1. 只有在构造函数返回一个对象的时候才可以不用 `super()`

###类方法

覆盖、隐藏父类方法

```javascript
class Square extends Rectangle {
  constructor ( length ) {
    super( length, length );
  }
  getArea () {
    return this.length * this.length;
  }
}
```

仍然可以使用 `super` 调用父类方法

```javascript
class Square extends Rectangle {
    constructor ( length ) {
        super( length, length );
    }
    getArea () {
        return super.getArea();
    }
}
```

类方法没有 `[[Construct]]` 这个内部方法，所以不能被 `new`。（[什么是`[[Construct]]`](http://www.ecma-international.org/ecma-262/5.1/#sec-13.2.2)）

###静态成员

相当于 ES5 中定义在构造函数上的方法（注意不是定义在构造函数的原型上），派生类显然也能调用

###extends 关键字后面可以使用表达式

除了 `null` 和生成器函数外

```javascript
// 使用函数
function base () {}
class foo extends base {}

// 使用表达式
function base () {}
function getBase () {
  return base
}
class foo extends getBase() {}

// 混合模式（多继承？!）
let AMinin = {
  aF = function () {}
}
let BMinin = {
  bF = function () {}
}
function mixin ( ...mixins ) {
  var base = function () {}
  Object.assign( base.prototype, ...mixins )
  return base
}
class foo extends mixin( AMinin, BMinin ) {}

// 内置类型
class foo extends Array {}
class foo extends String {}
```

###new.target

能够得知类的调用状态，应用例如：阻止抽象类被实例化

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error("This class cannot be instantiated directly.")
    }
  }
}
```

<a name="promises"></a>
##Promises[↑](#catalogue)

Promise 是老朋友了，所以没有什么好记录的，就记一下语法。

```javascript
let p1 = new Promise( function ( resolve, reject ) {
    resolve( 42 );
});

let p2 = Promise.resolve( 42 );

let p3 = Promise.reject( 43 );

let p4 = Promise.all( [p1, p2, p3] );  // 等待所有 Promise 返回

let p5 = Promise.race( [p1, p2, p3] );  // 最快的一个 Promise 返回就返回

p4.then( function ( value ) {
    console.log( value );
}).catch( function ( value ) {
    console.log( value );
})
```

<a name="modules"></a>
##模块（Modules）[↑](#catalogue)

> 注：本章的代码似乎有一些问题，基本参考 MDN 为准

1. 模块中的代码自动以严格模式运行
1. 模块中的顶层变量只是模块中顶层，并非全局顶层
1. 顶层中的 `this` 的值为 `undefined`
1. 代码中不允许 HTML 风格的注释
1. 模块必须有导出的东西

###基本导入导出

直接使用原书代码：

```javascript
// 导出数据
export var color = "red";
export let name = "Nicholas";
export const magicNumber = 7;

// 导出函数
export function sum(num1, num2) {
    return num1 + num1;
}

// 导出类
export class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
}

// 导出引用
function multiply(num1, num2) {
    return num1 * num2;
}
export multiply;

// 默认导出
export default function () {
    return;
}

// export as
export { multiply as foo }
```

1. 除非使用 `default` 语法，否则函数和类都不能使用匿名
1. `export` 只能用在顶层中

`as` 和 `default` 语法的情况，给出一个来自 [2ality](http://www.2ality.com/2015/07/es6-module-exports.html) 的表格

|Statement                      |Local name  |Export name|
|-------------------------------|------------|-----------|
|export {v as x};               | 'v'        | 'x'       |
|export default function f() {} | 'f'        | 'default' |
|export default function () {}  | '*default*'| 'default' |
|export default 123;            | '*default*'| 'default' |

可以看出，所谓的默认导出其实就是用了 `default` 作为名字罢了。

还能够将其他模块重新导出

|Statement                  | Module|Import name|Export name|
|---------------------------|-------|-----------|-----------|
|export {v} from 'mod';     |'mod'  |'v'        |'v'        |
|export {v as x} from 'mod';|'mod'  |'v'        |'x'        |
|export * from 'mod';       |'mod'  |'*'        |null       |

导入有很多方法，基本使用到的其实只有几种，以下来自 MDN：

```javascript
import name from "module-name";
import * as name from "module-name";
import { member } from "module-name";
import { member as alias } from "module-name";
import { member1 , member2 } from "module-name";
import { member1 , member2 as alias2 , [...] } from "module-name";
import defaultMember, { member [ , [...] ] } from "module-name";
import defaultMember, * as alias from "module-name";
import defaultMember from "module-name";
import "module-name";
```

最后那种导入是相当于将代码执行了一次。通常可以用来做 `polyfills` 和 `shims`。

<a name="miscellaneous"></a>
##杂七杂八[↑](#catalogue)

`Number.isInteger`，判断整数

`Number.isSafeInteger`，判断是否是有效整数

Math 中加入很多函数，例如双曲正弦、双曲余弦之类的

