# Redis专业环境安装

## MySQL下载

Redis下载唯一合法途径就是去官方下载，其它网上来源均不安全，作为dba人员请务必注意，推荐下载二进制包  
[下载地址](https://download.redis.io/releases/)  
[相关资料](https://www.cnblogs.com/xibuhaohao/p/12580331.html)


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

 ocker run -itd  -v /data/share:/data/share -p50000:50000  -v /data/app:/application -v  /etc/resolv.conf:/etc/resolv.conf  -v /etc/hosts:/etc/hosts -v /sys/fs/cgroup:/sys/fs/cgroup  --net myvpc  --ip 10.10.10.10  --cap-add=SYS_PTRACE --cap-add=NET_ADMIN  --privileged=true --name gcc_py   -h gcc_py  gcc_entos7:latest  /usr/sbin/init


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
cp  ../redis.conf  /data/redis/redis6379/conf

配置文件:
echo 'daemonize yes' >>  /data/redis/redis6379/conf/redis.conf
echo  'logfile "/data/redis/redis6379/log/redis.log"'  >> /data/redis/redis6379/conf/redis.conf
echo 'requirepass 123456'  >> /data/redis/redis6379/conf/redis.conf


设置密码:
CONFIG GET requirepass
CONFIG SET  requirepass 123456


```



