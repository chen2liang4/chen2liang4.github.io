---
title: Python逻辑运算结果的类型
---

Python的逻辑运算符如下：
```python
x and y # 如果x为False, 不计算y的值，直接返回x。否则返回y。
x or y  # 如果x为True，不计算y的值，直接返回x。否则返回y。
not x   # 如果x为False，返回True。否则返回False。
```

在and和or运算中，python使用了短路计算。即如果x的值已经决定了结果，将不执行y。x和y可以是变量或者表达式。

我们知道Python中，数字，字符串，列表等都能参与逻辑运算。0，空字符串，空列表当作False；而非空值当作True。

所以需要注意的是：and和or的运算结果不一定是布尔类型！具体类型是由返回的x或y决定的。只有not操作返回的才一定是布尔类型！
```python
>>>0 and 23
0
>>>type(0 and 23)
<type 'int'>
>>> '' or 'abcd'
''
>>> type('' or 'abcd')
<type 'str'>
>>>not ''
True
>>>type(not '')
<type 'bool'>
```

所以有了这样的用法：
```python
x > 0 and "Positive" or "Negative"
```

如果x>0的值是True，and的结果就是"Positive"。"Positive"和"Negative"进行or运算，因为“Positive”是True，所以继续返回“Positive”。  
如果x>0的值是False，and的结果就是False。False和”Negative“进行or运算，返回”Negative“。  
这和条件表达式的作用完全一样："Positive" if x > 0 else "Negative"。

还有这样：
```python
'''写一个程序，打印数字1到100，3的倍数打印“Fizz”来替换这个数，5的倍数打印“Buzz”，对于既是3的倍数又是5的倍数的数字打印“FizzBuzz”'''
for x in range(101): 
    print “fizz”[x%3*4:] + “buzz”[x%5*4:] or x
```
如果or前面的是空字符串，则返回数字x，因为空字符串是False。

所以在Python中，逻辑运算符并非只能用于逻辑运算。