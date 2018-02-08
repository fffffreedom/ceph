# rgw-tuning

Ceph配置参数详解  
http://www.yangguanjun.com/2017/05/15/Ceph-configuration/  

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


Ceph on Storage appliance Case Study and Performance for AWS S3 based object storage  
http://7xweck.com1.z0.glb.clouddn.com/cephdaybeijing201608/09-Ceph%E5%AD%98%E5%82%A8%E8%AE%BE%E5%A4%87%E6%A1%88%E4%BE%8B%E7%A0%94%E7%A9%B6%E4%B8%8ES3%E5%AF%B9%E8%B1%A1%E5%AD%98%E5%82%A8%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96-%E5%88%98%E5%BF%97%E5%88%9A.pdf


https://indico.cern.ch/event/613466/contributions/2473060/attachments/1422878/2181334/USATLAS-Ceph-Lightning-Talk.pdf  
rgw_thread_pool_size defaults to 100; try 400  
rgw_num_rados_handles defaults to 1; try 8  


