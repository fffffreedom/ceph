# install exapmle

```
###每台主机执行：
modprobe 8021q
vconfig add enp7s0f0 19
vconfig add enp7s0f1 21
ip a add 172.19.1.28/24 dev enp7s0f0.19
ip a add 172.20.1.28/24 dev enp7s0f1.21
ip l s enp7s0f0.19 up
ip l s enp7s0f1.21 up
ip l s enp7s0f0.1803 up
ip r add default via 10.202.83.126 dev enp7s0f0.1803
cd /etc/sysconfig/network-scripts
cp ifcfg-enp7s0f0 ifcfg-enp7s0f0.19
cp ifcfg-enp7s0f1 ifcfg-enp7s0f1.21
cp ifcfg-enp7s0f0 ifcfg-enp7s0f0.1803

[root@Controller-1 ~]# cat  /proc/net/vlan/config
VLAN Dev name	 | VLAN ID
Name-Type: VLAN_NAME_TYPE_RAW_PLUS_VID_NO_PAD
eth5.2         | 2  | eth5
eth4.2         | 2  | eth4


echo "IPADDR=172.19.1." >> /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.19
echo "IPADDR=172.20.1." >> /etc/sysconfig/network-scripts/ifcfg-enp7s0f1.21
echo "IPADDR=xxxxx" >> /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.1803
echo "NETMASK=255.255.255.0" >> /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.19
echo "NETMASK=255.255.255.0" >> /etc/sysconfig/network-scripts/ifcfg-enp7s0f1.21
echo "NETMASK=255.255.255.0" >> /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.1803

sed -i 's/ONBOOT=no/ONBOOT=yes/g' /etc/sysconfig/network-scripts/ifcfg-*
sed -i 's/BOOTPROTO=dhcp/BOOTPROTO=static/g' /etc/sysconfig/network-scripts/ifcfg-*
sed -i 's/NAME=enp7s0f0/NAME=enp7s0f0.19/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.19
sed -i 's/NAME=enp7s0f0/NAME=enp7s0f0.1803/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.1803
sed -i 's/NAME=enp7s0f1/NAME=enp7s0f1.21/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f1.21
sed -i 's/DEVICE=enp7s0f0/DEVICE=enp7s0f0.19/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.19
sed -i 's/DEVICE=enp7s0f0/DEVICE=enp7s0f0.1803/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.1803
sed -i 's/DEVICE=enp7s0f1/DEVICE=enp7s0f1.21/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f1.21
sed -i 's/UUID/\#UUID/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.19
sed -i 's/UUID/\#UUID/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f1.21
sed -i 's/UUID/\#UUID/g' /etc/sysconfig/network-scripts/ifcfg-enp7s0f0.1803

#将VLAN信息写入配置文件(/etc/rc.local)中，下次随机启动。
echo "modprobe 8021q">> /etc/rc.local 
echo "VLAN=yes" >> /etc/sysconfig/network
echo "vconfig add enp7s0f0 19" >>/etc/rc.local
echo "vconfig add enp7s0f1 21" >>/etc/rc.local
echo "vconfig add enp7s0f0 1803" >>/etc/rc.local
echo "ip a add 172.19.1.28/24 dev enp7s0f0.19" >>/etc/rc.local
echo "ip a add 172.20.1.28/24 dev enp7s0f1.21" >>/etc/rc.local
echo "ip a add 10.202.83.74/24 dev enp7s0f0.1803" >>/etc/rc.local
echo "ip l s enp7s0f0.19 up" >>/etc/rc.local
echo "ip l s enp7s0f1.21 up" >>/etc/rc.local
echo "ip l s enp7s0f0.1803 up" >>/etc/rc.local
echo "ip r add default via 10.202.83.126 dev enp7s0f0.1803" >>/etc/rc.local


###每台主机执行：
for i in {b..o};do parted -s /dev/sd$i mklabel gpt;done
for i in {b..m};do parted -s /dev/sd$i mkpart primary 2048s 100%;done
parted -s /dev/sdn mkpart primary 2048s 10G
parted -s /dev/sdn mkpart primary 10G 20G
parted -s /dev/sdn mkpart primary 20G 30G
parted -s /dev/sdn mkpart primary 30G 40G
parted -s /dev/sdn mkpart primary 40G 50G
parted -s /dev/sdn mkpart primary 50G 60G
parted -s /dev/sdo mkpart primary 2048s 10G
parted -s /dev/sdo mkpart primary 10G 20G
parted -s /dev/sdo mkpart primary 20G 30G
parted -s /dev/sdo mkpart primary 30G 40G
parted -s /dev/sdo mkpart primary 40G 50G
parted -s /dev/sdo mkpart primary 50G 60G

for i in {b..m};do sgdisk --typecode=1:4fbd7e29-9d25-41b8-afd0-062c0ceff05d  /dev/sd$i;done
for i in {1..6};do sgdisk --typecode=$i:45B0969E-9B03-4F30-B4C6-B4B80CEFF106  /dev/sdn;done
for i in {1..6};do sgdisk --typecode=$i:45B0969E-9B03-4F30-B4C6-B4B80CEFF106  /dev/sdo;done
echo noop > /sys/block/sdn/queue/scheduler
echo noop > /sys/block/sdo/queue/scheduler

###每台执行：
sed -i s'/SELINUX=enforcing/SELINUX=disabled'/g /etc/sysconfig/selinux
systemctl disable firewalld&systemctl stop firewalld
rpm -Uvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-6.noarch.rpm
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
rpm -Uvh http://download.ceph.com/rpm-hammer/el7/noarch/ceph-release-1-1.el7.noarch.rpm
rpm --import http://download.ceph.com/keys/release.asc
sed -i 's/gpgcheck=1/gpgcheck=0/g' /etc/yum.repos.d/*.repo

yum -y install yum-plugin-priorities
yum install -y redhat-lsb-core
yum -y install ceph-deploy ceph-common ceph ceph-radosgw


###Mon执行：
ceph osd crush add-bucket default root（已有）

ceph osd crush add-bucket Node-1 host
ceph osd crush move Node-1 root=default
ceph osd crush add-bucket Node-2 host
ceph osd crush move Node-2 root=default


ceph osd getcrushmap -o cm
crushtool -d cm -o cm.txt
vi cm.txt
   rule replicated_ruleset {
        ruleset 1
        type replicated
        min_size 1
        max_size 10
        step take default
        step chooseleaf firstn 0 type host
        step emit
	}

crushtool -c cm.txt -o cmm
ceph osd setcrushmap -i cmm



for i in {1..5};do ceph-deploy admin Node-$1;done

ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdb1:/dev/sdn1
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdb1:/dev/sdn1
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdc1:/dev/sdn2
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdc1:/dev/sdn2
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdd1:/dev/sdn3
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdd1:/dev/sdn3
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sde1:/dev/sdn4
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sde1:/dev/sdn4
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdf1:/dev/sdn5
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdf1:/dev/sdn5
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdg1:/dev/sdn6
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdg1:/dev/sdn6

ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdh1:/dev/sdo1
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdh1:/dev/sdo1
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdi1:/dev/sdo2
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdi1:/dev/sdo2
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdj1:/dev/sdo3
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdj1:/dev/sdo3
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdk1:/dev/sdo4
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdk1:/dev/sdo4
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdl1:/dev/sdo5
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdl1:/dev/sdo5
ceph-deploy --overwrite-conf osd prepare  Node-1:/dev/sdm1:/dev/sdo6
ceph-deploy --overwrite-conf osd activate Node-1:/dev/sdm1:/dev/sdo6
###以上重复6台节点，注意主机名大小写敏感


ceph osd pool create <poolname> 1024 1024 replicated <crush_ruleset>
--[root@Node-1 ~]# ceph osd pool create test 1024 1024 replicated Node-

ceph osd pool set <poolname> crush_ruleset 1





#######利用ceph-deploy创建rgw实例：
#######ceph-deploy install --rgw <gateway-node1> [<gateway-node2> ...]
#######ceph-deploy admin <node-name>
#######ceph-deploy rgw create <gateway-node1>
#######同步配置文件：ceph-deploy --overwrite-conf config push 主机名 ; ceph-deploy --overwrite-conf admin 主机名
#######



ceph auth get-or-create client.radosgw.gateway osd 'allow rwx' mon 'allow rwx' -o /etc/ceph/ceph.client.radosgw.keyring 

#创建密钥环
ceph-authtool --create-keyring /etc/ceph/ceph.client.radosgw.keyring
chmod +r /etc/ceph/ceph.client.radosgw.keyring
#为各例程生成 Ceph 对象网关用户名及其密钥
ceph-authtool /etc/ceph/ceph.client.radosgw.keyring -n client.radosgw.xxx-new --gen-key
#给各密钥增加能力，允许到监视器的写权限、允许创建存储池
ceph-authtool -n client.radosgw.xxx-new --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.client.radosgw.keyring
#创建密钥环及密钥，并授权 Ceph 对象网关访问 Ceph 存储集群之后，还需把各密钥导入存储集群
ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.radosgw.xxx-new -i /etc/ceph/ceph.client.radosgw.keyring

#在radosgw节点：
mkdir -p /var/lib/ceph/radosgw/ceph-radosgw.xxx-new


#vi ceph.conf
[client.radosgw.xxx-new]
host = Node-1
rgw region = xxx
rgw region root pool = .xxx.rgw.root
rgw zone = xxx-new
rgw zone root pool = .xxx-new.rgw.root
keyring = /etc/ceph/ceph.client.radosgw.keyring
rgw frontends = "civetweb port=80"

ceph-deploy --overwrite-conf config push {node1} {node2} {nodex}



【在各节点上安装ceph 客户端】
在 glance-api 节点，安装 librbd
sudo yum install python-rbd

在 nova-compute 和 cinder-volume 节点安装 ceph-common：
sudo yum install ceph-common


ceph osd pool create vms 2048 2048 replicated 

ceph osd pool create images 2048 2048 replicated 

ceph osd pool create volumes 2048 2048 replicated



```
