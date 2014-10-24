---
layout: post
title: "python中装饰器详解"
date: 2014-10-24 21:59:08 +0800
comments: true
categories: "Python"
---
一直对装饰器的概念很模糊，今天终于花时间重点研究了一下。Python中的装饰器就类似于Java中的面向切面编程，就是在函数执行前和执行后包装自定义的一些东西。关于装饰器的原理和简易实现可以参考这位兄弟的博客，写的很简单易懂。

http://www.cnblogs.com/huxi/archive/2011/03/01/1967600.html
##1. 装饰器入门
###1.1. 需求是怎么来的？
装饰器的定义很是抽象，我们来看一个小例子。

```
def foo():
    print 'in foo()'

foo()
```
这是一个很无聊的函数没错。但是突然有一个更无聊的人，我们称呼他为B君，说我想看看执行这个函数用了多长时间，好吧，那么我们可以这样做：

```
import time
def foo():
    start = time.clock()
    print 'in foo()'
    end = time.clock()
    print 'used:', end - start
 
foo()
```
很好，功能看起来无懈可击。可是蛋疼的B君此刻突然不想看这个函数了，他对另一个叫foo2的函数产生了更浓厚的兴趣。

怎么办呢？如果把以上新增加的代码复制到foo2里，这就犯了大忌了~复制什么的难道不是最讨厌了么！而且，如果B君继续看了其他的函数呢？

###1.2. 以不变应万变，是变也
还记得吗，函数在Python中是一等公民，那么我们可以考虑重新定义一个函数timeit，将foo的引用传递给他，然后在timeit中调用foo并进行计时，这样，我们就达到了不改动foo定义的目的，而且，不论B君看了多少个函数，我们都不用去修改函数定义了！

```
import time
 
def foo():
    print 'in foo()'
 
def timeit(func):
    start = time.clock()
    func()
    end =time.clock()
    print 'used:', end - start
 
timeit(foo)
```
看起来逻辑上并没有问题，一切都很美好并且运作正常！……等等，我们似乎修改了调用部分的代码。原本我们是这样调用的：foo()，修改以后变成了：timeit(foo)。这样的话，如果foo在N处都被调用了，你就不得不去修改这N处的代码。或者更极端的，考虑其中某处调用的代码无法修改这个情况，比如：这个函数是你交给别人使用的。

###1.3. 最大限度地少改动！
既然如此，我们就来想想办法不修改调用的代码；如果不修改调用代码，也就意味着调用foo()需要产生调用timeit(foo)的效果。我们可以想到将timeit赋值给foo，但是timeit似乎带有一个参数……想办法把参数统一吧！如果timeit(foo)不是直接产生调用效果，而是返回一个与foo参数列表一致的函数的话……就很好办了，将timeit(foo)的返回值赋值给foo，然后，调用foo()的代码完全不用修改！

```
#-*- coding: UTF-8 -*-
import time
 
def foo():
    print 'in foo()'
 
# 定义一个计时器，传入一个，并返回另一个附加了计时功能的方法
def timeit(func):
     
    # 定义一个内嵌的包装函数，给传入的函数加上计时功能的包装
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
     
    # 将包装后的函数返回, 记住一定要返回 ，不然外面调用foo的地方将会无函数可用。实际上此时foo=timeit(foo)
    return wrapper
 
foo = timeit(foo)
foo()
```
这样，一个简易的计时器就做好了！我们只需要在定义foo以后调用foo之前，加上foo = timeit(foo)，就可以达到计时的目的，这也就是装饰器的概念，看起来像是foo被timeit装饰了。在在这个例子中，函数进入和退出时需要计时，这被称为一个横切面(Aspect)，这种编程方式被称为面向切面的编程(Aspect-Oriented Programming)。与传统编程习惯的从上往下执行方式相比较而言，像是在函数执行的流程中横向地插入了一段逻辑。在特定的业务领域里，能减少大量重复代码。面向切面编程还有相当多的术语，这里就不多做介绍，感兴趣的话可以去找找相关的资料。

这个例子仅用于演示，并没有考虑foo带有参数和有返回值的情况，完善它的重任就交给你了 ：）

##2. Python的额外支持
###2.1. 语法糖
上面这段代码看起来似乎已经不能再精简了，Python于是提供了一个语法糖来降低字符输入量。

```
import time
 
def timeit(func):
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
 
@timeit
def foo():
    print 'in foo()'
 
foo()
```
重点关注第11行的@timeit，在定义上加上这一行与另外写foo = timeit(foo)完全等价，千万不要以为@有另外的魔力。除了字符输入少了一些，还有一个额外的好处：这样看上去更有装饰器的感觉。

