# Python闭包


## 定义

>闭包，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。
>
>有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。
>闭包在实现上是一个结构体，它存储了一个函数（通常是其入口地址）和一个关联的环境（相当于一个符号查找表）。
>环境里是若干对符号和值的对应关系，它既要包括约束变量（该函数内部绑定的符号），
>也要包括自由变量（在函数外部定义但在函数内被引用），有些函数也可能没有自由变量。
>
>闭包跟函数最大的不同在于，当捕捉闭包的时候，它的自由变量会在捕捉时被确定，这样即便脱离了捕捉时的上下文，它也能照常运行。

自由变量：没有在作用域中被定义的变量？

## 变量作用域

从一个例子分析：

```python
>>> b = 1
>>> def outer(a):
        print(a)
        print(b)
        b = 3
>>> outer(2)
2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in outer
UnboundLocalError: local variable 'b' referenced before assignment
```

变量搜索，当在函数中访问一个新的变量是，Python会在当前的命名空间中寻找变量是否存在。如果变量不存在则会从上一级命名空间中搜索，直到顶层命名空间。

调用`outer()`函数时，会正常打印出参数变量`a`。执行到变量`b`的时候，会在当前的函数作用域中查询，但是在执行打印变量的时候，检测到赋值语句在其之后，所以报错。以上函数想要正常运行的话，可以去掉函数中的赋值语句，由于所在函数内没有变量则会向上一级搜寻，则会打印出`b=1`的结果。或者也可以用在函数内用`global b`来将其申明为全局变量。

## 闭包函数

**函数对象的`__closure__`属性指示了该函数对象是否是闭包函数，若不是闭包函数，则该属性值为None，否则为一个非空元组。**

创建一个计算系列值的均值的逻辑。

```python
class Averager():
    
    def __init__(self):
        self.series = []
        
    def __call__(self, new_value):
        self.series.append(new_value)
        total = sum(self.series)
        return totle / len(self.series)

avg = Averager()
avg(10)  # 10.0
avg(11)  # 10.5
avg(12)  # 11.0
```

以上逻辑是通过类实现，`self.series`是实例变量。不属于自由变量，因为在类的作用域中已被定义。

```python
def make_averager():

    #-----闭包-----
    series = []
    
    def averager(new_value):
        # series是自由变量
        series.append(new_value)
        total = sum(series)
        return totle / len(series)
    #--------------
    
    return averager

avg = make_averager()
avg(10)  # 10.0
avg(11)  # 10.5
avg(12)  # 11.0
avg.__closure__   # (<cell at 0x7f3f4f4ecee0: list object at 0x7f3f4f634b80>,)
```

以上逻辑是通过函数实现，`make_averager()`在局部作用域中定义了`series`变量，它的内部函数`averager()`的自由变量`series`绑定了这个值，这就组成了所谓的闭包。如果没有闭包特性的话，自由变量`series`一定会报错找不到定义，因为在调用`avg(10)`时，`make_averager()`函数已经返回了，它的局部作用域也消失了。闭包是一种函数，它会保留定义时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍然能使用那些绑定。

## nonlocal

```python
def make_averager():
    count = 0
    total = 0
    
    def averager(new_value):
        count += 1
        total += new_value
        return total / count
        
    return averager

avg = make_averager()
avg(10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in averager
UnboundLocalError: local variable 'count' referenced before assignment
```

以上报错是因为`count +=1`等同于`count = count + 1`，存在赋值，`count`就变成局部变量了。当然，`total`变量也是如此。

这里如果把count和total通过global关键字声明为全局变量，显然是不合适的，它们作用域最多只扩展到make_averager()函数内。为了解决这个问题，Python3引入了nonlocal关键字声明：

```python
def make_averager():
    count = 0
    total = 0
    
    def averager(new_value):
        nonlocal count, total
        count += 1
        total += new_value
        return total / count
        
    return averager
```

nonlocal的作用是把变量标记为自由变量，即使在函数中为变量赋值了，也仍然是自由变量。

注意，对于列表、字典等可变类型来说，添加元素不是赋值，不会隐式创建局部变量。对于数字、字符串、元组等`不可变类型`以及`None`来说，赋值会隐式创建局部变量。示例：

```python
def make_averager():
    # 可变类型
    count = {}

    def averager(new_value):
        print(count)  # 成功
        count[new_value] = new_value
        return count

    return averager
```

## 闭包和装饰器

```python
def outer(x):
    def inner(y):
        print(x + y)
    return inner
```

以上的闭包函数的自由变量`x`是外层函数的参数变量。

我们可以利用闭包的特性得到一个对已有函数运行行为进行扩充或修改的新函数，而同时保留已有函数，不用对已有函数的代码进行修改。

做到这个的第一步是将函数作为参数传递到我们的“闭包创建函数”（例子中的`outer`）中，如`decorated = outer(foo)`，`decorated`是`foo`的装饰版，即给`foo`加上了一些东西，调用`decorated()`就是调用装饰器了。

