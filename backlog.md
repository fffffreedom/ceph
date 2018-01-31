有高并发的业务，就必须要调整backlog  
http://www.cnblogs.com/higkoo/articles/php-fpm_backlog_setting.html  
操作系统以CentOS为例，可通过默认配置 /etc/sysctl.conf 文件进行调整。比如：  
```
net.core.somaxconn = 1048576 # 默认为128
net.core.netdev_max_backlog = 1048576 # 默认为1000
net.ipv4.tcp_max_syn_backlog = 1048576 # 默认为1024
```

Linux之TCPIP内核参数优化  
http://www.cnblogs.com/fczjuever/archive/2013/04/17/3026694.html  

/proc/sys/net/core/somaxconn  
定义了系统中每一个端口最大的监听队列的长度，这是个全局的参数。  

