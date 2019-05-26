---
title: "使用nginx和keepalived构建高可用websocket"
date: 2019-05-24T20:11:54+08:00
draft: false
---
# 背景：
日常项目中的消息推送，im等需求会使用到websocket，项目对于高可用和扩展性要求比较高的时候就必须使用一些HA和负载均衡方案，本文将基于[keepalived](http://www.keepalived.org)和[nginx](http://nginx.org)构建高可用websocket部署。

# 部署架构图：
![部署架构](/images/ha_websocket.png)

# 软件配置：

server1：

keepalived配置

{{< highlight bash >}}

global_defs {
} 
vrrp_script chk_nginx {
       script "killall -0 nginx"     # cheaper than pidof
       interval 2                      # check every 2 seconds
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s3
    virtual_router_id 51
    priority 50
    advert_int 1
    virtual_ipaddress {
        192.168.1.110
    }
    track_script {
        chk_nginx
    }

}
vrrp_instance VI_2 {
    state MASTER
    interface enp0s3
    virtual_router_id 52
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.1.112
    }
    track_script {
        chk_nginx
    }

}
    {{< /highlight >}}

nginx配置：

{{< highlight bash >}}
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    upstream websocket {
        server 192.168.1.16:8010;
        server 192.168.1.17:8010;
        server 192.168.1.18:8010;
        #server 192.168.1.18:8030;
    }

    server {
        listen 8020;
        location / {
            proxy_pass http://websocket;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }
    }
{{< /highlight >}}

启动server1的keepalived和nginx

通过ip addr 检查vip是否注册成功
{{< highlight bash >}}
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ea:42:a9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.14/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 70455sec preferred_lft 70455sec
    inet 192.168.1.112/32 scope global enp0s3
       valid_lft forever preferred_lft forever

{{< /highlight >}}

server2：

keepalived配置

{{< highlight bash >}}
global_defs {
}

vrrp_script chk_nginx {
       script "killall -0 nginx"     # cheaper than pidof
       interval 2                      # check every 2 seconds
}
vrrp_instance VI_1 {
    state MASTER
    interface enp0s3
    virtual_router_id 51
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.1.110
    }
    track_script {
	chk_nginx
    }

}

vrrp_instance VI_2 {
    state BACKUP
    interface enp0s3
    virtual_router_id 52
    priority 50
    advert_int 1
    virtual_ipaddress {
        192.168.1.112
    }
    track_script {
        chk_nginx
    }

}

{{< /highlight >}}

nginx配置：

{{< highlight bash >}}
map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }
 
    upstream websocket {
	server 192.168.1.16:8010;
    server 192.168.1.17:8010;
    server 192.168.1.18:8010;
	#server 192.168.1.18:8030;  
  }
 
    server {
        listen 8020;
        location / {
            proxy_pass http://websocket;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }
    }

{{< /highlight >}}

启动server2的keepalived和nginx

通过ip addr 检查vip是否注册成功
{{< highlight bash >}}
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:00:0e:6d brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.15/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 70335sec preferred_lft 70335sec
    inet 192.168.1.110/32 scope global enp0s3
       valid_lft forever preferred_lft forever

{{< /highlight >}}


server3,server4,server5的应用程序代码：

{{< highlight javascript >}}
console.log("Server started at 8010");
var Msg = '';
var WebSocketServer = require('ws').Server
    , wss = new WebSocketServer({port: 8010});
    wss.on('connection', function(ws) {
        ws.on('message', function(message) {
        console.log('Received from client: %s', message);
        ws.send('Server 192.168.1.18  received from client: ' + message);
    });
 });
{{< /highlight >}}

分别启动3个sever的nodejs程序

{{< highlight bash >}}
    node server.js
{{< /highlight >}}

# 通过wscat验证环境:

安装wscat
{{< highlight bash >}}
    npm install -g wscat
{{< /highlight >}}

使用wscat分别连接两个vip

{{< highlight bash >}}
    wscat -c 192.168.1.110:8020
    wscat -c 192.168.1.112:8020
    wscat -c 192.168.1.110:8020
    wscat -c 192.168.1.112:8020
{{< /highlight >}}

运行结果如下：

![客户端测试](/images/client_test.png)

# keepalived HA测试

#### kill server1 nginx进程
{{< highlight bash >}}
    sudo killall -9 nginx
{{< /highlight >}}
通过ip addr查看ip的漂移

server1:
{{< highlight bash >}}
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ea:42:a9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.14/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 60959sec preferred_lft 60959sec
    inet6 fe80::a00:27ff:feea:42a9/64 scope link 
       valid_lft forever preferred_lft forever
{{< /highlight >}}
server2:
{{< highlight bash >}}
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:00:0e:6d brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.15/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 60775sec preferred_lft 60775sec
    inet 192.168.1.110/32 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.1.112/32 scope global enp0s3
       valid_lft forever preferred_lft forever
{{< /highlight >}}

#### server1关机

结果同上

当server1恢复正常时vip又会改变成初始状态

# 故障对客户端程序的影响

![故障对客户端程序的影响](/images/ha_test.png)

# 增加websocket server 节点

增加节点只需修改server1和server2的nginx配置添加响应的server配置，然后reload nginx就可以。

增加节点后就会有新的client与新sever建立连接

# 总结

基于keepalived和nginx的websocket的高可用方案只能实现基本的故障转移，以及负载均衡和扩容，对于整个的应用的高可用的功能的实现websocket客户端应用程序必须实现 ***重连功能*** 来解决故障转移过程中的连接的丢失问题。

同时在客户端测试的过程中发现一段时间，server端的连接就会超时自动断开，
因此在开发websocket客户端应用程序时要加上 ***心跳功能***。

# 附录

vrrp协议请求包

![vrrp](/images/vrrp.png)

websocket请求建立请求包

![websocket1](/images/websocket-1.png)
![websocket2](/images/websocket-2.png)
![websocket3](/images/websocket-3.png)



