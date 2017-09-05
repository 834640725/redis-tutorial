#### 1. redis的复制

[redis的复制](http://redisdoc.com/topic/replication.html)除去源码级的代码不说，要操作起来实在太简单，我们这里就不介绍它的定义了。

它具体的内容都可以在官方文档查到。

 这一节只是来介绍它的实践和与nginx的结合。

首先，已经有了一台主服务器，监听在6379端口。

它的配置文件位于`/usr/local/etc/redis.conf`。

现在我们要制造一个从服务器。

那很简单，只要监听在别的端口就可以了。

我们把配置文件复制一份，改成`/usr/local/etc/redis-slave.conf`

找到类似下面的内容改一下：

``` conf
pidfile /usr/local/var/run/redis-slave.pid

port 6380
```

我们把从服务器监听在6380端口。

再把它启动起来。

``` bash
$ redis-server /usr/local/etc/redis-slave.conf
```

现在我们需要开启一个客户端，连接上从服务器，再去同步主服务器的数据。

我们使用redis-cli客户端。

``` bash
$ redis-cli -p 6380
```

接着执行以下命令表示设置为从服务器。

```
slaveof 127.0.0.1 6379
```

现在已接完成了redis的复制了，至于效果，可以在主服务器上添加一些内容，再到从服务器上看是否有一样的结果。

如果要查看复制的情况，可以用下面的命令：

``` bash
$ info replication
```

输出：

```
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=753,lag=1
master_repl_offset:753
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:752
```

#### 2. nginx的tcp负载均衡

nginx自从1.9之后，就有tcp负载均衡了。

它使用的是[ngx_stream_core_module](http://nginx.org/en/docs/stream/ngx_stream_core_module.html)这个模块，默认是没有编译的，你要重新编译一下。重新编译的时候使用`--with-stream`这个选项即可。

在nginx配置文件中，添加类似这样的内容：

``` conf
stream {
    upstream backend {
        server 127.0.0.1:6379            max_fails=3 fail_timeout=30s;
        server 127.0.0.1:6380            max_fails=3 fail_timeout=30s;
    }

    server {
        listen 6378;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }
}
```

**注意：stream是和http同级的**

这样，用`redis-cli`连接上6378端口。就可以进行尝试了。

#### 3. 总结

**上面的例子只是一个案例，只是为了演示，在生产环境中你可能需要改造一下。**

一般来说，主服务器只有写的功能，从服务器就只有读的功能。

在程序里控制一下，写的功能指定为主服务器，从服务器那边做负载均衡就可以了。

有时候主服务器会挂掉，这个时候可以就需要`redis sentinel`的功能了。

本篇完结。
