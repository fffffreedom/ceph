## crush修改过程
- 提取已有的CRUSH map，使用-o参数，ceph将输出一个经过编译的CRUSH map到您指定的文件  
`ceph osd getcrushmap -o crushmap.txt`
- 反编译你的CRUSH map，使用-d参数将反编译CRUSH map到通过-o指定的文件中  
`crushtool -d crushmap.txt -o crushmap-decompile`
- 使用编辑器编辑CRUSH map  
`vim crushmap-decompile`
- 重新编译这个新的CRUSH map  
`crushtool -c crushmap-decompile -o crushmap-compiled`
- 将新的CRUSH map应用到ceph集群中  
`ceph osd setcrushmap -i crushmap-compiled`
- 修改ceph.conf文件并推送到各节点  
修改如下：  
osd crush update on start = false  
该配置设置为false，表明完全手动管理CRUSH Map  
## 编辑crush map
使用`ceph-deploy osd create`创建出来的ssd osd，默认的crush map为：
```
host ceph-node-10-101-5-14 {
        id -3           # do not change unnecessarily
        # weight 33.149
        alg straw
        hash 0  # rjenkins1
        item osd.9 weight 3.637
        item osd.10 weight 3.637
        item osd.11 weight 3.637
        item osd.12 weight 3.637
        item osd.13 weight 3.637
        item osd.14 weight 3.637
        item osd.15 weight 3.637
        item osd.16 weight 3.637
        item osd.17 weight 3.637
        #item osd.46 weight 0.417 <- 需要注释掉，并抽出来成为host，如下
}
root default {
        id -1           # do not change unnecessarily
        # weight 165.744
        alg straw
        hash 0  # rjenkins1
        item ceph-node-10-101-5-13 weight 33.149 -> 32.733
        item ceph-node-10-101-5-14 weight 33.149 -> 32.733
        item ceph-node-10-101-5-15 weight 33.149 -> 32.733
        item ceph-node-10-101-5-16 weight 33.149 -> 32.733
        item ceph-node-10-101-5-17 weight 33.149 -> 32.733
}

```
添加下面的代码：  
```
host ceph-node-10-101-5-13-ssd {
	id -1006
	# weight 0.417
	alg straw
	hash 0	# rjenkins1
	item osd.45 weight 0.417
}
host ceph-node-10-101-5-14-ssd {
	id -1007
	# weight 0.417
	alg straw
	hash 0	# rjenkins1
	item osd.46 weight 0.417
}
host ceph-node-10-101-5-15-ssd {
	id -1008
	# weight 0.417
	alg straw
	hash 0	# rjenkins1
	item osd.47 weight 0.417
}
host ceph-node-10-101-5-16-ssd {
	id -1009
	# weight 0.417
	alg straw
	hash 0	# rjenkins1
	item osd.48 weight 0.417
}
host ceph-node-10-101-5-17-ssd {
	id -1010
	# weight 0.417
	alg straw
	hash 0	# rjenkins1
	item osd.49 weight 0.417
}

root ssd {
	id -1002
	# weight 2.085
	alg straw
	hash 0
	item ceph-node-10-101-5-13-ssd weight 0.417
	item ceph-node-10-101-5-14-ssd weight 0.417
	item ceph-node-10-101-5-15-ssd weight 0.417
	item ceph-node-10-101-5-16-ssd weight 0.417
	item ceph-node-10-101-5-17-ssd weight 0.417
}

rule ssd-pool {
	ruleset 1
	type replicated
	min_size 1
	max_size 10
	step take ssd
	step chooseleaf firstn 0 type host
	step emit
}
```
## 验证crush修改是否生效
创建default.rgw.buckets.index，指定其使用ssd pool：
```
ceph osd pool create default.rgw.buckets.index 256 256
ceph osd pool set default.rgw.buckets.index crush_ruleset 1
```
创建好rgw后，创建s3用户，用s3cmd创建一个bucket：
```
s3cmd mb s3://test
```
查看pool中的文件：
```
rados -p default.rgw.buckets.index ls
.dir.c6d58823-ca7a-4728-8d81-81871591eddd.4367.1.15
......
```
然后再查看使用的osd，可见其使用的都是ssd硬盘（osd.45/46/47/48/49）：  
```
# ceph osd map default.rgw.buckets.index .dir.c6d58823-ca7a-4728-8d81-81871591eddd.4367.1.15  
osdmap e259 pool 'default.rgw.buckets.index' (7) object   
'.dir.c6d58823-ca7a-4728-8d81-81871591eddd.4367.1.15'   
-> pg 7.a5139605 (7.5) -> up ([48,47,45], p48) acting ([48,47,45], p48)
```
