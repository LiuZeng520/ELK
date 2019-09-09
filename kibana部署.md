kibana部署

1.下载安装

```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.2.0-linux-x86_64.tar.gz
tar zxf kibana-7.2.0-linux-x86_64.tar.gz
```

2.修改配置

```yaml
server.host: "0.0.0.0"  # 修改为本机IP
elasticsearch.hosts: ["http://0.0.0.0:9200"]  # 修改为ES集群地址
# ES集群的用户名密码
elasticsearch.username: "******"
elasticsearch.password: "******"
elasticsearch.requestTimeout: 90000  # 超时等待时间
i18n.locale: "zh-CN"  # kibana中文界面展示
```

3.启动

```shell
./bin/kibana &
```

4.愉快地使用kibana

(1).在 “管理”--"安全性" 管理用户权限

(2).在 “管理”--"kibana"--"索引模式" 创建 索引模式，使用正则匹配，如 web*

如果可以创建就说明ELK集群部署成功了。