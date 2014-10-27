---
layout: post
title: "Python中代码风格的改变和相应的性能优化"
date: 2014-10-27 09:58:32 +0800
comments: true
categories: "Python"
---
##使用现代风格改善你的代码
原文： http://python3porting.com/improving.html<br>
译者： TheLover_Z<br>
转自： http://pycoders-weekly-chinese.readthedocs.org/en/latest/issue13/improving-your-code-with-modern-idioms.html

一旦你开始使用 Python 3，你就有机会接触新的特性来改善你的代码。这篇文章中提到的很多东西实际上在 Python 3 之前就已经被支持了。但我还是要提一下它们，因为知道了这些以后你的代码可以从中获益。我说的包括修饰器，在 Python 2.2 开始提供支持； sorted() 方法，在 Python 2.4 开始提供支持；还有上下文管理，在 Python 2.5 开始提供支持。

这里提及的其它新特性在 Python 2.6 或者 2.7 都提供了支持，所以说如果你不是在用 Python 2.5 和之前的版本的话，你可以使用这里提到的几乎全部的新特性。

###使用 sorted() 来替代 .sort()
在 Python 中，列表有一个 .sort() 方法可以进行排序。 .sort() 会影响列表的结构。下面这么写是因为在 Python 2.3 之前只能这么写。
```
>>> infile = open('pythons.txt')
>>> pythons = infile.readlines()
>>> pythons.sort()
>>> [x.strip() for x in pythons]
['Eric', 'Graham', 'John', 'Michael', 'Terry', 'Terry']
```
Python 2.4 开始加入了新的支持 sorted() ，它会返回一个排好序的列表并且接受和 .sort() 一样的参数。使用 sorted() 你可以避免改变列表的结构。它还可以接受迭代器作为输入而不只是列表，这样可以让你的代码看起来更棒。
```
>>> infile = open('pythons.txt')
>>> [x.strip() for x in sorted(infile)]
['Eric', 'Graham', 'John', 'Michael', 'Terry', 'Terry']
```
然而，如果你把 mylist.sort() 替换为 mylist = sorted(mylist) 是没有用的，而且还会消耗更多的内存。

2to3 有时会把 .sort() 改为 sorted() 。

###使用上下文管理器来编写代码
从 Python 2.5 开始你可以使用上下文管理器，它允许你创造和管理运行时内容。如果你觉得听起来有点儿抽象，那就对了。上下文管理器确实很抽象并且很灵活，很容易被误用，我这就教你怎么正确运用它。

上下文管理器被用来当作 with 的一部分，在 with 的代码块内都有效。在代码块结束的时候上下文管理器退出。这可能听起来不是那么令人激动，除非我告诉你你可以使用它来实现资源分配。你进入上下文的时候资源管理器分配资源，你退出的时候它释放资源。

with 语句

with 语句是被设计用来简化“try / finally”语句的。通常的用处在于共享资源的获取和释放，比如文件、数据库和线程资源。它的用法如下：
```
with context_exp [as var]:

        with_suit
```
with 语句也是复合语句的一种，就像 if、try 一样，它的后面也有个“：”，并且紧跟一个缩进的代码块 with_suit。context_exp 表达式的作用是提供一个上下文管理器（Context Manager），整个 with_suit 代码块都是在这个上下文管理器的运行环境下执行的。context_exp 可以直接是一个上下文管理器的引用，也可以是一句可执行的表达式，with 语句会自动执行这个表达式以获得上下文管理对象。with 语句的实际执行流程是这样的：

