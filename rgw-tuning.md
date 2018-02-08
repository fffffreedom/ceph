# rgw-tuning

Ceph Rados Gateway Tuning  
http://www.osris.org/performance/rgw.html  

调用下面的参数，并没有什么用。  
```
[client.rgw.ceph-node-10-101-4-21]
rgw_dns_name = storage.s3.lan
rgw_num_rados_handles = 8
rgw_frontends = civetweb port=80 num_threads=512
rgw thread pool size = 1024
```

源码走读rgw内置civetweb的参数初始化过程  
https://cloud.tencent.com/developer/article/1032846  
