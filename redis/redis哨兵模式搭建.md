# redis哨兵模式搭建

## 一、redis介绍

redis一共有三种模式

1、主从模式（一主两从）

2、哨兵模式（一主两从、三哨兵）

3、集群模式

本文实战搭建一主两从三哨兵，通过使用[哨兵模式](https://so.csdn.net/so/search?q=哨兵模式&spm=1001.2101.3001.7020)，可以有效避免某台服务器的 Redis 挂掉出现的不可用问题，保障系统的高可用。

## 二、准备环境

|     IP地址      |          角色          |
| :-------------: | :--------------------: |
| 192.168.101.103 | redis-master，sentinel |
| 192.168.101.104 | redis-slave1，sentinel |
| 192.168.101.105 | redis-slave2，sentinel |

## 三、安装Redis

### 3.1、安装 C/C++ 环境，编译 Redis 安装包使用

```
# 三台服务器均需要安装
yum -y install gcc gcc-c++ make
```

### 3.2、下载 Redis 安装包

```
# 切换软件安装目录
cd /usr/local/

# 新建 redis 安装目录
mkdir redis

# 切换到 redis 安装目录
cd redis

# 下载 redis 安装包
wget http://download.redis.io/releases/redis-6.2.5.tar.gz

# 解压 redis 安装包
tar -zxvf redis-6.2.5.tar.gz
```

### 3.3、编译 redis

```
# 进入解压后的 redis 目录
cd redis-6.2.5/

# 编译
make

# 进入编译好的目录(编译成功后 src 目录下会出现编译后的 redis 服务程序 redis-server)
cd src
```

![image-20241016223759317](E:\GitHup\yunwei\redis\images\image-20241016223759317.png)

## 四、配置 Redis

### 4.1、配置主节点 & 哨兵

```
# 进入 Redis 的主目录
cd /usr/local/redis/redis-6.2.5

# 创建工作目录 tmp
mkdir tmp

# 创建日志目录 log
mkdir log

# 编辑 Redis 配置
vim redis.conf

# 编辑哨兵配置
vim sentinel.conf
```

redis.conf 配置信息如下(这里仅列举了需要修改的地方，其他地方保持默认即可)

```
# 表示redis允许所有地址连接。默认127.0.0.1，仅允许本地连接。
bind 0.0.0.0

# 允许redis后台运行
daemonize yes

# 设置redis日志存放路径
logfile "/usr/local/redis/redis-6.2.5/log/redis_6379.log"

# 设置为no，允许外部网络访问
protected-mode no

# 修改redis监听端口(可以自定义)
port 6379

# pid存放目录
pidfile /var/run/redis_6379.pid

# 工作目录，需要创建好目录,可自定义
dir /usr/local/redis/redis-6.2.5/tmp

# 设置redis密码
requirepass 123456

# 主从同步master的密码
masterauth 123456
```

sentinel.conf 哨兵的配置如下(这里仅列举了需要修改的地方，其他地方保持默认即可)

```
# 修改Sentinel监听端口
port 26380

# 允许Sentinel后台运行
daemonize yes

# 设置Sentinel日志存放路径
logfile "/usr/local/redis/redis-6.2.5/log/redis_6379_sentinel.log"

# 工作目录，需要创建好目录,可自定义
dir /usr/local/redis/redis-6.2.5/tmp

# Sentinel 监听 redis 主节点, mymaster：master名称可自定义,192.168.101.103 6379。 ：redis主节点IP和端口,2 ：表示多少个Sentinel认为redis主节点失效时，才算真正失效.
sentinel monitor mymaster 192.168.101.103 6379 2

# 配置失效时间，master会被这个sentinel主观地认为是不可用的，单位毫秒   
sentinel down-after-milliseconds mymaster 10000

# 若sentinel在该配置值内未能完成master/slave自动切换，则认为本次failover失败。
sentinel failover-timeout mymaster 60000

# 在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步。
sentinel parallel-syncs mymaster 2

# 设置连接master和slave时的密码，注意的是sentinel不能分别为master和slave设置不同的密码，因此master和slave的密码应该设置相同
sentinel auth-pass mymaster 123456
```

### 4.2、配置从节点 & 哨兵

> 这里只演示配置一个从节点，另一个从节点和这个从节点配置一样。
>
> 首先需要在从节点的服务器上安装 redis，安装 redis 的方法和在主节点服务器上安装 redis 方法一样。

```
# 进入 Redis 的主目录
cd /usr/local/redis/redis-6.2.5

# 创建工作目录 tmp
mkdir tmp

# 创建日志目录 log
mkdir log

# 编辑 Redis 配置
vim redis.conf

# 编辑哨兵配置
vim sentinel.conf
```

redis.conf 配置信息如下(这里比主节点只多了一行，replicaof的配置，用于追随主节点)

```
# 表示redis允许所有地址连接。默认127.0.0.1，仅允许本地连接。
bind 0.0.0.0

# 允许redis后台运行
daemonize yes

# 设置redis日志存放路径
logfile "/usr/local/redis/redis-6.2.5/log/redis_6379.log"

# 设置为no，允许外部网络访问
protected-mode no

# 修改redis监听端口(可以自定义)
port 6379

# pid存放目录
pidfile /var/run/redis_6379.pid

# 工作目录，需要创建好目录,可自定义
dir /usr/local/redis/redis-6.2.5/tmp

# 设置redis密码
requirepass 123456

# 主从同步master的密码
masterauth 123456

# 多了这一行，用于追随某个节点的redis，被追随的节点为主节点，追随的为从节点，Redis5.0前版本可使用slaveof
replicaof 10.211.55.4 6379
```

sentinel.conf  哨兵的配置如下(和主节点的配置一样)

```
# 修改Sentinel监听端口
port 26380

# 允许Sentinel后台运行
daemonize yes

# 设置Sentinel日志存放路径
logfile "/usr/local/redis/redis-6.2.5/log/redis_6379_sentinel.log"

# 工作目录，需要创建好目录,可自定义
dir /usr/local/redis/redis-6.2.5/tmp

# Sentinel 监听 redis 主节点, mymaster：master名称可自定义,127.0.0.1 6379 ：redis主节点IP和端口,2 ：表示多少个Sentinel认为redis主节点失效时，才算真正失效
sentinel monitor mymaster 127.0.0.1 6379 2

# 配置失效时间，master会被这个sentinel主观地认为是不可用的，单位毫秒   
sentinel down-after-milliseconds mymaster 10000

# 若sentinel在该配置值内未能完成master/slave自动切换，则认为本次failover失败。
sentinel failover-timeout mymaster 60000

# 在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步。
sentinel parallel-syncs mymaster 2

# 设置连接master和slave时的密码，注意的是sentinel不能分别为master和slave设置不同的密码，因此master和slave的密码应该设置相同
sentinel auth-pass mymaster 123456
```

## 五、启动 Redis 集群

#### 5.1、启动 redis 集群，主 -> 从

启动 redis 命令如下(从节点类似)，下面这条命令执行后没有出错，一般就是启动成功了

```
/usr/local/redis/redis-6.2.5/src/redis-server /usr/local/redis/redis-6.2.5/redis.conf
```

查看启动是否成功,如果失败可以看下log文件夹下的日志

```
ps -ef|grep redis
```

![image-20241016224530440](E:\GitHup\yunwei\redis\images\image-20241016224530440.png)

查看集群信息

```
#切换到主库目录下
/usr/local/redis/redis-6.2.5/src

#连接redis
./redis-cli 

#验证密码
auth 123456

#查看集群
info replication
```

master 节点上可以看到，此节点为master 节点。有两个从节点，以及从节点的ip

![image-20241016225129723](E:\GitHup\yunwei\redis\images\image-20241016225129723.png)

从节点查看集群信息，可以看到追随的主节点为192.168.101.103

![image-20241016233007685](E:\GitHup\yunwei\redis\images\image-20241016233007685.png)



### 5.2、启动哨兵

启动 redis 命令如下(从节点类似)，下面这条命令执行后没有出错，一般就是启动成功了

```
/usr/local/redis/redis-6.2.5/src/redis-sentinel /usr/local/redis/redis-6.2.5/sentinel.conf
```

查看启动是否成功,如果失败可以看下log文件夹下的日志

```
ps -ef|grep redis
```

![image-20241016225317871](E:\GitHup\yunwei\redis\images\image-20241016225317871.png)

## 六、哨兵模式测试

从master 节点进入数据库

```
#切换到主库目录下
/usr/local/redis/redis-6.2.5/src

#连接redis
./redis-cli 

#验证密码
auth 123456

# 插入一条数据
set test_key "This is a test data"

# 获取key test_key 的值，要在所有节点都查看一下数据是否主从同步
get test_key
```

![image-20241016225648214](E:\GitHup\yunwei\redis\images\image-20241016225648214.png)

## 6.2、主节点宕机测试

先模拟一下挂掉 redis 主节点。

> 1. 使用 ps aux | grep redis 找到 redis 主节点对应的进程 id
> 2. 使用 kill -9 xxx 杀掉 redis 主节点 id
> 3. 查看从库数据库的哨兵集群状态

此时 192.168.101.105 机器上的节点成为了master节点

![image-20241016230911287](E:\GitHup\yunwei\redis\images\image-20241016230911287.png)

另一个从节点追随的主机点变成了192.168.101.105

![image-20241016231005376](E:\GitHup\yunwei\redis\images\image-20241016231005376.png)