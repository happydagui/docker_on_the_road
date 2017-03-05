
## redis 配置

- 去除保护模式（或启用密码）

`protected-mode yes` = > `protected-mode no`

- 去除本地绑定，运行外部访问

`bind 127.0.0.1` => `# bind 127.0.0.1`

## 启动日志参考

```
redis_slave_2_1  | 1:S 05 Mar 00:54:36.333 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_slave_1_1  |       `-._    `-.__.-'    _.-'                                       
redis_slave_2_1  | 1:S 05 Mar 00:54:36.333 # Server started, Redis version 3.2.8
redis_slave_1_1  |           `-._        _.-'                                           
redis_slave_2_1  | 1:S 05 Mar 00:54:36.333 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis_slave_1_1  |               `-.__.-'                                               
redis_slave_1_1  | 
redis_slave_1_1  | 1:S 05 Mar 00:54:36.359 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_slave_2_1  | 1:S 05 Mar 00:54:36.333 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis_slave_2_1  | 1:S 05 Mar 00:54:36.333 * The server is now ready to accept connections on port 6379
redis_slave_2_1  | 1:S 05 Mar 00:54:36.334 * Connecting to MASTER redis_master:6379
redis_slave_1_1  | 1:S 05 Mar 00:54:36.359 # Server started, Redis version 3.2.8
redis_slave_1_1  | 1:S 05 Mar 00:54:36.360 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis_slave_1_1  | 1:S 05 Mar 00:54:36.360 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.

```



```
[min@t410 db]$ ~/opt/redis-3.2.6/src/redis-cli -h 172.19.0.2 -p 6379
172.19.0.2:6379> keys *
(error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
172.19.0.2:6379> 
```