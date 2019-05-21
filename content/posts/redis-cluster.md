---
title: "redis cluster环境搭建"
date: 2019-05-21T20:31:34+08:00
draft: false
---
# redis cluster是什么？

[官网解答](https://redis.io/topics/cluster-spec)

# 如何搭建集群环境？

1.官网下载源码
{{< highlight bash >}}
    wget http://download.redis.io/releases/redis-5.0.5.tar.gz
{{< /highlight >}}

2.编译安装程序
{{< highlight bash >}}
    tar -xzf redis-5.0.5.tar.gz
    cd redis-5.0.5
    make
    make make install
{{< /highlight >}}

3.使用源码包utils里的install_server.sh快速配置redis
  server分布如下：
  {{< highlight bash >}}
    192.168.1.2 6379
    192.168.1.6 6379 6380 6381 6382
    192.168.1.7 6379 6380 6381
  {{< /highlight >}}
4.修改生成的配置文件，开启redis集群功能
{{< highlight bash >}}
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
{{< /highlight >}}
5.通过redis-server /etc/redis/***.conf启动server

6.通过ps -ef | grep redis-server 确认server启动

7.通过redis-cli 连接相应的节点，执行info命令，确保节点可访问，并开启了集群功能

8.通过redis-cli --cluster 工具初始化集群
{{< highlight bash >}}
    redis-cli --cluster create 192.168.1.2:6379 192.168.1.6:6379 192.168.1.6:6380 192.168.1.6:6381 192.168.1.6:6382 192.168.1.7:6379 192.168.1.7:6380 192.168.1.7:6381 --cluster-replicas 1
{{< /highlight >}}
工具会提示如下信息，主要是用来显示集群的Slot是如何分配，以及复制关系
{{< highlight bash >}}
    >>> Performing hash slots allocation on 8 nodes...
    Master[0] -> Slots 0 - 4095
    Master[1] -> Slots 4096 - 8191
    Master[2] -> Slots 8192 - 12287
    Master[3] -> Slots 12288 - 16383
    Adding replica 192.168.1.6:6381 to 192.168.1.2:6379
    Adding replica 192.168.1.7:6381 to 192.168.1.6:6379
    Adding replica 192.168.1.6:6382 to 192.168.1.7:6379
    Adding replica 192.168.1.7:6380 to 192.168.1.6:6380
    M: ddd75e058dd4d3d8c6202f6fb1b90f2f6e3936f7 192.168.1.2:6379
    slots:[0-4095] (4096 slots) master
    M: ed8a5c0f4df226dbc8a5d534f49decb88b5838cd 192.168.1.6:6379
    slots:[4096-8191] (4096 slots) master
    M: 755800e1330a60fde74de939fe7b97daf53abff6 192.168.1.6:6380
    slots:[12288-16383] (4096 slots) master
    S: 974ded7561fa8fe9263cc5ebf87da12edd53be45 192.168.1.6:6381
    replicates ddd75e058dd4d3d8c6202f6fb1b90f2f6e3936f7
    S: 8de83041cf2ec43930111e93c0255e703e31f609 192.168.1.6:6382
    replicates 274ab3f3de33ed601cb33776f5b434ee91cd2288
    M: 274ab3f3de33ed601cb33776f5b434ee91cd2288 192.168.1.7:6379
    slots:[8192-12287] (4096 slots) master
    S: 9bf2b9e65d6e91c309926668619a0b613b1ed65b 192.168.1.7:6380
    replicates 755800e1330a60fde74de939fe7b97daf53abff6
    S: eee4a71f418caf7dd72f491cb2a54b40f4d749bc 192.168.1.7:6381
    replicates ed8a5c0f4df226dbc8a5d534f49decb88b5838cd
   {{< /highlight >}}
   输入yes回车，等待集群初始化

   {{< highlight bash >}}
    >>> Nodes configuration updated
    >>> Assign a different config epoch to each node
    >>> Sending CLUSTER MEET messages to join the cluster
    Waiting for the cluster to join
    .....
    >>> Performing Cluster Check (using node 192.168.1.2:6379)
    M: ddd75e058dd4d3d8c6202f6fb1b90f2f6e3936f7 192.168.1.2:6379
    slots:[0-4095] (4096 slots) master
    1 additional replica(s)
    M: 274ab3f3de33ed601cb33776f5b434ee91cd2288 192.168.1.7:6379
    slots:[8192-12287] (4096 slots) master
    1 additional replica(s)
    S: 4a8ca1ce2786c45610eb79d9936c09a3ec6fac98 192.168.1.6:6381
    slots: (0 slots) slave
    replicates ddd75e058dd4d3d8c6202f6fb1b90f2f6e3936f7
    M: 223eb187a9141c3d651deec15a8243ebf5d01fbf 192.168.1.6:6380
    slots:[12288-16383] (4096 slots) master
    1 additional replica(s)
    S: be352e6487c8fc647077610f0acfc5acbfd8025d 192.168.1.6:6382
    slots: (0 slots) slave
    replicates 274ab3f3de33ed601cb33776f5b434ee91cd2288
    S: 9bf2b9e65d6e91c309926668619a0b613b1ed65b 192.168.1.7:6380
    slots: (0 slots) slave
    replicates 223eb187a9141c3d651deec15a8243ebf5d01fbf
    M: 61f5192703e1258592ed8028fb67f8003d19de34 192.168.1.6:6379
    slots:[4096-8191] (4096 slots) master
    1 additional replica(s)
    S: eee4a71f418caf7dd72f491cb2a54b40f4d749bc 192.168.1.7:6381
    slots: (0 slots) slave
    replicates 61f5192703e1258592ed8028fb67f8003d19de34
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.

{{< /highlight >}}
    见到如上信息表示集群已经建立完成

9.通过redis-cli 连接任意一个节点执行 CLUSTER NODES 查看集群状态，也可以通过redis-cli --cluster check 检查集群状态

10.通过redis-cli向集群写入数据
{{< highlight bash >}}
redis-cli -c -h 192.168.1.2 -p 6379
192.168.1.2:6379> set foo bar
-> Redirected to slot [12182] located at 192.168.1.7:6379
OK
192.168.1.7:6379> get foo
"bar"
192.168.1.7:6379> INCR hello
-> Redirected to slot [866] located at 192.168.1.2:6379
(integer) 1
192.168.1.2:6379> INCR hello
(integer) 2
192.168.1.2:6379> get hello
"2"

{{< /highlight >}}

# 如何日常维护集群？
  Resharding the cluster
    交互式：
    {{< highlight bash >}}
    redis-cli --cluster reshard 127.0.0.1:7000
    {{< /highlight >}}
    命令行式：
    {{< highlight bash >}}
    redis-cli --cluster reshard <host>:<port> --cluster-from <node-id> --cluster-to <node-id> --cluster-slots <number of slots> --cluster-yes
    {{< /highlight >}}
 添加节点
    可以作为master也可以作为slave节点，作为master的节点添加后要Resharding才能使新节点存储数据
 {{< highlight bash >}}
    redis-cli --cluster add-node new_host:new_port existing_host:existing_port
                 --cluster-slave
                 --cluster-master-id <arg>
  {{< /highlight >}} 

更多维护见[官方文档](https://redis.io/topics/cluster-tutorial) 
