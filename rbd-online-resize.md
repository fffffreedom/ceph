# rbd在线扩容
> http://www.cnblogs.com/tonychiu/p/5766548.html
```
rbd showmapped
rbd info rbd/test
df -Th
rbd resize -s 2048 rbd/test
df -Th
blockdev --getsize64 /dev/rbd0
# for ext4
resize2fs /dev/rbd0
# for xfs
xfs_growfs /data
# finally check it
df -Th
```
