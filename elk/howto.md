## 启动
`docker-compose up -d`

等价的 `docker run` 命令
1. 启动 *springbootapp*
> ` docker run --rm -v "$PWD/spring-boot-sample-simple-0.0.0.jar:/usr/spring-boot-sample-simple-0.0.0.jar" -v "/home/min/opt:/opt:ro" centos /opt/jdk1.8.0_111/bin/java -jar /usr/spring-boot-sample-simple-0.0.0.jar
`
2. 启动 *elasticsearch*
> `docker run -p 9200:9200 docker.elastic.co/elasticsearch/elasticsearch:5.2.1`
> 
> 本机需要配置参数，添加　*vm.max_map_count=262144*，然后`sudo sysctrl -p `生效，通过
> `more /proc/sys/vm/max_map_count`

访问 <http://<ip>:9200>，默认账号： elastic/changeme

3. logstash vs elasticsearch

> [Configuration logstash vs elasticsearch](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticseach.html)


##　参考
- <https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html>
- `docker pull docker.elastic.co/elasticsearch/elasticsearch:5.2.1`
- [kibana](https://www.elastic.co/products/kibana)
- [logstash](https://github.com/elastic/logstash),also <https://www.elastic.co/downloads/logstash>

### Kibana Installation Steps
1. Download and unzip Kibana
Kibana can also be installed from our package repositories using apt or yum. See Repositories in the Guide.
2. Point to Elasticsearch
Open config/kibana.yml in an editor
Set elasticsearch.url to point at your Elasticsearch instance
3. Run bin/kibana (or bin\kibana.bat on Windows)
4. Point your browser at http://localhost:5601
5. Dive into the getting started guide and video.

### Logstash Installation Steps

1. Download and unzip Logstash
Logstash can also be installed from our package repositories using apt or yum. See Repositories in the Guide.
2. Prepare a logstash.conf config file
3. Run bin/logstash -f logstash.conf
4. Dive into the getting started guide and video.


## faq
- run `bin/logstash -e 'input { stdin { } } output { stdout { codec => rubydebug } }'`　出现权限错误
2017-02-28 20:16:11,190 main ERROR FileManager (/home/min/opt/logstash-5.2.1/logs/logstash-slowlog-plain.log) java.io.FileNotFoundException: /home/min/opt/logstash-5.2.1/logs/logstash-slowlog-plain.log (Permission denied) java.io.FileNotFoundException: /home/min/opt/logstash-5.2.1/logs/logstash-slowlog-plain.log (Permission denied)

发现 logs 目录属于 root 组，估计是那次使用root操作了，解决 `sudo chown -R min:min logs`
data 权限问题 `sudo chown -R min:min data`


- 集成 ik 分词
