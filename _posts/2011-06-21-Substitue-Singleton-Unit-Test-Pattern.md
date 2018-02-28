---
title: Substituted Singleton Unit Test Pattern
---

Singleton用的比较广了，一是因为节省资源，二是使用方便。但由于Signleton对象的实例往往是对外界不可修改的，这样造成了一种强依赖性，导致单元测试困难。其实也是造成了代码扩展的潜在问题。直接上代码吧
```c#
public class TripManager
{
    private static TripManager _tripManager;
 
    private TripManager() {...}
 
    public static TripManager Instance
    {
        get
        {
            if (null == _tripManager)
            {
                _tripManager = new TripManager();
            }
 
            return _tripManager;
        }
        private set
        {
            _tripManager = value;
        }
    }
 
    private TripRecord CurrentTrip { get; set; }
 
    public void GetCurrentTrip(Action<TripRecord> onGetTrip)
    {
        //retreive data from WCF services.
    }
}
```
这段代码为了节省版面有删节。同时，也是典型的一个Singleton实现。当我们想使用一个自定义的CurrentTrip时，发现很难做。因为CurrentTrip是私有的，而GetCurrentTrip会触发WCF调用，开销很大。我们并不想把CurrentTrip暴露出来，这样代码的封装性就被破坏了。Singleton和静态类相比，最大区别就是在于Singleton是可以继承的。利用这点，就可以解决这个问题。首先TripManager需要做点修改，见注释部分。主要是为了子类能访问这些基类对象。
```c#
public class TripManager
{
    // change privete to protected
    protected static TripManager _tripManager;
 
    // change privete to protected
    protected TripManager() {...}
 
    public static TripManager Instance
    {
        get
        {
            if (null == _tripManager)
            {
                _tripManager = new TripManager();
            }
 
            return _tripManager;
        }
        private set
        {
            _tripManager = value;
        }
    }
 
    private TripRecord CurrentTrip { get; set; }
 
    // add virtual for overwrite
    public virtual void GetCurrentTrip(Action<TripRecord> onGetTrip)
    {
        //retreive data from WCF services.
    }
}
 
```
然后实现一个子类，专为单元测试使用。
```c#
public class TripManagerStub : TripManager
{
    private TripManagerStub()
    { }
 
    public static TripManagerStub GetInstanceStub()
    {
        _tripManager = new TripManagerStub();
        return (TripManagerStub)_tripManager;
    }
 
    public TripRecord CurrentTrip { get; set; }
 
    public override void GetCurrentTrip(Action<TripRecord> onGetTrip)
    {
        onGetTrip(CurrentTrip);
    }
 
    public static void RemoveStub()
    {
        _tripManager = null;
    }
}
```
在GetInstanceStub方法中，基类TripManager的实例对象被替换成了子类对象。子类中的CurrentTrip和基类中的毫无关系，你要是喜欢可以使用其他名字。我们有这样一个使用TripManager的方法
```c#
//OfferDeclineViewModel class
public void Initialize()
{
    ConfirmCommand = new DelegateCommand(this.OnConfirm, () => true);
    TripManager.Instance.GetCurrentTrip((tripRecord) =>
    {
        this.CurrentTrip = tripRecord;
    });
}
```
现在看测试代码怎么写，测试给出一个TripRecord对象，在执行Initialize方法后，OfferDeclineViewModel对象的CurrentTrip应该和给定的对象是同一实例。
```c#
[TestMethod]
public void Initialize_GivenATripRecord_CurrentTripEqualsGivenInstance()
{
    //Arrange
    TripRecord expected = new TripRecord();
    var tripMgrStub = TripManagerStub.GetInstanceStub();
    tripMgrStub.CurrentTrip = expected;
 
    //Act
    var vm = new OfferDeclineViewModel();
    vm.Initialize();
 
    //Arrange
    Assert.AreSame(expected, vm.CurrentTrip);
}
```
此时执行OfferDeclineViewModel.Initialize()时，执行到TripManager.Instance.Get()，因为实例对象已经生成了（TripManagerStub.GetInstanceStub()起的作用），就不会再new一个TripManager实例了。这时再执行的是TripManagerStub.GetCurrentTrip()，而不是基类TripManager.GetCurrentTrip()。因为基类的实例对象已经被替换了，而且GetCurrentTrip是virtual的，所以也就被覆盖了。 
TripManagerStub还提供了一个RemoveStub方法。执行后，当代码执行到TripManager.Instance.Get()时，因为实例对象不存在了，就会新生成一个TripManager对象。这样行为又恢复到原样。

实际中，我们用proteced属性较少。实际通过把个别属性从private改成protected，即保证了安全性，又提供了扩展可能。由此推开来说，当我们谈到私有方法不可测，私有属性不可verify时，也可考虑转换为protected，通过继承类暴露出来进行测试。