1.执行 context_exp 以获取上下文管理器<br>
2.加载上下文管理器的 __exit__() 方法以备稍后调用<br>
3.调用上下文管理器的 __enter__() 方法<br>
4.如果有 as var 从句，则将 __enter__() 方法的返回值赋给 var<br>
5.执行子代码块 with_suit<br>
6.调用上下文管理器的 __exit__() 方法，如果 with_suit 的退出是由异常引发的，那么该异常的 type、value 和 traceback 会作为参数传给 __exit__()，否则传三个 None<br>
7.如果 with_suit 的退出由异常引发，并且 __exit__() 的返回值等于 False，那么这个异常将被重新引发一次；如果 __exit__() 的返回值等于True，那么这个异常就被无视掉，继续执行后面的代码<br>
 

即，可以把 __exit__() 方法看成是“try / finally”的 finally，它总是会被自动调用。Python 里已经有了一些支持上下文管理协议的对象，比如文件对象，在使用 with 语句处理文件对象时，可以不再关心“打开的文件必须记得要关闭”这个问题了：

```
>>> with open('test.py') as f:
    print(f.readline())
 
     
#!/usr/bin/env python
 
>>> f.readline()
Traceback (most recent call last):
  File "<pyshell#3>", line 1, in <module>
    f.readline()
ValueError: I/O operation on closed file.
```
可以看到在 with 语句完成后，f 已经自动关闭了，这个过程就是在 f 的__exit__() 方法里完成的。然后下面再来详细介绍一下上下文管理器：

最常用的例子是读写文件。在大多数的更面向底层的语言中你必须记得关闭已打开的文件，但在 Python 中你不需要这么做。然而有时候你必须确认你关掉了文件，比如说你在循环中打开了许多文件以至于你用完了文件名。
```
>>> f = open('/tmp/afile.txt', 'w')
>>> try:
...     n = f.write('sometext')
... finally:
...     f.close()
```
你也可以这么写，使用上下文管理器。
```
>>> with open('/tmp/afile.txt', 'w') as f:
...     n = f.write('sometext')
```
当你使用上下文管理器的时候，代码块结束的时候文件就会自动关闭，就算是有错误发生也是这样。正如你所看到的那样，代码量少了很多，但是更重要的是程序看起来干净多了，也易读了。

另一个例子是如果你想要重定向标准输出。正如前面一样，你会使用 try/except 。那样也不错，如果你只使用一次的话。但是如果你有很多次这样的需求的话，上下文管理器是你不二的选择。
```
>>> import sys
>>> from StringIO import StringIO
>>> class redirect_stdout:
...     def __init__(self, target):
...         self.stdout = sys.stdout
...         self.target = target
...
...     def __enter__(self):
...         sys.stdout = self.target
...
...     def __exit__(self, type, value, tb):
...         sys.stdout = self.stdout
...
>>> out = StringIO()
>>> with redirect_stdout(out):
...     print 'Test'
...
>>> out.getvalue() == 'Test\n'
True
```
碰到 with 语句以后 __enter__ 方法被调用，退出的时候 __exit__() 被调用，包括引发错误。

上下文管理器在很多地方都可以使用。你的任何使用例外的代码最好确保资源或者全局变量没有被分配或者设置。

contextlib 库有各种各样的函数帮助你使用上下文管理器。比如说，如果你有一个有 .close() 方法但不是上下文管理器的对象，你可以使用 closing() 函数来在 with 块结束的时候自动关闭它们。
```
>>> from contextlib import closing
>>> import urllib
>>>
>>> book_url = 'http://python3porting.com/'
>>> with closing(urllib.urlopen(book_url)) as page:
...     print len(page.readlines())
117
```
###高级字符串格式化
在 Python 3 和 2.6 中，一种新的字符串格式支持被引进了。它更灵活并且有更聪明的语法。

旧的字符串格式：
```
>>> 'I %s Python %i' % ('like', 2)
'I like Python 2'
```
新的字符串格式：
```
>>> 'I {0} Python {1}'.format('♥', 3)
'I ♥ Python 3'
```
使用这些新特性你可以实现一些比较疯狂的小东西，但是玩过火的话你旧失去了它易读的优点：
```
>>> import sys
>>> 'Python {0.version_info[0]:!<9.1%}'.format(sys)
'Python 300.0%!!!'
```
更详细的文档请参考 Common String Operations 。

