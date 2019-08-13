# Python

[参考链接](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

用Python可以做什么？可以做日常任务，比如自动备份你的MP3；可以做网站，很多著名的网站包括YouTube就是Python写的；可以做网络游戏的后台，很多在线游戏的后台都是Python开发的。总之就是能干很多很多事啦。

Python当然也有不能干的事情，比如写操作系统，这个只能用C语言写；写手机应用，只能用Swift/Objective-C（针对iPhone）和Java（针对Android）；写3D游戏，最好用C或C++。

# 安装Python

python 是跨平台的，mac\linux\win 上都有。

python 有两个版本，2.x 和 3.x，两个是不兼容的。一般普及的是3.x（笔记时是3.7版本）。

![](.\图片\安装Python.png)

> 注意将这个 `Add Python 3.5 to PATH` 勾上，然后安装即可。

**验证**

安装后在命令行输入 `python` 即可进入 Python 提供的交互环境，再次输入 `exit()` 即可退出。

如果没有出现正常的情况，可能是没有将 Python 添加到 path 中，要么修改环境变量，要么重新安装并勾选`Add Python 3.5 to PATH`

安装后，你会得到Python解释器（就是负责运行Python程序的），一个命令行交互环境（命令行输入 `python`然后回车），还有一个简单的集成开发环境。

## Python 解释器

我们编写的 Python 代码文件是以 `.py` 的格式保存的。Python 的解释器就是可以运行 `.py` 的工具。

### CPython

当我们从[Python官方网站](https://www.python.org/)下载并安装好Python 3.x后，我们就直接获得了一个官方版本的解释器：CPython。这个解释器是用C语言开发的，所以叫CPython。在命令行下运行`python`就是启动CPython解释器。

CPython是使用最广的Python解释器。教程的所有代码也都在CPython下执行。

### IPython

IPython是基于CPython之上的一个交互式解释器，也就是说，IPython只是在交互方式上有所增强，但是执行Python代码的功能和CPython是完全一样的。好比很多国产浏览器虽然外观不同，但内核其实都是调用了IE。

CPython用`>>>`作为提示符，而IPython用`In [序号]:`作为提示符。

### PyPy

PyPy是另一个Python解释器，它的目标是执行速度。PyPy采用[JIT技术](http://en.wikipedia.org/wiki/Just-in-time_compilation)，对Python代码进行动态编译（注意不是解释），所以可以显著提高Python代码的执行速度。

绝大部分Python代码都可以在PyPy下运行，但是PyPy和CPython有一些是不同的，这就导致相同的Python代码在两种解释器下执行可能会有不同的结果。如果你的代码要放到PyPy下执行，就需要了解[PyPy和CPython的不同点](http://pypy.readthedocs.org/en/latest/cpython_differences.html)。

### Jython

Jython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码执行。

### IronPython

IronPython和Jython类似，只不过IronPython是运行在微软.Net平台上的Python解释器，可以直接把Python代码编译成.Net的字节码。

# 第一个 Python 程序

![](.\图片\第一个Python程序.png)

> 注意：
>
> 1. print 可以打印内容，打印内容可以用单引号或双引号包括，但不能混用；
> 2. 执行一个`.py`文件**只能**在命令行模式执行，并使用 `pyhton <file>` 执行；
> 3. 在命令行模式运行`.py`文件和在Python交互式环境下直接运行Python代码有所不同。Python交互式环境会把每一行Python代码的结果自动打印出来，但是，直接运行Python代码却不会。这个就好比运行js和使用浏览器开发者工具的区别；
> 4. 交互模式主要用于调试Python代码，而不是真正的开发环境。
> 5. 在mac/linuxl 上通过一些方法可以直接运行 `.py` 文件

## Python 代码运行助手

可以让你在线输入 Python 代码，然后通过本机运行的一个 Python 脚本来执行代码。

运行 文件夹下的 learning.py 文件即可。

当然，只是在[这个网站](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432523496782e0946b0f454549c0888d05959b99860f000#0)中使用的

## 输出

> print()

- 将内容打印出来，内容需要使用引号包括起来。
- 可以接受多个输出内容，多个输出内容之间使用逗号隔开，Python遇到逗号会输出一个空格。

![](.\图片\print().png)

## 输入

> input()

- 可以让用户输入字符串，并存放到一个变量里
- 可以接受参数，参数为提示输入的语句

![](.\图片\input().png)

当输入 `name = input()` 回车之后，Python 会进入等待你输入的状态。

有了输入输出后，我们可以将之前的代码升级：

```python
name = input('Please input your name: ')
print("hello world", name)
```

# Python 基础

```python
# print absolute value of an integer:
a = 100
if a >= 0:
    print(a)
else:
    print(-a)
```

- 以`#`开头的语句是注释
- Python 是采用缩进方式的
- 每一行都是一个语句，当语句以冒号`:`结尾时，缩进的语句视为代码块。
- 好处是强迫你写出格式化的代码，但没有规定缩进是几个空格还是Tab。按照约定俗成的管理，应该始终坚持使用**4个空格**的缩进。
- 缩进的坏处就是“复制－粘贴”功能失效了，这是最坑爹的地方。当你重构代码时，粘贴过去的代码必须重新检查缩进是否正确
- Python 程序是**大小写敏感**的，如果写错了大小写，程序会报错

## 数据类型和变量

- 整数

  - Python 可以处理任意大小的整数，负整数

  - 整数运算永远是精确的（除法难道也是精确的？是的！），而浮点数运算则可能会有四舍五入的误差

  - Python 的整数没有大小限制，而某些语言的整数根据其存储长度是有大小限制的，例如Java对32位整数的范围限制在`-2147483648`-`2147483647`。

    Python 的浮点数也没有大小限制，但是超出一定范围就直接表示为`inf`（无限大）。

```python
# 在 Python 中有两种除法，“/” 与 “//”

# “/”除法计算结果是浮点数，即使是两个整数恰好整除，结果也是浮点数：
>>> 10 / 3
3.3333333333333335
>>> 9 / 3
3.0

# 是"//"，称为地板除，两个整数的除法仍然是整数
>>> 10 // 3
3
# 求余数
>>> 10 % 3
1
```

- 浮点数
- 字符串
  - Python还允许用`r''`表示`''`内部的字符串默认不转义
  - Python允许用`'''...'''`的格式表示多行内容，注意在输入多行内容时，提示符由`>>>`变为`...`，提示你可以接着上一行输入，注意`...`是提示符，不是代码的一部分。还可以配合 `r` 来不转义
  - 支持格式化，即使用%s\%d\%f\%x作为占位符，使用值替换占位符。

```python
>>> print('\\\t\\')
\       \
>>> print(r'\\\t\\')
\\\t\\

>>> print('''line1
... line2
... line3''')
line1
line2
line3

>>> print(r'''hello,\n
world''')
hello,\n 
world 

# 如果只有一个占位符，括号可以省略
>>> 'Hello, %s' % 'world'
'Hello, world'
>>> 'Hi, %s, you have $%d.' % ('Michael', 1000000)
'Hi, Michael, you have $1000000.'

# 有些时候，字符串里面的%是一个普通字符怎么办？这个时候就需要转义，用%%来表示一个%：
>>> 'growth rate: %d %%' % 7
'growth rate: 7 %'

# 另一种格式化字符串的方法是使用字符串的format()方法，它会用传入的参数依次替换字符串内的占位符{0}、{1}……，不过这种方式写起来比%要麻烦得多：
>>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
'Hello, 小明, 成绩提升了 17.1%'
```

| 占位符 | 替换内容     |
| ------ | ------------ |
| %d     | 整数         |
| %f     | 浮点数       |
| %s     | 字符串       |
| %x     | 十六进制整数 |

- 布尔值
  - 布尔值和布尔代数的表示完全一致，一个布尔值只有`True`、`False`两种值，要么是`True`，要么是`False`，在Python中，可以直接用`True`、`False`表示布尔值（请注意大小写）
  - 布尔值可以用`and`、`or`和`not`运算。
- 空值
  - 空值是Python里一个特殊的值，用`None`表示。`None`不能理解为`0`，因为`0`是有意义的，而`None`是一个特殊的空值。
- 变量
  - 变量名必须是大小写英文、数字和`_`的组合，且不能用数字开头
  - Python 的变量为动态语言

```python
# 变量本身类型不固定的语言称之为 动态语言
a = 123 # a是整数
print(a)
a = 'ABC' # a变为字符串
print(a)

# 静态语言在定义变量时必须指定变量类型，如果赋值的时候类型不匹配，就会报错，例如java语言中
int a = 123; // a是整数类型变量
a = "ABC"; // 错误：不能把字符串赋给整型变量
```

- 常量
  - Python中，通常用**全部大写的变量名**表示常量，但是 Python 没有任何措施阻止改变常量

## 编码问题

[参考链接](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431664106267f12e9bef7ee14cf6a8776a479bdec9b9000)

由于Python源代码也是一个文本文件，所以，当你的源代码中包含中文的时候，在保存源代码时，就需要务必指定保存为UTF-8编码。当Python解释器读取源代码时，为了让它按UTF-8编码读取，我们通常在文件开头写上这两行：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```

第一行注释是为了告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释；

第二行注释是为了告诉Python解释器，按照UTF-8编码读取源代码，否则，你在源代码中写的中文输出可能会有乱码。

## list 与 tuple

Python 提供了两种类型的集合列表，一种是 list，另一种是 tuple。两者都是有序集合。

list 可以随时添加、删除、替换其中元素。而 tuple 是不可变的（指向不可变）

### list

```python
>>> classmates = ['TJwoods', 'Liliyard', 'yueban', 'zhongqiu']
>>> classmates
['TJwoods', 'Liliyard', 'yueban', 'zhongqiu']

>>> len(classmates)
4

>>> classmates[0]
TJwoods
# 越界
>>> classmates[10]
...（python报错）

# 倒数第一个元素，-2，-3如此类推，同样会存在越界可能
>>> classmates[-1]
zhongqiu

# 添加元素 append()
>>> classmates.append('doubi')
>>> classmates
['TJwoods', 'Liliyard', 'yueban', 'zhongqiu', 'doubi']

# 插入元素，插入到索引号为1的位置 insert()
>>> classmates.insert(1, 'insert')
>>> classmates
['TJwoods', 'insert', 'Liliyard', 'yueban', 'zhongqiu', 'doubi']

# 删除list末尾元素 pop()
>>> classmates.pop()
doubi
>>> classmates
['TJwoods', 'insert', 'Liliyard', 'yueban', 'zhongqiu']
>>> classmates.pop(1)
insert
>>> classmates
['TJwoods', 'insert', 'Liliyard', 'yueban', 'zhongqiu']

# 替换元素
>>> classmates[1] = 'Liliyard is doubi'
>>> classmates
['TJwoods', 'Liliyard is doubi', 'yueban', 'zhongqiu']

# list的元素可以为不同类型
>>> L = ['Apple', 123, True]

# list元素也可以是另一个list，采用二维三维的形式，像java一样
>>> s = ['python', 'java', ['asp', 'php'], 'scheme']
>>> len(s)
4

# 如果一个list中一个元素也没有，就是一个空的list，它的长度为0
>>> L = []
>>> len(L)
0
```

### tuple

tuple 与list类似，但是tuple是不可变的，起码是指向不可变的。一旦初始化后就不能修改。它就像list，但是没有了修改的函数。

```python
# 初始化
>>> t = (1, 2)
>>> t
(1, 2)

# 定义一个空的tuple
>>> t = ()
>>> t
()

# 定义一个只有1个元素的 tuple
>>> t = (1,)
>>> t
(1,)
```

这里为什么不使用 `t = (1)` ? 因为如果使用 `(1)` ，则Python会认为这里的括号是数字运算括号，结果还是1，那么只是定义了一个 t 的变量，其值为1。所以Python规定了定义只有一个元素的tuple时需要如此写。

```python
# "可变的"tuple
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```

这个tuple，虽然是"可变的"，这是因为其中的list是可变的，因为tuple实际上并不是固定不变，是指向不变，即tuple指向的地址不变，而地址指向的内容无法固定，list可以改变地址内容。如果希望tuple是真实不变的，那么其中的元素必须都是不可变的。

![](.\图片\可变的tuple.png)

## 条件与判断

```python
# 判断
if xxx:
    ...
elif xxx:
    ...
else:
    ...
 
# 这个会报错，因为input的问题
birth = input('birth: ')
if birth < 2000:
    print('00前')
else:
    print('00后')
```

例子中会报错是因为input返回的类型是 `str` ，`str`不能直接和整数比较，需要把`str`转成整数再比较。Python提供了`int()`函数来完成这件事情：

```python
s = input('birth: ')
birth = int(s)
if birth < 2000:
	print('00前')
else:
    print('00后')
```

这样就不会报错了，但是如果输入的字符串无法转为整数，如 `abc` 时，则仍然会报错。报错的处理就需要涉及到异常的处理机制，这个后面说。

## 循环

### **for...in**

依次把list或tuple中的每个元素迭代出来：

```python
names = ['TJwoods', 'Liliyard']
for name in names:
    print(name)
    
# 结果
TJwoods
Liliyard

sum = 0
for x in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
    sum = sum + x
print(sum)

# 如果要计算1-100的整数之和，从1写到100有点困难，幸好Python提供一个range()函数，可以生成一个整数序列，再通过list()函数可以转换为list。比如range(5)生成的序列是从0开始小于5的整数：
>>> list(range(5))
[0, 1, 2, 3, 4]

sum = 0
for x in range(101):
    sum = sum + x
print(sum)
```

### while

只要满足条件，就不断循环，不满足时退出循环

```python
while xxx:
    ...
    
sum = 0
n = 99
while n > 0:
    sum = sum + n
    n = n - 2
print(sum)
```

### break

提前退出当前循环

### continue

跳过当前的这次循环

## dict 和 set

### dict

python内置了字典，dict的支持，dict全称dictionary，就像其他语言里的map，使用**键值对**存储。

dict的key必须是**不可变对象**。

```python
>>> scores = {'TJwoods': 99, 'Liliyard': 100}
>>> scores['Liliyard']
100

# 把数据放入dict的方法，除了初始化时指定外，还可以通过key放入
>>> d['Adam'] = 67
>>> d['Adam']
67

# 如果key不存在，dict就会报错
>>> d['Thomas']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'Thomas'
    
# 要避免key不存在的错误，有两种办法，一是通过in判断key是否存在：
>>> 'Thomas' in d
False
# 二是通过dict提供的get()方法，如果key不存在，可以返回None，或者自己指定的value
# 注意：返回None的时候Python的交互环境不显示结果
>>> d.get('Thomas')
>>> d.get('Thomas', -1)
-1

# 要删除一个key，用pop(key)方法，对应的value也会从dict中删除
>>> d.pop('Bob')
75
>>> d
{'Michael': 95, 'Tracy': 85}
```

### set

set和dict类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在set中，没有重复的key。

```python
>>> s = set([1, 2, 3])
>>> s
{1, 2, 3}

# 重复元素在set中自动被过滤
>>> s = set([1, 1, 2, 2, 3, 3])
>>> s
{1, 2, 3}

# 通过add(key)方法可以添加元素到set中，可以重复添加，但不会有效果：
>>> s.add(4)
>>> s
{1, 2, 3, 4}
>>> s.add(4)
>>> s
{1, 2, 3, 4}

# 通过remove(key)方法可以删除元素
>>> s.remove(4)
>>> s
{1, 2, 3}

# set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作
>>> s1 = set([1, 2, 3])
>>> s2 = set([2, 3, 4])
>>> s1 & s2
{2, 3}
>>> s1 | s2
{1, 2, 3, 4}
```

