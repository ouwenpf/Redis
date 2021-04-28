FROM source_centos7:latest

ENV basedir redis
ENV datadir redis6379

	
RUN if [ ! -d /usr/local/redis ];then ln -s /application/$basedir   /usr/local/redis; fi

RUN if ! id redis &> /dev/null ;then useradd redis ; fi
RUN if [ ! -d /data/redis/$datadir ];then mkdir -p /data/redis/$datadir/{data,log,conf}  &&  chown -R redis.redis /data/$basedir/$datadir ;fi

RUN sed   -ri 's#local/bin#local/bin:/application/redis/bin#g'     /home/redis/.bash_profile



COPY redis.conf /data/redis/$datadir/conf
#COPY sysctl.conf /etc/ 
COPY oneinstack.sh /etc/profile.d/
#ADD key.tar.gz /root/.ssh 
RUN chmod +x  /etc/rc.d/rc.local  \
&& echo "sed  -ri  '/sed/,/insecure/d'  /etc/rc.local" >> /etc/rc.d/rc.local 
WORKDIR /data/mysql/$datadir
EXPOSE 6379 16379






## 创建Dockerfile文件
# 宿主机上创建/data/app/mysql-8.0.24-linux-glibc2.12-x86_64        ln -s  ./data/app/数据库二进制文件  ./mysql-8
# 其它相关软件亦是如此

#- 构建镜像

#docker build -t mysql8-19:v1 -f Dockfile[默认此文件名称] [上下文:dockerfile文件中引用的文件]
#如：docker build -t mysql8-19:v1  .