旧的字符串格式基于 % 的这个特性可能最终会被移除，不过最终日期还没有定。

###类修饰器
修饰器在 Python 2.4 的时候被支持，然后有了内置的修饰器比如说 @property 和 @classmethod ，修饰器开始变的流行。Python 2.6 引入了类修饰器。

类修饰器可以用来包裹类或者修饰类。一个例子就是 functools.total_ordering ，可以让你实现最小的富比较操作符，然后增加到你的类。它们可以作为元类，类修饰器的例子就是修饰器可以把类变成一个单独的类。 zope.interface 类修饰器可以注册一个作为特定接口的类。

###集合
Python 3 中引入了一种新的集合语法。相对于 set([1, 2, 3]) 你可以使用更干净语法的 {1, 2, 3} 。两种语法在 Python 3 中都可以工作，但是更建议使用新的语法。
```
>>> set([1,2,3])
{1, 2, 3}
```
###yield 和 生成器
就像浮点除法操作符和 .sort() 的 key 参数，生成器已经在不知不觉深入了我们的编码生活。虽然不多见，但它们还是非常实用的，可以帮你节省内存，简化代码。我们来看看这个例子：
```
>>> def allcombinations(starters, endings):
...    result = []
...    for s in starters:
...         for e in endings:
...             result.append(s+e)
...     return result
```
这么写就优雅多了：
```
>>> def allcombinations(starters, endings):
...     for s in starters:
...         for e in endings:
...             yield s+e
```
生成器在 Python 2.2 开始加入支持，但是 Python 2.4 进行了一些改进。看起来很像是列表表达式，但并不返回列表而是返回表达式。它们在有列表表达式的地方几乎都可以使用。
```
>>> sum([x*x for x in xrange(2000000)])
2666664666667000000L
```
可以写作：
```
>>> sum(x*x for x in xrange(2000000))
2666664666667000000L
```
###更多的推导式
在 Python 3 和 2.6 中，生成器推导式被引进。它就是简单的一个带括号的生成器表达式，可以和列表推导式一样工作，返回一个生成器而不是列表。
```
>>> (x for x in 'Silly Walk')
<generator object <genexpr> at ...>
```
在 Python 3 中生成器推导式不仅仅是一个新的漂亮的特性，而是一个重要的改变，因为生成器推导式现在是其它所有内置推导式的基础。在 Python 3 中列表推导式只是一个给 list 类型的构造器提供生成器表达式的语法糖。
```
>>> list(x for x in 'Silly Walk')
['S', 'i', 'l', 'l', 'y', ' ', 'W', 'a', 'l', 'k']

>>> [x for x in 'Silly Walk']
['S', 'i', 'l', 'l', 'y', ' ', 'W', 'a', 'l', 'k']
```
这也意味着循环变量再也不会掺入附近的命名空间了。

生成器推导式也可以用 Python 2.6 及其以后版本的 dict() 和 set() 构造器生成。但是在 Python 3 还有 Python 2.7 中，你可以用新的语法来定义字典和列表推导式：
```
>>> department = 'Silly Walk'
>>> {x: department.count(x) for x in department}
{'a': 1, ' ': 1, 'i': 1, 'k': 1, 'l': 3, 'S': 1, 'W': 1, 'y': 1}

>>> {x for x in department}
{'a', ' ', 'i', 'k', 'l', 'S', 'W', 'y'}
```
###新的模块
还有许多新的模块值得你一看。在这里我就不多说了，因为大多数如果你不重写软件的话可能获益不多，但你应该知道它们存在。你可以翻看一下 Python 文档来了解一下。

abc

abc 模块包含了对生成抽象的基础类的支持，你可以 标记 一个基础类的方法或者属性为“抽象”，意思是你必须在子类中进行实现，否则无法实例化。

