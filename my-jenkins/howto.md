```
cd build
docker-compose up -d
```

> 通过获得`docker logs <container_id>`查找登录密码
> 通过 `docker inspect <container_id>`获得容器 ip
> 通过 `docker exec -it <container_id　bash`进入容器的命令行

注意：修改　*docker-compose.yml* 文件后重新运行 `docker-compose up -d`命令，如果有变更会自动删除原始的容器，生成新的；没有变更，则启动原有容器。

> 更多内容擦参考 <https://hub.docker.com/_/jenkins/>`docker run -it -d --name my-jenkins -v /home/min/opt:/shares jenkins`
 

##简单运行一下　
`docker run -it -d --rm --name my-jenkins -v /home/min/opt:/shares jenkins`
 
