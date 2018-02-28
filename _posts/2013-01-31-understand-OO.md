---
title: Re:如此理解面向对象
---

在酷壳上看到的一篇文章[如此理解面向对象](http://coolshell.cn/articles/8745.html)，吐槽一个大师的OO例子。那个例子的确糟糕，倒不是说杀鸡用了牛刀，过度设计等等。毕竟只是一个例子，来说明如何面向对象编程。但就其面向对象实现来说，我觉得有几点真得需要商榷。我画了一个依赖关系图，比较简陋将就看吧。  
![OO-Example](/images/understanding-oo-example.jpg)  

1. 类工厂的实现  
OSDiscriminator类不但依赖了BoxSepcifier接口，同时还依赖了BoxSepecifier接口的实现WindowsBox, UNIXBox和MacBox。 
作为类工厂这也说的过去。但这也意味每增加一个BoxSepecifier的实现就需要修改OSDiscriminator类！类似的问题我们以前在Blog上也讨论过，比较实践的做法我觉得还是1)数据驱动；2)依靠惯例/约定；3)IOC。
双向依赖   
WindowsBox, UNIXBox和MacBox同时也依赖OSDiscriminator，形成了双向依赖！如果我们把这些类放入到不同的组件中，那么编译就都会有问题了。双向依赖往往说明了设计时的抽象或者类耦合上出现了问题。保持单向依赖，是保持设计简单的一个重要原则。
2. 环境上下文依赖  
OSDiscriminator.getBoxSpecifier里面直接调用了System.getProperty。要是写单元测试，这下又要抓头了。问题是作者把这个类标注为Factor Pattern。按我的想法至少得把key作为参数，从外部传进来。这样就减少了和System的耦合。  
3. 抽象不完全  
除DefualtBox外，其余BoxSpecifier的实现都有一个静态方法register。这也导致了OSDiscriminator中多了两句，if (value == null) return DefaultBox.value。其实只要DefaultBox中加一个register不就解决了，还保持了实现的一致性。不过因为register是静态方法，这种一致也只能依靠人工。按我的想法，register应该是个虚方法，只返回key信息，由外部进行注册，自然方法名也因此需要修改。
4. 静态方法  
单例就不说了，这个单例实现真的很脆弱。我是想说register方法。那WindowsBox为例，如果register是虚方法，我们完全就可以继承实现Windows95Box，WindowsNTBox，重载register和getStatement。OO中静态方法的使用一定要慎重，它是对扩展性的一种破坏。

我觉得原文的实现在类的职责识别和定义， 类耦合关系上还真得是需要重新考虑下。