# docker-ceph-rbd
## readme
https://github.com/yp-engineering/rbd-docker-plugin 
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
然后运行make，编译成功后，会在dist目录下生成文件rbd-docker-plugin
# cp dist/rbd-docker-plugin /root/go/bin

4. 进程启动方式
可使用手动启动：
rbd-docker-plugin --create --remove delete &
或者配置serivce:
配置文件可在github上下载
https://github.com/yp-engineering/rbd-docker-plugin/blob/21b9c0080675e0f770f1079e139fedb099f2e40d/etc/systemd/rbd-docker-plugin.service
```
## 测试
rbd-docker-plugin的参数说明：
```
Usage of rbd-docker-plugin:
  -cluster string
    	Ceph cluster
  -config string
    	Ceph cluster config (default "/etc/ceph/ceph.conf")
  -create
    	Can auto Create RBD Images
  -debug
    	Debug output
  -fs string
    	FS type for the created RBD Image (must have mkfs.type) (default "xfs")
  -logdir string
    	Logfile directory (default "/var/log")
  -mount string
    	Mount directory for volumes on host (default "/var/lib/docker-volumes")
  -name string
    	Docker plugin name for use on --volume-driver option (default "rbd")
  -plugins string
    	Docker plugin directory for socket (default "/run/docker/plugins")
  -pool string
    	Default Ceph Pool for RBD operations (default "rbd")
  -remove value
    	Action to take on Remove: ignore, delete or rename (default ignore)
  -size int
    	RBD Image size to Create (in MB) (default: 20480=20GB) (default 20480)
  -user string
    	Ceph user (default "admin")
  -version
    	Print versio
```
```
You can control the RBD Pool and initial Size using this syntax sugar:
  foo@1024 => pool=rbd (default), image=foo, size 1GB
  deep/foo => pool=deep, image=foo and default --size (20GB)
  deep/foo@1024 => pool=deep, image=foo, size 1GB
  pool must already exist
```
### 启动rbd-docker-plugin
命令行方式：
```
1. rbd-docker-plugin &
  此方式不会自动创建rbd image
2. rbd-docker-plugin --create &
  此方式会自动创建rbd image
3. rbd-docker-plugin --create --remove delete &
  此方式会自动创建rbd image，并且在容器退出时，会根据情况，是否删除volume
4. rbd-docker-plugin --create --remove rename &
  此方式会自动创建rbd image，并且在容器退出时，不会删除volume，但会根据情况，在volume前面添加字符串：zz_
  (docker volume rm foo，会使得volume重命名为：zz_foo)
```
### 启动容器
docker run --rm --volume-driver=rbd -v imagename:/mnt/dir IMAGE [CMD]
docker run命令中，如果指定了imagename，则容器删除后，rbd image不会被删除，
如果不指定，则在容器退出时，会自动删除rbd image.
```
# image会自动删除
docker run --rm --volume-driver=rbd --volume /mnt -it busybox /bin/sh
# image不会自动删除
docker run --rm --volume-driver=rbd --volume foo:/mnt -it busybox /bin/sh
docker run --rm --volume-driver=rbd --volume foo@1024:/mnt -it busybox /bin/sh
```
## 问题
```
1. 确保创建rbd image的默认feature为1（rbd_default_features = 1），否则会因为功能不支持而出错；
2. 注意volume在容器退出后，自动删除的条件；
3. 经常会碰到，暂时还不清楚原因：
2018/01/17 10:54:22 api.go:174: Entering go-plugins-helpers hostVirtualPath
2018/01/17 10:54:22 driver.go:507: INFO: API Path request(foo) => /var/lib/docker-volumes/rbd/rbd/foo
2018/01/17 10:54:22 api.go:160: Entering go-plugins-helpers mountPath
2018/01/17 10:54:22 driver.go:309: INFO: API Mount(&{foo@1024 940ca729c9d6b64e659aa864cd4d2d5fa10187c7652d1520adee041cb39fe530})
2018/01/17 10:54:22 driver.go:325: ERROR: locking RBD Image(foo): exit status 2
2018/01/17 10:54:22 api.go:202: Entering go-plugins-helpers unmountPath
2018/01/17 10:54:22 driver.go:525: INFO: API Unmount(&{foo@1024 940ca729c9d6b64e659aa864cd4d2d5fa10187c7652d1520adee041cb39fe530})
2018/01/17 10:54:22 driver.go:545: WARN: Volume is not in known mounts: ignoring request to unmount: rbd/foo
```
## reference
http://www.sebastien-han.fr/blog/2015/08/17/getting-started-with-the-docker-rbd-volume-plugin/  
http://blog.opskumu.com/rbd-docker-plugin.html  
