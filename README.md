###### 注: 系统是centos7,转载自
###### https://blog.csdn.net/qq_36514588/article/details/83856795
##### https://www.cnblogs.com/small-bud/p/9160679.html
##### https://blog.csdn.net/xiyujianxia/article/details/80679187

#### 一.redis的安装
#####   编译安装redis
| 操作 | 说明 |
| :------ | :------ |
| tar -zxvf redis-5.0.0.tar.gz | 官网下载并解压 http://www.redis.cn/download.html |
| yum install gcc -y | 安装gcc编译器 |
| make && make install | 进入redis-5.0.0编译安装 |
    
#### 二.创建节点
#####     1.建6个节点测试,将配置文件拷贝到各个文件夹下
    # pwd
    /root
    # mkdir cluster
    # cd cluster
    # mkdir 7001 7002 7003 7004 7005 7006
    # cd ..
    # cp redis-5.0.0/redis.conf cluster/7001/
    # cp redis-5.0.0/redis.conf cluster/7002/
    # cp redis-5.0.0/redis.conf cluster/7003/
    # cp redis-5.0.0/redis.conf cluster/7004/
    # cp redis-5.0.0/redis.conf cluster/7005/
    # cp redis-5.0.0/redis.conf cluster/7006/
    
#####     2.分别修改配置文件(启动时使用)并启用各redis

|修改记录 | 说明|
|:----- | :------ |
|dir /root/cluster/7001/|生成的数据库文件位置|
|daemonize    yes|redis后台运行
|pidfile  /var/run/redis_7001.pid | pidfile文件对应7001,7002...7006
|port  7001|端口7001,7002...7006
|cluster-enabled  yes|开启集群  把注释#去掉
|cluster-config-file  nodes_7001.conf|集群的配置  配置文件首次启动自动生成 7001,7002...7006
|cluster-node-timeout  5000|请求超时
|appendonly  yes|aof日志开启  有需要就开启，它会每次写操作都记录一条日志

#####     3.启动redis
    redis-server cluster/7001/redis.conf
    redis-server cluster/7002/redis.conf
    redis-server cluster/7003/redis.conf
    redis-server cluster/7004/redis.conf
    redis-server cluster/7005/redis.conf
    redis-server cluster/7006/redis.conf
    
    ps -ef|grep redis 查看启动成功与否


#### 三、创建集群
#####     redis5.0开始不再使用ruby搭建集群,使用 redis-cli --cluster create
#####     cluster-replicas 1: 代表主从比是1;这里是3主3从
    
    redis-cli --cluster create --cluster-replicas 1 192.168.137.131:7001 192.168.137.131:7002 192.168.137.131:7003 192.168.137.131:7004 192.168.137.131:7005 192.168.137.131:7006 

#####     查看集群
    cluster info    查看集群信息
    cluster nodes   查看节点
    
     
#### 四、python3操作集群库
##### pip install rediscluster
    
    from rediscluster import StrictRedisCluster
    startup_nodes = [
        {"host":"192.168.137.131", "port":7001},
        {"host":"192.168.137.131", "port":7002},
        {"host":"192.168.137.131", "port":7003},
        {"host":"192.168.137.131", "port":7004},
        {"host":"192.168.137.131", "port":7005},
        {"host":"192.168.137.131", "port":7006},
    ]
    rc = StrictRedisCluster(startup_nodes=startup_nodes, decode_responses=True)
    rc.set('name','zhangsan')
    rc.set('age',22)
    print ("name is: ", rc.get('name'))
    print ("age  is: ", rc.get('age'))
    
##### 注:python连接redis数据,建议使用 redis.StrictRedis,StrictRedisCluster
    redis.Redis 好像有点问题
    redis.StrictRedis
