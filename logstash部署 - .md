# logstash部署说明

### 一.环境准备

##### 1.下载安装包：

```shell
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.2.0.tar.gz
```

##### 2.解压：

```shell
tar zxf logstash-7.2.0.tar.gz
```

##### 3.JAVA环境：

elasticsearch-7.2.0里面自带的jdk版本为12.0.1，而logstash-7.2.0需要的java版本为1.8.0，安装java：

```shell
yum install java-1.8.0-openjdk -y
```

### 二.配置logstash

###### 1.配置logstash启动参数

vim config/logstash.yml

```shell
# pipeline线程数，官方建议是等于CPU内核数
pipeline.workers: 4
# 每次发送的事件数
pipeline.batch.size: 500
# 发送延时
pipeline.batch.delay: 5
```

###### 2.调整堆内存

```shell
# Xms表示总堆空间的初始大小
# Xmx表示总堆空间的最大大小
# 设置两个值相等，至少为系统预留1G，最大不能超过32G
-Xms4g
-Xmx4g
```

优化：提高每次批处理的数量，调节传输间歇时间。当batch.size增大，es处理的事件数就会变少，写入也就越快了，但是logstash占用的内存也会越高，需要慢慢调整慢慢测试，观察以调整到最优值。

###### 3.配置文件

vim config/logstash.conf

```shell
# 输入插件，从filebeat接收数据
# Beats -> Logstash -> Elasticsearch
input {
  beats {
    port => 5044
  }
}

# 过滤插件
# 参考文档 https://www.elastic.co/guide/en/logstash/7.2/filter-plugins.html
filter {
  # 把json格式日志处理,把各属性都拆出来
  json {
    source => "message"
  }
}
```

```shell
# 输出插件，发送数据到ES
output {
    elasticsearch {
        hosts => ["es1:9200","es2:9200","es3:9200"]
        # 在filebeat里面定义tags，以tags做索引名来区分各服务
        index => "%{[tags][0]}-%{+YYYY.MM.dd}"
    }
}
```

### 三.启动

（1）测试配置文件是否可用：

```shell
./bin/logstash -f config/logstash.conf --config.test_and_exit
```

（2）监视配置并重新加载，默认每3s一次，可加 --config.reload.interval 参数修改

```shell
./bin/logstash -f config/logstash.conf --config.reload.automatic
```

在后台启动：

```shell
./bin/logstash -f config/logstash.conf
```

