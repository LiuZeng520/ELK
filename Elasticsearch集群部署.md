# **Elasticsearch**集群部署

### **一.** **环境准备**

##### 1.下载安装包：

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-linux-x86_64.tar.gz
```

#####  2.解压：

```sh
tar -xvf elasticsearch-7.2.0-linux-x86_64.tar.gz
```

##### 3.先决条件：

（1）.不能以root用户运行，先创建用户：

```shell
useradd elk
```

（2）.需要用root用户修改系统参数：

```shell
vim /etc/security/limits.conf
```

```
#在配置文件里面添加：
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096

vim /etc/sysctl.conf
#在配置文件里面添加：
vm.max_map_count=655360

（3）.JAVA环境
7.2版本的elasticsearch自带jdk，目录为
elasticsearch/jdk
```

##### 4.启动head插件（可选）

```shell
需要node环境，安装：
yum install -y nodejs git

下载插件：
git clone git://github.com/mobz/elasticsearch-head.git

运行：
cd elasticsearch-head
npm install
npm run start
```

打开http://127.0.0.1:9100访问

### 二.配置elasticsearch

###### 1.vim config/elasticsearch.yml

```shell
#监听的IP(ip需要修改)
network.host: 0.0.0.0
#外部访问的端口
http.port: 9200
#允许head插件访问
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,Content-Type
#集群名,需要集群里所有的节点都设置一样，集群名一样才会加入
cluster.name: mpm-elk
#节点名
node.name: node-1
#节点功能
node.master: true #可选为mastar节点
node.data: true #数据节点
#内部IP
transport.tcp.port: 9300
#集群master候选节点(ip需要修改)
discovery.zen.ping.unicast.hosts:["0.0.0.0:9300","0.0.0.0:9300","0.0.0.0:9300"]
#防止脑裂，设置为(master候选节点数/2+1),向下取整
discovery.zen.minimum_master_nodes: 2
#默认
gateway.recover_after_nodes: 3
#tcp传输压缩
transport.tcp.compress: true
```

###### 2.设置JVM选项

vim config/jvm.options

```shell
# Xms表示总堆空间的初始大小
# Xmx表示总堆空间的最大大小
# 尽量设置两个参数的值为一致的,因为内存切换的代价是昂贵的
-Xms2g
-Xmx2g
# 设置回收机制为G1回收器回收
-XX:+UseG1GC
-XX:MaxGCPauseMillis=50
```

###### 3.禁用交换

```sh
sudo swapoff -a
```

### 三.启动

授权：

```shell
chown -R elk.elk elasticsearch/
```

 切换到普通用户：

```shell
su -- elk
```

作为守护进程运行：

```shell
./bin/elasticsearch -d
```

