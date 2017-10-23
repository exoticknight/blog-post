# 不要再只会用 for 了

> No Silver Bullet

## 老黄牛的 for

![for-for-everywhere](https://i.imgflip.com/1y3pxa.jpg)

几乎每一个编程语言都有 `for`，JavaScript 也不例外。

在 JavsScript 中，`for` 广泛用于遍历数组中，也能用于遍历对象的属性。

语法：

```javascript
for ([initialization]; [condition]; [final-expression])
   statement
```

`initialization` 是初始化语句，通常用于初始化计数变量（比如，你们最爱的 `i`）；`condition` 是判断本次是否执行 `statement`；`final-expression` 在 `statement` 执行完后执行，通常用于对计数变量进行变换。

例如，要遍历一个数组 `arr`，那么可以这样写：

```javascript
for (var i=0; i<arr.length; i++) {
  ;// 做你爱做的事
  // arr[i] 就能在每一次运行过程中取到在 arr 中的元素
}
```

或者经常见到所谓的性能优化：

```javascript
for (var i=0, len=arr.length; i<len; i++) {
  ;// 做你爱做的事
  // arr[i] 就能在每一次运行过程中取到在 arr 中的元素
}
```

> 强烈建议使用 `const` 或 `let` 声明变量而不是使用 `var`，因为 `var` 会在 `for` 语句外声明变量，结果就是变量可能会在意外的地方被读取到。如果你不能使用 ES2015 或更新的版本，下文同样有解决方法（同样是本文的主要内容）。

如果你写过 C 系列，那么你有可能忍不住自己的麒麟臂，写出「炫技」的代码来。比如 MDN 上的[这个例子][for-on-mdn]。

```javascript
function showOffsetPos(sId) {

  var nLeft = 0, nTop = 0;

  for (

    var oItNode = document.getElementById(sId); /* initialization */

      oItNode; /* condition */

    nLeft += oItNode.offsetLeft, nTop += oItNode.offsetTop, oItNode = oItNode.offsetParent /* final-expression */

  ); /* semicolon */ 

  console.log('Offset position of \'' + sId + '\' element:\n left: ' + nLeft + 'px;\n top: ' + nTop + 'px;');

}

/* Example call: */

showOffsetPos('content');

// Output:
// "Offset position of "content" element:
// left: 0px;
// top: 153px;"
```

这么写看着很酷，但是实际上不要这样写，尤其是在团队合作中。这样的代码一是混杂难懂，二是难以维护。代码首先是写给人看的，接着才是给机器运行的。

或许你已经非常习惯写 `for` 了，习惯到了看见一个数组就自然而然打出 `for (...)` 来。但是你有没有想过，很多时候，遍历数组其实跟索引并没有什么关系，代码只是要将数组里面的元素按顺序处理完。然而，**数组天然就应该是顺序的**，根本无需要一个额外的 `i` 来保证。换句话说，数组应该利用自身属性，提供无需索引的顺序读取方法，而索引只是在顺序读取的过程中的一个记录变量。

那这样有什么优势呢？

从处理流程上说，举个例子：假设你是一个接待员，工作是处理一列队伍的咨询。使用 `for` 的处理方法是：先计算整个队伍的长度，然后喊第一个人开始处理；每处理完一个人，就将序号加一再喊；直到序号等于队伍长度。而使用直接顺序读取的处理方法是：从队伍最前开始处理，每处理完一个，直接转到下一个重新开始处理，直到队伍没有下一个需要处理。这样以来就能省去了对队伍长度和索引的处理。

从代码逻辑上说，在数组上提供顺序读取方法，是将全局的语法转化为了相当于成员函数的执行，解除了和全局的耦合的同时，结合链式调用和灵活的回调函数能解放出极大的数组处理潜力。你能轻松在一行代码内基于数组进行非常多而灵活的处理，并且将「肮脏」的处理过程隐藏起来，直接得到一个处理得「连你阿妈都唔识」的结果，干净利落。

下面来认识一下这些顺序读取方法吧。

[for-on-mdn]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for

## 优雅的 forEach, map, filter, reduce

![forEach,map,filter,reduce](https://i.imgflip.com/i/1y3h2m.jpg)

在 ES5（ES5.1） 中，JavaScript 新增了多个数组方法，包括：forEach, map, filter, reduce。

每个方法都接受一个回调函数作为参数传入，每个方法都会在取得一个元素的时候调用此回调函数，不同在于不同方法对待回调函数的结果上。

### forEach

forEach 返回值为 undefined，适合通过数组来操作其他对象。

```javascript
arr.forEach(function callback(currentValue, index, array) {
    //your iterator
}[, thisArg]);
```

### map

map 返回值为回调函数返回值组成的数组，适合处理数组变换。

```javascript
var new_array = arr.map(function callback(currentValue, index, array) {
    // Return element for new_array
}[, thisArg])
```

### filter

filter 返回值为回调函数返回真所对应的元素组成的数组，适合处理数组筛选。

```javascript
var new_array = arr.filter(function callback(currentValue, index, array) {
    // Return true or false
}[, thisArg])
```

### reduce

reduce 返回值为初始值经过和每个元素作用后得到的最终值，适合遍历数组后得到一个值或者一个对象的情况。

```javascript
var new_array = arr.reduce(function callback(accumulator, currentValue, index, array) {
    // Return accumulator
}[, initialValue])
```

> 别忘了在回调函数中返回结果！

## 新方法新思路

配合几个编写代码时的常见场景，看看不使用 `for` 的解决方法。

### 循环多次执行某些动作

给定一个数组，打印其元素。

```javascript
var arr = [1, 2, 3]
```

for

```javascript
for (var i=0; i<arr.length; i++) {
  console.log(arr[i])  // 1 2 3
}
```

炫技

```javascript
for (
  var i=0;
  i<arr.length;
  console.log(arr[i++])
  );
```

改写

```javascript
arr.forEach(el => console.log(el))  // 1 2 3
```

### 对数组的每一个元素进行变换

给定一个数组，将其元素都加一。

```javascript
var arr = [1, 2, 3]
```

for

```javascript
for (var i=0; i<arr.length; i++) {
  arr[i] = arr[i] + 1
}
```

炫技

```javascript
for (
  var i=0;
  i<arr.length;
  arr[i] = arr[i++] + 1
  );
```

改写

```javascript
arr.forEach((el, i, ar) => ar[i] = ar[i] + 1)
```

更好

```javascript
const newArr = arr.map(el => el + 1)
```

> 在允许的情况下尽量不要去修改原数据，而是返回一个新的数组。

### 提取数组中符合某个标准的元素

给定一个数组，筛选出大于 2 的元素。

```javascript
var arr = [1, 2, 3]
```

for

```javascript
var newArr = []
for (var i=0; i<arr.length; i++) {
  if (arr[i] > 2) {  // e.g. should be larger than 2
    newArr.push(arr[i])
  }
}
```

改写

```javascript
const newArr = arr.filter(el => el > 2)
```

### 使用数组生成新数组

给定一个数组，要求使用其元素内容作为键，元素下表作为值，生成一个新数组

```javascript
var arr = ['a', 'b', 'c']
```

for

```javascript
var newArr = []
for (var i=0; i<arr.length; i++) {
  var obj = {}
  obj[arr[i]] = i
  newArr.push(obj)
}
// can access 'i' and 'obj' here
```

改写

```javascript
var newArr = arr.map(function(el, i) {
  var obj = {}
  obj[el] = i
  return obj
})
// cannot access 'obj' here, hell yeah!
```

ES2015

```javascript
const newArr = arr.map((el, i) => { return { [el]: i } })
// less code in one line! fuck yeah!
```

### 遍历数组，得到一个最终值

给定一个数字数组，将其包含的数字累加

```javascript
var arr = [1, 2, 3]
```

for

```javascript
var result = 0
for (var i=0; i<arr.length; i++) {
  result += arr[i]
}
```

改写

```javascript
const result = arr.reduce((ret, el) => ret + el, 0)
```

给定一个键值数组，将其转换为一个对象

```javascript
var arr = [
  { key: 'a', value: 1 },
  { key: 'b', value: 2 },
  { key: 'c', value: 3 },
  { key: 'd', value: 4 },
]
```

for

```javascript
var result = {}
for (var i=0; i<arr.length; i++) {
  result[arr[i].key] = arr[i].value
}
```

改写

```javascript
const result = arr.reduce((obj, { key, value }) => {
  obj[key] = value
  return obj
}, {})
```

从以上例子中可以看到，ES2015 的代码更加清晰可读，而且代码打起来流畅省时（你自己试试！）。如果你还没使用上 ES6，那么应该赶紧去学！或许[这篇文章][Why You Should Be Writing ECMA Script 6 Now]和[这篇文章][6 reasons Web developers need to learn JavaScript ES6 now]能说服你。

也可以看看本人写的 [《Understanding ECMAScript 6》笔记][《Understanding ECMAScript 6》笔记]。

[Why You Should Be Writing ECMA Script 6 Now]: https://www.wintellect.com/why-you-should-be-writing-ecma-script-6-now/
[6 reasons Web developers need to learn JavaScript ES6 now]: https://thenextweb.com/dd/2016/03/09/6-reasons-need-learn-javascript-es6-now-not-later/
[《Understanding ECMAScript 6》笔记]: https://raw.githubusercontent.com/exoticknight/blog-post/master/%E3%80%8AUnderstanding%20ECMAScript%206%E3%80%8B%E7%AC%94%E8%AE%B0.md

## 不灭的 for

尽管数组新增的方法十分强大，但是 `for` 除了会在遍历数组中使用，还会在处理对象的时候使用，比如使用 `for...in` 遍历对象的属性（及其原型上的属性）。在这些场合上，就需要具体情况具体分析了。

### 遍历对象问题

给出 app 的版本以及版本的使用量，统计最新两个大版本的使用量。版本命名符合 semver 标准，形如 'x.x.x'。

```javascript
var apps = {
  '6.6.0': 53695,
  '6.10.0': 47319,
  '5.4.0': 42601,
  '5.8.5': 41320,
  '5.5.5': 40322,
  '5.8.1': 38509,
  '5.1.5': 26473,
  '5.2.1': 24267,
  '6.10.1': 17042,
  '5.8.0': 13878,
  '5.5.1': 12887,
  '5.1.0': 9836,
  '6.5.0': 8909,
  '5.0.0': 6704,
  '4.7.0': 5915,
  '4.5.0': 5300,
  '4.3.0': 4213,
  '5.7.0': 4000,
  '4.6.1': 3647,
  '4.4.0': 1921,
  '4.6.0': 1802
}
```

for

```javascript
let largest = 0
const versions = {}
for (const key in apps) {
  const major = parseInt(key.split('.')[0], 10)
  if (!versions[major]) {
    versions[major] = {}
  }
  versions[major][key] = apps[key]

  largest = largest < major ? major : largest
}
const newApps = {}
for (let i=0; i<2; i++) {
  Object.assign(newApps, versions[largest-i])
}
```

ES2015

```javascript
let largest = 0
const versions = Object.keys(apps).reduce((obj, key) => {
  const major = parseInt(key.split('.')[0], 10)
  if (!obj[major]) {
    obj[major] = {}
  }
  obj[major][key] = apps[key]

  largest = largest < major ? major : largest

  return obj
}, {})
const newApps = [...Array(2).keys()].map(x => largest - x).reduce((obj, key) => {
  return Object.assign(obj, versions[key])
}, {})
```

使用 `for` 和不使用 `for`相比，相差不大，甚至代码看起来更清晰，而且有 ES2015 的加成，消除了变量泄露的影响。所以如果是遍历对象，就没必要去用数组的方法了。

### 性能问题

截止目前位置（2017-10-17），从 [benchmark][for-vs-foreach-vs-for-of] 来看，在性能上，`for` > `forEach` > `for...of`。

因此在一些对性能要求比较高的代码中，使用 `forEach` 和 `for...of` 需要谨慎，这有可能会成为性能瓶颈。另外 `for...of` 在浏览器上的支持度不高，所以还是可以暂时不使用，除非你清楚自己在干什么。

不过，仍然是那句话，代码首先是写给人看的，性能优化应该在功能实现之后再考虑。

[for-vs-foreach-vs-for-of]: https://jsperf.com/for-vs-foreach-vs-for-of

### 循环中断问题

一般来说，使用 `for` 和使用数组方法在功能实现上是一样的，但是由于 `for` 是编程语言层面的实现，可以使用 `break` 和 `return` 手段进行中断；上文中的数组方法由于是遍历调用函数，并不存在什么停止的条件，因此肯定是会将所有元素都过一遍。在这种情况下，就乖乖使用 `for` 吧。

> 当然也可以使用 `Array.some` 等方法模拟中断效果，但要是那样做还不如直接 `for` 呢。

### Promise 的问题

使用数组方法时，最容易出错的地方是和 Promise 一起使用的时候。

比如需要从不同的 URL 请求数据，极其容易写成以下**错误的**代码。

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
const results = URLs.map(url => {
  return $.ajax(url)
})
```

你会发现 `results` 的内容只是 Promise 实例，根本不是期望的值。代码的问题在于几乎所有的网络请求 API，返回的都是一个 Promise 实例。

正确的做法是使用 `Promise.all` 将多个 Promise 实例包装成一个 Promise 实例：

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
let result
Promise.all(URLs.map(url => {
  return $.ajax(url)
})).then(res => result = res)
```

如果你使用 async／await，**千万不要这样写**：

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
const results = URLs.map(async url => {
  await $.ajax(url)
})
```

同样这种写法只能得到一个 Promise 实例数组。应该这样：

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
const results = await Promise.all(URLs.map(url => {
  return $.ajax(url)
}))
```

> ⚠️ 注意你不能在没有 `async` 标识的函数中使用 `await`，因此在各种全局状态下是无法使用 `await` 的。幸好 async／await 处理的就是 Promise，你只需要改用 `.then` 就好了。

举一个更极端的例子，URL 请求需要按顺序发送，上一次的结果需要作为下一次请求的参数。怎么写呢？

Promise 方法：

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
let result
URLs
.map(url => {
  return params => {
    return $.ajax(url + params)
  }
})
.reduce((prev, next) => {
  return prev.then(next)
}, Promise.resolve(''))
.then(ret => result = ret)
```

async／await 方法：

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
const result = await URLs.map(url => {
  return params => {
    return $.ajax(url + params)
  }
}).reduce((prev, next) => {
  return prev.then(next)
}, Promise.resolve(''))
```

但是看看用 `for` 会如何？

```javascript
const URLs = [
  'http://www.a.com',
  'http://www.b.com',
  'http://www.c.com',
]
let result
for (const url of URLs) {
  result = await $.ajax(url + result)
}
```

意外的简洁。这归功于数组的有序性，以及 `for` 在语言层面上的可被打断性。

可见在处理顺序的异步请求上，`for` 有着很大的优势，但在并发请求上，还是乖乖用 `.map` 吧（比 `for` + `.push` 要好）。
