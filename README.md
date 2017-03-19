内容列表

## 使用　Docker和Docker Compose　组合应用

- [nginx-tomcat2x-redis](nginx-tomcat2x-redis/howto.md)

> 使用 nginx 作为负载均衡服务器，两个 tomcat 节点作为应用服务器，redis　作为共享的 session 服务器

- [my-jenkins](my-jenkins/howto.md)

> 使用 jenkins 作为持续集成服务器

- [ELK](elk/howto.md)

> 搭建日志集中管理平台，Logstash 采集日志到 Elasticsearch 系统，通过 Kibana　对日志进行查询和统计。使用　Springboot 构建 web 应用作为日志源。

- [Redis 集群] (redis-cluster/howto.md)

> 配置 Redis 集群

- [Mysql主从结构 + Amoeba 读写分离](mysql5/howto.md)

- [使用 gitlab+redmine+phabricator做项目管理](my_pm/howto.md)

## 运行准备

`docker network create -d bridge mynet`

> 创建共用的网络，有共享需求的话，将docker-compose产生的容器加入该网络
> mysql5集群和redis集群已经加入了此网络，其他需要使用数据库的需要加入此网络。


*不同 docker 容器之间跨网段访问*

> docker-compose　创建的网络默认不在一个网段上，不加入 mynet　网络的话，可以在宿主机上执行下面的操作来互通

```
sudo su -
# iptables -F
# iptables -A FORWARD -j ACCEPT
# iptables -t nat -A POSTROUTING -j MASQUERADE 
# service iptables save
# iptables-save > /etc/sysconfig/iptables
```

> 最后保存规则的语句使用一条即可

来看一下保存的规则是啥子：

```
# Generated by iptables-save v1.6.0 on Thu Mar 16 15:29:57 2017
*mangle
:PREROUTING ACCEPT [27941:20941105]
:INPUT ACCEPT [17930:16529153]
:FORWARD ACCEPT [26:2184]
:OUTPUT ACCEPT [18178:1621872]
:POSTROUTING ACCEPT [18394:1642321]
-A POSTROUTING -o virbr0 -p udp -m udp --dport 68 -j CHECKSUM --checksum-fill
COMMIT
# Completed on Thu Mar 16 15:29:57 2017
# Generated by iptables-save v1.6.0 on Thu Mar 16 15:29:57 2017
*nat
:PREROUTING ACCEPT [358:156663]
:INPUT ACCEPT [3:252]
:OUTPUT ACCEPT [136:8844]
:POSTROUTING ACCEPT [0:0]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING -s 172.22.0.0/16 ! -o br-c9b9a2efebc8 -j MASQUERADE
-A POSTROUTING -s 192.168.122.0/24 -d 224.0.0.0/24 -j RETURN
-A POSTROUTING -s 192.168.122.0/24 -d 255.255.255.255/32 -j RETURN
-A POSTROUTING -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p tcp -j MASQUERADE --to-ports 1024-65535
-A POSTROUTING -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p udp -j MASQUERADE --to-ports 1024-65535
-A POSTROUTING -s 192.168.122.0/24 ! -d 192.168.122.0/24 -j MASQUERADE
-A POSTROUTING -s 172.18.0.0/16 ! -o br-80835a06d8ec -j MASQUERADE
-A POSTROUTING -s 172.21.0.0/16 ! -o br-1a3deb4a45f6 -j MASQUERADE
-A POSTROUTING -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
-A DOCKER -i br-c9b9a2efebc8 -j RETURN
-A DOCKER -i br-80835a06d8ec -j RETURN
-A DOCKER -i br-1a3deb4a45f6 -j RETURN
COMMIT
# Completed on Thu Mar 16 15:29:57 2017
# Generated by iptables-save v1.6.0 on Thu Mar 16 15:29:57 2017
*filter
:INPUT ACCEPT [1518:1033419]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [1762:210531]
:DOCKER - [0:0]
:DOCKER-ISOLATION - [0:0]
-A FORWARD -j ACCEPT
COMMIT
# Completed on Thu Mar 16 15:29:57 2017

```
