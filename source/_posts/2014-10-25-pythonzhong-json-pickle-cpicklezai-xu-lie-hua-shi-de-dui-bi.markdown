---
layout: post
title: "Python中json_pickle_cPickle在序列化时的性能对比"
date: 2014-10-25 10:51:57 +0800
comments: true
categories: "Python"
---
在上篇介绍python中装饰器的文章中(http://yindashan.github.io/blog/2014/10/24/pythonzhong-zhuang-shi-qi-xiang-jie/)提到一个装饰器实例，原来的打算是利用redis缓存减少请求的响应时间，第一次从数据库中获取数据，把数据回种到缓存，然后返回数据，第二次请求的时候直接从redis缓存中读取就可以了，经过实践发现性能并没有提升，反而下降了，这到底是什么情况呢？后来经过代码分析发现时间都花费在下面这行代码上了，足足会有4s.
```
json.loads(ret_data)
```
这个函数平常用的时候性能可以啊，完全能够满足自己的需求，这次到底是怎么回事？又经过一番分析，发现是由于ret_data数据量太大导致的，数据大概在15M左右，所以要把15M的字符数据转换成json对象这个太耗时了。从数据库中取数据才需要2s左右，现在从redis中取并解析完成需要将近5s,这显然不能满足需求。


既然在数据量比较大的情况下json.loads()函数性能不能满足需求，那就得寻找其他的解决方案了，最终发现python中的内建序列化库pickle和cPickle是一个不错的选择，想到就测，发现在同等数据量的情况下json.loads()需要消耗4s左右，pickle只需要消耗1.5s左右，而cPickle更快，只需要消耗300ms左右,取其中一组测试结果：
```
hget cost time:  0:00:00.052490
json.loads cost time:  0:00:04.859132
pickle.loads cost time:  0:00:01.458769
cPickle.loads cost time:  0:00:00.361623
```

测试代码如下：
```
import json
import time
import datetime
import traceback
import pickle
import cPickle

import redis

def test_redis():
    """
    测试redis
    """
    redis_db = redis.StrictRedis(host='127.0.0.1', port=6379, db=0, password='N6MXWf')
    
    d1 = datetime.datetime.now()
    ret_data = redis_db.hget('forecast_redis', '20141023')
    d2 = datetime.datetime.now()
    print 'hget cost time: ', d2 - d1
    ret_data = json.loads(ret_data)
    d3 = datetime.datetime.now()
    print 'json.loads cost time: ', d3 - d2
    ret_data = redis_db.hget('forecast_redis', '20141024')
    d4 = datetime.datetime.now()
    ret_data = pickle.loads(ret_data)
    d5 = datetime.datetime.now()
    print 'pickle.loads cost time: ', d5 - d4
    ret_data = redis_db.hget('forecast_redis', '20141025')
    d6 = datetime.datetime.now()
    ret_data = cPickle.loads(ret_data)
    d7 = datetime.datetime.now()
    print 'cPickle.loads cost time: ', d7 - d6
    print 'success'
    
    
def main():
    test_redis()

if __name__ == "__main__":
    main()
```

