# 添加osd节点的步骤
## 步骤
- 配置环境（hostname设置，软件安装，调优配置等）
- 推送配置到新OSD（ceph-deploy --overwrite-conf admin HOSTNAME）
- 添加硬盘（ceph-deploy disk zap; ceph-deploy osd create）
- 更新crush map信息
```
ceph osd crush add-bucket HOSTNAME host
ceph osd crush move HOSTNAME root=default
ceph osd crush add osd.X 1.0 host= HOSTNAME
```
- reboot新OSD，查看是否可正常加入集群（可略）  

## 添加优化（未尝试）
在上面更新crush map信息之前，运行下面的命令：  
```
ceph osd set nobackfill && ceph osd set norebalance && ceph osd set norecover && ceph osd set noout \
&& ceph tell osd.* injectargs  --osd_op_thread_suicide_timeout 300 --osd_op_thread_timeout 120 \
--osd_heartbeat_interval 60 --osd_heartbeat_grace 180 --osd_max_backfills 1 --osd_recovery_max_active 1
```

然后iostat -x -m 1看看磁盘负载 ssd journal 能不能撑得住。ceph perf高不高  

不高可以慢慢提高 osd_max_backfills osd_recovery_max_active的数值  

之后再unset nobackfill norebalance norecover  