抽象基础类也可以创建没有实体方法的类，用于定义接口。

abc 模块在 Python 2.6 及其以后的版本被支持。

multiprocessing 和 future

multiprocessing 是一个新的模块，用于进行多进程操作，它允许你拥有进程队列和使用锁，还有用于同步进程的 信号标 。

multiprocessing 在 Python 2.6 以后被加入支持。在 2.4 和 2.5 你可以使用 CheeseShop 。

如果你要做并发你可以看一下 future 模块，在 Python 3.2 引入了这个模块，在 Python 2.5 及以后的版本可以用 参考这里 。

numbers 和 fractions

Python 3 加入了这个库。大多数情况下你不会注意到它，但是很有趣的是 fractions 模块，在 Python 2.6 被支持。
```
>>> from fractions import Fraction
>>> Fraction(3,4) / Fraction('2/3')
Fraction(9, 8)
```
还有 numbers 模块，包含支持所有数字类型的抽象基础类。如果你正在实现你自己的数字类型的话，那么它非常有用。

中英文对照
生成器推导式 - generator comprehension

列表推导式 － list comprehension

生成器 － generator

抽象的基础类 － abstract base classes



##Python性能优化的20条建议
转自：http://segmentfault.com/blog/defool/1190000000666603

###1.优化算法时间复杂度

算法的时间复杂度对程序的执行效率影响最大，在Python中可以通过选择合适的数据结构来优化时间复杂度，如list和set查找某一个元素的时间复杂度分别是O(n)和O(1)。不同的场景有不同的优化方式，总得来说，一般有分治，分支界限，贪心，动态规划等思想。

###2.减少冗余数据

如用上三角或下三角的方式去保存一个大的对称矩阵。在0元素占大多数的矩阵里使用稀疏矩阵表示。

###3.合理使用copy与deepcopy

对于dict和list等数据结构的对象，直接赋值使用的是引用的方式。而有些情况下需要复制整个对象，这时可以使用copy包里的copy和deepcopy，这两个函数的不同之处在于后者是递归复制的。效率也不一样：（以下程序在ipython中运行）
```
import copy
a = range(100000)
%timeit -n 10 copy.copy(a) # 运行10次 copy.copy(a)
%timeit -n 10 copy.deepcopy(a)
10 loops, best of 3: 1.55 ms per loop
10 loops, best of 3: 151 ms per loop
```
timeit后面的-n表示运行的次数，后两行对应的是两个timeit的输出，下同。由此可见后者慢一个数量级。

###4.使用dict或set查找元素

python dict和set都是使用hash表来实现(类似c++11标准库中unordered_map)，查找元素的时间复杂度是O(1)
```
a = range(1000)
s = set(a)
d = dict((i,1) for i in a)
%timeit -n 10000 100 in d
%timeit -n 10000 100 in s
10000 loops, best of 3: 43.5 ns per loop
10000 loops, best of 3: 49.6 ns per loop
```
dict的效率略高(占用的空间也多一些)。

###5.合理使用生成器（generator）和yield
```
%timeit -n 100 a = (i for i in range(100000))
%timeit -n 100 b = [i for i in range(100000)]
100 loops, best of 3: 1.54 ms per loop
100 loops, best of 3: 4.56 ms per loop
```
使用()得到的是一个generator对象，所需要的内存空间与列表的大小无关，所以效率会高一些。在具体应用上，比如set(i for i in range(100000))会比set([i for i in range(100000)])快。

