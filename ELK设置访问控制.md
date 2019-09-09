### 一、加密elasticsearch通信

##### 1.在elasticsearch.yml打开安全设置

```shell
echo "xpack.security.enabled: true" >>  elasticsearch.yml
```

##### 2.生产节点证书

###### (1).为Elasticsearch集群创建证书颁发机构

```shell
bin/elasticsearch-certutil ca
```

默认输出文件名称为: elastic-stack-ca.p12,请保留该文件和记住生成该文件所输入的密码。

###### (2).为群集中的每个节点生成证书和私钥

```shell
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

###### (3).将节点证书复制到适当的位置

将适用的`.p12`文件复制到**<u>每个节点</u>**上Elasticsearch配置目录中的目录中

比如： elasticsearch-7.2.0/config/certs/

##### 3.加密集群中节点之间的通信

###### (1).请将以下信息添加到`elasticsearch.yml`每个节点上的 文件中：

```yaml
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12 
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```

###### (2).执行以下命令，将密码添加到Elasticsearch密钥库：

```shell
bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

##### 4.在elasticsearch配置安全性

###### (1).在执行完“3”之后重启es集群

关闭es节点尽量不要直接使用

```sehll
kill -9 $PID
```

第一步：先禁止分片分配

```shell
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d' { "persistent": { "cluster.routing.allocation.enable": "primaries" } } '
```

第二步：再停止索引并执行同步刷新，执行同步刷新可加快碎片恢复速度

```shell
curl -X POST "localhost:9200/_flush/synced?pretty"
```

执行同步刷新时，请检查响应以确保没有失败。虽然请求本身仍返回200 OK状态，但响应正文中列出了由于挂起的索引操作而失败的同步刷新操作。如果有失败，请重新发出请求。

第三步：关闭所有节点

```shell
kill $PID
```

第四步：启动每个节点

```shell
su - elk
bin/elasticsearch -d
exit
```

第五步：等待所有节点加入群集并报告黄色状态

打开localhost:9100观察es集群状态

或者执行

```shell
curl -X GET "localhost:9200/_cat/health?pretty"
```

观察集群健康值，刚启动时是red，等待它变成yellow

第六步：重新启用分配

```shell
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d' { "persistent": { "cluster.routing.allocation.enable": null } } '
```

完成重启完成。

###### (2).设置所有内置用户的密码

```shell
bin/elasticsearch-setup-passwords interactive
```

<u>**所有用户都提示修改密码成功才算修改成功**</u>

### 二.在logstash里面添加验证

在logstash.conf配置文件里面的输出插件配置里面添加user和password，如:

```shell
output {
    elasticsearch {
        ......
        user => "xxxx"
        password => "xxxxxx"
    }
}
```

然后重启logstash

### 三.配置kibana

修改kibana.yml

```yaml
elasticsearch.username: "xxxx"
elasticsearch.password: "xxxxxx"
```

重启kibana

然后用之前在“4.(2)”设置的用户名密码登录

http://localhost:5601

在管理---》用户  ，可以创建、修改用户，更改权限等

![1566456184838](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1566456184838.png)

