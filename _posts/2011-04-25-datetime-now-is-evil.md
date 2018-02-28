---
title: DateTime.Now is evil
---

和所有的静态方法一样，DateTime.Now在做单元测试的时候是很邪恶的。你无法讲其从依赖的真实环境中剥离出来。如果我们想指定当前时间的话，可以从以下方法入手。

## 使用接口  
```c#
public interface IEnviroment
{
    DateTime GetDateTimeNow();
}
```
优点：接口能在单元测试中使用Mock框架模拟出我们想要的时间。  
缺点：需要实现接口，并在代码中创建接口的实例。

## 使用委托  
```c#
public class ItineraryViewModel
{
    public ItineraryViewModel() : this(DateTime.Now) {}
    public ItineraryViewModel(Func<DateTime> getDateTimeNow)
    {
        _getDateTimeNow = getDateTimeNow;
    }

    private Func<DateTime> _getDateTimeNow;

    public string AlertMessage()
    {
        TimeSpan ts = _geetDateTimeNow() - this.Entitry.StartTime;
        //implementation code
    }
 }
```
优点：和使用接口相比，就是少实现了接口。并且每一个类独享自己获取当前时间的方法。  
缺点：因为每个需要使用当前时间的类，都要维护一个委托，并且由使用者注入进去。所以频繁大量使用的话，还是比较费工作量。  
note：此方法还有变形，如使用方法注入委托或具体的值
```c#
public string AlertMessage(Func<DateTime> getDateTimeNow)
{
    TimeSpan ts = getDateTimeNow - this.Entity.StartTime;
}

public string AlertMessage(DateTime now)
{
    TimeSpan ts = now - this.Entity.StartTime.
}
```

## 使用静态委托  
```c#
public static class SystemUtil
{
    public static Func<DateTime> GetDateTimeNow = () => DateTime.Now;
}
```
优点：因为是静态类方法，所以调用很方便，不用去new。  
缺点：静态方法的顽疾。如果修改了GetDateTimeNow，那么对其他的调用者也有影响，而且调用者还不清楚是否发生了变化。

## 使用Moles框架   
http://research.microsoft.com/en-us/projects/pex/default.aspx   
Moles allows to replace any .NET method with a delegate!!!需要在测试工程中引入Moles，并简单配置下，就可以“替换”DateTime.Now.
```c#
MDateTime.NowGet = () => new DateTime(2000, 1, 1);
```
上面这条语句会让应用中使用DateTime.Now的地方，返回值都是1/1/2000。 

优点：对现有代码没修改，可以像原始的方法一样使用DateTime.Now。  
缺点：暂时还不能在Silverlight中使用。 

通过简单讨论，我们使用的#3方法。目前可以想到的是，在Test Class的Setup方法中，重置方法来减少调用者直接的相互影响。
```c#
[TestInitialize]
public void Setup()
{
    SystemUtil.GetDateTimeNow = () => DateTime.Now;
}
```