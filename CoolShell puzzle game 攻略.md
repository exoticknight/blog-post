CoolShell puzzle game 攻略
=======================

coolshell出了个游戏，网址点[这里](http://fun.coolshell.cn/)，第一时间奉上攻略。

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

