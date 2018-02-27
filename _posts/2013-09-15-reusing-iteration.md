---
title: 遍历算法的重用
---

如何重用实现的遍历算法，这是我在面试中常问的一个问题。其实问题并不难，主要考察对语言特性的掌握，以及处理这种常见场景的经验。或许是交谈中没有实例的原因，满意的答案不多。

假设用Node类来表示树形结构中的节点。
```c#
public class Node
{
    public Node(string name)
    {
        this.Name = name;
        this.Children = new List<Node>();
    }

    public string Name { get; set; }
    public Node Parent { get; set; }
    public IList<Node> Children { get; set; }

    public void Add(Node child)
    {
        child.Parent = this;
        this.Children.Add(child);
    }
}
```

使用这个类来构建我们的行政区域。
```c#
var nodes = new Dictionary<string, Node> ()
{
    {"China", new Node("China")},
    {"SiChuan", new Node("Si Chuan")},
    {"GuangDong", new Node("Guang Dong")},
    {"ChengDu", new Node("Cheng Du")},
    {"ChongQin", new Node("Chong Qin")}
};
nodes["China"].Add(nodes["SiChuan"]);
nodes["China"].Add(nodes["GuangDong"]);
nodes["SiChuan"].Add(nodes["ChengDu"]);
nodes["SiChuan"].Add(nodes["ChongQin"])
```

现在遍历给定的一个节点，将其及所有子节点的名称打印出来。
```c#
public void Iterate(Node node)
{
    Console.WriteLine(node.Name);
    foreach (var child in node.Children)
    {
        this.Iterate(child);
    }
}
```

这个例子中的迭代算法很简单，仅使用递归来遍历所有的子节点。实际情况会复杂，比如同时要遍历其兄弟节点。如果我们还需要遍历节点做其他事情，那就要考虑如何重用这个遍历算法。

## 函数参数
C#是以委托的形式把函数作为参数来使用。
```c#
public class NodeIterator
{
    public void Iterate(Node node, Action<Node> action)
    {
        action(node);
        foreach (var child in node.Children)
        {
            this.Iterate(child, action);
        }
    }
}
```

定义一个参数为Node返回值为空的函数，作为Iterate的第二个参数。为省去函数定义，可以使用匿名函数。
```c#
var it = new NodeIterator();
it.Iterate(nodes["China"], delegate(node node)
    {
        Console.WriteLine(node.Name);
    }
);
```

使用lambda更简洁。
```c#
it.Iterate(nodes["China"], (node) => 
    {
        Console.WriteLine(node.name);
    }
);
```

大多编程语言都支持这种方式。对于脚本语言来说，使用起来更方便。以Python为例。
```python
def print_node(node):
    print node.name

def iterate(node, fn):
    fn(node)
    for child in node.children:
        iterate(child, fn)

iterate(nodes["China"], print_node)
```

不过Python不支持匿名函数，lambda也仅仅支持单行表达式。所以往往还需要函数定义。而JavaScript是可以使用匿名函数的。
```javascript
iterate(nodes["China"], function(node){
    console.log(node.name);
});
```

对于C/C++来说就得用到函数指针和仿函数了。

## 观察者模式
就是GoF 23种模式之一。
```c#
public interface IObserver
 {
     void Notify(Node node);
 }
  
public class NodeObserver : IObserver
{
     public void Notify(Node node)
     {
        Console.WriteLine(node.Name);
    }
}

public class NodeIterator
{
    public NodeIterator()
    {
        this.Observers = new List<IObserver>();
    }

    public IList<IObserver> Observers {get; set;}

    public void Iterate(Node node)
    {
        foreach (var observer in this.Observers)
        {
            observer.Notify(node);
        }

       foreach (var child in node.Children)
       {
           this.ObserverIterate(child);
        }
    }
}
```

代码有点多，但也支持了遍历时调用多个函数。
```c#
var it = new NodeIterator();

it.Observers.Add(new NodeObserver());    
it.Observers.Add(new NodeObserver());

it.Iterate(nodes["China"]);
```

因为添加了两个观察者，所以上面代码会打印两次节点名称。

## 迭代器
迭代器可以使数据看起来更像一个集合.
```c#
public IEnumerable<Node> Iterate(Node node)
{
    yield return node;
    foreach (var child in node.Children)
    {
        foreach (var grandChild in this.Iterate(child))
        {
            yield return grandChild;
        }
    }
}
```

因为Iterate是有返回值的，并由这个IEnumerable返回值完成遍历的。所以在递归调用中，需要多一个foreach。
```c#
foreach (var note in it.Iterate(nodes["China"]))
{
    Console.WriteLine(node.Name);
}
    
IEnumerator<Node> iter = it.Iterate(nodes["China"]).GetEnumerator();
while (iter.MoveNext())
{
    Console.WriteLine(iter.Current.Name);
}
```

上面是两种调用方式。不仅使用起来更简洁了，而且遍历算法和操作节点的函数完全解耦了。在函数参数方式中，遍历算法需要知道操作节点函数。在观察者模式中，遍历算法也是需要知道操作接口的。

Python实现迭代器也很容易。
```python
def iterate(node):
    for child in node.children:
        yield child
        for item in iterate(child):
            yield item

for node in iterate(nodes["China"]):
    print node.name
```

## 事件
事件或者其他消息机制也能很好地完成复用。
```c#
public delegate void IterateEventHandler(object sender, IterateEventArgs e);
 
public class IterateEventArgs : EventArgs
{
    public IterateEventArgs(Node node)
{
        this.Target = node;
    }
    
    public Node Target {get; set;}
}

public class NodeIterator
{
    public event IterateEventHandler Iterated;

    public void Iterate(Node node)
    {
        this.Iterated(this, new IterateEventArgs(node));
        foreach (var child in node.Children)
        {
            Iterate(child);
        }
    }
}
```

绑定事件处理函数，这里同样使用最方便的lambda为例。

```c#
var it = new NoteIterator();

it.Iterated += new IterateEventHandler((sender, e) => 
    {
        Console.WriteLine(e.Target.Name);
    }
);

it.Iterate(nodes["China"]);
```
还有其他方式吗？但那种遍历后，把所有节点放到一个集合中的方式就不用列举了。