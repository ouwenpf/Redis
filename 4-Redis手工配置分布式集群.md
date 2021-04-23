
# Redis专业环境安装

## Redis下载

Redis下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[Redis下载地址](https://download.redis.io/releases/)   
[Ruby下载地址](https://www.ruby-lang.org/zh_cn/downloads/)


- 机器故障 
```
redis集群如果发生故障将会自动切换,主库宕机,从库自动接管
```


- 性能测试

```
redis-benchmark  -h 10.0.10.11 -c 200 -r 1000000 -n 2000000 -t get,set,lpush,lpop -P 16 -q 

-- 其它测试案例

1. redis-benchmark  -h 10.0.10.11 -c 200 -n 100000
100个并发,100000请求

2. redis-benchmark  -h 10.0.10.11  -q -d 100
测试存储大小为100为字节的数据包的性能

3. redis-benchmark  -h 10.0.10.11 -t set,lpush -n 100000 -q
只测试某些操作的新能

4. redis-benchmark -h 10.0.10.11 -n 100000 -q scrip load "redis.call('set','foo','bar')"
只测试某些数值存储的性能

```


- 手工配置redis集群
```
1. 服务器信息
10.0.10.11:6379
10.0.10.12:6379
10.0.10.13:6379
10.0.10.14:6379
10.0.10.15:6379
10.0.10.16:6379
10.0.10.17:6379
10.0.10.18:6379
10.0.10.19:6379
10.0.10.20:6379

2. 加入集群节点
cluster meet 10.0.10.11 6379
cluster meet 10.0.10.12 6379
cluster meet 10.0.10.13 6379
cluster meet 10.0.10.14 6379
cluster meet 10.0.10.15 6379
cluster meet 10.0.10.16 6379
cluster meet 10.0.10.17 6379
cluster meet 10.0.10.18 6379
cluster meet 10.0.10.19 6379
cluster meet 10.0.10.20 6379


```