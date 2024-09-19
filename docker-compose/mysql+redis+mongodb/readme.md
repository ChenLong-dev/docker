#  mysql+redis+mongodb



学习地址：https://blog.csdn.net/qq_45463500/article/details/132311999

## 创建持久化目录

```
mkdir -pv {mysql/{data,logs,conf},redis/{conf,data,logs},mongodb/data}
```

### 编辑docker-compose.yml文件

```
version: '3'
services:
  mysql:
    # 基础镜像
    image: mysql:5.7
    #容器名称
    container_name: mysql
    # 开机启动，失败也会一直重启
    restart: always
    # 端口
    ports:
      - "23306:3306"
    environment:
    # 密码设置
      - MYSQL_ROOT_PASSWORD=xDDeE53JNXK@V#
      - MYSQL_LOG_CONSOLE=true
    # 持久化数据挂载
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d
      - ./mysql/logs:/logs

  redis:
    image: redis:5.0
    container_name: redis
    privileged: true
    environment:
      TZ: Asia/Shanghai
    ports:
      - 26379:6379
    command: ["redis-server","/etc/redis/redis.conf"]
    restart: always
    volumes:
      - ./redis/conf:/etc/redis/
      - ./redis/data:/data
      - ./redis/logs:/logs

  mongodb:
    image: mongo:4.2.2
    container_name: mongodb
    environment:
      TZ: Asia/Shanghai
    ports:
      - 27017:27017
    restart: always
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=xDDeE53JNXK@V#
    volumes:
      - ./mongodb/data:/data/db
```

## 启动命令

```
docker-compose -f mysql_redis_mongo.yml up -d

docker-compose -f mysql_redis_mongo.yml restart redis
```

## 镜像可提前下载

```
docker pull mysql:5.7
docker pull redis:5.0
docker pull mongo:4.2.2
```

### 数据库无法访问

```
# 4.2.1
# 查看是否开启本机防火墙
# centos7
systemctl status firewalld
# 关闭防火墙
systemctl stop firewalld
# 如果不能停止防火墙就加入白名单
# 临时添加
firewalld-cmd --add-port=tcp/13306
firewalld-cmd --add-port=tcp/16379
firewalld-cmd --add-port=tcp/27017
# 永久添加
firewall-cmd --add-port=13306/tcp --permanent
firewall-cmd --add-port=16379/tcp --permanent
firewall-cmd --add-port=27017/tcp --permanent
#重新载入配
firewall-cmd --reload
# 4.2.2
# 根据日志进行排查
docker logs -f mysql
```

## 常用命令

### 删除

```
# 单独删除
docker rm -f mysql
# 全部删除
docker-compose down
```

### 启动

```
# 单独启动
docker-compose up -d mysql
# 全部启动
docker-compose up -d
```

### 查看

```
# 查看容器
docker ps
# 实时查看
docker logs -f mysql
# 实时查看后500行
docker logs -f --tail=500 mysql
```

## redis 问题

### redis配置

将目录下的redis.conf拷贝至./redis/conf目录下

### 目录权限问题

给data和logs赋予最高权限 

```
chmod -R 777 ./redis/data*
chmod -R 777 ./redis/logs*
```

### 密码修改问题

密码（requirepass）自己改一下：(在redis.conf的508行)

```
508  requirepass xDDeE53JNXK@V*#*
```

### 进程老是启动不起来

```
CONTAINER ID   IMAGE                        COMMAND                   CREATED         STATUS                         PORTS                                                    NAMES
e25bd0a1e0ff   redis:5.0                    "docker-entrypoint.s…"   8 seconds ago   Restarting (0) 2 seconds ago                                                            redis
```

**原因：**

redis.conf 中设置了 daemonize yes
当daemonize 设置了yes，表示redis在后台运行，当执行docker-compose执行启动redis进程时，docker发现自己无事可做，容器自动结束，所以导致redis启动失败。

**解决方案：**

将 redis.conf 中的 `daemonize yes` 注释/删掉。

然后重新启动。 `docker-compose up -d`

参考文章：

`https://blog.csdn.net/a1368783069/article/details/132241808`

## mongo问题

### 在yml文件中重置密码不生效

**原因：** 

mysql容器绑定的卷或者挂载的本地目录是没有变的，本地目录里面配置的密码一直是原来的密码。
**解决：**

删除绑定的本地目录，重新创建容器