###2.2. 内置的装饰器
内置的装饰器有三个，分别是staticmethod、classmethod和property，作用分别是把类中定义的实例方法变成静态方法、类方法和类属性。由于模块里可以定义函数，所以静态方法和类方法的用处并不是太多，除非你想要完全的面向对象编程。而属性也不是不可或缺的，Java没有属性也一样活得很滋润。从我个人的Python经验来看，我没有使用过property，使用staticmethod和classmethod的频率也非常低。

```
class Rabbit(object):
     
    def __init__(self, name):
        self._name = name
     
    @staticmethod
    def newRabbit(name):
        return Rabbit(name)
     
    @classmethod
    def newRabbit2(cls):
        return Rabbit('')
     
    @property
    def name(self):
        return self._name
```
这里定义的属性是一个只读属性，如果需要可写，则需要再定义一个setter：

```
@name.setter
def name(self, name):
    self._name = name
```

###2.3. functools模块
functools模块提供了两个装饰器。这个模块是Python 2.5后新增的，一般来说大家用的应该都高于这个版本。但我平时的工作环境是2.4 T-T

####2.3.1. wraps(wrapped[, assigned][, updated]): 
这是一个很有用的装饰器。看过前一篇反射的朋友应该知道，函数是有几个特殊属性比如函数名，在被装饰后，上例中的函数名foo会变成包装函数的名字wrapper，如果你希望使用反射，可能会导致意外的结果。这个装饰器可以解决这个问题，它能将装饰过的函数的特殊属性保留。

```
import time
import functools
 
def timeit(func):
    @functools.wraps(func)
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
 
@timeit
def foo():
    print 'in foo()'
 
foo()
print foo.__name__
```
首先注意第5行，如果注释这一行，foo.__name__将是'wrapper'。另外相信你也注意到了，这个装饰器竟然带有一个参数。实际上，他还有另外两个可选的参数，assigned中的属性名将使用赋值的方式替换，而updated中的属性名将使用update的方式合并，你可以通过查看functools的源代码获得它们的默认值。对于这个装饰器，相当于wrapper = functools.wraps(func)(wrapper)。


##3. 下面是我本地测试的一些示例
###3.1. 普通装饰器

```
def common(func):
    '''普通装饰器'''
    def _deco(*args, **kwargs):
        print 'args:', args
        return func(*args, **kwargs)
    return _deco

@common
def test_common(p):
    print p

def main():
	test_common(1)

if __name__ == "__main__":
    main()
```

###3.2. 给函数的类装饰器(避免在装饰器对象上保留状态)
```
class Common(object):
    '''给函数的类装饰器(避免在装饰器对象上保留状态)'''
    def __init__(self, func):
        self.func = func
    def __call__(self, *args, **kwargs):
        print 'args:', args
        return self.func(*args, **kwargs)

@Common
def test_common_class(p):
    print p

def main():
	test_common_class(2)

if __name__ == "__main__":
    main()
```

###3.3. 带参数的装饰器
```
def common_arg(*args, **kw):
    '''带参数的装饰器'''
    a = args
    b = kw
    def _common_arg(func):
        def _deco(*args, **kwargs):
            print 'args:', args, a, b
            return func(*args, **kwargs)
        return _deco
    return _common_arg


@common_arg('c', 'd', e=1)
def test_common_arg(p):
    print p

def main():
	test_common_arg(3)

if __name__ == "__main__":
    main()

```

###3.4. 一个比较实用的示例
模拟从数据库中获取数据，第一次从数据库中获取，获取成功后保存到redis里面，以后每次都从redis里面获取
```
import json
import redis

def redis_cache(*args, **kwargs):
    redis_link_dict = kwargs
    print redis_link_dict
    redis_db = redis.StrictRedis(redis_link_dict['host'], redis_link_dict['port'], redis_link_dict['db'], redis_link_dict['password'])
    def _decorator(func):
        def _wrapped(*args, **kwargs):
            key, hash_key = args
            print key, hash_key
            # 判断当前key是否存在
            is_key = redis_db.exists(key)
            if is_key:
                # 判断当前key下是否有hash_key
                is_hash_key = redis_db.hexists(key, hash_key)
                if is_hash_key:
                    ret_data = redis_db.hget(key, hash_key)
                    if ret_data:
                        print u'从redis中获取数据'
                        return json.loads(ret_data)
            print u'从数据库中获取并写入到redis'
            ret_data = func(*args, **kwargs)
            redis_db.hset(key, hash_key, json.dumps(ret_data))
            return ret_data
        return _wrapped
    return _decorator


@redis_cache(host='127.0.0.1', port=6379, db=0, password='N6MXWf')
def get_data_from_redis_or_db(key, hash_key):
    '''
    获取数据
    '''
    ret_data = {
        "username":"dashan",
        "datetime":"20141024"
    }
    return ret_data

def main():
    key = 'dashan_hash'
    hash_key = '20141024'
    print get_data_from_redis_or_db(key, hash_key)

if __name__ == "__main__":
    main()

```