但是对于需要循环遍历的情况：
```
%timeit -n 10 for x in (i for i in range(100000)): pass
%timeit -n 10 for x in [i for i in range(100000)]: pass
10 loops, best of 3: 6.51 ms per loop
10 loops, best of 3: 5.54 ms per loop
```
后者的效率反而更高，但是如果循环里有break,用generator的好处是显而易见的。yield也是用于创建generator：
```
def yield_func(ls):
    for i in ls:
        yield i+1

def not_yield_func(ls):
    return [i+1 for i in ls]

ls = range(1000000)
%timeit -n 10 for i in yield_func(ls):pass
%timeit -n 10 for i in not_yield_func(ls):pass
10 loops, best of 3: 63.8 ms per loop
10 loops, best of 3: 62.9 ms per loop
```
对于内存不是非常大的list，可以直接返回一个list，但是可读性yield更佳(人个喜好)。

python2.x内置generator功能的有xrange函数、itertools包等。

###6.优化循环

循环之外能做的事不要放在循环内，比如下面的优化可以快一倍：
```
a = range(10000)
size_a = len(a)
%timeit -n 1000 for i in a: k = len(a)
%timeit -n 1000 for i in a: k = size_a
1000 loops, best of 3: 569 µs per loop
1000 loops, best of 3: 256 µs per loop
```
###7.优化包含多个判断表达式的顺序

对于and，应该把满足条件少的放在前面，对于or，把满足条件多的放在前面。如：
```
a = range(2000)  
%timeit -n 100 [i for i in a if 10 < i < 20 or 1000 < i < 2000]
%timeit -n 100 [i for i in a if 1000 < i < 2000 or 100 < i < 20]     
%timeit -n 100 [i for i in a if i % 2 == 0 and i > 1900]
%timeit -n 100 [i for i in a if i > 1900 and i % 2 == 0]
100 loops, best of 3: 287 µs per loop
100 loops, best of 3: 214 µs per loop
100 loops, best of 3: 128 µs per loop
100 loops, best of 3: 56.1 µs per loop
```
###8.使用join合并迭代器中的字符串
```
In [1]: %%timeit
   ...: s = ''
   ...: for i in a:
   ...:         s += i
   ...:
10000 loops, best of 3: 59.8 µs per loop

In [2]: %%timeit
s = ''.join(a)
   ...:
100000 loops, best of 3: 11.8 µs per loop
```
join对于累加的方式，有大约5倍的提升。

###9.选择合适的格式化字符方式
```
s1, s2 = 'ax', 'bx'
%timeit -n 100000 'abc%s%s' % (s1, s2)
%timeit -n 100000 'abc{0}{1}'.format(s1, s2)
%timeit -n 100000 'abc' + s1 + s2
100000 loops, best of 3: 183 ns per loop
100000 loops, best of 3: 169 ns per loop
100000 loops, best of 3: 103 ns per loop
```
三种情况中，%的方式是最慢的，但是三者的差距并不大（都非常快）。(个人觉得%的可读性最好)

###10.不借助中间变量交换两个变量的值
```
In [3]: %%timeit -n 10000
    a,b=1,2
   ....: c=a;a=b;b=c;
   ....:
10000 loops, best of 3: 172 ns per loop

In [4]: %%timeit -n 10000
a,b=1,2
a,b=b,a
   ....:
10000 loops, best of 3: 86 ns per loop
```
使用a,b=b,a而不是c=a;a=b;b=c;来交换a,b的值，可以快1倍以上。

###11.使用if is
```
a = range(10000)
%timeit -n 100 [i for i in a if i == True]
%timeit -n 100 [i for i in a if i is True]
100 loops, best of 3: 531 µs per loop
100 loops, best of 3: 362 µs per loop
```
使用 if is True 比 if == True 将近快一倍。

###12.使用级联比较x < y < z
```
x, y, z = 1,2,3
%timeit -n 1000000 if x < y < z:pass
%timeit -n 1000000 if x < y and y < z:pass
1000000 loops, best of 3: 101 ns per loop
1000000 loops, best of 3: 121 ns per loop
```
x < y < z效率略高，而且可读性更好。

