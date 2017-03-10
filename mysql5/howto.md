# How to

## 运行

运行 `cp -r mysql/conf.d ~/.config/ && chmod -R 644 ~/.config/conf.d `
运行 `docker-compose up -d`

## Mysql 配置说明

master 服务器开启binlog,slow log（可选general log,tcpdump）

> 务必`chmod 644 etc/conf.d/custom.cnf`，否则　*Warning: World-writable config file '/etc/mysql/conf.d/custom.cnf' is ignored*，如果挂载的是 NTFS 磁盘，这个搞不了，除非把该配置放到　ubuntu 的文件系统上。
> TODO: 可能的解决方法，使用 entrypoint 搞定这一切（先摸清mysql5镜像的 CMD 内容）；
> 或使用 build　指令定制　mysql 镜像

## 集群配置: 使用　amoeba　做读写分离

针对 JDK 1.7 +,需要调整 amoeba　的jvm.properties　

```
# JVM_OPTIONS="-server -Xms256m -Xmx1024m -Xss196k -XX:PermSize=16m -XX:MaxP    ermSize=96m"
 33 JVM_OPTIONS="-server -Xms256m -Xmx256m -Xss256k"
```

运行 `mysql -h<amoeba_container_ip> -P8066 -uroot -p` 连接该服务即可
从服务器连接主服务器需要运行相关命令并启动同步进程

> TODO: 是否支持事务，事务中包含读和写

## 命令参考
临时运行：　`docker run --rm --name some-mysql -e MYSQL_ROOT_PASSWORD=changeit mysql:5`

运行　MySQL cli：　`docker exec -it some-mysql mysql -uroot -p`

远程连接：　`mysql -h<target ip> -P<port> -uroot -p`

查看日志：　`mysqlbinlog --verbose mysql5-master-bin.000001` 

使用 mysqldump　导出：　

`mysqldump -h172.20.0.2 -uroot -p --databases testingonly>dump.sql`

导入数据

```
mysql -h172.20.0.3 -uroot -p
mysql> source dump.sql;
```

