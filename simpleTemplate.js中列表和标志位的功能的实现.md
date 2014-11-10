simpleTemplate.js中列表和标志位的功能的实现
========================

在[上一篇文章](http://blog.e10t.net/a-piece-of-simple-javascript-for-template-render/)中，为了一些实际需求，我写了一个非常理想化而基础的模板渲染js代码。但当我尝试将其实际使用的时候，却发现代码中不但问题不少，而且功能也不够，于是就只能继续改进。

> 文中所有代码都截取自js文件，稍有修改。你可以到[github项目](https://github.com/exoticknight/jswarehouse/tree/master/simpleTemplate)上找到完整代码，边对比边看本文。

##绕开split函数

在上一篇文章的更新里面提到，split函数在IE下有问题，只能放弃使用。

其实在使用正则匹配数据域（field，在模板中的形式是`{field}`）的时候，是能够同时获得最近一次匹配到的数据域的位置的。比如：

```javascript
var re=/\{t\}/g;
re.exec('test{t}');  // ["{t}"]
re.lastIndex;  // 7
```

或者去看[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/lastIndex)加深了解。

于是这个`lastIndex`减去匹配出的数据域长度就可以确定数据域前一小节模板文本的结尾，然后对原始模板字符串使用slice函数切割出此一小节模板文本。下面是大概的代码。

```javascript
var lastIndex = 0;
while ( ( mark = fieldRe.exec( str ) ) !== null ) {
    /*
    other code...
    */

    templateText.push( str.slice( lastIndex, fieldRe.lastIndex - mark[0].length ) );
    lastIndex = fieldRe.lastIndex;

    /*
    other code...
    */
}
    
if ( lastIndex < str.length ) {
    templateText.push( str.slice( lastIndex, str.length ) );
}
```



最后记得要检查一下完成匹配后的`lastIndex`值，因为有可能在最后一个数据域后还有一小段模板文本。

好了，现在已经绕开了split函数来将模板分成“文本”和“数据域”两个数组了。

##.表示法

原来的js代码中，作为`field`的命名，只能使用一般的命名，也就是带`_`的英文字母和数字的混合，但是用以填充的json数据不一定是“扁平化”的，换言之有可能是嵌套的，比如`{'a':{'b':1}}`。普通js代码中用`a.b`就能访问`b`的值。在模板解析中，我思考了这么一个方法来实现（如果有更好的方法请告诉我！）

将访问的路径用`.`分开，再逐层赋值，写成代码就是如下：

```javascript
var _getDataViaPath = function( path, json ) {
    var fieldPath = path.split( '.' ),
        data = json,
        index;

    for ( index = 0; index < fieldPath.length; index++ ) {
        data = data[fieldPath[index]];
        if ( data === undefined ) {
            return '';
        }
        if ( data === null ) {
            return index + 1 < fieldPath.length ? '' : null; // maybe not necessary
        }
    }

    return typeof data === 'function' ? data.call( json/* maybe better than 'data' */ ) : data;
},
```

在通过路径访问数据的时候，如果：

1. 路径不存在，返回空字符串
1. 路径存在，返回数据

第二行，以`.`为分割符将访问路径拆成数组。

第六行，通过遍历数组来逐层访问数据，如果途中遇到undefinded，即路径不存在，那么就直接返回空字符（如果直接返回undefined，那么在拼合字符串的时候，调用toString方法会返回字符串`undefined`，显然不是我们想要的）。

然而，路径存在的情况下，数据有可能为`null`，而`null`是不能再读取属性的，于是就看看是不是最后一个路径，不是就返回空字符（因为再走下去路径也不存在了），是就返回数据`null`。这里可能有点绕，并且其实空字符和`null`最后渲染出来的效果是一样的，似乎也没必要这么深究，但还是谨慎地区分一下比较好。

最后，返回数据的时候如果发现数据是函数，那么就执行了之后再返回。执行函数的时候总是要留意这个函数执行的context（上下文），这里给它绑定最顶层的数据好了，在函数体里面它喜欢访问哪个嵌套的数据都行。

> 使用.call()来调用函数，第一个参数是函数的context（上下文）。

然后问题来了，之前识别数据域的正则在加入.表示法功能后就不适用了。重写一下。

```javascript
/\{\s*(([\w\d]+)(\.[\w\d]+)*)\s*\}/gm
```

这样就既能匹配`{field}`，也能匹配`{field.field}`了。

##标志位 & 列表

###模板设计

嗯，不知道这里说“标志位”是否准确，或者大家是否明白我要表达的意思，可能说“flag”会更容易理解？

这里的标志位起这么一个作用，渲染的时候查看这个标志位，根据值（真/假）来决定是否渲染某一小段模板。

看代码：

```markup
<p>{!flag}</p>
<p>{field}</p>
<p>{!flag}</p>
```

这里的`{!flag}`表明渲染的时候先查看一下`flag`的值，如果结果为假，那么两个`{!flag}`所包围着的一小段模板就不渲染了。

至于列表，这样：

```markup
<p>{@list}</p>
<p>{something}</p>
<p>{-list}</p>
```

跟标志位类似，`{@list}`表明渲染的时候遍历`list`列表，每次循环都将`{@list}`和`{-list}`之间的模板渲染一次。

为什么使用`@`和`-`两个不同的符号？因为要支持嵌套，循环的头尾用不同符号便于配对。

```javascript
<p>{@list}</p>
    <p>{@list1}</p>
    <p>{something}</p>
    <p>{-list1}</p>
<p>{-list}</p>
```

###修改正则识别

再次修改识别数据域的正则表达式：

```javascript
/\{\s*([@|\-|!]?)(([\w\d]+)(\.[\w\d]+)*)\s*\}/gm
```

###编写渲染过程

接下来是渲染的过程。先回顾一下一个生成好的模板对象的结构：

```javascript
template = {
    templateText: [],  // 模板文本数组
    fields: [],  // 数据域数组
    data: {}  // 数据对象
}
```

所以模板对象 = 模板文本数组 + 数据域数组 + 数据对象。

我们稍微修改一下数据域数组的结构。

从类似`['field1', 'field2', 'field1']`

改成`[['!', 'field1'], ['', 'field2'], ['!': 'field1']]`

也就是说在每一个数据域中添加一个标识，用来辨别此数据域是否有特殊功能。这在生成模板的时候并不难实现。

当渲染进程遇到一个特殊功能的数据域，那么就应该去定位配对的下一个特殊数据域，两个特殊数据域中间的模板和数据域就需要特殊处理。

我们可以在生成时就记录好这个信息。在模板对象中增加一个`functions`对象，结构如下：

```javascript
functions: {
    loop: {},
    flag: {}
}
```

循环/标记位的首尾就以键值对的方式记录在`loop`/`falg`中，这样在渲染时一查就行。

所以模板对象 = 模板文本数组 + 数据域数组 + 数据对象 + functions对象。

先将渲染函数独立出来成一个内部函数：

```javascript
_render = function( scope, start, end ) {
    var tempFragment = [],
        strings = this.getTemplateText(),
        fields = this.getFields(),
        functions = this.getFunctions(),
        field,
        data,
        i;

    for ( i = start; i < end; i++ ) {
        tempFragment.push( strings[i] );

        field = fields[i];
        data = _getDataViaPath( field[1], scope );
        switch ( field[0] ) {
            case '': // normal data
            tempFragment.push( data );
            break;

            case '@': // begin of list
            break;

            case '!': // begin of flag
            break;

            default:
        }
    }

    tempFragment.push( strings[i] ); // dont forget the last of strings

    return tempFragment.join( '' );
}
```

我来慢慢解释。

`_render`函数定义为传入数据、待渲染数据域开头的下标和待渲染数据域结尾的下标，会返回数据域数组两下标段间（包括开头不包括结尾）的渲染结果（一个字符串）。

开头各种变量定义自不用解释，主体部分是一个遍历，最后返回字符串。

要注意，因为分割原始模板字符串使用的分隔符是不同的数据域，所以分割出来的**模板文本**总是比匹配到的**数据域**数量多1。

> 举个例子，`'sgewgwgw,,seyer,jhrepbo,'.split(/,/g)`，匹配到的分隔符数量为4，分割后文本的数量为5。而任意两个分隔符之间的字符串也可以单独又看作一个待分割的字符串，继续分割后也跟整个字符串具有同样的性质。

画出图来的话就是如下：

![结构图](https://i.imgur.com/4FopsTK.png)

整个函数执行过程看下图：

![执行流程](https://i.imgur.com/ODPd80S.png)

可以看到，循环要做的第一件事是压入模板文本，对应循环体内第一条语句`tempFragment.push( strings[i] );`。剩下的全是在处理数据，可以看到之前写的`_getDataViaPath`函数在这里用上了。最后在循环外压入最后一个模板文本，对应结束循环之后第一条语句`tempFragment.push( strings[i] );`。

来编写数据处理中遇到标志位的情况，对应`switch`语句中`case '!'`：

```javascript
case '!': // begin of flag
if ( data ) {
    tempFragment.push( _render.call( this, scope, i + 1, functions['flag'][i]));
}

// reset index
i = functions['flag'][i];
break;
```

十分简单，按照要求，判断值，再决定是否渲染两标志位间的模板。新加入的`functions`对象就在这里起重要作用了。

最后的`i = functions['flag'][i];`是为了重置当前循环处理的位置（注意这个位置是整个数据域数组中的位置）。这里无需考虑超出下标的问题。

> 为什么不考虑？`functions`对象中指示的位置必须是正确，否则整个渲染过程就毫无运行的必要。

如果遇到列表：

```javascript
case '@': // begin of list
if ( Object.prototype.toString.call( data ) === '[object Array]' ) {
    for ( var loopIndex = 0; loopIndex < data.length; loopIndex++ ) {

        // recursively render
        tempFragment.push( _render.call( this, scope, i + 1, functions['loop'][i]) );
    }
}

// reset index
i = functions['loop'][i];
break;
```

这里需要先判断数据是否为数组，然后遍历数组，循环中渲染两个列表标志间的模板。其实跟标志位的过程差不多。

如果遇到普通数据：

```javascript
case '': // normal data
tempFragment.push( data );
break;
```

直接压入即可。

###编写解析模板的过程

嘿，先别高兴得太早了，虽然编写好了渲染过程，但是渲染是要基于已经生成好的模板的！

别忘了在渲染中指路的重要的`functions`对象是还没有生成出来的！我们刚才只是在假设它已经能工作的前提下编程的！

回到本文一开头切割原始模板字符串的代码中，我们需要在那里为以后的一切铺路。还记得那个处理正则匹配的`while`语句吗？

前方代码高能注意。

```javascript
var templateText = [],
    fields = [],
    lastIndex = 0,
    functions = {
        'loop': {},
        'flag': {}
    },
    flags = [],  // flag stack
    loops = [],  // loop stack
    flag,
    loop,
    mark;

// initial data and index
this.data = {};

while ( ( mark = fieldRe.exec( str ) ) !== null ) {
    /*
     * mark[0] = '{@fo.fo}'
     * mark[1] = '@'
     * mark[2] = 'fo.fo'
     * mark[3] = 'fo'
     * mark[4] = '.fo'
     */
    fields.push( [mark[1], mark[2]] );

    templateText.push( str.slice( lastIndex, fieldRe.lastIndex - mark[0].length ) );
    lastIndex = fieldRe.lastIndex;

    switch ( mark[1] ) {
        case '@':
        loops.push( [mark[2], fields.length-1] );
        break;

        case '-':
        if ( loops[loops.length-1] && loops[loops.length-1][0] === mark[2] ) {
            loop = loops.pop();
            functions['loop'][loop[1]] = fields.length - 1;
        } else {
            return;
        }
        break;

        case '!':
        if ( flags[0] && flags[0][0] === mark[2] ) {
            flag = flags.pop();
            functions['flag'][flag[1]] = fields.length - 1;
        } else {
            flags.push( [mark[2], fields.length-1] );
        }
        break;

        default:
    }
    
}

if ( lastIndex < str.length ) {
    templateText.push( str.slice( lastIndex, str.length ) );
}

if ( flags.length !== 0 || loops.length !== 0 ) {
    return;
}
```

我们最后使用正则表达式是

```javascript
/\{\s*([@|\-|!]?)(([\w\d]+)(\.[\w\d]+)*)\s*\}/gm
```

在执行了`exec`之后，每一次匹配出来的结果都是一个数组，在注释当中我已经明确地指出每一个位置上的内容了。好好记住，开始解释代码。

首先第一句，生成数据域数组。在编写渲染过程一节中我已经说过了这个的数据结构已经改为每一个元素都是“特殊功能符号”和“数据路径”了。这一句非常好理解。

接着的两句就是熟悉的模板文本数组生成，是由于要绕开`split`函数所写。继续看下去。

好了，*数据域数组*和*模板文本数组*处理好了，模板对象 = 模板文本数组 + 数据域数组 + 数据对象 + functions对象，接下来是functions对象。

> 这里使用栈来检查特殊功能数据域是否匹配。

查看一下数据的特殊功能符号，如果遇到的一个列表的头：

```javascript
case '@':
loops.push( [mark[2], fields.length-1] );
break;
```

暂时先将它和它的位置压入栈`loops`。

如果遇到一个列表的尾：

```javascript
case '-':
if ( loops[loops.length-1] && loops[loops.length-1][0] === mark[2] ) {
    loop = loops.pop();
    functions['loop'][loop[1]] = fields.length - 1;
} else {
    return;
}
break;
```

检查栈`loops`中最近一次压入的数据，不存在或者不等于这个列表尾的情况都属于模板格式错误，直接退出。

否则就是匹配成功了，将列表头的位置作为键，列表尾的位置作为值放入`functions`对象的`loop`属性中。

如果遇到标志位：

```javascript
case '!':
if ( flags[0] && flags[0][0] === mark[2] ) {
    flag = flags.pop();
    functions['flag'][flag[1]] = fields.length - 1;
} else {
    flags.push( [mark[2], fields.length-1] );
}
break;
```

查找栈`flags`中最近一次压入的数据，跟本次标志位相等即匹配成功，不相等继续压入。

好了最后再检查一下两个栈是否为空，不为空则有些数据域没有匹配成功，也就是模板格式错误，打回。

```javascript
if ( flags.length !== 0 || loops.length !== 0 ) {
    return;
}
```

呼！写到这里，我都怀疑是不是说得太罗嗦了。画公仔都画出肠了。

##还没完！

其实在渲染列表的时候，只是循环是没有多大意义的。更多时候，我们想输出的是列表中的内容。然而每次循环中需要输出的数据都不一样，怎么破？

再写下去我估计你也不想看了，这个问题在下一篇文章中详细解释。