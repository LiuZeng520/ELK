filebeat部署

1. 下载安装

```shell
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.2.0-linux-x86_64.tar.gz
tar zxf filebeat-7.2.0-linux-x86_64.tar.gz
mv filebeat-7.2.0-linux-x86_64 filebeat-7.2.0
```

2. 修改配置

```shell
cd filebeat-7.2.0
vim filebeat.yml
```

```yml
filebeat.inputs:
 -  type: log
  enabled: true   # 把flase 改为 true
  paths:
    - /data/logs/*.log  # 改成日志的路径
......
tags: ["web"] # 可以用标签来做日志区分 logstash引用字段内容：%{[tags][0]}
......
output.logstash:
  hosts: ["logstash:5044"]  #改成logstash的ip端口
......
processors:
  - add_host_metadata: 
      netinfo.enabled: true  #默认为flase,打开可以添加IP、mac等主机元数据
      cache.ttl: 5m
```

3. 启动

   ```shell
   ./filebeat -e -c filebeat.yml &
   ```

   

