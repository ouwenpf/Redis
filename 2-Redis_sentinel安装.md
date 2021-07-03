# Redis_sentinel专业环境安装

## Redis下载

Redis下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[下载地址](https://download.redis.io/releases/)



## 安装前准备

- 关闭numa  
```
1. 关闭numa：bios设置--memory setting--node interleaving设置enabled
numactl --hardware 查询numa是否开启
2. numactl --interleave=all mysqld --defaults-file=/data/mysql/mysql3306/my.cnf & 系统运行后不能进行BIOS操作可以采取此办法启动mysql
   在mysql.server脚本中262行位置加上numactl --interleave=all
3. 修改 /etc/grub.conf 配置文件，在 kernel 那行增加一个配置后重启生效
kernel /vmlinuz-2.6.32-754.17.1.el6.x86_64 ro root=UUID=8ea2724c-08d3-4a47-97b3-65c38f56dc2a rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM  elevator=deadline numa=off  rhgb quiet
 

```



- 限制设置/etc/security/limits.conf && 网络优化

```
echo "*                -       nofile          65535" >>/etc/security/limits.conf
echo "*                -       nproc          65535" >>/etc/security/limits.conf
#echo 'DefaultLimitNOFILE=65535' >>/etc/systemd/system.conf
#echo 'DefaultLimitNPROC=65535' >>/etc/systemd/system.conf

sysctl -w vm.overcommit_memory=1  
内存分配策略
0 表示内核将检查是否有足够的可用内存供进程使用;如果有足够的可用内存,内存允许申请,否则内存申请失败,并把错误返回给进程
1 表示内核允许分配所有的物理内存,而不管当前的内存状态
2 表示内核允许分配超过所有的物理内存和交换空间总和的内存

echo never > /sys/kernel/mm/transparent_hugepage/enabled 
解决:redis做rdb时候会有部分请求超时的case

sysctl -w net.core.somaxconn=2048

sysctl -w vm.overcommit_memory=1
echo never > /sys/kernel/mm/transparent_hugepage/enabled 
sysctl -w net.core.somaxconn=2048

```

- 安装

```
docker run -itd   -v /application:/application -v   /data/tools:/data/tools  -v /etc/resolv.conf:/etc/resolv.conf  --net mysqlnet --ip 10.0.10.11  --cap-add=SYS_PTRACE --cap-add=NET_ADMIN  --privileged=true --name redis-1 -h redis-1 redis/centos7:v1  /usr/sbin/init

useradd redis
mkdir  -p /application/redis 
mkdir  -p  /data/redis/redis6379/{data,log,conf}
chown redis:redis  /application/redis/
chown -R  redis:redis  /data/redis/
chown redis:redis  /home/redis/
su - redis 
sed   -ri 's#local/bin#local/bin:/application/redis/bin#g'     .bash_profile


tar xf redis-6.2.1.tar.gz -C  /home/redis
cd  /home/redis
make 
cd src
make install PREFIX=/application/redis
cp  ../redis.conf  /data/redis/redis6379/conf




```


- 配置Redis主从

```
1.主节点配置

daemonize yes
logfile "/data/redis/redis6379/log/redis.log"
dir /data/redis/redis6379/data
requirepass 123456
masterauth 123456
timeout 300
#bind 127.0.0.1 -::1
bind 10.0.10.11 -::1	#当前从节点的地址
save ""   禁用持久化




2.从节点配置
daemonize yes
logfile "/data/redis/redis6379/log/redis.log"
dir /data/redis/redis6379/data
requirepass 123456
masterauth 123456
timeout 300
#replica-serve-stale-data yes #从节点只读6.0版本默认自动配置好了无需配置
#bind 127.0.0.1 -::1
bind 10.0.10.12 -::1	#当前从节点的地址
slaveof 10.0.10.11 6379 #主节点的地址

3. 查询主从相关信息
10.0.10.11:6379> info

# Replication
role:master
connected_slaves:2
slave0:ip=10.0.10.12,port=6379,state=online,offset=2282,lag=1
slave1:ip=10.0.10.13,port=6379,state=online,offset=2296,lag=0
master_failover_state:no-failover
master_replid:d002f4caa55f2370751f4c6daa8d6c4759bd03d9
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:2296
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:2296

```


- redis+哨兵(主从切换)配置
```
如果主节点挂了怎么办，所以redis提供了一个sentinel(哨兵),以此来实现主从切换

哨兵的作用
1. 监控:监控主从是否正常
2. 通知:出现问题时,可以通知相关人员
3. 故障迁移:自动主从切换
4. 统一配置管理:连接者询问sentinel取得主从地址

sentinel monitor mymaster 10.0.10.11 6379 2 #mymaster别名,10.0.10.11 6379主节点的地址和端口,2表示投票的数量(sentinel/2+1)
sentinel down-after-millseconds mymaster 10000 #10秒主节点还未响应认为主节点挂掉
sentinel parallel-syncs mymaster 1 #主从切换后,并行同步机器的数量
sentinel failover-timeout-mymaster 15000 #15主节点还没有活过来,进行切换
sentinel auth-pass mymaster 123456 #哨兵验证主节点需要密码
bind 10.0.10.11 哨兵所在的服务器的ip
----------------------------------------------------------------------


设置配置文件sentinel.conf
sentinel monitor mymaster 10.0.10.11 6379 2 
sentinel down-after-milliseconds mymaster 1000
sentinel parallel-syncs mymaster 1 
sentinel failover-timeout mymaster 180000 
sentinel auth-pass mymaster 123456 
bind 10.0.10.11
port 26379
daemonize yes
logfile "/data/redis/redis6379/logs/sentinel.log"
dir /data/redis/redis6379/data

启动查看
redis-sentinel  /data/redis/redis6379/conf/sentinel.conf 
redis-cli -h 10.0.10.11 -p 26379 sentinel masters 查看当前的master节点情况
redis-cli -h 10.0.10.11 -p 26379 info 查看master地址,几个slave,几个监控


```
