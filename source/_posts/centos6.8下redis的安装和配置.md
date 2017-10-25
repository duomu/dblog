---
title: centos6.8下redis的安装和配置
date: 2017-10-25 11:30:46
tags: redis
categories: redis
---

# 下载、安装

在[redis官网](https://redis.io/)可以获取到最新版本的redis

进入/usr/local/目录，执行如下命令

    wget http://download.redis.io/releases/redis-4.0.2.tar.gz
	tar xzf redis-4.0.2.tar.gz
	cd redis-4.0.2
	make
	
执行make构建redis时报如下错误，这是因为没有安装gcc,执行如下命令即可解决

    错误：	make[3]: gcc：命令未找到
	解决：	yum install -y wget gcc make tcl //安装gcc

继续执行make又报错，这是因为构建redis的默认内存分配器是jemalloc，如果系统中没有jemalloc，就会报错，可以在构建时将内存分配器设置成libc

    错误：	zmalloc.h:50:31: 错误：jemalloc/jemalloc.h：没有那个文件或目录
	解决：	make MALLOC=libc  //构建时指定内存分配器

# 配置、启动

## 启动redis服务
### 使用默认配置文件启动redis服务
执行完make命令后，redis就安装完毕了，在安装目录/usr/local/redis-4.0.2目录下执行下面的命令，如果成功启动redis服务，说明redis安装成功

    redis-server

### 指定配置文件启动redis服务
创建如下目录，存放配置文件、日志文件、进程文件、工作文件（如数据备份）

    mkdir /etc/redis
	mkdir /var/redis
	mkdir /var/redis/log
	mkdir /var/redis/run
	mkdir /var/redis/6379

复制一份配置文件到/etc/redis目录

    cp redis.conf /etc/redis/6379.conf
	
修改配置文件6379.conf

	daemonize yes	//将redis服务设成守护进程
	requirepass 123456	//设置认证密码
	bind 0.0.0.0	//设置监听所有ip，默认为bind 127.0.0.1，只监听本机ip，其他主机无法访问此redis，因为我要远程操作redis，所以暂时改成0.0.0.0
	protected-mode no	//关闭保护模式，默认启用保护模式，同样要想远程访问redis，必须设成no
	pidfile /var/redis/run/redis_6379.pid
	logfile /var/redis/log/redis_6379.log
	dir /var/redis/6379

使用6379.conf启动redis服务

    redis-server /etc/redis/6379.conf

### 使用redis启动脚本启动
复制redis启动脚本到/etc/init.d目录

    cp redis_init_script /etc/init.d/redis

修改启动脚本

由于启动脚本的一些变量使用的是默认值，而我刚刚修改了redis的默认配置，所以启动脚本也要进行一致性修改，否则使用启动脚本执行命令时会报错。

我这里需要修改脚本里的pidfile和关闭服务时的命令，pidfile的目录要和6379.conf里的一致，关闭服务命令要设置认证密码，由于每个人使用可能不同，具体需要修改哪些地方还要具体分析。

    PIDFILE=/var/redis/run/redis_${REDISPORT}.pid
	$CLIEXEC -a "123456" -p $REDISPORT shutdown

启动redis服务

    service redis start

### 设置开机自启动

在以上基础上修改启动脚本，在启动脚本的开头增加下行注释（不加的话执行chkconfig 命令会报"redis 服务不支持 chkconfig"）

    # chkconfig:   2345 90 10

修改后如下：

    # !/bin/sh
	# chkconfig:   2345 90 10
	# Simple Redis init.d script conceived to work on Linux systems
	# as it does use of the /proc filesystem.

执行命令，设置开机自启

    chkconfig redis on

关闭开机自启

    chkconfig redis off

## 关闭redis服务

### 直接杀死redis服务进程

    #查看运行的redis服务，得到redis服务的进程号，假设是1000
	ps -ef|grep redis
	#杀死redis进程
	kill -9	1000

### 使用redis客户端关闭

    redis-cli –h localhost –p 6379 –a 123456 shutdown

### 使用启动脚本关闭
	
    service redis stop

## 启动redis客户端
在安装目录下执行如下命令，启动redis客户端，如果不设ip，默认连接本机的redis的服务，如果不设port默认连接端口为6379的redis服务，
redis默认没有配置认证密码，如果配置了认证密码requirepass，就需要使用参数-a进行认证。

	redis-cli -h localhost -p 6379 -a 123456    

# 测试
## 本机测试
启动redis客户端，输入如下命令，可以成功存取数据

	[root@128-157 redis-4.0.2]# redis-cli -a 123456
	127.0.0.1:6379> set name duomu
	OK
	127.0.0.1:6379> get name
	"duomu"
	127.0.0.1:6379> 

## 远程测试
在远程主机上编写如下java程序访问redis，可以成功执行（注意要引入相关的jar包）

	public class RedisMainTest {
	    private static final Logger logger = LoggerFactory.getLogger(RedisMainTest.class);
	
	    public static void main(String[] args) {
	        Jedis jedis = new Jedis("192.x.x.x", 6379);
	        jedis.auth("123456");
	        jedis.set("name", "duomu");
	        String name = jedis.get("name");
	        logger.info("name：" + name);
	    }
	}

注意：远程访问redis服务，redis主机需要对外开放6379端口号或者直接关闭防火墙，否则会连接失败

开放6379端口号

	/sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT #开启6379端口
	/etc/rc.d/init.d/iptables save #保存配置
	/etc/rc.d/init.d/iptables restart #重启服务

关闭防火墙

	service iptables stop 	#关闭防火墙
	service iptables status	#查看防火墙状态
	service iptables start	#启动防火墙

# 参考资料


1. http://www.cnblogs.com/haoxinyue/p/3620648.html
2. https://www.tuicool.com/articles/aQbQ3u
3. http://www.cnblogs.com/wangqingyi/articles/5575419.html
4. http://www.cnblogs.com/jeffen/p/6068745.html
5. http://linux.it.net.cn/CentOS/fast/2014/0622/1602.html








