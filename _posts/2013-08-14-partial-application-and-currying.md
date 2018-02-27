---
title: 偏函数应用(Partial Application）和函数柯里化(Currying)
---

**偏函数**应用指的是固化函数的一个或一些参数，从而产生一个新的函数。比如我们有一个记录日志的函数:
```python
def log(level, message):
    print level + ": " + message

# usage
log("Warning", "this is one warning message")
log("Error", "this is one error message")
```
在这个函数基础上我们可以固化level参数，产生新的具有特定意义的log函数：
```python
def logWarning(message):
    log("Warning", message)

def logError(message):
    log("Error", message)

# usage
logWarning("this is one warning message")
logError("this is one error message")
```
这就叫偏函数应用。  

在Python中可以借助functools模块来完成偏函数应用，而不用手工编写代码。
```python
from functools import partial
logWarning = partial(log, "Warning")
 
# usage
logWarning("this is one warning message")
```
partial是一个高阶函数（参数含有函数类型，返回值也是函数类型），所以对于3个或更多参数的函数我们还可以这样用：
```python
def log(level, message, stack):
    print level + ": " + message
    print stack
 
logUnknownError = partial(partial(log, "Error"), "Unknown")
logUnkownError("stack content")
 
# output
# Error: Unkown
# stack content
```
**Currying**指的是将一个具有多个参数的函数，转换成能够通过一系列的函数链式调用，其中每一个函数都只有一个参数。我们对log函数进行Currying：
```python
def log(level):
    def logMessage(message):
        print level + ": " + message
 
    return logMessage
 
# usage
log("Warning")("this is one warning message")
 
# or like this
logError = log("Error")
logError("this is one error message")
```
对于curried函数是不能使用多参数的调用方式:
```python
log(“warning”, “this is one warning message”)
```
这样会得到错误，因为log只接受一个参数。  
偏函数和Currying看起来有点类似，区别是什么呢？偏函数是用特定的值来具体化参数，而Currying是变成了一条函数链。
```python
from functools import partial
 
def log(level, message, stack):  
    print level + ": " + message
    print stack   
      
# partial application
partialLogWarning = partial(log, "Warning")
 
# currying
def curriedLog(level):
    def logMsg(message):
        def logStack(stack): 
            print level + ": " + message
            print stack
 
        return logStack
 
    return logMsg
    
curriedLogWarning = curriedLog("Warning")
 
# 1. 偏函数后依然可以使用多参数，只要还有参数未固化；而Currying只能使用函数链
# work  
partialLogWarning("message", "stack")
curriedLogWarning("message")("stack")
 
# 2. Currying得到简化的函数方式更自然，而偏函数必须借用高阶函数或手动生成
curriedLogError  = curriedLog("Error")
curriedLogUnknownError = curriedLog("Error")("Unknown")
curriedLogUnknownError2 = curriedLogError("Unknown")
 
partialLogError = partial(log, "Error")
partialLogUnknownError = partial(log, "Error", "Unknown")
partialLogUnknownError2 = partial(partial(log, "Error"), "Unknown")
```

偏函数和Currying**有什么用**？主要就是从能一个通用函数得到更特定的函数。有一些编程经验的，一定都手工写过偏函数应用吧。  
Currying提供了另外一种实现方式。这种方式在函数式编程中更常见。函数式编程思想，不仅在Lisp这样的函数式编程语言中，在更多的语言中也得到了实现和发展，像Python，Javascript乃至C#这样的命令式语言(imperative language)。所以有机会不妨考虑下使用Currying，能否更好地解决问题。

延伸阅读
1. http://msmvps.com/blogs/jon_skeet/archive/2012/01/30/currying-vs-partial-function-application.aspx， Jon Skeet写的C#中偏函数应用和Currying。
2. http://mtomassoli.wordpress.com/2012/03/18/currying-in-python/, Python中的Partial Application和Currying，里面实现了一个函数，能把普通函数转换成Curried函数。就像Partial一样能把普通函数进行偏函数应用。
3. http://www.aqee.net/currying-partial-application/，这篇文章中有javascirpt的例子。