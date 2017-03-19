
#　运行

```
docker pull hachque/phabricator
docker pull sameersbn/gitlab:8.16.6
docker pull redmine:3.1
docker-compose up -d
```

浏览[http://172.22.0.6/]访问gitlab，设置root账户密码，比如changeit
浏览[http://172.22.0.6:3000/]访问 redmine，默认账号 admin/admin
浏览[http://172.22.0.4/auth/register/] 访问 phabricator,设置管理账号，密码要求比较严格: admin/changethepassword
（容器中app dir: /srv/phabricator/phabricator）

# 整合 gitlab 和 redmine

> 原理：　redmine 为每个 project 克隆远程仓库的一个镜像，redmine_gitlab_hook插件提供了远程接口。gitlab在有提交时回调此接口更新本地的镜像，redmine从本地镜像中获取提交日志。

## 使用 gitlab 创建第一个 project，例如　*my-first-project*，克隆到本机准备以后修改

`cd ~/temp && git clone http://172.22.0.6/root/my-first-project`

##　配置 redmine

### 配置问题状态和跟踪标签
按需增加一些问题状态，例如打开/进行中/已解决/关闭等

### 配置版本库
开启 scm 和　web service，这里需要生成　API Key。
另外，可以根据跟踪标签和关键字的设置，自动设置issue状态。

### 安装readmine插件redmine_gitlab_hook

在宿主机中下载插件

```
cd <your plugin dir>
git clone --depth=1 https://github.com/phlegx/redmine_gitlab_hook
```

> <your plugin dir>指 *docker-compose.yml*中`/media/dev/db/redmine_home/plugins:/usr/src/redmine/plugins`中的宿主机目录

在容器中安装

```
docker exec -it my_redmine bash
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
```

> 更多插件 [http://www.redmine.org/plugins]

在 redmine　中创建一个项目，例如 *My First Project*
设置参数如下：
SCM: git
勾选主版本库
库路径： **/gitrepo/my-first-project/**
路径编码：　utf8

### 配置 *Redmine GitLab Hook plugin*　插件

配置/插件/Redmine GitLab Hook plugin
勾选：All branches, Auto create, Fetch updates from repository
设置： Local repositories path = /gitrepo

### 在容器中克隆 gitlab 仓库的镜像
```
docker exec -it my_remine
cd /gitrepo
git clone --mirror http://172.22.0.6/root/my-first-project    
```

> 在容器中操作，故/gitrepo　实际指向 *docker-compose.yml*中`/media/dev/db/redmine_home/gitrepo:/gitrepo`的容器目录为前半部分


## 在 gitlab 中配置　webhook，有提交时通知 redmine 同步
<your project>/Integrations添加 Webhook，参数如下：
 http://my_redmine:3000/gitlab_hook?project_id=<project_id>&key=<your key> 

> <project_id>指向 redmine 工程的id，例如my-first-project
> <your key>指向 redmine　中配置的 web service的key

# docker 资源
*使用的 docker 资源*
[https://hub.docker.com/_/redmine/]
[https://hub.docker.com/r/sameersbn/gitlab/]
[https://hub.docker.com/r/hachque/phabricator/] and [https://github.com/RedpointGames/phabricator]

*其他资源*
[https://hub.docker.com/r/gitlab/gitlab-ce/] 官方gitlab
[http://www.open-open.com/lib/view/open1438009785316.html]


















todo:
按照phabricator提示修改MySQL参数
max_allowed_packet=4194304 # 现值33554432
sql_mode=STRICT_ALL_TABLES # 现值"ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
