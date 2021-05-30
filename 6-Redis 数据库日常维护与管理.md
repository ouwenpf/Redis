
# Redis专业环境安装

## Redis下载

Redis下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[Redis下载地址](https://download.redis.io/releases/)   
[Ruby下载地址](https://www.ruby-lang.org/zh_cn/downloads/)



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
 mv     redis-6.2.1  /home/redis/
 chown redis:redis  /home/redis/
sed   -ri 's#local/bin#local/bin:/application/redis/bin#g'     .bash_profile


tar xf redis-6.2.1.tar.gz -C  /home/redis
cd  redis-6.2.1.tar.gz
make 
cd src
make install PREFIX=/application/redis
cp  ../redis.conf  /data/redis/conf

配置文件:
echo 'daemonize yes' >>  /data/redis/redis6379/conf/redis.conf
echo  'logfile "/data/redis/redis6379/logs/redis.log"'  >> /data/redis/redis6379/conf/redis.conf
echo 'requirepass 123456'  >> /data/redis/redis6379/conf/redis.conf


设置密码:
CONFIG GET requirepass
CONFIG SET  requirepass 123456


```


- 配置Redis-cluster多主多从

```


1. 服务器信息
主					从
10.0.10.11:6379 <-- 10.0.10.14:6379,10.0.10.15:6379 
10.0.10.12:6379 <-- 10.0.10.16:6379,10.0.10.17:6379
10.0.10.13:6379 <-- 10.0.10.18:6379,10.0.10.19:6379




yum  install zlib-devel curl-devel openssl-devel   -y
gem sources  --add source -a  https://gems.ruby-china.com/  --remove https://rubygems.org/

redis-cli  --cluster help 查询集群命令帮助
--[ERR] Node 10.0.10.11:6379 NOAUTH Authentication required配置文件中需要把密码认证注释掉
redis-cli --cluster create  --cluster-replicas 2 \
10.0.10.11:6379 10.0.10.14:6379 10.0.10.15:6379 \
10.0.10.12:6379 10.0.10.16:6379 10.0.10.17:6379 \
10.0.10.13:6379 10.0.10.18:6379 10.0.10.19:6379
-----提升信息如下
>>> Performing hash slots allocation on 9 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 10.0.10.16:6379 to 10.0.10.11:6379
Adding replica 10.0.10.17:6379 to 10.0.10.11:6379
Adding replica 10.0.10.13:6379 to 10.0.10.14:6379
Adding replica 10.0.10.18:6379 to 10.0.10.14:6379
Adding replica 10.0.10.19:6379 to 10.0.10.15:6379
Adding replica 10.0.10.12:6379 to 10.0.10.15:6379
M: 96a63077bfae3219ce39075cf4c58e75f106cbc3 10.0.10.11:6379
   slots:[0-5460] (5461 slots) master
M: 99eb54572d0de3fbf462da020f8b3e74d6a573cf 10.0.10.14:6379
   slots:[5461-10922] (5462 slots) master
M: 3cad4b882ba834b8ba6dd01d421537a86793cfd6 10.0.10.15:6379
   slots:[10923-16383] (5461 slots) master
S: 8a9ef7ddbb82af62c030e5289b1525462bb8e658 10.0.10.12:6379
   replicates 3cad4b882ba834b8ba6dd01d421537a86793cfd6
S: effb9029a3598251cb211365353681ff0abb16cf 10.0.10.16:6379
   replicates 96a63077bfae3219ce39075cf4c58e75f106cbc3
S: 9e306d162f3846aa4890f836f37daa1badd24e92 10.0.10.17:6379
   replicates 96a63077bfae3219ce39075cf4c58e75f106cbc3
S: 1a0f0ee43da8dc7abd2cb4ae19eae473422f7482 10.0.10.13:6379
   replicates 99eb54572d0de3fbf462da020f8b3e74d6a573cf
S: daa0dc4e362983a914f21b76914de8672541c76a 10.0.10.18:6379
   replicates 99eb54572d0de3fbf462da020f8b3e74d6a573cf
S: 21b48113fb7dd1a0944d57f2ae6d729e495d0e58 10.0.10.19:6379
   replicates 3cad4b882ba834b8ba6dd01d421537a86793cfd6
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 10.0.10.11:6379)
M: 96a63077bfae3219ce39075cf4c58e75f106cbc3 10.0.10.11:6379
   slots:[0-5460] (5461 slots) master
   2 additional replica(s)
M: 99eb54572d0de3fbf462da020f8b3e74d6a573cf 10.0.10.14:6379
   slots:[5461-10922] (5462 slots) master
   2 additional replica(s)
S: 9e306d162f3846aa4890f836f37daa1badd24e92 10.0.10.17:6379
   slots: (0 slots) slave
   replicates 96a63077bfae3219ce39075cf4c58e75f106cbc3
S: 21b48113fb7dd1a0944d57f2ae6d729e495d0e58 10.0.10.19:6379
   slots: (0 slots) slave
   replicates 3cad4b882ba834b8ba6dd01d421537a86793cfd6
S: 1a0f0ee43da8dc7abd2cb4ae19eae473422f7482 10.0.10.13:6379
   slots: (0 slots) slave
   replicates 99eb54572d0de3fbf462da020f8b3e74d6a573cf
S: daa0dc4e362983a914f21b76914de8672541c76a 10.0.10.18:6379
   slots: (0 slots) slave
   replicates 99eb54572d0de3fbf462da020f8b3e74d6a573cf
S: effb9029a3598251cb211365353681ff0abb16cf 10.0.10.16:6379
   slots: (0 slots) slave
   replicates 96a63077bfae3219ce39075cf4c58e75f106cbc3
M: 3cad4b882ba834b8ba6dd01d421537a86793cfd6 10.0.10.15:6379
   slots:[10923-16383] (5461 slots) master
   2 additional replica(s)
S: 8a9ef7ddbb82af62c030e5289b1525462bb8e658 10.0.10.12:6379
   slots: (0 slots) slave
   replicates 3cad4b882ba834b8ba6dd01d421537a86793cfd6
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
---------表示集群创建成功


redis-cli -c -h 10.0.10.11 
10.0.10.11:6379> CLUSTER NODES



```
