---
title: segmentfault光棍节闯关攻略
categories:
  - [技术, 黑客]
  - [编程语言, javascript]
tags: [11.11, segmentfault, walkthrough, quiz]
permalink: segmentfault-1111-quiz-walkthrough
id: 25
updated: '2014-04-15 15:39:31'
date: 2014-01-23 03:32:59
---

### 第一关/第二关
密码都在页面的源代码里面。

### 第三关
页面里面啥都没有，js和css文件也没有，还能藏东西的地方是cookie和localstorage什么的？最后发现http header里面有个the-key-is，好吧……

### 第四关
第三关的时候就发现get的k=值好像是加密的字符串，32位，猜测应该是md5，不过第三关的值去解密解不出来。这关解出来是4，那么将5加密后复制到地址栏就是啦。

### 第五关
二维码……这么显然，扫出来果然是个坑。又是什么都没有，唯一能用的资源就是那图片了，嘛以前玩过类似的游戏的都知道密码基本就在图片里面，下载回来用notepad差不多在文件末尾就看到了。

### 第六关
f4de502e58723e6252e8856d4dc8fc3b，扔去解密竟然要收钱，靠，直接Google之，找不到什么有用的，百度之，找到了……就是这一串：1573402aa6086d9ce42cfd5991027022，替换后下一关。

### 第七关
按照提示继续Google/百度，啥都没有，于是，直接将那串东西替换到地址栏，过关。

### 第八关
将get方法改成post方法提交……我的做法是直接用chrome改页面代码。

### 第九关
一大坨1和0，八个一组，有的有空缺，二进制嗯。大概看了一下，左边第一位都是0，可能是ascii码。应该能转出字符来，空缺的地方补1（这里比较坑，补0是不行的）。写段js搞出来看看。
```javascript
var tag= document.getElementsByTagName('pre ')[0].innerText.replace(/_/g, '1 ').split(/\n|\s/g).forEach(function(ele){tag=tag+String.fromCharCode(parseInt(ele,2))})
```

最后tag是一串东西，注意到有+、/还有最后有一个=，哈哈哈，应该是base64了。

上http://www.opinionatedgeek.com/dotnet/tools/base64decode/ 解出一个文件。

接下来应该是判断文件类型了咯？估计也是gzip什么的压缩包，手头没工具，直接丢sublime text里面，搜索了一下文件开头的1f8b 0800，似乎真的是gzip的格式（以后要记住哈）。7zip打开后看见里面有图片，解压出来看知道下一关了。顺便说图片里面的似乎蒼井そら？
