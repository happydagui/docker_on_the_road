
#　运行

假定已经安装好 docker 和 docker-compose。

```
docker pull redpointgames/phabricator
docker pull sameersbn/gitlab:8.16.6
docker pull redmine:3.1

docker start my_mysql5_master
docker start my_redis_master
docker-compose up -d
```

使用 `docker inspect <container_name>`查看各个容器 ip，假定分配如下 ip:

- gitlab: 172.22.0.4
- redmine: 172.22.0.5
- phabracator: 172.22.0.6

```
docker exec -it my_phabricator /srv/phabricator/phabricator/bin/config set phabricator.base-uri 'http://172.22.0.6/'
```

> 若 my_phabricator 容器 ip 地址变更时，需要使用下面的命令来设置参数和恢复管理员账户
> `docker exec -it my_phabricator /srv/phabricator/phabricator/bin/auth recovery admin`

浏览[http://172.22.0.4/]访问gitlab，首次访问需设置root账户密码，比如changeit
浏览[http://172.22.0.5:3000/]访问 redmine，默认账号 admin/admin
浏览[http://172.22.0.6] 访问 phabricator,首次访问需要设置管理账号，密码要求比较严格，例如 admin/changethepassword

# 整合 gitlab 和 redmine

> 原理：　redmine 为每个 project 克隆远程仓库的一个镜像，redmine_gitlab_hook插件提供了远程接口。gitlab在有提交时回调此接口更新本地的镜像，redmine从本地镜像中获取提交日志。

## 使用 gitlab 创建第一个 project，例如　*my-first-project*，克隆到本机准备以后修改

`cd ~/temp && git clone http://172.22.0.4/root/my-first-project`

##　配置 redmine

**配置问题状态和跟踪标签以及设置优先级**

- 通过“管理/问题状态”菜单，定义问题状态，例如打开/进行中/已解决/关闭等
- 通过“管理/跟踪标签”菜单，定义跟踪标签，需要关联对应的项目，否则在项目页面无法创建该类 issue
- 通过"管理/枚举值/优先级"　菜单定义优先级，例如低、中、高、非常高、紧急等

**配置版本库**

通过“管理/配置/版本库”菜单，启用 SCM 和　web service，这里需要生成　API Key。
另外，可以根据跟踪标签和关键字的设置，自动设置issue状态。

**安装readmine插件redmine_gitlab_hook**

1. 宿主机中下载插件并安装

```
cd <your host plugin dir>
git clone --depth=1 https://github.com/phlegx/redmine_gitlab_hook
docker exec -it my_redmine bundle exec rake redmine:plugins:migrate RAILS_ENV=production
```

> <your host plugin dir>指 *docker-compose.yml*中`/media/dev/db/redmine_home/plugins:/usr/src/redmine/plugins`中的宿主机目录
> 更多插件 [http://www.redmine.org/plugins]

2. 配置 *Redmine GitLab Hook plugin*　插件

使用“管理/插件/Redmine GitLab Hook plugin/配置”菜单进入设置界面：

- 勾选：All branches, Auto create, Fetch updates from repository三项
- 设置： Local repositories path = /gitrepo
 
3. 范例：　创建 redmine 项目并关联到 gitlab　代码库

**创建redmine项目**
在 redmine　中创建一个项目，例如 *My First Project*，设置参数如下：

- SCM: git
- 勾选主版本库
- 库路径： **/gitrepo/my-first-project/**
- 路径编码：　utf8

**在 gitlab 中配置　webhook**
有提交时通知 redmine 同步，使用“<your project>/Integrations”菜单，添加 Webhook，参数如下：
 http://my_redmine:3000/gitlab_hook?project_id=<project_id>&key=<your key> 

> <project_id>指向 redmine 工程的id，例如my-first-project
> <your key>指向 redmine　中配置的 web service的key

**在容器中克隆 gitlab 仓库的镜像**

```
docker exec -it my_remine
cd /gitrepo
git clone --mirror http://172.22.0.4/root/my-first-project    
```

> 在容器中操作，故/gitrepo　实际指向 *docker-compose.yml*中`/media/dev/db/redmine_home/gitrepo:/gitrepo`的容器目录为前半部分。
> 在本机中操作也可以，`cd /media/dev/db/redmine_home/gitrepo && git clone --mirror http://172.22.0.4/root/my-first-project`

# 整合 gitlab 和 Phabricator

1. 创建项目
进入首页，通过 “Create a Repository”菜单在 Phabricator中创建一个项目，例如 *My First Project*，选择创建一个 git 项目。参数如下：

- Name: My First Project
- Callsign: MIN

> Callsign参数后面会用到

完成之后通过“URIs/Add New URI”添加一个新的URI，指向gitlab代码库的地址，参数如下：

- URI: http://172.22.0.4/root/my-first-project/
- I/O Type: Observe: Copy from a remote

2.　测试

使用 `docker exec -it my_phabricator /srv/phabricator/phabricator/bin/repository update -- rMIN` 命令立即更新。进入项目页面就可以看到代码和修改记录。

> MIN　是项目的　Callsign　参数名称。上述命令会在容器的 /repos　目录（已经挂载到了宿主机目录）克隆远程代码库的镜像，并有后台 daemon 程序自动同步。

3. 使用phabicator作代码审核

> TODO


# FAQs 

## For phabricator

> 提示： phabricator日志位置 /var/tmp/phd/log/daemon.log，有问题查日志

**更新 repository 的命令`/srv/phabricator/phabricator/bin/repository update -- rMIN`出现异常**

```
 EXCEPTION: (Exception) Define 'phabricator.base-uri' in your configuration to continue. at [<phabricator>/src/infrastructure/env/PhabricatorEnv.php:473]
arcanist(head=master, ref.master=3b6b523c2b23), phabricator(head=master, ref.master=a36b1e8f64ab), phutil(head=master, ref.master=13a200ca7621)
  #0 PhabricatorEnv::getAnyBaseURI() called at [<phabricator>/src/infrastructure/env/PhabricatorEnv.php:384]
  #1 PhabricatorEnv::getURI(string) called at [<phabricator>/src/applications/repository/storage/PhabricatorRepository.php:2233]
  #2 PhabricatorRepository::newBuiltinURIs() called at [<phabricator>/src/applications/repository/storage/PhabricatorRepository.php:2146]
  #3 PhabricatorRepository::attachURIs(array) called at [<phabricator>/src/applications/repository/query/PhabricatorRepositoryQuery.php:373]
  #4 PhabricatorRepositoryQuery::didFilterPage(array) called at [<phabricator>/src/infrastructure/query/policy/PhabricatorPolicyAwareQuery.php:273]
  #5 PhabricatorPolicyAwareQuery::execute() called at [<phabricator>/src/applications/repository/management/PhabricatorRepositoryManagementWorkflow.php:18]
  #6 PhabricatorRepositoryManagementWorkflow::loadRepositories(PhutilArgumentParser, string) called at [<phabricator>/src/applications/repository/management/PhabricatorRepositoryManagementWorkflow.php:40]
  #7 PhabricatorRepositoryManagementWorkflow::loadLocalRepositories(PhutilArgumentParser, string) called at [<phabricator>/src/applications/repository/management/PhabricatorRepositoryManagementUpdateWorkflow.php:47]
  #8 PhabricatorRepositoryManagementUpdateWorkflow::execute(PhutilArgumentParser) called at [<phutil>/src/parser/argument/PhutilArgumentParser.php:441]
  #9 PhutilArgumentParser::parseWorkflowsFull(array) called at [<phutil>/src/parser/argument/PhutilArgumentParser.php:333]
  #10 PhutilArgumentParser::parseWorkflows(array) called at [<phabricator>/scripts/repository/manage_repositories.php:22]
```

查看代码如下：

```
public static function getAnyBaseURI() {
    $base_uri = self::getEnvConfig('phabricator.base-uri');

    if (!$base_uri) {
      $base_uri = self::getRequestBaseURI();
    }

    if (!$base_uri) {
      throw new Exception(
        pht(
          "Define '%s' in your configuration to continue.",
          'phabricator.base-uri'));
    }

    return $base_uri;
  }
```

解决：使用`bin/config set phabricator.base-uri <right url>`确保参数设置正确


*进入Phabricator首页异常*

```
Authentication Failure
This Phabricator install is not configured with any enabled authentication providers which can be used to log in. If you have accidentally locked yourself out by disabling all providers, you can use `phabricator/bin/auth recover <username>` to recover access to an administrative account.
```

按照提示操作即可，参考前面的内容。

# docker 资源

<https://hub.docker.com/_/redmine/>
<https://hub.docker.com/r/sameersbn/gitlab/>
<https://hub.docker.com/r/hachque/phabricator/>
<https://github.com/RedpointGames/phabricator>
(官方gitlab)[https://hub.docker.com/r/gitlab/gitlab-ce/]

---------------------------------
todo:
按照phabricator提示修改MySQL参数
max_allowed_packet=4194304 # 现值33554432
sql_mode=STRICT_ALL_TABLES # 现值"ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"





