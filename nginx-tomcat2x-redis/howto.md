## 启动
`docker-compose up -d`

修改一下文件，查看效果
```
docker ps
docker exec -it <container_id> bash
<container_id># cd /usr/local/tomcat/webapps/ROOT
<container_id># cp index.jsp index.jsp.bak
<container_id># echo helloworld > index.jsp
```

使用 `docker inspect <container_id>`查询各个容器 ip，使用<http://<tomcat1 ip:8080>和 <http://<tomcat2 ip>:8080>　访问 tomcat 服务器，然后使用　<http://<nginx1 ip>访问 nginx　服务器，会发现交替显示两个 tomcat　服务器的首页。使用 `docker stop <container_id>`停用一个 tomcat ，再访问 nginx 服务器只能显示一个 tomcat 服务器页面


##　参考
<https://hub.docker.com/_/nginx/>
<https://hub.docker.com/_/tomcat/>
<https://hub.docker.com/_/redis/>

<https://github.com/jcoleman/tomcat-redis-session-manager>
<https://github.com/jcoleman/tomcat-redis-session-manager/downloads>


## faq

jedis版本过高 应改为2.2.1

严重: Error deploying web application directory 
java.lang.VerifyError: Bad type on operand stack
Exception Details:
Location:
com/radiadesign/catalina/session/RedisSessionManager.initializeDatabaseConnection()V @28: invokespecial
Reason:
Type 'redis/clients/jedis/JedisPoolConfig' (current frame, stack[3]) is not assignable to 'org/apache/commons/pool/impl/GenericObjectPool$Config'
