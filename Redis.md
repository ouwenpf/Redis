##Radis
<pre>
Redis:
    KV cache and store
    in-memory
    持久化
    主从(借助于sentinel实现一定意义上的HA)
    Clustering(分布式)

数据结构服务器：
    string，list，Hash，Set，Sored Set 


存储系统有三类：
    RDBMS
    NoSQL
        Key-Value NoSQL
            memcached Redis
        Column family NoSQL
            HBase
        Document NoSQL
            MongoDB
        Graph NoSQL：Neo4j
    
    NewSQL

 Redis的组件：   
    redis-server
    redis-cli
    redis-benchmark
    redis-check-dump & redis-check-aof

Redis守护进程：
    监听的端口：6379/tcp
    
    string:
        SET key value [EX seconds设置多少秒钟过期][NX不存在创建|XX存在即修改]
        GET
        INCR
        DECR
        EXIST
    list：
        LPUSH
        RPUSH
        LPOP
        RPOP
        LINDEX
        LSET

    set：
        SADD
        SINTER
        SUNION
        SPOP
        SISMEMBER
    sorted set有序集合
        ZADD 
        ZRANGE
        ZCARD
        ZRANK

    hashe：
        HSET
        HGET
        HDEL
        HSETNX
        HVALS
        HKEYS
        
bitmaps hyperloglog    
认证实现方法：
    1. redis.conf
        requirepass PASSWORD
    2. redis-cli
        AUTH PASSWORD
清空数据库：
    FLUSHDB：清空当前库
    FLUSHALL：清空所有库

事务：
    通过MULTI，EXEC，WAtCH等命令实现事务功能；将一个或多个命令归并为一个操作提请服务器按顺序执行的机制，不支持回滚操作
    MULTI：启动一个事务
    EXEC：执行事务
        一次性将事务中所有操作完成后返回给你客户端
    WATCH：乐观锁，在exec命令执行之前，用于监视指定数量的键，如果监视中的某任意键数据被修改，则服务器拒绝执行事务

   
connection相关命令

server相关的命令：
    CLIENT GETNAME  
    CLIENT KILL ip：prot
    CLIENT SETNAME
    
    config set
    config set paramenter value
    config rewrite

发布于订阅(publish/subscribe)
    频道：消息队列
    subscribe：订阅一个或多个队列
    PUBLISH：向频道发布消息
    unsubscribe：退订此前订阅的频道
    psubscribe：模式订阅
</pre>


Redis的持久化：
    RDB和AOF
        RDB：snapshot，二进制格式，按事先定制的策略，周期性地将数据保存至磁盘，数据文件默认为dump.rdb
             客户端也可显示使用SAVA或BGSAVE命令启用快照保存机制
                SAVE：同步；在主线程中保存快照，此时会阻塞所有客户端请求
                BGSAVE：异步，
                RDB工作模式：fork一个子进程，主进程继续处理各个客户端发来的请求，子进程负责把内存的数据快照到磁盘上面来，
                
                相关参数：
                    RDB：
                    save 900 1
                    save 300 10
                    save 60 10000
                
                    stop-writes-on-bgsave-error yes
                    rdbcompression yes
                    rdbchecksum yes
                    dbfilename dump.rdb 
                    dir ./

        AOF：Append Only File
             记录每一次写操作至指定的文件尾部实现持久化，当redis重启时，可通过重新执行文件中的命令在内存重建数据库
                bgrewritaof：aof文件重写，不会读取正在使用AOF，而通过将内存中的数据以命令的方式保存到临时文件，完成之后替换原来的aof文件.
            AOF重写过程：
                1. redis主进程通过fork创建子进程
                2. 子进程根据redis内存中的数据创建数据库重建命令序列于临时文件中
                3. 父进程基础clinet的请求，并会把这些请求的写操作继续追加至原来的AOF文件；额外地，这些新的写请求还会被置于一个缓冲队列中
                4. 子进程重写完成，会通知父进程，父进程把缓冲的命令写到临时文件中
                5. 父进程用临时文件替换老的aof文件
            相关参数：
                AOF：
                appendonly no
                appendfilename "appendonly.aof"
                appendfsync {always|everysec|no}
                no-appendfsync-on-rewrite no
                    
            注意：持久本身不能取代备份，还应该定制备份策略，对redis数据库定期进行备份

        RDB有AOF同时启动
            1. BGSAVE和BGREWRITAOF不会同时执行
            2. 在redis服务启动用于回复数据时，会优先使用AOF

复制：
    特点：
        一个Master可以有多个Slave
        文件链式复制
        Master以非阻塞方式同步数据至slave





Clustering：
    分布式数据，通过分片机制进行数据分布，clustering内的每个节点仅数据库的一部分数据
               
![](https://i.imgur.com/19zMn3G.png)
