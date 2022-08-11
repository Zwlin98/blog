---

title: "[笔记] Python 与函数式编程"
date: 2019-05-27 20:31:53

---

## 匿名函数定义

``` python
lambda argument1, argument2 ,... argument3 : expression
```
<!--more-->
## 用法示例
1. 类似函数
``` python
>> up = lambda s : s.upper() 
>> up("you")
YOU
```
2. 列表生成式
``` python
>> [(lambda x: x**2)(x) for x in range(10)]
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```
3. 用作参数
``` python
#将列表按照元组的第二个值进行排序
>> l = [(1, 20), (3, 0), (9, 10), (2, -1)]
>> l.sort(key=lambda x: x[1])
>> print(l)
[(2, -1), (3, 0), (9, 10), (1, 20)]
#将字典按键值从大到小排序
>> d = {'mike': 10, 'lucy': 2, 'ben': 30}
>> dict(sorted(d.items(), key=lambda x: x[1], reverse=True))
{'ben': 30, 'mike': 10, 'lucy': 2}
```

## `map()` `filter()` `reduce()`与lambda表达式

### map() 
#### 原型
`map(function, iterable, ...)`
#### 官方文档说明
> 返回一个将 `function` 应用于 `iterable` 中每一项并输出其结果的迭代器。 如果传入了额外的 `iterable` 参数，`function` 必须接受相同个数的实参并被应用于从所有可迭代对象中并行获取的项。 当有多个可迭代对象时，最短的可迭代对象耗尽则整个迭代就将结束。
#### 举例
``` python
>> l = [1, 2, 3, 4, 5]
>> list(map(lambda x: x * 2, l))
[2, 4, 6, 8, 10]
```

### filter()
#### 原型
`map(function, iterable, ...)`
#### 官方文档说明
> 用 `iterable` 中函数 `function` 返回真的那些元素，构建一个新的迭代器。`iterable` 可以是一个序列，一个支持迭代的容器，或一个迭代器。如果 `function` 是 `None` ，则会假设它是一个身份函数，即 `iterable` 中所有返回假的元素会被移除。
请注意， `filter(function, iterable)` 相当于一个生成器表达式，当 `function` 不是 `None` 的时候为 `(item for item in iterable if function(item))`；`function` 是 `None` 的时候为 `(item for item in iterable if item)` 。
#### 举例
``` python
>> l = [1, 2, 3, 4, 5]
>> list(filter(lambda x: x % 2 == 0, l))
[2, 4]
```

### reduce() 官方文档说明
#### 原型
`reduce(function, iterable[, initializer])`
#### 官方文档说明
> 将两个参数的 `function` 从左至右积累地应用到 `iterable` 的条目，以便将该可迭代对象缩减为单一的值。 例如，`reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])` 是计算 `((((1+2)+3)+4)+5)` 的值。 左边的参数 x 是积累值而右边的参数 y 则是来自 iterable 的更新值。 如果存在可选项 initializer，它会被放在参与计算的可迭代对象的条目之前，并在可迭代对象为空时作为默认值。 如果没有给出 initializer 并且 iterable 仅包含一个条目，则将返回第一项。
#### 举例
``` python
>> reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])
15
>> reduce(lambda x, y: x+y, [1, 2, 3, 4, 5],100)
115
```
