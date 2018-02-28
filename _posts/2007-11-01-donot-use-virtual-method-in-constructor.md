---
title: 不要在构造函数中使用虚方法
---

在网上看到一篇文章http://blog.csdn.net/xamcsdn2/archive/2004/08/11/71766.aspx，结果和想象不一样。按样copy代码，编译执行，果不其然。  
再仔细一想，是没有错的。当new一个SubClass时，内存的对象应该是这样:
```
类名：SubClass
变量区：int intCount, int subCount
代码区: (重载后的）Increate
```
在调用SubClass的构造函数时，先调用基类构造函数，基类构造函数再调用Increate，因为被重载了，所以subCount+1，而intCount仍然是0  
然后调用自身构造函数，自身构造函数调用Increate， subCount再次+1，intCount仍然是0
```c#
public class MyClass
{
    public static void Main()
    {
        SubClass sc = new SubClass();
        Console.WriteLine(sc.intCount + "..." + sc.subCount);
    }
}

public class BaseClass
{
    public int intCount=0;

    public BaseClass()
    {
        Increate();
    }

    public virtual void Increate()
    {
        intCount++;
    }
}

public class SubClass: BaseClass 
{
    public int subCount=0;

    public SubClass()
    {
        Increate();
    }

    public override void Increate()
    {
        subCount++;
    }
}
```