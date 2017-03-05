# 本机运行

临时运行

```
docker run --rm --name some-mysql -e MYSQL_ROOT_PASSWORD=changeit -d mysql:5
```

运行　MySQL cli

```
docker exec -it some-mysql mysql -uroot -pchangeit
```

## 配置
开启binlog,general log,slow log和tcpdump收集日志

> 务必`chmod 644 etc/conf.d/custom.cnf`，否则　*Warning: World-writable config file '/etc/mysql/conf.d/custom.cnf' is ignored*，如果挂载的是 NTFS 磁盘，这个搞不了，除非把该配置放到　ubuntu 的文件系统上。
> cp -rf etc/conf.d ~/.config
> sudo chmod 644 ~/.config/conf.d/custom.cnf
> 可能的解决方法，使用 entrypoint 搞定这一切（先摸清mysql5镜像的 CMD 内容）；
> 或使用 build　指令定制　mysql 镜像

## Refs
- [](https://hub.docker.com/_/mysql/)
- [ MySQL性能剖析工具](https://www.percona.com/doc/percona-toolkit/2.2/pt-query-digest.html)

## 解决customer.cnf 权限的问题
mysql 官方镜像启动命令位于　`/usr/local/bin/docker-entrypoint.sh`,修改他即可
