---
title: 不使用if或switch，根据类型来创建实例
---

在实际应用中，经常会根据类型，或者枚举，字符串等一切可以代表指定类型的参数来创建实例，如：
```c#
public Base Create(Option option)
{
    Base result = null;
    switch(option)
    {
        case Option.A:
             result = new SubClassA();
             break;
        case Option.B:
             result = new SubClassB();
             break;
        default:
             result = new DefaultClass();
             break;
    }
    return result;
}
``` 

换成if语句，也是同样的复杂度。想减少这样的代码，首先想到的是通过反射来创建。
```c#
 public Base Create(OPtion option)
 {
     string typeName = "SubClass" + option.ToString();
     return (Base)Activator.CreateInstance(Type.GetType(typeName));
 }
``` 

这段代码不但使用了反射，还使用了Convention is better than configuration的原则。即需要满足类名和SubClass+Option的值匹配的惯例。即使以后增加了Base的子类，这段代码也不用修改。其带来的风险性，可以通过测试来排除。

但视乎大家对反射总有点顾虑。想了一下，还可以利用委托来完成。
```c#
public Base Create(Option option)
{
    var dic = new Dictionary<Option, Func<Base>();
    dic.Add(Option.A, () => { return new SubClassA(); });
    dic.Add(Option.B, () => { return new SubClassB(); });
 
    return dic[option]();
}
```
和方法1相比，有了新的子类后，不用增加Case语句，只需要维护字典即可。