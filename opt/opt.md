# 运维相关
## 修改时间和时区
`date -s "2019-02-18 09:59:30" && clock -w`

## 性能分析命令
```
cpu : vmstat,iostat,topas,nmon,ps,sar,time,timex,netpmon,trace,trcrpt; 
内存：vmstat,topas,nmon,ps,svmon,lsps,filemon,trace,trcrpt; 
磁盘：iostat,topas,nmon,lvmstat,iostat -d, lvmstat, lsps,filemon,lsattr,lsdev; 
网络：netstat,topas,nmon,entstat,nfsstat,ifconfi

g,iptrace,ipreport,trace,trcrpt; 
监视CPU使用情况：vmstat 2 ; iostat -t 2 6;sar -P ALL 2 3; 
监视内存使用情况： vmstat 2 10;ps aux;svmon -G;svmon -Pau 10; 
监视I/O使用情况： iostat 5;sar -d 3 3; 
监视网络使用情况： netstat -i ;netstat -m;netstat -v
```
## 查看端口连接情况
`lsof -i : 17017`

## 开放端口
```
sudo firewall-cmd --permanent --add-port=18080/tcp
sudo firewall-cmd --reload
```

## kafka运维
```
删除Topic 
./kafka-topics.sh -delete -zookeeper localhost:12181 -topic TrxTopic
查看消费情况 
./kafka-consumer-groups.sh --bootstrap-server localhost:19092  --describe --group trx
列出所有Topic信息 

./kafka-topics.sh --describe --zookeeper localhost:12181
列出所有消费组 
./kafka-consumer-groups.sh --bootstrap-server localhost:19092  --list 
增加分区 
./kafka-topics.sh  --alter --zookeeper localhost:12181 --topic TrxTopic --partitions 16
```

## druid运维
```
出现Getting warning on coordinator.log - Not enough [_default_tier] servers or node capacity to assign错误
进入druid cluster（8081）UI界面，点击datasources的default rules后的edit按钮，将_default_tier修改为1
```
## mongo运维

### 常用命令
1. 登录
`mongo admin -u apptrace -p APMmongo8$ --port 17017`
2. mongotop和mongostat
```
mongostat --port 27001
mongotop --port 27001
mongostat -u apptrace -p APMmongo8$ --port 17017 --authenticationDatabase admin
mongotop -u apptrace -p APMmongo8$ --port 17017 --authenticationDatabase admin
```
3. mongo清空日志
`true | sudo tee  shard.log`

### mongo加密失败
1. 修改Mongo配置取消安全验证
```
vi /etc/mongod.conf
注释
#security:
#  authorization: enabled
sudo systemctl stop mongod
sudo systemctl start mongod
```
2. 创建用户
```
mongo admin --port 17017 --eval "db.setProfilingLevel(1, { slowms:\" 100000\" })"
mongo admin --port --eval "db.createUser({user:\"admin\",customData:{description:\"superuser\"},pwd:\"APMadmin8$\",roles:[{role:\"userAdminAnyDatabase\",db:\"admin\"}]})"
mongo admin --port --eval "db.createUser({user:\"apptrace\",pwd:\"APMmongo8$\",roles:[\"root\"]})"
```
3. 修改配置增加安全验证
```
vi /etc/mongod.conf
取消注释
security:
  authorization: enabled
sudo systemctl stop mongod
sudo systemctl start mongod
```

### 索引
```
src/parts/mgmt/createMongoIndex.js 
```
### sharding
```
sh.shardCollection('lingcloudapm5b9717486443175005d658cf.app_calltreeraws', { trxid: 'hashed' });
sh.shardCollection('lingcloudapm5b9717486443175005d658cf.app_httpreqs', { shid: 'hashed' });
sh.shardCollection('lingcloudapm5b9717486443175005d658cf.app_sqlraws', { trxid: 'hashed' });
```

## 查看open files
1. 查看系统open file参数
`cat /etc/security/limits.conf` 
```
*  soft  nofile  640000
*  hard  nofile  640000
*  soft  nproc  32000
*  hard  nproc  32000
```
2. 查看进程的open file
```
ps -ef | grep kafka
cat /proc/15601/limits
```
3. 修改服务的启动参数  
`vi /usr/lib/systemd/system/kafka.service`  
在[service]后添加如下内容  
```
# file size
LimitFSIZE=infinity
# cpu time
LimitCPU=infinity
# virtual memory size
LimitAS=infinity
# open files
LimitNOFILE=64000
# processes/threads
LimitNPROC=64000
# locked memory
LimitMEMLOCK=infinity
```
重新加载并重启服务
```
sudo systemctl daemon-reload
sudo systemctl stop kafka
sudo systemctl start kafka
```

## Agent和Collector所在机器握手失败
有些客户为了防止攻击，开启了arp防火墙，导致agent与collector所在机器建立握手失败，利用` netstat -an | grep 9994`命令可以看到有些监听状态为SYN-REC。  
先用tcpdump抓包看是不是arp没有回
```
tcpdump -i <eth name> arp and host <agent IP>
```
然后看arp表，agent对应的IP是不是incomplete
```
arp -n
```
最后手动添加arp记录。因为我们这边的表现是有syn消息过来，所以可以从syn里提取到agent机器的MAC地址，命令是
```
tcpdump -i <eth name>  port 9994 -e -nn
```
找到IP对应的MAC地址，然后再用下面的命令添加
```
arp -s <agent IP> <MAC address>
```
