## 如何支持　logstash

``` logback.xml
    <appender name="stash" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
        <file>logs/elk.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/elk.log.%d{yyyy-MM-dd}</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>
```


``` logstash-logback.conf
input {
file {
path => "/media/dev/workspaces/docker/mysamples/elk/springbootapp/logs"
codec => "json"
}
}

output {
stdout{codec =>rubydebug}
}
```

> run `bin/logstash -f logstash-logback.conf`

Ref
- <https://github.com/logstash/logstash-logback-encoder>
- [在线gork正则](http://grokdebug.herokuapp.com/) 
- [Logstash基础正则地址](https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns )

> 如果你是做运维的，那么很幸运，内置的120多个正则，对运维人员来说非常方面，比如常见的Apache的log格式，Nginx的log格式 
  上面的正则库都有成型的正则式，省去了自己编写正则一大部分的工作。 
> 如果你是做开发的，那么稍微麻烦点，开发需要面对各种业务log+系统log，这时候，掌握gork用法，使用正则提取任意内容，变得比较重要了，
不过相信大部分人都会一些基础的正则，所以还是比较轻松的。 