###13.while 1 比 while True 更快
```
def while_1():
    n = 100000
    while 1:
        n -= 1
        if n <= 0: break
def while_true():
    n = 100000
    while True:
        n -= 1
        if n <= 0: break    

m, n = 1000000, 1000000 
%timeit -n 100 while_1()
%timeit -n 100 while_true()
100 loops, best of 3: 3.69 ms per loop
100 loops, best of 3: 5.61 ms per loop
```
while 1 比 while true快很多，原因是在python2.x中，True是一个全局变量，而非关键字。

###14.使用**而不是pow
```
%timeit -n 10000 c = pow(2,20)
%timeit -n 10000 c = 2**20
10000 loops, best of 3: 284 ns per loop
10000 loops, best of 3: 16.9 ns per loop
```
**就是快10倍以上！

###15.使用 cProfile, cStringIO 和 cPickle等用c实现相同功能（分别对应profile, StringIO, pickle）的包
```
import cPickle
import pickle
a = range(10000)
%timeit -n 100 x = cPickle.dumps(a)
%timeit -n 100 x = pickle.dumps(a)
100 loops, best of 3: 1.58 ms per loop
100 loops, best of 3: 17 ms per loop
```
由c实现的包，速度快10倍以上！

###16.使用最佳的反序列化方式

下面比较了eval, cPickle, json方式三种对相应字符串反序列化的效率：
```
import json
import cPickle
a = range(10000)
s1 = str(a)
s2 = cPickle.dumps(a)
s3 = json.dumps(a)
%timeit -n 100 x = eval(s1)
%timeit -n 100 x = cPickle.loads(s2)
%timeit -n 100 x = json.loads(s3)
100 loops, best of 3: 16.8 ms per loop
100 loops, best of 3: 2.02 ms per loop
100 loops, best of 3: 798 µs per loop
```
可见json比cPickle快近3倍，比eval快20多倍。(这个跟前文测试有冲突，数据量的问题)

###17.使用C扩展(Extension)

目前主要有CPython(python最常见的实现的方式)原生API, ctypes,Cython，cffi三种方式，它们的作用是使得Python程序可以调用由C编译成的动态链接库，其特点分别是：

CPython原生API: 通过引入Python.h头文件，对应的C程序中可以直接使用Python的数据结构。实现过程相对繁琐，但是有比较大的适用范围。

ctypes: 通常用于封装(wrap)C程序，让纯Python程序调用动态链接库（Windows中的dll或Unix中的so文件）中的函数。如果想要在python中使用已经有C类库，使用ctypes是很好的选择，有一些基准测试下，python2+ctypes是性能最好的方式。

Cython: Cython是CPython的超集，用于简化编写C扩展的过程。Cython的优点是语法简洁，可以很好地兼容numpy等包含大量C扩展的库。Cython的使得场景一般是针对项目中某个算法或过程的优化。在某些测试中，可以有几百倍的性能提升。

cffi: cffi的就是ctypes在pypy（详见下文）中的实现，同进也兼容CPython。cffi提供了在python使用C类库的方式，可以直接在python代码中编写C代码，同时支持链接到已有的C类库。

使用这些优化方式一般是针对已有项目性能瓶颈模块的优化，可以在少量改动原有项目的情况下大幅度地提高整个程序的运行效率。

###18.并行编程

因为GIL的存在，Python很难充分利用多核CPU的优势。但是，可以通过内置的模块multiprocessing实现下面几种并行模式：

多进程：对于CPU密集型的程序，可以使用multiprocessing的Process,Pool等封装好的类，通过多进程的方式实现并行计算。但是因为进程中的通信成本比较大，对于进程之间需要大量数据交互的程序效率未必有大的提高。

多线程：对于IO密集型的程序，multiprocessing.dummy模块使用multiprocessing的接口封装threading，使得多线程编程也变得非常轻松(比如可以使用Pool的map接口，简洁高效)。

分布式：multiprocessing中的Managers类提供了可以在不同进程之共享数据的方式，可以在此基础上开发出分布式的程序。

