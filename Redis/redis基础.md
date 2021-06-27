##### redis 常用的数据结构

- String 可以用来计数，缓存，分布式锁
- Hash 可以用来保存用户的一些信息，用户的详情页
- list 可以用来做队列 ，可以用来做栈，可以用来做数据，可以维护一个评论表。
- Set 可以用来做 交集 并集 差集 ，微博抽奖，随机事件问题。无序、去重
- Sorted set 可以用来做排行榜，带权重的队列

##### 分布式锁

最早的 redis 设置锁

```
setnx lock:codehole true
expire lock:codehole 5
del lock:codehole
```

如果setnx 和 expire 之间服务器进程突然挂掉了，可能是因为机器掉电或者是人为造成的，就会导致 expire 得不到执行，也会造成死锁。

解决方案

Redis2.8 加入了set 指令的扩展参数，使得setnx和e xpire指令可以一起执行。

```
set lock:codehole true ex 5 nx
del lock:codehole
```

NX : 保证只有键不存在的时候插入

PX timeout: 设置键的过期毫米数

ex timeout: 设置键的过期秒数