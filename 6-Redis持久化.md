
# Redis专业环境安装

## Redis下载

Redis下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[Redis下载地址](https://download.redis.io/releases/)   
[Ruby下载地址](https://www.ruby-lang.org/zh_cn/downloads/)

- Redis持久化

```
redis 支持RDB 和AOF 两种持久化机制，持久化可以避免因进程退出而造成数据丢失
1. RDB 持久化
RDB 持久化把当前进程数据生成快照（.rdb）文件保存到硬盘的过程，有手动触发和自动触发
手动触发有save 和bgsave 两命令
save 命令：
阻塞当前Redis，直到RDB 持久化过程完成为止，若内存实例比较大会造成长时间阻塞，
线上环境不建议用它
bgsave 命令：
redis 进程执行fork 操作创建子线程，由子线程完成持久化，阻塞时间很短（微秒级），
是save 的优化,在执行redis-cli shutdown 关闭redis 服务时，如果没有开启AOF 持久化，
自动执行bgsave,显然bgsave 是对save 的优化。
RDB 文件的操作
命令：
config set dir /data/redis/redis6379/data   //设置rdb 文件保存路径
备份：
bgsave //将dump.rdb 保存到/data/redis/redis6379/data  下
恢复：
将dump.rdb 放到redis 安装目录与redis.conf 同级目录，重启redis 即可
优点：
1，压缩后的二进制文文件适用于备份、全量复制，用于灾难恢复
2，加载RDB 恢复数据远快于AOF 方式
缺点：
1，无法做到实时持久化，每次都要创建子进程，频繁操作成本过高
2，保存后的二进制文件，存在老版本不兼容新版本rdb 文件的问题


2. RDB参数配置

save 900 1    # 900 秒（15 分钟）内至少1 个key 值改变（则进行数据库保存--持久化）
save 300 10       # 300 秒（5 分钟）内至少10 个key 值改变（则进行数据库保存--持久化）
save 60 10000   # 60 秒（1 分钟）内至少10000 个key 值改变（则进行数据库保存--持久化）

stop-writes-on-bgsave-error yes  # 后台存储错误停止写。
rdbcompression yes   # 在进行镜像备份时,是否进行压缩。yes：压缩，但是需要一些cpu 的消耗;no不压缩，需要更多的磁盘空间
rdbchecksum yes  #一个CRC64 的校验就被放在了文件末尾，当存储或者加载rbd 文件的时候会有一个10%左右的性能下降，为了达到性能的最大化，你可以关掉这个配置项
dbfilename dump.rdb  # 快照的文件名
dir /data/redis/redis6379/data

上面提出了RDB 持久化不能实时持久化数据，要是不允许数据丢失，则需要用AOF来持久化
如果不做持久化用replication 去保证可用性，另外最后可以通过应用从数据库同步最新数据，
因此注释掉所有持久化策略，添加一条带空字符串参数的save 指令也能移除之前所有配置的
save 指令,除非手动执行gsave数据才会持久化
save ""

总结1：
开启RDB 持久化，在满足save 条件、手动bg save、kill,pkill,正常关闭的时候数据都会被持久化，而异常关闭终止的时候数据会丢失,kill -9



3. AOF参数配置
针对RDB 不适合实时持久化，redis 提供了AOF 持久化方式来解决

流程说明：
1，所有的写入命令(set hset)会append 追加到aof_buf 缓冲区中
2，AOF 缓冲区向硬盘做sync 同步
3，随着AOF 文件越来越大，需定期对AOF 文件rewrite 重写，进行压缩
4，当redis 服务重启，可load 加载AOF 文件进行恢复
AOF 持久化流程：
命令写入(append),文件同步(sync),文件重写(rewrite),重启加载(load)


如何从AOF 恢复？
1. 设置appendonly yes；
2. 将appendonly.aof 放到dir 参数指定的目录；
3. 启动Redis，Redis 会自动加载appendonly.aof 文件。
redis 重启时恢复加载AOF 与RDB 顺序及流程：
1，当AOF 和RDB 文件同时存在时，优先加载AOF
2，若关闭了AOF，加载RDB 文件
3，加载AOF/RDB 成功，redis 重启成功
4，AOF/RDB 存在错误，redis 启动失败并打印错误信息
总结3：
redis 重启载入数据的时候，读取aof 的文件要先于rdb 文件，所以尽量一开始开启aof选项，不要在中途开启。
在开启aof之前需要执行BGREWRITEAOF命令保存.rdb文件然后数据合并到.aof中,重新启动优先加载.aof


AOF 配置详解
appendonly yes #启用aof 持久化方式,默认不开启，为no,默认文件名appendfilename "appendonly.aof"
appendfsync always  #每收到写命令就立即强制写入磁盘，最慢的，但是保证完全的持久化，不推荐使用
appendfsync everysec #每秒强制写入磁盘一次，性能和持久化方面做了折中，推荐
appendfsync no  #完全依赖os，性能最好,持久化没保证（操作系统自身的同步）
no-appendfsync-on-rewrite yes #正在导出rdb 快照的过程中,要不要停止同步aof
auto-aof-rewrite-percentage 100 #aof文件大小比起上次重写时的大小,增长率100%时,重写
auto-aof-rewrite-min-size 64mb #aof 文件至少超过64M 时,重写



```

- redis数据备份和恢复
```
备份很简单,只需要把RDB,AOF的文件复制备份起来就可以,相同的办法可以任意恢复,不同的版本可能不兼容

```
[更多参数参考](https://www.cnblogs.com/ysocean/p/9114267.html)
