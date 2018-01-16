# docker-ceph-rbd
## install software
```
1. 配置yum源
cat << EOF > /etc/yum.repos.d/ceph.repo
[ceph]
name=ceph
baseurl=http://mirrors.aliyun.com/ceph/rpm-jewel/el7/x86_64/
gpgcheck=0
[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.aliyun.com/ceph/rpm-jewel/el7/noarch/
gpgcheck=0
EOF

2. 安装相关软件
yum install -y ceph-common librados2-devel librbd1-devel xfsprogs
```
## rbd-docker-plugin
```
1. 下载源码
cd $GOPATH/src/
git clone https://github.com/yp-engineering/rbd-docker-plugin.git

2. 配置路径
PATH=$PATH:$HOME/bin:/usr/local/go/bin/:/root/go/bin
source ~/.bash_profile

3. 编译
由于被墙，注释掉Gopkg.lock文件中的如下行（自己下载后放到指定路径）：
#[[projects]]
#  branch = "master"
#  name = "golang.org/x/net"
#  packages = ["proxy"]
#  revision = "57efc9c3d9f91fb3277f8da1cff370539c4d3dc5"
#
#[[projects]]
#  branch = "master"
#  name = "golang.org/x/sys"
#  packages = ["windows"]
#  revision = "2d6f6f883a06fc0d5f4b14a81e4c28705ea64c15"
然后运行make，编译成功后，会在dist目录下生成文件：
# file dist/rbd-docker-plugin 
dist/rbd-docker-plugin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
# cp dist/rbd-docker-plugin /root/go/bin
```
## 测试
