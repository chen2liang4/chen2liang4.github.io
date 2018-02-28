---
title: Asynchronous assertion in Unit test
---

上周有同学提到对于异步事件的Assertion要放到响应事件的代码体中，否则在事件被fire前，测试就结束了，而断言自然也就不起作用了。空想无结果，写下代码测试下就知道了。
```c#
public class BusinessObject
{
    public event EventHandler DoCompleted;
    private IService _client;
 
    public BusinessObject(IService client)
    {
        _client = client;
    }
 
    public void DoAsynchronously()
    {
        _client.AsynCall(() =>
        {
            DoCompleted(this, EventArgs.Empty);
        });
    }
}
 
public interface IService
{
    void AsynCall(Action completedAction);
}
 
public class ServiceClient : IService
{
    private Action _completedAction = null;
 
    public void AsynCall(Action completedAction)
    {
        Thread thread = new Thread(new ThreadStart(AsynThread));
        thread.Start();
        _completedAction = completedAction;
    }
 
    public void AsynThread()
    {
        Thread.Sleep(1000 * 2);
        if (null != _completedAction)
            _completedAction();
    }
}
```
ServiceClient的AsynCall方法会启动一个线程，并暂停两秒钟后调用传入的completedAction，以此来模拟异步调用。首先写的测试代码如下：
```c#
[TestMethod]
public void DoAsynchronously_GivenDoCompletedHandler_EventFired()
{
    bool actual = false;
    BusinessObject bo = new BusinessObject(new ServiceClient);
    bo.DoCompleted += (sender, e) => { actual = true; };
 
    bo.DoAsynchronously();
 
    Assert.IsTrue(actual);
}
```
因为是异步的，当程序就执行到断言语句的时候，actual还没有被设置为true，所以断言失败了。把断言移入到lambda表达式后，测试通过。 
```c#
public void DoAsynchronously_GivenDoCompletedHandler_EventFired()
{
    bool actual = false;
    BusinessObject bo = new BusinessObject(new ServiceClient);
    bo.DoCompleted += (sender, e) =>
    {
        actual = true;
        Assert.IsTrue(actual);
    };
 
    bo.DoAsynchronously();
}
```
但是当把第7行， actual = true; 注释后，测试仍然通过！这是因为assert失败的异常在子线程中被抛出，主线程无法捕获所以造成了测试通过的假象。于是在测试方法中加入信号量以同步操作， 
```c#
public void DoAsynchronously_GivenDoCompletedHandler_EventFired()
{
    bool actual = false;
    BusinessObject bo = new BusinessObject(new ServiceClient);
    Semaphore sem = new Semaphore(0, 1);
    bo.DoCompleted += (sender, e) =>
    {
        actual = true;
        sem.Release();
    };
 
    bo.DoAsynchronously();
    
    sem.WaitOne();
    Assert.IsTrue(actual);
}
```
sem.WaitOne();会阻止测试代码的运行，直到事件响应代码中Release掉这个信号量。测试结果通过！但这样测试效率很低，想想如果这个异步操作需要更多的时间，或者我们要测试有一堆这种有异步调用的方法。

但实际想想我们其实测试的时候使用的是Mock对象，而并非ServiceClient的实例。 
```c#
 public void DoAsynchronous_GivenDoCompletedHandlerWithMock_EventFired()
 {
     bool actual = false;
     Mock<IService> mock = new Mock<IService>();
     mock.Setup(p => p.AsynCall(It.IsAny<Action>()))
         .Callback((Action action) => { action(); });
  
     BusinessObject bo = new BusinessObject(mock.Object);
     bo.DoCompleted += (sender, e) =>
     {
         actual = true;
     };
  
     bo.DoAsynchronously();
  
     Assert.IsTrue(actual);
 }
```
测试通过。去掉第11行，actual = true; 测试失败！看来mock对象的调用是同步的，所以我们并不用担心这个问题。

例子中ServiceClient是异步的，如果被测试对象BusinessObject是异步的，那该咋办呢？  
答案很简单，把异步方法抽出去，放入到ServiceClient这样的类中。