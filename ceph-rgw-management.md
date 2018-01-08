# ceph对象存储之管理手册
> http://docs.ceph.org.cn/radosgw/admin/
## 用户管理
有两种类型的用户：
- 用户: ‘用户’ 这个表示这个是使用 S3 接口的一个用户.
- 子用户: ‘子用户’ 这个词表示使用的是 Swift 接口的一个用户. 一个子用户 是和一个用户关联的
### 新建一个用户
```
radosgw-admin user create --uid={username} --display-name="{display-name}" [--email={email}]
eg:
radosgw-admin user create --uid=johndoe --display-name="John Doe" --email=john@example.com
```
> 有时 radosgw-admin 会生成一个 JSON转义字 符(\),但是有些客户端不道如何处理JSON 转义字符!
### 新建一个子用户
```
radosgw-admin subuser create --uid={uid} --subuser={uid} --access=[ read | write | readwrite | full ]
eg:
radosgw-admin subuser create --uid=johndoe --subuser=johndoe:swift --access=full
```
### 获取用户信息
`radosgw-admin user info --uid=johndoe`
### 修改用户信息
```
eg:
radosgw-admin user modify --uid=johndoe --display-name="John E. Doe"
radosgw-admin subuser modify --uid=johndoe:swift --access=full
```
### 用户 启用/停用
```
radosgw-admin user suspend --uid=johndoe
radosgw-admin user enable --uid=johndoe
```
>  停用一个用户后，它的子用户也会一起被停用.
### 删除用户
```
radosgw-admin user rm --uid=johndoe
# del subuser only
radosgw-admin subuser rm --subuser=johndoe:swift
```
其它可选操作：
- Purge Data: 加 --purge-data 选项可清除与此 UID 相关的所有数据。
- Purge Keys: 加 --purge-keys 选项可清除与此 UID 相关的所有密钥。
### 新建一个密钥
要为用户新建一个密钥，你需要使用 key create 子命令。  
对于用户来说，需要指明用户的 ID 以及新建的密钥类型为 s3。  
要为子用户新建一个密钥，则需要指明子用户的 ID以及密钥类型为 swift。  
`radosgw-admin key create --subuser=johndoe:swift --key-type=swift --gen-secret`  
### 新建/删除 ACCESS 密钥
用户和子用户要能使用 S3 和Swift 接口，必须有 access 密钥。  
在你新建用户或者子用户的时候，如果没有指明 access 和 secret 密钥，这两 个密钥会自动生成。    
你可能需要新建 access 和/或 secret 密钥，不管是 手动指定还是自动生成的方式。你也可能需要删除一个 access 和 secret。  
可用的选项有：
- --secret=<key> 指明一个 secret 密钥 (e.即手动生成).
- --gen-access-key 生成一个随机的 access 密钥 (新建 S3 用户的默认选项).
- --gen-secret 生成一个随机的 secret 密钥.
- --key-type=<type> 指定密钥类型. 这个选项的值可以是: swift, s3  
要新建密钥，需要指明用户 ID:  
`radosgw-admin key create --uid=johndoe --key-type=s3 --gen-access-key --gen-secret`  
要删除一个 access 密钥, 也需要指定用户 ID:  
`radosgw-admin key rm --uid=johndoe`  
### 添加/删除 管理权限
Ceph 存储集群提供了一个管理API，它允许用户通过 REST API 执行管理功能。  
默认情况下，用户没有访问 这个 API 的权限。要启用用户的管理功能，需要为用 户提供管理权限。
执行下面的命令为一个用户添加管理权限:  
`radosgw-admin caps add --uid={uid} --caps={caps}`  
你可以给一个用户添加对用户、bucket、元数据和用量(存储使用信息)等数据的 读、写或者所有权限。举例如下:  
`--caps="[users|buckets|metadata|usage|zone]=[*|read|write|read, write]"`  
```
eg:
radosgw-admin caps add --uid=johndoe --caps="users=*"
radosgw-admin caps rm --uid=johndoe --caps={caps}
```
## 配额管理
Ceph对象网关允许你在用户级别、用户拥有的 bucket 级别设置配额。  
配额包括一个 bucket 内允许的最大对象数和最大存储容量，大小单位 是兆字节。  
- Bucket: 选项 --bucket 允许你为用户的某一个 bucket 设置配额。
- Maximum Objects: 选项 --max-objects 允许 你设置最大对象数。负数表示不启用这个设置。
- Maximum Size: 选项 --max-size 允许你设置一个 最大存储用量的配额。负数表示不启用这个设置。
- Quota Scope: 选项 --quota-scope 表示这个配额 生效的范围。这个参数的值是 bucket 和 user.  
  Bucket 配额作用于用户的某一个 bucket。而用户配额作用于一个用户。  
