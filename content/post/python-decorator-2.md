---
title: "[笔记] Python Decorator (2)"
date: 2020-01-04 13:46:33

---
## 可接受参数的装饰器

假设编写一个为函数添加日志功能的装饰器，但是又允许用户指定日志的等级以及一些其他的细节作为参数，下面是定义这个装饰器的可能做法：
<!--more-->
```python
from functools import wraps

import logging


def logged(level, name=None, message=None):
    '''
    Add logging to a function,if name and message is aren't specified,they default to the function's module and name.
    :param level: the logging level
    :param name:  the logger name
    :param message: the log message
    '''

    def decorate(func):
        logname = name if name else func.__module__
        log = logging.getLogger(logname)
        logmsg = message if message else func.__name__

        @wraps(func)
        def wrapper(*args, **kwargs):
            log.log(level, logmsg)
            return func(*args, **kwargs)

        return wrapper

    return decorate


# Example use
@logged(logging.DEBUG)
def add(x, y):
    return x + y


@logged(logging.CRITICAL, 'example')
def spam():
    print("Spam!")
```

初看上去这个实现很有技巧性，但是其中的思想相对来说是很简单的，最外层的 `logged()` 函数接受所需的参数，并让他们对装饰器的内层函数可见，内层的 `decorate()` 函数接受一个参数并给他加上一个包装层，关键部分在于这个包装层可以使用传递给 `logged()` 的参数。

编写一个可接受参数的装饰器是需要一定技巧的，因为这会涉及底层的调用顺序，具体来说，如果有这样的代码：
```python
@decorator(x,y,z)
def func(a,b):
    pass
```
装饰的过程会按照下列方式来进行：
``` python
def func(a,b):
    pass
func = decorator(x,y,z)(func)
```
`decorator(x,y,z)` 的结果必须是一个可调用对象，这个对象反过来接受一个函数作为参数输入，并对其进行包装。

## 属性可由用户修改的装饰器

编写一个装饰器来来包装函数，但是可以让用户调整装饰器的属性，这样运行时能够控制装饰器的行为。接下来给出的解决方案对上一节的示例作了扩展，引入了访问器函数 `accessor function`，通过使用 nonlocal 关键字声明变量来修改装饰器内部的属性，之后把访问器函数作为函数属性附加到包装函数上。
```python
from functools import wraps, partial
import logging


# Utility decorator to attach a function as an attribute of obj
def attach_wrapper(obj, func=None):
    if func is None:
        return partial(attach_wrapper, obj)
    setattr(obj, func.__name__, func)
    return func


def logged(level, name=None, message=None):
    '''
    Add logging to a function,if name and message is aren't specified,
    they default to the function's module and name.
    :param level: the logging level
    :param name:  the logger name
    :param message: the log message
    '''

    def decorate(func):
        logname = name if name else func.__module__
        log = logging.getLogger(logname)
        logmsg = message if message else func.__name__

        @wraps(func)
        def wrapper(*args, **kwargs):
            log.log(level, logmsg)
            return func(*args, **kwargs)

        @attach_wrapper(wrapper)
        def set_level(newlevel):
            nonlocal level
            level = newlevel

        @attach_wrapper(wrapper)
        def set_message(newmsg):
            nonlocal logmsg
            logmsg = newmsg

        return wrapper

    return decorate


# Example use
@logged(logging.DEBUG)
def add(x, y):
    return x + y

@logged(logging.CRITICAL, 'example')
def spam():
    print("Spam!")
```
运行示例
```python
import logging
>>> logging.basicConfig(level=logging.DEBUG)
>>> add(2,3)
DEBUG:__main__:add
5
>>> spam()
CRITICAL:example:spam
Spam!
>>> add.set_level(logging.WARNING)
>>> add(5,6)
WARNING:__main__:add
11
>>> add.set_message("add called")
>>> add(4,5)
WARNING:__main__:add called
9
```
本节的关键在于访问器函数 (`set_message()` 和 `set_level()`)，它们以属性的方式附加到了包装函数上，每个访问器函数允许对 `nonlocal` 变量赋值来调整内部参数。

之所以要使用访问器函数，不使用对函数属性的直接访问，如下：
```python
@wraps(func)
def wrapper(*args,**kwargs):
    warpper.log.log(wrapper.level,wrapper.logmsg)
    return func(*args,*kwargs)
#Attach adjustment attributes
wrapper.level = level
wrapper.logmsg = logmsg
wrapper.log = log
```
是因为这种方法只能用在最顶层的装饰器，如果当前顶层的装饰器上有添加了一个装饰器，这样就会隐藏下层的属性使得无法被修改，而使用访问器函数可以绕过这个限制。
本方法可以作为类装饰器的一种替代方案。
