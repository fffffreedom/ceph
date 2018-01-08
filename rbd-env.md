# rbd客户端初始化（rbd map)
```
####################
# ceph-install.sh
####################
#!/bin/sh

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

yum install -y ceph-common

####################
# 在deploy节点上 
####################
# 拷贝配置文件到client
# 如：
ceph-deploy --overwrite-conf admin \
    ceph-client-10-101-17-77

####################
# user and keyring 
####################

# 在client上，删除已有的keyring
rm /etc/ceph/ceph.client.admin.keyring

# create user and keyring
# username 替换成自己的名字
ceph auth get-or-create client.username mon 'allow r' osd 'allow rwx pool=rbd' > ceph.client.username.keyring

# copy keyring to client
scp ceph.client.username.keyring root@ip:/etc/ceph/

# 创建image
rbd create rbd/test --name client.username

#!/bin/sh

# image命名规则
# vol-[username首字母]-[标识字符串]

IMAGE=rbd/vol-xxx-$1

rbd map $IMAGE
mkfs -t xfs /dev/rbd0

mkdir /data

echo "/dev/rbd/$IMAGE /data xfs noauto,_netdev 0 0" >> /etc/fstab
echo "$IMAGE id=admin,keyring=/etc/ceph/ceph.client.admin.keyring" >> /etc/ceph/rbdmap

systemctl enable rbdmap.service
systemctl start rbdmap.service
systemctl status rbdmap.service
```
