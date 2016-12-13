说说牛客上的一道 JavaScript 题目
=======================

牛客上有这么一道 JavaScript 的[题目](https://www.nowcoder.com/questionTerminal/9c76e58c2ce94eb9b8168b43adef4f50)。

```javascript
//填写内容让下面代码支持a.name = “name1”; b.name = “name2”;
function obj(name){
    【1】
}
obj.【2】 = "name2";
var a = obj("name1");
var b = new obj;
```

【1】和【2】是填写的内容，【2】的答案是 `prototype.name`，没争议。

问题是【1】，参考答案居然是 `if(name){ this.name = name;}return this;`，这么随便地玩弄 `this` 不就是明摆着污染全局变量吗？暴力赋值不可取。

下面的一些高票讨论还说了一大堆解释的废话，连他自己都说自己好罗嗦。对，你不但罗嗦，而且还没有改错。注释里都说了给 window 的属性赋值，还不自知出问题，真是误人子弟。

先来分析一下题目，a 和 b 都从 obj 来，为什么同名的属性值不一样？可以看出，是对 obj 这个函数的调用方式不一样，a 是 obj 函数的调用结果，而 b 则是 obj 作为*构造函数*调用的结果。所以这题的重点应该是如何区分_函数调用_和_构造函数调用_。

一个关键字 `new` 决定了不同。`new` 的作用是什么呢？[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new) 上说了，面试也会考你的，简单来说是三步，`new foo`：

1. 生成一个继承于 foo.prototype 的对象
2. foo 会被调用，其中的 `this` 值会被绑定为 1 中的对象
3. 如果 foo 没有返回一个对象（注意是对象！），则返回 1 的对象

从 2 就可以看出 `this` 值会被 `new` 绑定为一个确定的对象，而不是像普通函数调用中那样自己不可预料，要看上下文的进程。

于是就可以在这里做文章。先来判断 `this` 的值。

```javascript
if (this instanceof obj) {}
```

`instanceof` 会检查 `this` 的原型链上是否存在 `foo.prototype`。也就是说能判断是否满足第 1 条，确保了对象能从 `prototype` 中读取到 `name` 属性。（毕竟代码中并没有给 b 的赋值中传入）

>`instanceof` 并不是完美的判断方法，但是在这里足够了，后面会谈到这个问题。

```javascript
if (this instanceof obj) {
    // new 调用
} else {
    // 非 new 调用
    return {
        name: name
    }
}
```

非 new 调用的情况下，直接返回一个新对象就 OK 了。

而在 new 调用的情况下，可以看到 `function obj(name)` 定义的时候是有参数的，调用的时候却没参数，这就要小心了，为了安全起见，还是判断一下为妙。

```javascript
if (this instanceof obj) {
    // new 调用
    if (name !== undefined) {
        this.name = name
    }
} else {
    // 非 new 调用
    return {
        name: name
    }
}
```

一般来说，判断会写成 `if (name)`，但是碰到 `null`、`0`、`false` 就 GG 了，所以还是谨慎点吧。

问题到这里就可以比较完美地解答了。

##bonus: instanceof 的问题
『`instanceof` 会检查 `this` 的原型链上是否存在 `foo.prototype`』，为什么说得这么拗口，是因为需要表达出 `instanceof` 本来就不是真的用来检测是否调用 `new` 的方法。

在题目里面，要求的是 a 需要从原型链上读取到特定的属性值，所以 `instanceof` 的作用刚好在这里能符合要求而已。

函数调用除了题目中的方法还有第三种方法，那就是 `foo.call`、`foo.apply`，而且也能为函数指定 `this` 的值（所以还有 `bind`）。因此是存在方法调戏 `instanceof` 的。

```javascript
foo.prototype.name = 'foo'
var midman = new foo('fake foo')
var a = foo.call(midman)
var b = foo.call(midman, 'b')
a  // undefined, WTF?!
b  // undefined, WTF?!
```

这里的 `foo` 调用的方式是作为函数来调用，但是为 `this` 绑定的值是从 `foo` 上 `new` 出来的，换句话说，其原型链上存在 `foo.prototype`，于是就骗过了 `instanceof`。

于是 ES2015 来搭救你了，新增了一个 `new.target`。于是修改成：

```javascript
if (new.target !== undefined) {
    // new 调用
    if (name !== undefined) {
        this.name = name
    }
} else {
    // 非 new 调用
    return {
        name: name
    }
}
```

[1]: 
[2]: 