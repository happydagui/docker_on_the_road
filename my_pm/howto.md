
```
docker pull hachque/phabricator
docker pull sameersbn/gitlab:8.16.6
docker pull redmine:3.1
docker-compose up -d
```

浏览[http://172.22.0.5/]初始化，设置root账户密码，比如changeit
浏览[http://172.22.0.3:3000/]访问 redmine，默认账号 admin/admin
浏览[http://172.22.0.2/auth/register/] 访问 phabricator,设置管理账号，密码要求比较严格: admin/changethepassword（容器中app dir: /srv/phabricator/phabricator）

安装readmine插件
```
cd <your plugin dir>
git clone --depth=1 https://github.com/phlegx/redmine_gitlab_hook
```

```
docker exec -it my_redmine bash
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
```

更多插件 [http://www.redmine.org/plugins]

## docker 资源
*使用的 docker 资源*
[https://hub.docker.com/_/redmine/]
[https://hub.docker.com/r/sameersbn/gitlab/]
[https://hub.docker.com/r/hachque/phabricator/] and [https://github.com/RedpointGames/phabricator]

*其他资源*
[https://hub.docker.com/r/gitlab/gitlab-ce/] 官方gitlab
[http://www.open-open.com/lib/view/open1438009785316.html]






按照提示修改MySQL参数
max_allowed_packet=4194304 # 现值33554432
sql_mode=STRICT_ALL_TABLES # 现值"ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
