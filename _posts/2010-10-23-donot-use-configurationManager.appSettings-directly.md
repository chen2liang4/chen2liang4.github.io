---
title: 避免在代码中直接任意使用ConfigurationManager.AppSettings
---

ConfigurationManager.AppSettings可以很方便的获取应用程序配置文件中的内容，如Web.Config。这也导致了在代码中，经常能看见对ConfigurationManager.AppSettings的随意调用。  

我们**不能假设配置总是正确的**，假设配置项是一定存在的，配置项的值一定是正确的。因为手工编辑配置文件，本身没有有效的验证机制，全靠人员自己掌控。当这种不可靠的假设不成立的时候，ConfigurationManager.AppSettings不会给你一个准确的参考。如下面这行代码：
```c#
int MarginX = int.Parse(ConfigurationManager.AppSettings["MarginX"]);
```
如果没有配置MarginX的话，会得到一个ArgumentNullException；如MarginX配置的不是一个数字，会得到一个ArgumentException。这时也不要假设能很快地定位到问题所在，在日志中看到这样的Exception，是不会明白其实这是一个简单的配置错误；也不是什么环境下都能让我们去debug，打个断点就能跟踪进去。所以我们可以对这种操作进行封装一下：
```C#
public class ConfigSetting
{
    public static int GetMarginX()
    {
        return GetIntAppSettings("MarginX");
    }

    private static String GetStringAppSettings(string name)
    {
      if (string.IsNullOrEmpty(ConfigurationManager.AppSettings[name]))
      {
        throw new ConfigurationErrorsException(string.Format("Missing ConfigSetting \"{0}\"", name));
      }
      return ConfigurationManager.AppSettings[name];
    }
 
    private static int GetIntAppSettings(string name)
    {
      string value = GetStringAppSettings(name);
      int result = 0;
      if (!Int32.TryParse(value, out result))
      {
        throw new ConfigurationErrorsException(string.Format("Invalid value in ConfigSetting \"{0}\"", name));
      }
      return result;
    }
}
```
这样，我们就能得到比较清楚的错误提示了。  

此外，类不应依赖配置项。在类中处处调用ConfigurationManager.AppSettings，会导致类对配置文件的依赖。比如写单元测试代码时，又得去维护一个配置文件。其实配置项大多配置的是某个对象的属性默认值。我们应该用一个配置类或接口读取配置项值，再注入到要使用配置项的类中。至于采取构造注入，属性注入还是方法注入，就看具体情况了。普通类最好也不要去依赖配置类或配置接口，这种依赖也无太多业务意义。