### 设置用户配额
```
radosgw-admin quota set --quota-scope=user --uid=<uid> [--max-objects=<num objects>] [--max-size=<max size>]
eg:
radosgw-admin quota set --quota-scope=user --uid=johndoe --max-objects=1024 --max-size=1024
```
最大对象数和最大存储用量的值是负数则表示不启用指定的配额参数。
### 启用/禁用用户配额
```
radosgw-admin quota enable --quota-scope=user --uid=<uid>
radosgw-admin quota disable --quota-scope=user --uid=<uid>
```
### 获取配额信息
`radosgw-admin user info --uid=<uid>`
### 更新配额统计信息
配额的统计数据的同步是异步的。你也可以通过手动获取最新的配额统计数据为所有用户和所有 bucket 更新配额统计数据：  
`radosgw-admin user stats --uid=<uid> --sync-stats`  
### 获取用户用量统计信息
执行下面的命令获取当前用户已经消耗了配额的多少:  
`radosgw-admin user stats --uid=<uid>`  
> 你应该在执行 radosgw-admin user stats 的时候带上 --sync-stats 参数来获取最新的数据.  
### 读取/设置全局配额
你可以在 region map中读取和设置配额。执行下面的命 令来获取 region map:  
`radosgw-admin regionmap get > regionmap.json`  
要为整个 region 设置配额，只需要简单的修改 region map 中的配额设置。然后使用 region set 来更新 region map即可:  
`radosgw-admin region set < regionmap.json`  
> 在更新 region map 后，你必须重启网关!
## 用量
Ceph 对象网关会为每一个用户记录用量数据。你也可以通过指定 日期范围来跟踪用户的用量数据。  
可用选项如下:  
- Start Date: 选项 --start-date 允许你指定一个起始日期来过滤用量数据 (format: yyyy-mm-dd[HH:MM:SS]).
- End Date: 选项 --end-date 允许你指定一个截止日期来过滤用量数据 (format: yyyy-mm-dd[HH:MM:SS]).
- Log Entries: 选项 --show-log-entries 允许你 指明显示用量数据的时候是否要包含日志条目。 (选项值: true | false).  
>  你可以指定时间为分钟和秒，但是数据存储是以一个小时的间隔存储的。  
### 展示用量信息
显示用量统计数据，使用 usage show 子命令。显示某一个特定 用户的用量数据，你必须指定该用户的 ID。  
你也可以指定开始日期、结 束日期以及是否显示日志条目:  
`radosgw-admin usage show --uid=johndoe --start-date=2012-03-01 --end-date=2012-04-01`  
通过去掉用户的 ID，你也可以获取所有用户的汇总的用量信息:  
`radosgw-admin usage show --show-log-entries=false`  
### 删除用量信息
对于大量使用的集群而言，用量日志可能会占用大量存储空间。  
你可以为所有用户或者一个特定的用户删除部分用量日志，也可以为删除操作指定日期范围。  
```
radosgw-admin usage trim --start-date=2010-01-01 --end-date=2010-12-31
radosgw-admin usage trim --uid=johndoe
radosgw-admin usage trim --uid=johndoe --end-date=2013-12-31
```
