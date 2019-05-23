---
title: 'C#实现简单的控件动画'
categories:
  - [编程语言, C#]
tags: [C#]
permalink: csharp-simple-animation-component
id: 2
updated: '2014-04-15 15:37:21'
date: 2014-01-23 03:01:34
---

在学习.net编程的时候，做作业经常要求有一个界面。而在做局域网五子棋游戏这个作业的时候，客户端的界面既有登录界面又有用户列表界面，做一个动画来切换就再好不过了。不过翻查了一下MSDN和一些资料，对于C#怎么实现控件的动画效果都没怎么提及，于是参考了一下javascript在网页上实现动画的原理，自己尝试着写了。

动画原理什么的不详细解释了，大家都知道是每一张静态的图片在同等时间间隔内快速地播放，利用视觉停留现象造成一系列视觉印象，从而出现会动的感觉。

而应用到控件的动画上又是如何呢？从最简单的直线匀速移动开始考虑吧。比如，需要将实现一个标签从距离窗体左方30px移动到距离左方300px，在500毫秒内完成，那么，就需要具体计算出每一个时间间隔需要将这个标签移动多少距离，然后在每经过这样的一段时间之后，通过改变控件的距离左边的属性的值来实现控件的微小移动。只要在500毫秒内移动的次数足够的多，那么控件看起来就是在连续地移动。

OK，其中重要的概念有几个。1、将那最小的一小段时间叫做一帧，也就是说整体的移动是通过一帧一帧的小移动叠加出来。2、时间控制。需要一个函数，在每一帧将控件的属性重新设置。这里可以通过使用计时器来实现。3、动画状态。需要将当前动画进行的状态记录下来，以便下一帧到达的时候被更改到下一个帧的状态。

了解了基本原理之后，下面开始编写代码。

首先定义一个存储动画状态的类AnimationStatus，其中：_attribute是控件的属性名称，_initValue是控件动画前的值，_endValue是控件动画后的最终值，_totalValue是整个动画变化的值，_totalFrames是动画所有帧的数量，_currentFrames是代表动画进行到多少帧。将字段封装好，能从外面修改的只有_currentFrames。构造函数没什么好说的，就是初始赋值，注意_totalValue是正数而_currentFrames默认是1。
```csharp
/// <summary>
/// A class that store a set of animation of the control
/// </summary>
class AnimationStatus
{
	AnimationType _animationType;
	string _attribute;
	int _initValue;
	int _endValue;
	int _totalValue;
	int _totalFrames;
	int _currentFrames;

	/// <summary>
	/// type of the animation, such as liner, Ease...
	/// </summary>
	public AnimationType AnimationType
	{
		get { return _animationType; }
	}
	/// <summary>
	/// attribute of control that the contrl will change
	/// </summary>
	public string Attribute
	{
		get { return _attribute; }
	}
	/// <summary>
	/// current value of the attribute that is ready to change
	/// </summary>
	public int InitValue
	{
		get { return _initValue; }
	}
	/// <summary>
	/// final value of the attribute that is ready to change
	/// </summary>
	public int EndValue
	{
		get { return _endValue; }
	}
	/// <summary>
	/// total value that changed
	/// </summary>
	public int TotalValue
	{
		get { return _totalValue; }
	}
	/// <summary>
	/// total frames the animation should play, READONLY
	/// </summary>
	public int TotalFrames
	{
		get { return _totalFrames; }
	}
	/// <summary>
	/// current frames the animation has played
	/// </summary>
	public int CurrentFrames
	{
		get { return _currentFrames; }
		set { _currentFrames = value; }
	}

	// contructor
	public AnimationStatus(string attribute, int initValue, int endValue, int totalFrames, AnimationType animationType)
	{
		this._attribute = attribute;
		this._animationType = animationType;
		this._initValue = initValue;
		this._endValue = endValue;
		this._totalValue = Math.Abs(this._endValue - this._initValue);
		this._totalFrames = totalFrames;
		this._currentFrames = 1;
	}
}
```

接下来写的是处理每一帧的通用动画函数。
```csharp
/// <summary>
/// common function of moving control
/// </summary>
/// <param name="contorl">the control to be moved</param>
/// <param name="timer">the timer that control the time of animation</param>
/// <param name="animationStatue">current statue of animation</param>
private static void Animate(
	System.Windows.Forms.Control contorl,
	System.Windows.Forms.Timer timer,
	AnimationStatus animationStatue)
{
	if (contorl == null
		|| contorl.IsDisposed
		|| animationStatue.CurrentFrames > animationStatue.TotalFrames)
	{
		timer.Enabled = false;
		return;
	}
	// perform animation
	Type _tp = contorl.GetType();
	System.Reflection.PropertyInfo _pi = _tp.GetProperty(animationStatue.Attribute);
	if (_pi != null)
	{
		double _progress = (double)animationStatue.CurrentFrames / (double)animationStatue.TotalFrames;
		int _newValue =
			animationStatue.InitValue < animationStatue.EndValue ?
			animationStatue.InitValue + Convert.ToInt32(Math.Round(animationStatue.TotalValue * CalculateValue(animationStatue.AnimationType, _progress))) :
			animationStatue.InitValue - Convert.ToInt32(Math.Round(animationStatue.TotalValue * CalculateValue(animationStatue.AnimationType, _progress)));
		_pi.SetValue(contorl, _newValue, null);
	}
	else
	{
		timer.Enabled = false;
		return;
	}
	animationStatue.CurrentFrames++;
}
```

一开始的判断是防止被操作的控件已经销毁了，并且在动画已经播完了的时候将定时器暂停并返回。从”// perform animation”这个注释之后的代码应用到C#的反射机制。为什么？简单来说就是因为要写出一个通用的动画函数，所以不能在函数里面事先规定要修改的Control对象的属性，但是在实际运行的时候还是需要得到Control对象的具体属性才能修改其中的值，于是使用反射来得到这个对象的某个属性，进而修改这个属性的值。

那么这个“值”如何计算出来呢？首先需要得知动画现在的进度，于是就用当前的动画进度（animationStatue.CurrentFrames）除以总进度（animationStatue.TotalFrames）。然后是利用这个进度（一个百分比）计算出在此进度下属性的值，只要在这里稍微调整计算的方程y=f(x)，就能实现各种各样的效果了，例如最常见的线性和回弹效果。这里的函数CalculateValue就是做这样的事情，它根据传入的AnimationType决定使用什么方程，而进度_progress作为方程的x值，然后返回计算后的y值，这个y值同样是一个百分比，要和动画总变化值（totalValue）相乘，最后加上属性初始值就是当前进度的动画变化值。注意这里设定的方程定义域应该是(0,1]，而值域则最好是从零开始，最后当x=1时，y=1，这样动画效果才不会奇奇怪怪。还有就是属性值的变化不一定是从小到大的所以要做判断决定是加还是减。

CalculateValue函数：
```csharp
private static double CalculateValue(AnimationType animationType, double x)
{
	double _y = 1;
	switch (animationType)
	{
		case AnimationType.Liner:
			_y = x;
			break;
		case AnimationType.Ease:
			_y = Math.Sqrt(x);
			break;
		case AnimationType.Ball:
			_y = Math.Sqrt(1.0 - Math.Pow(x - 1, 2));
			break;
		case AnimationType.Resilience:
			_y = -10.0 / 6.0 * x * (x - 1.6);
			break;
	}
	return _y;
}
```

OK，现在可以写实际动画的函数了。以下是水平移动的例子：
```csharp
public static void HorizontalMove(
	System.Windows.Forms.Control control,
	int endLeft,
	int lastTime,
	AnimationType animationType)
{
	System.Windows.Forms.Timer _timer = new System.Windows.Forms.Timer();
	_timer.Interval = 15; // CAUSION, this value may not work for you
	int _frames = lastTime % _timer.Interval > 0 ? lastTime / _timer.Interval + 1 : lastTime / _timer.Interval;
	AnimationStatus animationStatue = new AnimationStatus("Left", control.Left, endLeft, _frames, animationType);
	_timer.Tick += delegate { Animate(control, _timer, animationStatue); };
	_timer.Enabled = true;
	_timer.Start();
}
```

做的事情就是初始化定时器、动画状态，然后将通用动画函数和参数作为一个委托绑定到定时器上，打开定时器。

时间控制使用System.Windows.Forms.Timer这个定时器。C#中有三种定时器，这个是唯一一个单线程执行的定时器，也就是说会在UI主线程上执行绑定触发事件，而且它的精度是55ms，也就是说不能很精确地控制动画的进度。最重要的一点，因为是在UI主线程上执行的关系，如果绑定的触发事件执行时间过长，会造成UI假死。但是这里做的控件动画是应用在界面切换或者为控件增加小型动态效果上，动画量少而且持续时间不长，途中的假死是可以无视的。

最后来看一下效果吧。
GUI：
![GUI图](http://i.imgur.com/yvLJGnP.jpg)

按钮事件：
```csharp
private void button1_Click(object sender, EventArgs e)
{
  ControlAnimation.HorizontalMove(label1, 300, 500, AnimationType.Resilience);
}
```

动画效果：
![动画效果](http://i.imgur.com/YKoP3CJ.gif)
