python × Qt应用开发 · 0 -- 序
==================================
14 Mar 20 12:46

##python？
python是个很好用的语言，即使学校没有教，很多同学也会自己学。虽然比较少用python来做桌面程序，但是某些情况下需要用到python的数学或者统计功能的时候，又需要用有GUI的程序来交差。个人比较推荐用C#来做桌面应用程序（Windows平台的话），只是VS这庞大的IDE和各种库还是会让某些有独特癖好的人侧目。这个时候用大家都比较喜欢的python就最好啦，搭配Qt来做GUI实在是方便。

于是就有了本系列。对比起其他教程/指南里只有单独的控件编写实例，在系列中我会使用python和Qt真实地编写出一个应用来，一边写应用一边写本系列的博文。途中遇到的问题也会记录下来并且尽量还原解决过程，希望能让读者有开发的真实感。实际上我在自主学习的时候在找资料和控件的测试使用上已经疲于奔命，也希望本系列能够总结出一个比较有效和固定的流程。

##Qt？
其实要在python实现GUI并不一定要使用Qt，python原生自带的Tkinter和下载一个wxPython库也可以。只是Tkinter嘛，做出来的界面实在难看，你看python自带的IDLE就知道了；wxPython嘛，似乎没有什么比较成型的GUI设计工具。Qt的话有一个QtDesigner，比较好用。于是这里就是用Qt了。（实际上是以前写C++的时候GUI用Qt，不想转了= =）

OK，现在选定了Qt之后还有一件事，就是用PyQt还是PySide。为什么有两个呢？嗯，我没有仔细去研究过，Qt本身的历史就比较复杂。这里我选用PySide来，没别的，因为之前的一个科创项目[Micro XenServer Manager](https://github.com/exoticknight/Micro-XenServer-Manager)中已经用了PyQt，这里就尝试使用另外一个。在网上搜寻过相关资料，PySide和pyqt的差别可以说在普通情况下影响不大，所以打算使用pyqt的读者也可以看本系列的博文来共同学习。

至于安装python和PySide，不在这里详述，在Windows下python和PySide的安装就是两个exe文件的事情而已，Linux下的话会用的人比我还专业。注意QtDesigner也要安装上，方法自己Google去。（最新版的PySide似乎会附带上）

系列博文使用的是python2.7.3，PySide1.2.1。