不同的业务场景可以选择其中的一种或几种的组合实现程序性能的优化。

###19.终级大杀器：PyPy

PyPy是用RPython(CPython的子集)实现的Python，根据官网的基准测试数据，它比CPython实现的Python要快6倍以上。快的原因是使用了Just-in-Time(JIT)编译器，即动态编译器，与静态编译器(如gcc,javac等)不同，它是利用程序运行的过程的数据进行优化。由于历史原因，目前pypy中还保留着GIL，不过正在进行的STM项目试图将PyPy变成没有GIL的Python。

如果python程序中含有C扩展(非cffi的方式)，JIT的优化效果会大打折扣，甚至比CPython慢（比Numpy）。所以在PyPy中最好用纯Python或使用cffi扩展。

随着STM，Numpy等项目的完善，相信PyPy将会替代CPython。

###20.使用性能分析工具

除了上面在ipython使用到的timeit模块，还有cProfile。cProfile的使用方式也非常简单： python -m cProfile filename.py，filename.py 是要运行程序的文件名，可以在标准输出中看到每一个函数被调用的次数和运行的时间，从而找到程序的性能瓶颈，然后可以有针对性地优化。

参考

[1] http://www.ibm.com/developerworks/cn/linux/l-cn-python-optim/

[2] http://maxburstein.com/blog/speeding-up-your-python-code/


备注：<br>
1.降低方法调用次数，如果你有一个列表需要操作，传递整个列表，而不是遍历整个列表并且传递每个元素给函数并返回。<br>
2.使用 xrange 代替 range。（在 Python2.x 中这样做，因为 Python 3.x 中是默认的）xrange 是 range 的 C 实现，着眼于有效的内存使用。<br>
3.对于大数据，使用 numpy，它比标准的数据结构好很多。<br>
4."".join(string) 比 + or += 好<br>
5.while 1 比 while True 快<br>
6.list comphrension > for loop > while<br>
7.列表推导比循环遍历列表快，但 while loop 是最慢的，需要使用一个外部计数器。<br>
8.使用 cProfile，cStringIO 和 cPickle<br>
一直使用 C 版本的模块<br>
9.使用局部变量<br>
局部变量比全局变量，内建类型以及属性快。<br>
10.列表和迭代器版本存在 - 迭代器是内存效率和可伸缩性的。使用 itertools<br>
11.创建生成器以及尽可能使用 yeild，它们比正常的列表方式更快。<br>
12.使用 Map ，Reduce 和 Filter 代替 for 循环<br>
13.校验 a in b， 字典 或 set 比 列表 或 元组 更好<br>
14.当数据量大的时候，尽可能使用不可变数据类型，他们更快 元组 > 列表<br>
15.在一个列表中插入数据的复杂度为 O(n)<br>
16.如果你需要操作列表的两端，使用 deque<br>
17.del - 删除对象使用如下<br>
1） python 自己处理它，但确保使用了 gc 模块<br>
2） 编写 __del__ 函数<br>
3） 最简单的方式，使用后调用 del<br>
18.time.clock()<br>
19.GIL(http://wiki.python.org/moin/GlobalInterpreterLock) - GIL is a daemon<br>
GIL 仅仅允许一个 Python 的原生线程来运行每个进程。阻止 CPU 级别的并行，尝试使用 ctypes 和 原生的 C 库来解决它，当你达到 Python 优化的最后，总是存在一个选项，可以使用原生的 C 重写慢的函数，通过 Python 的 C 绑定使用它，其他的库如 gevent 也是致力于解决这个问题，并且获得了成功。
TL,DR：当你写代码了，过一遍数据结构，迭代结构，内建和为 GIL 创建 C 扩展，如有必要。
更新：multiprocessing 是在 GIL 的范围之外，这意味着你可以使用 multiprocessing 这个标准库来运行多个进程。




