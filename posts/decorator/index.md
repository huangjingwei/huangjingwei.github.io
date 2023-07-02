# Python装饰器


## 一句话定义

装饰器本质上是一个`Python`函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。

在闭包的笔记中提到装饰器的形式：`decorated = outer(foo)`，类似“套娃”。

## 函数装饰器

```python
def decorator(fn):
    def wrapper(arg):
        print("In wrapper, arg is %s, fn is %s"%(arg, fn.__name__))
        return fn(arg)
    return wrapper

@decorator
def outer(arg):
    print("In outer, arg is %s"%arg)

print("Finished decorating outer()")
print(outer)
outer("walle")

#output
Finished decorating outer()
<function decorator.<locals>.wrapper at 0x7f897871e3b0>
In wrapper, arg is walle, fn is outer
In outer, arg is walle
```

由`print(outer)`的输出是`wrapper`函数可以看出，`decorator` 这个函数套在`outer`外面，即原来的`outer`函数被装饰了，以上的封装过程是：

```python
outer = decorator(outer)  # 这里的装饰器返回的是一个函数
```

`outer("walle")`等价于`decorator(outer)("waller")`等价于`wrapper("waller")`。
所以`walle in wrapper, fn is outer`是`wrapper`的`print()`输出的。`walle in outer`是在`wrapper`的`fn(arg)`输出的。

由此可以看出，装饰器的价值就是在`wrapper`的函数中搞事情，而且还可以继续调用被装饰的函数。

## 类装饰器

```python
class Decorator(object):
     def __init__(self, fn):
         print("In __init__, fn is %s"%fn.__name__)
         self.fn = fn
     def __call__(self, arg):
         print("In __call__, arg %s"%arg)
         self.fn(arg)
 
@Decorator
def outer(arg):
    print("In outer, arg %s"%arg)
    
print("Finished decorating outer()")
print(outer)
outer("walle")

#output
In __init__, fn is outer
Finished decorating outer()
<__main__.Decorator object at 0x7f6d773d31f0>
In __call__, arg is walle
In outer, arg is walle
```

由此执行结果可见，装饰过程只调用了该类的`__init__`函数，被装饰后实际返回的是一个`Decorator`的一个实例，以上的封装过程是：

```python
outer = Decorator(outer) # 这里的装饰器返回的是一个类的实例对象
```

因为装饰的过程是类的实例化过程，所以称之为类装饰。

`__call__()`是一个非常特殊的实例方法。该方法的功能类似于在类中重载`()`运算符，使得类实例对象可以像调用普通函数那样，以“对象名()”的形式使用。

所以`outer("walle")`就等价于`Decorator(outer)("walle")`，`Decorator(outer)`是一个实例化对象（这里我们用`outer_obj`表示），所以最后就是`outer_obj('walle')`，这就调用到了`__call__()`的逻辑。

## 对象装饰器

```python

class Decorator(object):
     def __init__(self, arg):
         print("In __init__, arg is %s"%arg)
         self.arg = arg
     def __call__(self, fn):
         print("In __call__, fn is %s"%fn.__name__)
         def wrapper(arg):
            arg = self.arg + ' ' +arg
            fn(arg)
         return wrapper
 
@Decorator("Hello")
def outer(arg1):
    print("%s In outer"%arg1)
    
print("Finished decorating outer()")
print(outer)
outer("walle")

#output
In __init__, arg is Hello
In __call__, fn is outer
Finished decorating outer()
<function Decorator.__call__.<locals>.wrapper at 0x7f468ba19c60>
Hello walle In outer
```

由此执行结果可见，装饰过程只调用了该类的`__init__`函数和实例函数`__call__`，装饰后的结果就是执行了`__call__`的结果，即返回了一个函数，以上的封装过程是：

```python
outer = Decorator('hello')(outer) # 这里的装饰器返回的是一个函数
```

因为装饰的过程是类实例化的对象被调用的过程，所以称之为对象装饰器。

所以`outer("walle")`就等价于`Decorator("Hello")(outer)("walle")`，`Decorator("Hello")`是一个实例化对象，`Decorator("Hello")(outer)`是实例化对象调用了`__call__()`，该过程返回了`wrapper`函数。

## 装饰器顺序

```python
def decorator_a(func):
    print('In decorator_a')
    def inner_a(*args, **kwargs):
        print('In inner_a')
        func(*args, **kwargs)
        print('In inner_a, post')
    return inner_a

def decorator_b(func):
    print('In decorator_b')
    def inner_b(*args, **kwargs):
        print('In inner_b')
        func(*args, **kwargs)
        print('In inner_b, post')
    return inner_b

@decorator_b
@decorator_a
def outer(arg):
    print("In outer, arg is %s"%arg)

print("Finished decorating outer()")
print(outer)
outer("walle")

#output
In decorator_a
In decorator_b
Finished decorating outer()
<function decorator_b.<locals>.inner_b at 0x7f6b711d6170>
In inner_b
In inner_a
In outer, arg is walle
In inner_a, post
In inner_b, pos
```

由执行结果中先执行了`decorator_a`的逻辑，再执行`decorator_b`的逻辑，最终返回的是`inner_b`，可见以上的封装过程为：

```python
outer = decorator_b(decorator_a(outer)) # 这里的装饰器返回的是一个函数
```

`decorator_a(outer)`的执行结果是返回了`inner_a`，`decorator_b(inner_a)`的执行结果是返回一个`inner_b`。


当`outer("walle")`传入参数进行调用时，就是调用`inner_b("walle")`，它会先打印`In inner_b`，然后在`inner_b`内部调用了`fun`即`inner_a`，所以会再打印`In inner_a`, 然后再`inner_a`内部调用的原来的`fun`即`outer`。
