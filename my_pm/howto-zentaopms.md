
#　运行

假定已经安装好 docker 和 docker-compose。

```
docker start my_mysql5_master
docker start my_redis_master
docker pull tutum/lamp
docker-compose -f docker-compose-with-zentaopms.yaml up -d
```

[http://<container_ip>/zentaopms/www/]
安装和设置管理员账号，例如 admin/changeit
