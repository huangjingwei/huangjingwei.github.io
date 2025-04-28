# Python类的方法解析顺序MRO


## MRO

```python
class A:
    def __init__(self, a):
        super(A, self).__init__()
        self.a = a

class B(A):
    def __init__(self, b):
        super(B, self).__init__(b)
        self.b = b

class C:
    def __init__(self, c):
        self.c = c

class D(B, C):
    def __init__(self, c):
        print(self.__class__.__mro__)
        print(self.__class__.__bases__)
        B.__init__(self, c)

if __name__ == "__main__":
    D(1)

>>>
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.A'>, <class '__main__.C'>, <class 'object'>)
(<class '__main__.B'>, <class '__main__.C'>)
<__main__.D object at 0x7fa5e4752ed0>
Traceback (most recent call last):
  File "xxx.py", line 23, in <module>
    D(1)
  File "xxx.py", line 20, in __init__
    B.__init__(self, c)
  File "xxx.py", line 9, in __init__
    super(B, self).__init__(b)
  File "xxx.py", line 3, in __init__
    super(A, self).__init__()
TypeError: C.__init__() missing 1 required positional argument: 'c'
```

可以看出，在多继承中，Python 3.x的MRO（Method Resolution Order）采用C3算法，示例中打印出了子类的父类解析顺序，当C继续调用的其父类的初始化函数时，其父类并非Object，而是C，所以报错。