## 参考资源
- [](https://hub.docker.com/_/mysql/)
- [ MySQL性能剖析工具](https://www.percona.com/doc/percona-toolkit/2.2/pt-query-digest.html)
- [https://launchpad.net/mysql-replicant-python]
- [](https://sourceforge.net/projects/amoeba/files/)
- [](https://github.com/vispractice/Amoeba-Plus-For-MySQL)
- [](http://docs.hexnova.com/amoeba/)
- [mycat cobar amoebar报告](https://wenku.baidu.com/view/92688b890066f5335b812153.html)
- [mycat](http://mycat.io/)

## 操作记录

*启动容器，开启主从同步*

```
docker-compose up -d                    # 启动 docker　容器

mysql -h 172.20.0.2 -uroot -pchangeit   # 登录 master 服务器
mysql> exit

mysqldump -h172.20.0.2 -uroot -p --single-transaction --all-databases>dump.sql

mysql -h172.20.0.3 -uroot -p < dump.sql # 导入备份，如果以前开启过同步，先关闭同步

mysql -h 172.20.0.3 -uroot -pchangeit   # 登录　slave 服务器并开启同步
mysql> change master to master_host='master',master_port=3306,master_user='root',master_password='changeit';           # 连接 master 服务器
mysql> start slave;                     # 启动同步
//mysql> stop slave;                    # 关闭同步
```

> `start slave` 执行一次就好，数据同步事先要执行一次全量导入，然后再主从同步。

*连接amoeba客户端*

```
mysql -h172.20.0.4 -P8066 -uroot -p
mysql> insert into testingonly.person values ("ame", "ame");
Query OK, 1 row affected (0.10 sec)

mysql> select * from testingonly.person;
ERROR 1044 (42000): poolName=readPool, no valid pools # 还未为从数据库导入数据
mysql> exit

mysql -h172.20.0.2  -uroot -p

mysql> select * from testingonly.person;
+-----------+----------+
| firstname | lastname |
+-----------+----------+
| yaping    | wang     |
| jing      | lei      |
| xiaomin   | wang     |
| yan       | wang     |
| foo       | bar      |
| foo       | bar      |
| foo       | bar      |
| ame       | ame      |
+-----------+----------+
8 rows in set (0.00 sec)
mysql> exit

mysqldump -h172.20.0.2 -uroot  -p<dump.sql

mysql -h172.20.0.3  -uroot -p
mysql> change master to master_host='master',master_port=3306,master_user='root',master_password='changeit';
mysql> start slave;
//mysql> delete from testingonly.person where firstname="foo";
// 测试一下，这里删除数据，会导致同步的时候报错，由于MySQL默认采用基于row的同步，master删除一行的时候，在slave上也删除，发现要删除的行不存在。呵呵！

mysql -h172.20.0.4 -P8066 -uroot -p
mysql> select * from testingonly.person;
+-----------+----------+
| firstname | lastname |
+-----------+----------+
| yaping    | wang     |
| jing      | lei      |
| xiaomin   | wang     |
| yan       | wang     |
| ame       | ame      |
| ame       | ame      |
| h         | h        |
+-----------+----------+
7 rows in set (0.01 sec)

mysql -h172.20.0.2 -uroot  -p
mysql> select * from testingonly.person;
+-----------+----------+
| firstname | lastname |
+-----------+----------+
| yaping    | wang     |
| jing      | lei      |
| xiaomin   | wang     |
| yan       | wang     |
| foo       | bar      |
| foo       | bar      |
| foo       | bar      |
| ame       | ame      |
| h         | h        |
+-----------+----------+
9 rows in set (0.00 sec)

```

## TODOs
## 解决customer.cnf 权限的问题
mysql 官方镜像启动命令位于　`/usr/local/bin/docker-entrypoint.sh`,修改他即可




> my_mysql5_slave_1 | 2017-03-07T07:21:00.794338Z 9 [Warning] Slave SQL for channel '': If a crash happens this configuration does not guarantee that the relay log info will be consistent, Error_code: 0
my_mysql5_slave_1 | 2017-03-07T07:21:00.794542Z 9 [Note] Slave SQL thread for channel '' initialized, starting replication in log 'mysql5-master-bin.000003' at position 154, relay log './mysql5-slave-relay-bin.000004' position: 383

> my_mysql5_slave_1 | 2017-03-07T07:07:17.252439Z 6 [ERROR] Slave SQL for channel '': Error executing row event: 'Table 'testingonly.person' doesn't exist', Error_code: 1146
my_mysql5_slave_1 | 2017-03-07T07:07:17.252457Z 6 [Warning] Slave: Table 'testingonly.person' doesn't exist Error_code: 1146
my_mysql5_slave_1 | 2017-03-07T07:07:17.252460Z 6 [ERROR] Error running query, slave SQL thread aborted. Fix the problem, and restart the slave SQL thread with "SLAVE START". We stopped at log 'mysql5-master-bin.000003' position 154


发现一个日志无法应用，
2017-03-09T12:07:33.870635Z 172 [ERROR] Slave SQL for channel '': Could not execute Delete_rows event on table testingonly.person; Can't find record in 'person', Error_code: 1032; handler error HA_ERR_END_OF_FILE; the event's master log mysql5-master-bin.000010, end_log_pos 1027, Error_code: 1032
2017-03-09T12:07:33.870658Z 172 [Warning] Slave: Can't find record in 'person' Error_code: 1032
2017-03-09T12:07:33.870666Z 172 [ERROR] Error running query, slave SQL thread aborted. Fix the problem, and restart the slave SQL thread with "SLAVE START". We stopped at log 'mysql5-master-bin.000010' position 699

查看一下配置
binlog_format                           | ROW         

说明删除的时候该行找不到了，通过 `mysqlbinlog mysql5-master-bin.000010` 找到 *699*发现是一条删除语句，下面想办法跳过该条语句，继续执行日志：

`mysqlbinlog -v mysql5-master-bin.000010 |grep -A4 -B4 699`

mysqlbinlog --start-position=699 mysql5-master-bin.000010 | mysql -h172.20.0.3 -uroot -p



mysql> SHOW SLAVE STATUS \G



发现，主从同步异常，在从库上执行 ：show slave status\G;
出现了如下错误
Last_SQL_Error.Could not execute Delete_rows event on table ….Error_code:1032……
表示的是，主库删除了一行记录，但是在同步到从库的时候 要删除时却找不到这条记录。

> ps: 不知道谁谁谁在从库上删除了该记录，呵呵

修复方式很简单，只需要在从库的mysql数据库上执行三行语句：

mysql> stop slave; 
mysql> set global sql_slave_skip_counter=1; 
mysql> start slave;

跳过一次（还不行的话，要再调整该值）错误，继续同步即可。

默认值如下
| sql_slave_skip_counter | 0     |

> 需要再恢复原来的值 `mysql> set global sql_slave_skip_counter=0;``

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: master

ｏｋ


可以查看日志，每次重启会新建一个日志文件，日志文件超出设置大小也会新建。
