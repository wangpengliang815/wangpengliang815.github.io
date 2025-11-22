# 环境准备

| IP | 操作系统 |
| :---: | :---: |
| 192.168.252.191 | CentOS Linux release 7.9.2009 (Core) |
| 192.168.252.192 | CentOS Linux release 7.9.2009 (Core) |
| 192.168.252.193 | CentOS Linux release 7.9.2009 (Core) |
| 192.168.252.194 | CentOS Linux release 7.9.2009 (Core) |
| 192.168.252.195 | CentOS Linux release 7.9.2009 (Core) |
| 192.168.252.196 | CentOS Linux release 7.9.2009 (Core) |
<!-- more -->
# 整体搭建步骤

整体搭建步骤主要分为以下几步：

1. 下载 Redis 镜像（可以省略，创建容器时如果本地镜像不存在就会去远程拉取）
2. 更改 Redis 配置文件
3. 创建并运行 Redis 容器
4. 使用集群命令创建 Redis Cluster 集群

# Step1：下载镜像

> 省略

# Step2：更改配置文件

获取 `redis.conf`  修改默认配置		

- `bind 127.0.0.1` ：  修改为 `0.0.0.0` 或指定`IP`（`127.0.0.1` 限制了 Redis 只能本地访问）
- `protected-mode no`： 改为`yes` ：开启保护模式限制为本地访问
- `daemonize no`  ：改为 `yes`：是否以守护进程方式启动可后台运行
- `dir  ./` ：Redis 数据存放文件夹（可选）
- `appendonly yes` ：Redis 持久化（可选）
- `cluster-enabled `：设置为集群节点
# Step3：创建并启动容器
```shell
docker run -d --net host --name redis  -v /usr/confs/redis.conf:/etc/redis/redis.conf -v /usr/redisdata:/data redis redis-server /etc/redis/redis.conf --appendonly yes
```
命令解释：

- `--name redis` ：指定容器名称
- `-v` ：挂载目录，前表示主机部分/后表示容器部分
- `-d` ：表示后台启动
- `redis-server /etc/redis/redis.conf` ：以配置文件启动 `Redis` ，加载容器内的 `conf` 文件，最终找到的是 `/usr/confs/redis.conf` 
- `appendonly yes`：开启 `Redis`  持久化
- `--requirepass`："如果有密码" 

在每台虚机上重复执行 `Step2` 和 `Step3` 启动6个redis节点。

> 注意：如果在执行创建集群命令时卡在" Waiting for the cluster to join ......"，启动容器时添加 `--net host`

# Step4：创建Redis集群

格式：

```bash
docker exec -it redis redis-cli --cluster create 192.168.252.191:6379 192.168.252.192:6379 192.168.252.193:6379 192.168.252.194:6379 192.168.252.195:6379 192.168.252.196:6379 --cluster-replicas 1
```

查看结果
```shell
[root@centos-01 ~]# docker exec -it redis /bin/bash
root@centos-01:/data# redis-cli 
127.0.0.1:6379> CLUSTER NODES
ea3bd63f54103bf7873bfa1b8bea4b7c1190d489 192.168.252.191:6379@16379 myself,master - 0 1631258399000 1 connected 0-5460
9d512a4a1bb7b2a92f09005298065c974d3cd545 192.168.252.194:6379@16379 master - 0 1631258399500 9 connected 10923-16383
752d6f9633db932d8347f69d3a4eb52f640d2bdb 192.168.252.192:6379@16379 slave 05debcade3fbaae47c561fefd6c982641a74185c 0 1631258398000 10 connected
7020b7697b4c0c3f5237f8d56a8ee24b41e1faa2 192.168.252.193:6379@16379 slave 9d512a4a1bb7b2a92f09005298065c974d3cd545 0 1631258400520 9 connected
05debcade3fbaae47c561fefd6c982641a74185c 192.168.252.196:6379@16379 master - 0 1631258398478 10 connected 5461-10922
60f3a530a42c1f984d82f93fbb9a4017f40fb498 192.168.252.195:6379@16379 slave ea3bd63f54103bf7873bfa1b8bea4b7c1190d489 0 1631258397457 1 connected

127.0.0.1:6379> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:10
cluster_my_epoch:1
cluster_stats_messages_ping_sent:6497
cluster_stats_messages_pong_sent:6333
cluster_stats_messages_fail_sent:32
cluster_stats_messages_auth-ack_sent:4
cluster_stats_messages_sent:12866
cluster_stats_messages_ping_received:6328
cluster_stats_messages_pong_received:6488
cluster_stats_messages_meet_received:5
cluster_stats_messages_fail_received:6
cluster_stats_messages_auth-req_received:4
cluster_stats_mes
```


