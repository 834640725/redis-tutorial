#### 1. 介绍

最近有个朋友问我redis持久化的问题，由于比较忙，虽然跟他说了解决方法，但是不够细，本来准备想写篇文章来介绍redis持久化，可是发现已经有人写好了文章，且是比较精华的，故而找出来分享。

* [redis的持久化](https://segmentfault.com/a/1190000002906345)

* [翻译的官方文档](http://redisdoc.com/topic/persistence.html)

* [设计与实现一书中的rdb章节](http://redisbook.readthedocs.io/en/latest/internal/rdb.html)

* [设计与实现一书中的aof章节](http://redisbook.readthedocs.io/en/latest/internal/aof.html)

redis 3.2.5 默认情况下关于redis的配置:

```
save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes

rdbcompression yes

rdbchecksum yes

dbfilename dump.rdb

dir /usr/local/var/db/redis/
```

关于aof的配置:

```
appendonly no

appendfilename "appendonly.aof"

appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-load-truncated yes
```

可见默认只开了rdb模式，没有开启aof模式。

#### 2. 读取rdb文件

rdb文件是个二进制形式的，只能用特殊的工具读写里面的内容，有人写好了，就是下面这个：

https://github.com/sripathikrishnan/redis-rdb-tools

还有一个nodejs写的工具：

https://github.com/jeremyfa/node-redis-dump


