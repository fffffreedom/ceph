# 监控搭建说明

```
prometheus
alertmanager
ceph_exporter
grafana
```

## 文件说明
```
alertmanager/config.yml
	告警配置文件，自己测试使用，使用自已的邮件来发送告警
alertmanager/config-对接告警中心.yml
	对接公司告警中心的配置，不需要配置global
alert.rules
	告警配置规则
ceph-alarm.py
	对接线上告警中心脚本
ceph-cluster.json
	ceph集群监控的grafana界面配置
docker-compose.yml
	docker up -d
host-node-ceph.json
	ceph物理机grafana界面配置
prometheus.yml
	prometheus配置文件
```

## 集群监控步骤
```
0. 安装依赖包，拉取相应的镜像
docker

docker pull prom/alertmanager:v0.14.0
docker pull prom/prometheus:v1.7.1
docker pull digitalocean/ceph_exporter:1.1.0
docker pull grafana/grafana:4.5.2
```

1. 把上面的文件拷贝到prometheus目录下，并创建grafana_data和prometheus_data：  
```
tree -L 2
.
├── alertmanager
│   └── config.yml
├── alert.rules
├── docker-compose.yml
├── grafana_data
├── prometheus_data
└── prometheus.yml
```

2. 根据集群情况，修改prometheus.yml文件，ceph-exporter配置指定的ip主机上，需要有ceph集群的配置和keyring（/etc/ceph/）:    
```
# prometheus.yml

# Place in same directory as docker-compose.yml and replace $DOCKERHOST with your desired host IP where ceph_exporter is running

global:
    scrape_interval: 5s
    external_labels:
        monitor: ceph-monitor
rule_files:
- "alert.rules"
scrape_configs:
    - job_name: prometheus
      static_configs:
          - targets: ['localhost:9090']
    - job_name: ceph-exporter
      static_configs:
          - targets: ['ceph-exporter-ip:9128']
    - job_name: ceph-node
      static_configs:
          - targets: ['telegrah-node-ip:9273']
```
 
3. 配置好docker-compose.yml文件，docker-compose up -d

4. 登录grafana界面（ip:3000），这里的ip为ceph-exporter配置的ip，admin/test

5. 配置Data Sources
```
Edit data source

config Dashboards
Name: xxx
Type: Prometheus <-- 选择prometheus

Http settings
Url: http://grafana-node-ip:9090
Access: proxy
```
 
6. 创建Dashboards，并导入ceph-cluster.json配置模板。

## osd监控步骤

1. 安装telegraf-ceph-xxx.x86_64.rpm  

2. 打开/etc/telegraf/telegraf.conf配置文件中配置项  
```
# # Monitor process cpu and memory usage
 [[inputs.procstat]]
##   ## Must specify one of: pid_file, exe, or pattern
##   ## PID file to monitor process
##   pid_file = "/var/run/nginx.pid"
##   ## executable name (ie, pgrep <exe>)
exe = "ceph-osd"
```

3. systemctl restart telegraf  

4. 重新配置prometheus  
添加ceph-node配置，添加三个osd节点的ip  
```
scrape_configs:
    ......
    - job_name: ceph-node
      static_configs:
          - targets: ['telegrah-node-ip:9273']
```

5. docker restart prometheus-container  

6. 查看采集到的数据  
```
http://xxx:9090/targets  
http://xxx:9273/metrics  
```

7. 展示到grafana  
创建新的Dashboard，导入配置模板host-node-ceph.json即可。  

【注意】需要根据node修改配置模板，即把ip改成osd的ip。

## 对接告警中心

0. 安装所需RPM包  
```
yum -y install epel-release
yum -y install python-pip
pip install Flask
```

1. 在公司的告警中心，申请账号，并创建告警事件；  

2. 使用alertmanager/config-对接告警中心.yml做为alertmanager的配置，重启alertmanager容器；  

3. 在监控机器运行如下命令，启动告警转发（公司告警中心的信息格式和alertmanager的告警格式不一样）：  
```
nohup python -u ceph-alarm.py > alarm.log 2>&1 &  
```

