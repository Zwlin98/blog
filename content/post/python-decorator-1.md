---

title: "[笔记] Python Decorator (1)"
date: 2020-01-03 20:40:53

---
## 给函数添加一个包装
给函数添加一个包装层（wrapper layer），以添加额外的处理，例如记录日志，计时统计等，可以通过自定义一个装饰器函数。举例如下:
<!--more-->
``` python
import time

from functools import wraps

def timethis(func):
    '''
    Decorator that reports the execution time
    :param func:
    :return:
    '''

    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end - start)
        return result

    return wrapper

@timethis
def countdown(cnt):
    while cnt > 0:
        cnt -= 1


if __name__ == '__main__':
    countdown(100000)
    countdown(1000000)
```
装饰器就是这样一个函数，它可以接受一个函数作为输入并返回一个新的函数作为输出，当像这样编写代码时:
``` python
@timethis
def countdown(cnt):
    ...
```
和单独执行下列步骤的效果是一样的：
``` python
def countdown(cnt):
    ...
countdown = timethis(countdown)
```
同时，内建的装饰器比如`@staticmethod`,`@classmethod`,`@property`的工作方式也是一样的。

装饰器内部一般会涉及创建一个新的函数，利用`*arg`和`**kwargs`来接受任意的参数。示例中的`wrapper`函数正是这样做的。在这个函数内部，我们需要调用原来的输入函数（即被包装的那个函数，它是装饰器的输入参数）并返回它的结果。但是，也可以添加任何想要添加的额外代码（例如计时处理）。这个新创建的`wrapper`函数会作为装饰器的结果返回，取代了原来的函数。

需要重点强调的是，**装饰器一般来说不会修改调用签名（参数个数，类型，顺序等），也不会修改被包装函数返回的结果。** 这里对`*arg`和`**kwargs`的使用是为了确保可以接受任何形式的输入参数。装饰器的返回值几乎总是同调用`func(*args,*kwargs)`的结果一致。

## 编写装饰器时要保存函数的元数据

编写好了一个装饰器，但当他用在一个函数上时，一些重要的元数据比如函数名，文档字符串，函数注解以及调用签名都丢失了，因此，每当定义一个装饰器时，应该总是记得为底层的包装函数添加`functools`库中的`@warps`装饰器。示例如下：
``` python
import time

from functools import wraps

def timethis(func):
    '''
    Decorator that reports the execution time
    :param func:
    :return:
    '''

    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end - start)
        return result

    return wrapper
```
## 对装饰器进行解包装

`@wraps`装饰器的一个重要特性就是它可以通过`__warpped__`属性来访问被包装的那个函数。例如，若果希望直接访问被包装的函数，这可以通过这样做：
```python
countdown.__wrapped__(10000)
#or
origin_countdown = countdown.__wrapped__
origin_countdown()
```
这种技术只有在实现装饰器时利用了`functools`库中的`@warps`对元数据进行了适当的拷贝，或者直接设定了`__warpped__`属性时才有用。

同时，嵌套时的装饰器链，也是逐层解包的：
```python
from functools import wraps


def decorator1(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator1")
        return func(*args, **kwargs)

    return wrapper


def decorator2(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator2")
        return func(*args, **kwargs)

    return wrapper


@decorator1
@decorator2
def add(x, y):
    return x + y


if __name__ == '__main__':
    print(add(2, 3))
    print(add.__wrapped__(2, 3))
    print(add.__wrapped__.__wrapped__(2, 3))

# 输出
Decorator1
Decorator2
5
Decorator2
5
5
```
