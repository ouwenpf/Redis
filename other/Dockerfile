FROM source_centos7:latest

ENV basedir redis
ENV datadir redis6379

	
RUN if [ ! -d /usr/local/redis ];then ln -s /application/$basedir   /usr/local/redis; fi

RUN if ! id redis &> /dev/null ;then useradd redis ; fi
RUN if [ ! -d /data/redis/$datadir ];then mkdir -p /data/redis/$datadir/{data,logs,conf}  &&  chown -R redis.redis /data/$basedir/$datadir ;fi

RUN sed   -ri 's#local/bin#local/bin:/application/redis/bin#g'     /home/redis/.bash_profile



COPY redis.conf /data/redis/$datadir/conf
#COPY sysctl.conf /etc/ 
COPY oneinstack.sh /etc/profile.d/
#ADD key.tar.gz /root/.ssh 
RUN chmod +x  /etc/rc.d/rc.local  \
&& echo "sed  -ri  '/sed/,/insecure/d'  /etc/rc.local" >> /etc/rc.d/rc.local 
WORKDIR /data/mysql/$datadir
EXPOSE 6379 16379













