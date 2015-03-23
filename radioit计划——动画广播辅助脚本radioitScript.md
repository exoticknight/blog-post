radioit计划——动画广播辅助脚本radioitScript
===========================

在[这篇文章](http://blog.e10t.net/how-to-save-internet-radio/)的实践基础和[这篇文章](http://blog.e10t.net/append-a-python-script-for-last-post-about-saving-internet-radio/)的代码基础上，我重新将代码整理一下，并添加了一些功能，将原来三个脚本的共同点提炼，写成一个网络广播查看和获取的框架。继而在这个框架上，写出对应三个广播站的脚本的ver 2.0。

原来的脚本的功能就不再详述了，来说说一下新版脚本三大功能，分别是：探索广播、下载广播和查看广播。

探索广播是指能够列出某一个广播站上所有的/当天的/星期x的/最新的广播，旨在能够帮助使用者发现自己喜欢的广播和新推出的广播。

下载广播是指能提取出广播音频/图片的地址，供第三方播放器播放或保存。

查看广播是指能够列出某广播的主要信息：包括更新日期、最新一期的内容、主持人等。

基本可以说，有了新版的脚本，基本就不需要用浏览器浏览广播站的网页了。并且，因为脚本只是需要抓单个页面和将网页内容整理好再输出，所以对比起网页，能更快更高效地呈现有用的信息。

##子命令

对比旧版，新版脚本的一大改进是引入了子命令。最直观的反映就是，调用方式的不同。

旧版
```language-python
python www.py -x -y zzz
```

新版
```language-python
python www.py xxx -y zzz
```

就是类似`git`命令行的那样，`argparse`库非常给力地支持这种方法。

重点是使用`add_subparsers`函数和`add_parser`函数，详细使用看[文档](https://docs.python.org/2/library/argparse.html)和[github上的代码](https://github.com/exoticknight/radioitScript/blob/master/_radioit_script_template.py)。

多亏了能够这样嵌套命令，即使再多的功能也能变得清晰分明。

##整体框架

ver 2.0 的起点是一个脚本框架，承载参数的解释和自定义函数调用的重要功能。

脚本的一次执行基本流程是：

入口 → 解析参数 → 调用自定义函数 → 执行自定义函数

前三个都能够定下来，不同的广播站只需要各自填充自定义函数就可以了。

最简框架代码如下：

```language-python
"""自定义函数
"""
def foo():
    pass


"""子命令和参数跳转
"""
def process(option):
    if option.sp_name == "www":
        if option.xxx:
            foo()


"""入口
"""
if __name__ == "__main__":
    parser = argparse.ArgumentParser(version="2.0")

    # 子命令
    sp = parser.add_subparsers(title="commands", description="support commands", help="what they will do", dest="sp_name")

    # 添加子命令
    sp_www = sp.add_parser("www")

    # 添加子命令参数
    sp_www.add_argument("-x", "--xxx", action="store_true", dest="xxx")

    try:
        args = parser.parse_args(sys.argv[1:])
    except argparse.ArgumentError, e:
        print("bad options: {0}".format(e))
    except argparse.ArgumentTypeError, e:
        print("bad option value: {0}".format(e))
    else:
        process(args)
```

##自定义函数骨架

框架还基于库`urllib`、`urllib2`、`argparse`和`bs4`，制定了自定义函数的骨架。

脚本做的事情，无非就是获取网页资源，接着在html结构中筛选有用信息，然后格式化成文本输出。因此，自定义函数也可以制定出骨架，不同的广播站就只需要指定筛选的规则就行了。

```language-python
# 获取网页
try:
    soup = BeautifulSoup(urllib2.urlopen(u"http://url", timeout=60)) # 1
except Exception, e:
    handle_error(e, "Network Error.")
    return

# 筛选
content = soup.select("html") # 2

# 组织内容
table = [(u"ID", u"Name")] + [a for a in content] # 3
# 格式化文本
text = prettify_table(table)

# 输出
print(text.encode("gb18030"))
```

虽说骨架是定下来了，但在特殊函数比如下载函数就需要另外编写。

#总结

虽然新版脚本框架是为了radioit计划而写的代码，但是其中`argparse`模块的使用的代码也是能被借鉴在其他需要带参数运行的python脚本中。

新版对比旧版，增加了功能，但是也增加了使用的复杂度和代码的长度。旧版还是可以保留下来，供日常快速使用。