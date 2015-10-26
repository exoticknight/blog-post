《Understanding ECMAScript 6》笔记
===============================

> 在线免费阅读：https://leanpub.com/understandinges6/read/

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

```language-javascript
let a = 1
let b = 2
let s = `${a} ${a + b}`  // '1 3'
```

<a name="tagged-templates"></a>
##标签模板（tagged templates）[↑](#catalogue)

```language-javascript
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

```language-javascript
function foo ( bar = 1 ) {
  console.log( bar )
}
```

###剩余参数

```language-javascript
function foo ( bar, ...rest ) {  // ✓
  ;
}

function foo ( bar, ...rest, last ) {  // ×
  ;
}
```

###函数属性 name

各种例子

```language-javascript
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

```language-javascript
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

```language-javascript
var foo = value => value;  // input value, output value
var foo = () => {};
var foo = ( x, y ) => x + y;
var foo = id => ({ x: 'x' });
```

```language-javascript
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

```language-javascript
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

```language-javascript
function foo ( text ) {
  return {
    name  // name: name
  }
}
```

###对象方法简写（Method Initializer Shorthand）

```language-javascript
var foo = {
  bar () {}
}
```

###计算属性名语法

对象的属性可以使用中括号 `[]` 表示需要「被计算」，结果转换为字符串作为属性名使用。

```language-javascript
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

```language-javascript
console.log( +0 === -0);             // true
console.log( Object.is( +0, -0 ) );     // false

console.log( NaN === NaN );           // false
console.log( Object.is( NaN, NaN ) );   // true
```

###Object.assign()

```language-javascript
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

```language-javascript
var {a, b} = c
var [a, [b, c]] = d
```

<a name="symbols"></a>
##Symbols（不知道如何翻译，是第七种原始类型）[↑](#catalogue)

```language-javascript
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

```language-javascript
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

```language-javascript
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();
iterator.next();  // 1
```

###自定义迭代器

```language-javascript
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

```language-javascript
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

```language-javascript
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

```language-javascript
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

```language-javascript
function *foo () {
  yield * "hello"
}
let it = foo()
console.log( it.next() )  // Object {value: "h", done: false}
console.log( it.next() )  // Object {value: "e", done: false}
```

###异步任务调度

以下是书中的例子，写得并不好，变量 `task` 的管理容易出问题：

```language-javascript
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

```language-javascript
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

```language-javascript
let foo = class {}
```

```language-javascript
let foo = class foo2 {}  // foo === foo2
```

匿名类作为参数

```language-javascript
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

```language-javascript
let foo = new class {
  constructor ( name ) {
    this.name = name
  }
}( 'foo' )
```

###存取器属性

```language-javascript
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

```language-javascript
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

```language-javascript
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

```language-javascript
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



<a name="promises"></a>
##Promises[↑](#catalogue)

<a name="modules"></a>
##模块（Modules）[↑](#catalogue)

<a name="miscellaneous"></a>
##杂七杂八[↑](#catalogue)