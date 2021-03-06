# MongoDB on QingCloud AppCenter 用户指南

<extoc></extoc>

## 简介

青云QingCloud MongoDB 服务提供的是原生 [MongoDB](https://www.mongodb.com/) replica set 云服务，MongoDB 是一个开源的文档型数据库，具有高性能、高可用等优点。


## 创建 MongoDB

在青云上，您可以很方便的创建和管理一个 MongoDB 集群。青云的 MongoDB 集群支持横向与纵向在线伸缩，同时具有自我诊断与自我修复功能，即当系统发现某节点坏死时会自我修复，无需人为干预。 另外我们还提供了监控告警等功能来帮助您更好的管理集群。集群将运行于私有网络内，结合青云提供的高性能硬盘，在保障高性能的同时兼顾您的数据安全。

> 为了保障数据安全, MongoDB 集群需要运行在受管私有网络中。所以在创建一个 MongoDB 集群之前，需要创建一个 VPC 和一个受管私有网络，受管私有网络需要加入 VPC，并开启 DHCP 服务（默认开启）。

第一步：创建集群

在创建的对话框中，您需要填写名称 (可选)，选择 MongoDB 版本号、CPU、节点配置和数量、私有网络等。

![](snapshots/step1.png)

> 目前集群节点数支持1、3、5、7，其中1个节点的 MongoDB 仅供测试使用。

> 用户默认创建在admin数据库下，连接时需要指定认证数据库为admin。


第二步：创建成功

当 MongoDB 创建完成之后，您可以查看每个节点的运行状态。当节点的服务状态显示为“正常”状态，表示该节点启动正常。 当每个节点都启动正常后 MongoDB 集群显示为“活跃”状态，表示您已经可以正常使用 MongoDB 服务了。

![](snapshots/step2.png)

## 测试 MongoDB

### 使用 mongo 客户端进行连接

MongoDB 创建完成之后可以进行连接测试。参考文档 https://docs.mongodb.com/manual/administration/install-on-linux/ 下载并安装 `mongodb-org-shell`，您可以在 MongoDB 同一私有网络或跨网络的客户端上测试。现假设客户端和 MongoDB 在同一私有网络，MongoDB 集群有三个节点，IP 地址分别为`192.168.100.10,192.168.100.11,192.168.100.12`， 您创建的用户名为`qc_test`，密码为`Pwd000`，可以通过如下命令连接 MongoDB：

```shell
mongo mongodb://qc_test:Pwd000@192.168.100.10,192.168.100.11,192.168.100.12/admin?replicaSet=foobar
```

连接成功后将出现下面的命令行：

```text
foobar:PRIMARY>
```

### 创建用户

> 创建集群时输入的用户名对应的是 readWriteAnyDatabase 权限的普通账号，无法创建用户。如果需要创建用户，请使用 root 账号，密码与你输入的密码相同。连接时请使用 3.4 以上的版本。

下面演示如何在 `db1` 中，创建一个用户名`test_user1`，密码为`Pwd001`，具有`readWrite`权限的用户。首先，使用 root 账号进行连接：

```shell
mongo mongodb://root:Pwd000@192.168.100.10,192.168.100.11,192.168.100.12/admin?replicaSet=foobar
```

连接成功后执行如下的命令：

```javascript
use db1;
db.createUser({ user: "test_user1", pwd: "Pwd001", roles: [ "readWrite" ]});
```

创建成功后可以`Ctrl+C`退回到 shell 中，使用如下命令以`test_user1`身份连接到`db1`。

```shell
mongo mongodb://test_user1:Pwd001@192.168.100.10,192.168.100.11,192.168.100.12/db1?replicaSet=foobar
```

更多权限管理可以参考: https://docs.mongodb.com/manual/tutorial/enable-authentication/

### 使用代码进行连接

如果使用代码进行连接, 那么 MongoDB 连接字符串为 mongodb://USER:PASSWORD@IP/DB?replicaSet=foobar&authSource=admin

其中:

- USER 是你所创建的用户名
- PASSWORD 是你设置的密码
- IP 是 replica set 中的节点 IP, 使用逗号分隔, 比如 192.168.100.10,192.168.100.11,192.168.100.12
- DB 是你需要进行连接的数据库
- replicaSet=foobar 是青云默认的 replicaSet 配置, 值为 foobar
- authSource=admin 是指定验证数据库为 admin
- 更多参数可以参考: https://docs.mongodb.com/manual/reference/connection-string/


## 服务功能

点开基本属性旁边的下拉按钮，可以看到提供的服务功能。

![](snapshots/feature.png)

### 配置参数

这里列出了可以修改并持久化的配置参数。修改配置参数将会使集群重启，请在业务低峰时进行修改

![](snapshots/env.png)

### 增删节点

可以根据需要增加集群节点，增加的节点数必需为偶数。添加节点的任务执行时间跟集群的数据量有关系，数据量大时，任务执行时间会久一些，添加节点不影响集群的读写。

删除节点有如下限制：

- 数量必须为偶数，至少保留一个节点
- 无法删除主节点
- 无法删除 `qc_sid` 为 1 的节点

> `qc_sid` 可以使用 root 用户通过 mongo 连接之后执行 `rs.conf().members` 命令查看。`qc_sid` 为 1 的节点的选举优先级被我们设置为 2，所以这个节点通常是主节点。

![](snapshots/add_nodes.png)

### 备份

如果您打开了『自动备份』功能，那么 MongoDB 集群会在你指定的时间段进行每日的自动备份； 您可以从这些备份创建出全新的 MongoDB 集群；

如果您关闭了自动备份，可以在集群列表右键 MongoDB 集群后点击『创建备份』选项， 或者在集群详情页的『备份』标签下点击『创建备份』按钮。

> 如果备份后进行过增删节点，那么再次备份时需要创建新的备份链。

![](snapshots/create_snapshot.png)

如果需要从备份创建出一个独立于原有 MongoDB 集群的新 MongoDB 集群， 可以在集群详细页的『备份』标签下右键相应的备份点，再选择『从备份创建集群』即可。

![](snapshots/create_cluster_from_snapshot.png)

### 扩容集群

可以对一个运行中的集群进行在线扩容，调整CPU/内存/磁盘空间大小。

![](snapshots/scale1.png)

注解：扩容需要在开机状态下进行，扩容会导致重新选主，请在业务低峰时进行

### 同步日志

同步日志功能可以将 mongod.log 拷贝到系统的 FTP 目录，同步后可以在内网下载到本地进行分析。

![](snapshots/copy_log.png)

### 清理日志

清理日志功能可以将 mongod.log 清空，减少日志的磁盘空间占用。

![](snapshots/clean_log.png)

### 监控告警

目前提供的监控项包括：

- CPU 使用率
- 内存利用率
- 磁盘使用量
- MongoDB 各项操作数量
- MongoDB 复制操作数量
- MongoDB 连接数

目前提供的告警项包括：

- CPU 利用率
- 内存利用率
- 磁盘使用量
- MongoDB 连接数

## 数据导出和导入

使用 `mongodb-org-tools` 中的工具 `mongodump`, `mongorestore` 可以很方便的执行 MongoDB 数据的导出和导入。

- 使用 `mongodump` 工具可以将 MongoDB 中的数据导出，使用说明可以参考官方文档 https://docs.mongodb.com/manual/reference/program/mongodump/
- 使用 `mongorestore` 工具可以将 `mongodump` 导出的数据导入到 MongoDB 中，使用说明可以参考官方文档 https://docs.mongodb.com/manual/reference/program/mongorestore/

### 在线导入

下面演示如何将`192.168.110.101`的数据直接在线导入到`192.168.110.102`中。在同一个VPC下创建一台主机，安装 `mongodb-org-tools` 工具。在命令行中执行以下的命令：

```shell
mongodump --archive --username=user001 --password=Pwd001 --authenticationDatabase=admin --host=192.168.110.101 --db=db1 | mongorestore --archive --username=user002 --password=Pwd002 --authenticationDatabase=admin --host=192.168.110.102 --db=db2
```

其中: 使用 `192.168.110.101`的用户`user001`，密码`Pwd001`，验证数据库`admin`，导出数据库`db1`，导入到`192.168.110.102`的用户`user002`，密码`Pwd002`，验证数据库`admin`，导入数据库`db2`

## 基准测试

我们用 [Yahoo! Cloud Serving Benchmark](https://github.com/brianfrankcooper/YCSB/tree/master/mongodb) 工具套件进行了基准测试，下面分别是各配置的测试结果。

> YCSB 不支持用户名密码登录，下面的结果是在关闭认证后测试得到的，仅供参考。由于安全原因，我们禁止您关闭 MongoDB 的认证，建议在实际性能评估时以模拟正式场景为准。

- 1核2G 100G 超高性能型硬盘

```shell
./bin/ycsb load mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 193602.0
[OVERALL], Throughput(ops/sec), 5165.2358963233855
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 254871.0
[OVERALL], Throughput(ops/sec), 3923.553483919316
```

- 2核4G 100G 超高性能型硬盘

```shell
./bin/ycsb load mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 73817.0
[OVERALL], Throughput(ops/sec), 13547.014915263422
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 122344.0
[OVERALL], Throughput(ops/sec), 8173.674230039887
```

- 4核8G 100G 超高性能型硬盘

```shell
./bin/ycsb load mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 40768.0
[OVERALL], Throughput(ops/sec), 24529.042386185243
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 47416.0
[OVERALL], Throughput(ops/sec), 21089.92745064957
```

> 下面同时运行了两个 YCSB 实例来对 8核16G 和 16核32G 进行基准测试， 需要注意的是两个 YCSB 实例实际上也并不能压满 MongoDB 的性能，所以 OPS 仅供参考之用。

- 8核16G 100G 超高性能型硬盘

```shell
./bin/ycsb load mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 35088.0
[OVERALL], Throughput(ops/sec), 28499.772001823985
```

```shell
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 50013.0
[OVERALL], Throughput(ops/sec), 19994.80135164857
```

```shell
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 51555.0
[OVERALL], Throughput(ops/sec), 19396.76074095626
```

- 16核32G 100G 超高性能型硬盘

```shell
./bin/ycsb load mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 33882.0
[OVERALL], Throughput(ops/sec), 29514.196328433976
```

```shell
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 37720.0
[OVERALL], Throughput(ops/sec), 26511.134676564157
```

```shell
./bin/ycsb run mongodb-async -s -P workloads/workloada -p operationcount=1000000 -p recordcount=1000000 -threads 100 -p mongodb.url="mongodb://192.168.100.26/ycsb"
[OVERALL], RunTime(ms), 38600.0
[OVERALL], Throughput(ops/sec), 25906.73575129534
```
