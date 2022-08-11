---

title: "[笔记] Python Generator"
date: 2020-01-05 18:23:39

---
## 简单生成器
`Generator`是一个用于创建迭代器的简单而强大的工具。 它们的写法类似标准的函数，但当它们要返回数据时会使用`yield`语句。 每次对生成器调用`next()`时，它会从上次离开位置恢复执行（它会记住上次执行语句时的所有数据值）。 显示如何非常容易地创建生成器的示例如下:
<!--more-->
``` python
def reverse(data):
    for index in range(len(data) - 1, -1, -1):
        yield data[index]


print(''.join(reverse("hello")))
#output
olleh
```
一个关键特性在于局部变量和执行状态会在每次调用之间自动保存。除了会自动创建方法和保存程序状态，当生成器终结时，它们还会自动引发`StopIteration`。 这些特性结合在一起，使得创建迭代器能与编写常规函数一样容易。
这里只是简单复习一下生成器，重头戏是`yield`表达式。

## yield表达式

`yield`表达式在定义`generator`函数或是`asynchronous generator`的时候才会用到。 因此只能在函数定义的内部使用`yield`表达式。 

在一个函数体内使用`yield`表达式会使这个函数变成一个生成器，并且在一个`async def`定义的函数体内使用`yield` 表达式会让协程函数变成异步的生成器。

当一个生成器函数被调用的时候，它返回一个迭代器，称为生成器。然后这个生成器来控制生成器函数的执行。当这个生成器的某一个方法被调用的时候，生成器函数开始执行。这时会一直执行到第一个`yield`表达式，在此执行再次被挂起，给生成器的调用者返回 `expression_list`的值。

挂起后，我们说所有局部状态都被保留下来，包括局部变量的当前绑定，指令指针，内部求值栈和任何异常处理的状态。通过调用生成器的某一个方法，生成器函数继续执行。此时函数的运行就和`yield`表达式只是一个外部函数调用的情况完全一致。

恢复后`yield`表达式的值取决于调用的哪个方法来恢复执行。 如果用的是`__next__()`(通常通过语言内置的`for`或是`next()`来调用) 那么结果就是`None`. 否则，如果用`send()`, 那么结果就是传递给send方法的值。这里举一个最简单的例子来说明`next()`和`send()`方法的用法:
``` python
def gen():
    num = -1
    while True:
        result = yield num + 1
        num += 1
        print(result)
>>> g = gen()
>>> g.send(None)
0
>>> g.send("hello")
hello
1
>>> g.send("world")
world
2
```
上面第一个值就是调用`next()`的返回值(**`yield`之后的表达式的值**)，而第二个值就是`result`，是`send()` 方法的值作为了`yield`的表达式的值，赋值给了`result`，并打印。
接下来附上官方文档中对`next()`和`send()`的说明：

`generator.__next__()`

> 开始一个生成器函数的执行或是从上次执行的 `yield` 表达式位置恢复执行。 当一个生器函数通过 `__next__()` 方法恢复执行时，**当前的 `yield` 表达式总是取值为 `None`**。随后会继续执行到下一个`yield`表达式，其`expression_list`的值会返回给`__next__()` 的调用者。 如果生成器没有产生下一个值就退出，则将引发`StopIteration`异常。此方法通常是隐式地调用，例如通过`for`循环或是内置的`next()`函数。

`generator.send(value)`

> 恢复执行并向生成器函数“发送”一个值。`value`参数将成为当前`yield`表达式的结果。 `send()`方法会返回生成器所产生的下一个值，或者如果生成器没有产生下一个值就退出则会引发`StopIteration`。 **当调用`send()`来启动生成器时，它必须以`None`作为调用参数，因为这时没有可以接收值的`yield`表达式**。

再看一个例子：
```python
def echo(value=None):
    print("Execution starts when 'next()' is called for the first time.")
    try:
        while True:
            try:
                value = yield value
            except Exception as e:
                value = e
    finally:
        print("Don't forget to clean up when 'close()' is called.")

>>> gen = echo(1)
>>> print(next(gen))
Execution starts when 'next()' is called for the first time.
1
>>> print(next(gen))
None
>>> gen.throw(TypeError,'spam')
TypeError('spam')
>>> gen.close()
Don't forget to clean up when 'close()' is called.
```
这里用到了`throw()`和`close()`方法,根据文档描述,这里 `gen.throw(TypeError,'spam')`的引发里一个异常，并且在生成器中被捕获，当前`value`值即为`'spam'`,是生成器函数所产生的下一个值，`throw()`和`send()`、`next()`相同，都是驱动生成器继续执行，故这里输出为`TypeError('spam')`。`close()`比较好理解,这里正常退出了生成器。

附上文档：

`generator.throw(type[, value[, traceback]])`    

> 在生成器暂停的位置引发`type`类型的异常，并返回该生成器函数所产生的下一个值。 如果生成没有产生下一个值就退出，则将引发`StopIteration`异常。 如果生成器函数没有捕获传入的异常，或引发了另一个异常，则该异常会被传播给调用者。

 `generator.close()`

> 在生成器函数暂停的位置引发`GeneratorExit`。 如果之后生成器函数正常退出、关闭或引发`GeneratorExit`(由于未捕获该异常) 则关闭并返回其调用者。 如果生成器产生了一个值，关闭引发 `RuntimeError`。 如果生成器引发任何其他异常，它会被传播给调用者。 如果生成器已经由异常或正常退出则`close()`不会做任何事。

## `yield from` 委托给子生成器的语法
PEP 380添加了`yield from`表达式，从而允许生成器将其部分操作委托给另一生成器。 这允许将包含`yield`的一段代码分解出来并放置在另一个生成器中。 此外，允许子生成器返回一个值，并且该值可用于委派生成器。
虽然主要用于委托给子生成器，但`yield from`实际上允许委派给任意子迭代器。

对于简单的迭代器，`yield from`本质上只是` for item in iterable: yield item:`的缩写形式。
``` python
def g(x):
    yield from range(x, 0, -1)
    yield from range(x)

print(list(g(5)))
# output
[5, 4, 3, 2, 1, 0, 1, 2, 3, 4]
```
用`yield from`实现深度优先遍历，在接下来这份代码中，`depth_first()`的实现非常易于阅读，描述起来也很方便。利用子节点的`depth_first()`方法，通过`yield from`语句来产生其他元素。
``` python
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

    def depth_first(self):
        yield self
        for c in self:
            yield from c.depth_first()


if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    child1.add_child(Node(3))
    child1.add_child(Node(4))
    child2.add_child(Node(5))

    print(list(root.depth_first()))

#outputs
[Node(0), Node(1), Node(3), Node(4), Node(2), Node(5)]
```
然而，不像通常的循环，`yield from`语句支持子生成器接受来自外界调用的`send()`和`throw()`的值，并将最终的值返回给外部的生成器：
```python
def g(x):
    yield from range(x, 0, -1)
    yield from range(x)


# print(list(g(5)))

def accumulate():
    tally = 0
    while True:
        next = yield
        if next is None:
            return tally
        tally += next


def gather_tallies(tallies):
    while True:
        tally = yield from accumulate()
        tallies.append(tally)


tallies = []
acc = gather_tallies(tallies)
next(acc)
for i in range(4):
    acc.send(i)

acc.send(None)

for i in range(5):
    acc.send(i)

acc.send(None)

print(tallies)
#outputs
[6, 10]
```
