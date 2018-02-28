---
title: Use generic method to take type parameter of drived type
---

项目中某个页面比较复杂，需要展现不同类型的item。因此有了GiftViewModel, ShowViewModel等，分别对应Gift， Show类型的item，并抽象出了一个基类BaseViewModel。（名称可能不对，不影响要讨论的问题）。所有类型的item放入对应的集合，ObservableColletion<GiftViewModel> giftVMs, ObservableCollection<ShowViewModel> showVMs。现在有个算法要对集合操作，并能在各种item类型的集合中重用。于是就有了这个方法：
```c#
void Foo(ObservableCollection<BaseViewModel> vms)
{}
```
可惜泛型不支持协变性的，即这个方法不接受ObservableCollection<GiftViewModel>或者ObservableCollection<ShowViewModel>类型的参数，以及任何派生于BaseViewModel的ObservableCollection集合。数组是支持协变性的，但我们并不打算把参数改为BaseViewModel[]。关于协变性，可参考《C# in Depth》 – 3. 6. 1节。也可写Helper方法，如把ObservableCollection<GiftViewModel>转换为ObservableCollection<BaseViewMode>，纯体力活。

因为泛型的类型是可以约束的，所以还可以使用泛型方法来达到目的
```c#
void Main()
{
    ObservableCollection<ShowViewModel> vms = new ObservableCollection<ShowViewModel>();
    vms.Add(new ShowViewModel());
    Foo(vms);
}

public void Foo<T>(ObservableCollection<T> vms) where T : BaseViewModel
{
    foreach(T vm in vms)
    {
        vm.Print();
    }
}

public class BaseViewModel
{
    public virtual void Print()
    {
        Console.WriteLine(this.GetType());
    }
}

public class GiftViewModel : BaseViewModel {}
public class ShowViewModel : BaseViewModel {}
```
上面的代码使用LinqPad测试通过。

因为ObservableCollection<T>继承于Collection<T>，而后者又实现了IList， ICollection，IEnumerable。可以通过这些接口获取内容后进行类型转换，以IList为例：
```c#
public Foo(IList vms)
{
    foreach (var vm in arg)
    {
        BaseViewModel bvm = vm as BaseViewModel;
        if (null == bvm)
            throw new ArgumentException("can't handle the type");
        bvm.Print();
    }
}
```
这种方法也可以工作，只不过在编译期间的检查不如第一种强。如传入的参数不是BaseViewModel或其派生类型，则会抛出异常。