---
title: Color Color problem
---

在最近一次code review中，再次提到了属性名称和类型名称一样，降低了可读性的问题。i.e., 
```c#
public ActivityViewModel ActivityViewModel {get; set;} 
```
那么当看到ActivityViewModel时，会先反应下到底是类还是类实例。如ActivityViewModel.DoSomething()到底是实例方法还是静态方法，无法一眼立即辨别出来。

我们最初想的是在命名上做点文章，如：
```c#
ActivityViewModel ActivityViewModelInstance;
ActivityViewModel ActivityVM;
```
虽然将类型和变量区别了，但大家对这样的命名视乎并不感冒。

上网搜索了下，see http://blogs.msdn.com/b/ericlippert/archive/2009/07/06/color-color.aspx. 这个就是所谓的Color Color问题的（悲催的是我怎么没查出Color type有个公共属性Color？）。不知道文章中的两个问题大家答对了几个，反正我是都错了…下面的评论也有意思，有在澳大利亚的说用别名 Color Colour，有提醒说namespace用复数的…

首先名字还是具有一定业务意义的好，如ActivityViewModel SelectedActivityViewModel。如果实在是不好取，我觉得ActivityViewModel ActivityViewModel也能接受，因为没有更好的办法:(。但是在使用属性的时候，最好加上this关键字区别。如this.ActivityViewModel.DoSomething()。这样很清楚这是个类实例变量，而非类。在VS中，写了this后，借助智能感知码字速度也要快点。