CoolShell puzzle game 攻略
=======================

coolshell出了个游戏，网址点[这里](http://fun.coolshell.cn/)，奉上攻略。

##第0关

brainfuck语言，很久以前就觉得有意思。

自己看就算了，直接网上执行一次。点[这个](http://esoteric.sange.fi/brainfuck/impl/interp/i.html)或[这个](http://www.brainfuck.tk/)或[这个](http://copy.sh/brainfuck/)都可以。

得出是welcome.html。

##第1关

2, 3, 6, 18, 108, ?

2×3=6，3×6=18，6×18=108，18×108=1944

输入1944.html知道x=1944。

“生命、宇宙以及任何事情的终极答案”，42。输入42.html知道y=42。

1944×42=81648

得81648.html。

##第2关

那个键盘排布是dvorak方案，不同于我们平常的QWERT键盘方案。

直接上网找[转换](http://wbic16.xedoloh.com/dvorak.html)，那串字符转过来就是

`main() { printf(&unix["\021%six\012\0"],(unix)["have"]+"fun"-0x60);}`

看起来就是C++语言。继续在线编译运行。运行出错，好吧。

按照collshell好像很喜欢IOCCC的尿性，用Google搜一些关键字，比如奇怪的`"&unix"`，记得加上双引号。找到一个比较有用的[解释](http://www.di-mgt.com.au/src/korn_ioccc.txt)。

学以致用，得出答案：unix

##第3关

扫描QR Code，得`[abcdefghijklmnopqrstuvwxyz] <=> [pvwdgazxubqfsnrhocitlkeymj]`。简单的字符替换。

在线解析QR code也可以，[网址](http://zxing.org/w/decode.jspx)。

因为在浏览器下，直接用javascript写脚本了。OK……注意是后一个的字母替换前一个。

```javascript
function decode(text) {
    var newText='';
    for (i=0;i<text.length;i++) {
        if (dict[text[i]]) {
            newText += dict[text[i]];
        } else {
            newText += text[i];
        }
    }
    return newText;
};
dict={};
ciper='abcdefghijklmnopqrstuvwxyz';
alphe='pvwdgazxubqfsnrhocitlkeymj'
for (i=0;i<alphe.length;i++) {
    dict[alphe[i]]=ciper[i];
}
rawText='Wxgcg txgcg ui p ixgff, txgcg ui p epm. I gyhgwt mrl lig txg ixgff wrsspnd tr irfkg txui hcrvfgs, nre, hfgpig tcm liunz txg crt13 ra "ixgff" tr gntgc ngyt fgkgf.';
console.log(decode(rawText));
```

得到一段话：`Where there is a shell, there is a way. I expect you use the shell command to solve this problem, now, please try using the rot13 of "shell" to enter next level.`

就是说要用ROT13来转换`shell`这个字符串，得出是furyy

##第4关

左边就是告诉我们提取字符的模式：

每5个字符，看前两个和后两个，必须满足：

1. 必须含一个大写字母和一个数字
1. 是回文

而提取的字符则必须是小写字母。

`The answer has been lost in the source`则是说密文在源代码里面。有好一大串。

用正则吧，写出来就是`([A-Z])([0-9])[a-z](\2)(\1)|([0-9])([A-Z])[a-z](\6)(\5)`。

抱歉，更高级的写不出来了。

直接开sublime text 2，复制那段大串字符进去，搜索开启正则模式和区分大小写，得：

```
E1v1E
4FaF4
9XrX9
O3i3O
0MaM0
4GbG4
M5l5M
0WeW0
Y0s0Y
```

取中间的variables就是答案。

##第5关

点图片，按后跳转，得一串数字，复制到地址栏，又是一串，好吧这模式都太无聊了。而且页面没有jQuery，先引入。

```javascript
var jq = document.createElement('script');
jq.src = "https://ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js";
document.getElementsByTagName('head')[0].appendChild(jq);
jQuery.noConflict();
```

接着模仿我之前写过的另外一个[攻略](http://blog.e10t.net/answers-for-alibaba-quiz3/)里面的做法。

```javascript
url = 'http://fun.coolshell.cn/n/';
urlreal=url + '32222';
function g() {
    $.get(urlreal, function(data){
        urlreal = url + data;
        console.log(data);
    })
}
```

之后就不断执行`g()`吧，最后答案是tree。

##第?关

最后tree.html跳回一开始了，不知道是不是我太渣已经过时了……

突然想到可能会看reference，于是访问了一下前一个地址：http://fun.coolshell.cn/n/20446，再回头访问tree.html，可以了。

##第6关

算法……我等只会Google的渣渣就棘手了。

好歹用python写了还原树结构的代码，但是它说找deepest path就不懂了，看看有什么人有解释……

最后还是得靠自己画树……借助python代码将节点关系打出来。

```language-python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'draco'

class Node:
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right


def binaryTreeFromInPostOrder(in_order, post_order):
    if len(in_order) == 0 or len(post_list) == 0:
        return None
    node = Node(post_order[-1])
    node_index = in_order.index(node.value)
    node.left = binaryTreeFromInPostOrder(in_order[:node_index], post_order[:node_index])
    node.right = binaryTreeFromInPostOrder(in_order[node_index+1:], post_order[node_index:-1])
    return node


def printLevel(q):
    global tree_structure
    next_level = list()
    this_structure = list()
    while len(q) != 0:
        node = q.pop(0)
        this_structure.append((node[0], node[1].value))
        if node[1].left:
            next_level.append((node[1].value + "-l", node[1].left))

        if node[1].right:
            next_level.append((node[1].value + "-r", node[1].right))

    tree_structure.append(this_structure)

    return next_level


def printTree(tree):
    current_level = list()
    n = 0

    current_level.append(("None", tree))

    while n < 14:
        current_level = printLevel(current_level)
        n += 1

in_list   = "T b H V h 3 o g P W F L u A f G r m 1 x J 7 w e 0 i Q Y n Z 8 K v q k 9 y 5 C N B D 2 4 U l c p I E M a j 6 S R O X s d z t".split(" ")

post_list = "T V H o 3 h P g b F f A u m r 7 J x e w 1 Y Q i 0 Z n G L K y 9 k q v N D B C 5 4 c l U 2 8 E I R S 6 j d s X O a M p W t z".split(" ")

tree_structure = list()

tree = binaryTreeFromInPostOrder(in_list, post_list)

printTree(tree)

for i in tree_structure:
    print(i)
```

输出结果：

```
[('None', 'z')]
[('z-l', 'W'), ('z-r', 't')]
[('W-l', 'b'), ('W-r', 'p')]
[('b-l', 'T'), ('b-r', 'g'), ('p-l', '8'), ('p-r', 'M')]
[('g-l', 'h'), ('g-r', 'P'), ('8-l', 'L'), ('8-r', '2'), ('M-l', 'I'), ('M-r', 'a')]
[('h-l', 'H'), ('h-r', '3'), ('L-l', 'F'), ('L-r', 'G'), ('2-l', '5'), ('2-r', 'U'), ('I-r', 'E'), ('a-r', 'O')]
[('H-r', 'V'), ('3-r', 'o'), ('G-l', 'u'), ('G-r', 'n'), ('5-l', 'v'), ('5-r', 'C'), ('U-l', '4'), ('U-r', 'l'), ('O-l', 'j'), ('O-r', 'X')]
[('u-r', 'A'), ('n-l', '0'), ('n-r', 'Z'), ('v-l', 'K'), ('v-r', 'q'), ('C-r', 'B'), ('l-r', 'c'), ('j-r', '6'), ('X-r', 's')]
[('A-r', 'f'), ('0-l', '1'), ('0-r', 'i'), ('q-r', 'k'), ('B-l', 'N'), ('B-r', 'D'), ('6-r', 'S'), ('s-r', 'd')]
[('1-l', 'r'), ('1-r', 'w'), ('i-r', 'Q'), ('k-r', '9'), ('S-r', 'R')]
[('r-r', 'm'), ('w-l', 'x'), ('w-r', 'e'), ('Q-r', 'Y'), ('9-r', 'y')]
[('x-r', 'J')]
[('J-r', '7')]
[]
```

然后手动画树！最后得到最深路径是zWp8LGn01wxJ7。

Linux下执行：

`echo U2FsdGVkX1+gxunKbemS2193vhGGQ1Y8pc5gPegMAcg=|openssl enc -aes-128-cbc -a -d -pass pass:zWp8LGn01wxJ7`

得到nqueens。终于过了。

##第7关

好吧八皇后。说实话我从来都不太想在做类似的游戏时写算法，又不是什么ACM做题，唉。直接Google N皇后算法的python算法，解出N=9的所有解，然后

只是它这个code和通常表示八皇后解的数字不一样，正常是从上到下从左到右记录，它这个是从下到上从右到左记录……

暴力出全部解之后其实都一样啦。

结合sha1解密验证的代码：

```language-python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'draco'

from itertools import *
import hashlib

c = (1,2,3,4,5,6,7,8,9)
nq = [v for v in permutations(c) if 9==len({v[i-1]+i for i in c})==len({v[i-1]-i for i in c})]
for q in nq:
    code = "".join(map(str, q))
    s = hashlib.sha1()
    s.update("zWp8LGn01wxJ7" + code + "\n")
    if s.hexdigest() == "e48d316ed573d3273931e19f9ac9f9e6039a4242":
        print(code)
        break
```

得出答案：953172864

##第8关

好吧26进制

A=1, B=2, ... , Z=26

于是AA=26^1*1+1, ZZ=26^1*26+26, AAA=26^2*1+26^1*1+1, ...

算出来是85165，然后还要转字符……是DUYO。

##第9关

根据图片搜出来是Pigpen Cipher，上wiki查到对应英文。

答案是helloworld。

啊终于通关了，看top100，排55。