# CDH on QingCloud AppCenter 用户指南

## 简介

CDH (*Cloudera's Distribution Including Apache Hadoop*) 是 Cloudera 的 Hadoop 发行版。CDH 提供了 Hadoop 生态圈很多重要开源产品及组件如 Hadoop, Spark, HBase, Hive, Pig, Hue, Oozie, Kafka, Zookeeper, Kudu, Accumulo, Flume, Sqoop, Sentry 等。CDH 还提供了用于 CDH 集群管理的 Cloudera Manager 以及用于数据管理的 Cloudera Navigator 。

安装配置 CDH 集群是一项十分复杂的工作，涉及到操作系统环境配置、特定版本 JDK 的安装、数据库的安装及配置、Cloudera Manager server 及 agent 的安装及配置、各个大数据组件的安装及配置等，并且安装过程中需要联网下载众多比较大的安装包，会使得整个集群的安装过程复杂、缓慢并且容易出错。

> 手动安装 CDH 步骤详见 [官方安装指南](https://www.cloudera.com/documentation/enterprise/latest/topics/installation.html)

*CDH on QingCloud* 将安装 CDH 需要的操作系统环境的配置、各个依赖产品/组件的安装及配置等众多纷繁复杂的工作自动化，提前下载好了各个组件的安装包并做了相关分发配置，使得整个 CDH 集群的安装配置过程可以在十分钟左右快速完成，并可以很方便的横向及纵向扩展集群，极大地方便了用户的使用。

>*CDH on QingCloud* 目前提供的 CDH 版本是与 CentOS 7.3 兼容的 *CDH 5.13.0* （Impala 和 Solr 暂没有与 CentOS 7.3 相兼容的版本），完整组件列表及版本详见[官方文档](https://www.cloudera.com/documentation/enterprise/release-notes/topics/cm_vd_cdh_package_tarball_513.html#cm_vd_cdh_package_tarball_513)。 
>
>CDH 某些服务需要 license 才可以使用，用户可以自行获取 license。
>
>*CDH on QingCloud* 提供的是 CDH 在青云上的自动化安装和部署服务，CDH 使用中遇到问题请参考[官方指南](https://www.cloudera.com/documentation/enterprise/latest/topics/introduction.html) 或者联系Cloudera 客服。



## 创建部署CDH所需集群

### 第1步：基本设置

![第1步：基本设置](../../images/CDH/basic_config.png)

填写服务`名称`和`描述`，选择版本

### 第2步：主节点设置

![第2步：HDFS主节点设置](../../images/CDH/cm_config.png)

填写主节点 CPU、内存、节点类型、数据盘类型及大小等配置信息。

> 主节点运行了诸多服务比如 Cloudera Manager Server, Name Node, Resource Manager 等，需要比较大的内存空间，8G 以下的内存配置仅供测试。

### 第3步：从节点设置

![第4步：从节点设置](../../images/CDH/slave_config.png)

填写 从节点 CPU、内存、节点类型、数据盘类型及大小等配置信息。

> 从节点如需安装除了 HDFS data node，YARN NodeManager 之外的其他服务如 Kafka，则至少需要 8G 内存。

> 主节点及从节点均为用户可访问的节点，可通过用户名 root，初始密码为 p12cHANgepwD 进行访问。

### 第4步：网络设置

![第6步：网络设置](../../images/CDH/network_config.png)

出于安全考虑，所有的集群都需要部署在私有网络中，选择自己创建的已连接路由器的私有网络。

CDH 主节点及从节点 IP 应该「手动指定」，此为官方推荐做法。

> 更多细节详见[网络资源配置](http://appcenter-docs.qingcloud.com/user-guide/apps/docs/network-config/)

### 第5步：服务环境参数设置

![第8步：服务环境参数设置](../../images/CDH/env_config.png)

由于从节点上通常保存有集群数据，并运行着相关服务，因此默认是不允许删除从节点的。如需删除从节点，请将该节点上的数据及服务妥善处理后，设置配置参数「允许横向缩容」为 true, 再进行相关操作。

操作完成后请将「允许横向缩容」设回 false 以避免误操作导致数据丢失。

![更改配置参数](../../images/CDH/change_env.png)

### 第 6 步: 用户协议

阅读并同意青云 AppCenter 用户协议之后即可开始创建部署 CDH 所需集群。

创建完成后点击集群列表页面相应集群即可查看集群详情：

![查看服务详情](../../images/CDH/cluster_detail.png)



## CDH 安装部署

### 1. 登录 Cloudera Manager

集群创建成功后，Cloudera Manager 服务将会部署到主节点中，待各节点服务状态均变为正常后，即可通过访问 `http://<主节点ip>:7180` 开始部署 CDH 组件到整个集群。

> 默认用户名及密码admin/admin

![登录](../../images/CDH/login.png)



### 2. 接受许可条款和条件

![条款](../../images/CDH/agreement.png)



### 3. 选择版本

![版本](../../images/CDH/version.png)



### 4. 安装组件列表

![组件](../../images/CDH/component_list.png)



### 5. 指定安装主机

输入新建集群的各节点 ip (包括主节点和所有从节点)，点击搜索列出需要安装 CDH 的主机列表并全部选中

![指定主机](../../images/CDH/hosts.png)



### 6. 选择配置安装包

- 选择默认安装方法为`「使用 Parcel」`

- 选择 CDH 版本`「CDH-5.13.0-1.cdh5.13.0.p0.29」`

  > CDH on QingCloud 目前预装的 CDH 版本是 `「CDH-5.13.0-1.cdh5.13.0.p0.29」` ， 请勿选择其他版本，以避免连外网下载比较大的 Parcel 安装包。
  >
  > CDH 如有新版本发布，我们会提供青云相应区的镜像站点的链接，请关注本指南的更新。

- 选择其他需要安装组件的 Parcel 包，比如 Kafka

  ![安装包](../../images/CDH/select_parcel.png)

>    如需安装 Accumulo 请选择已经提前下载好的 1.7.2 版

- 点击`「使用 Parcel」`旁边的`更多选项`进入 `Parcel 存储库设置` 

  ![Parcel 存储库设置](../../images/CDH/parcel_config.png)



与 CentOS 7.3 兼容的 CDH 5.13.0 各组件已经提前下载并存储到主节点，包括 cdh5, Kafka, kudu, accumulo-c5, spark2 和 sqoop 。因此需要将这几个组件的`远程 Parcel 存储库 URL`配置为主节点相应路径。假设主节点 ip 为 192.168.100.19 ，则各组件到`远程 Parcel 存储库 URL`需按如下 url 进行配置：

```shell
http://192.168.100.19/cdh5/parcels/5.13.0/
http://192.168.100.19/kafka/parcels/3.0.0/
http://192.168.100.19/kudu/parcels/1.4.0/
http://192.168.100.19/accumulo-c5/parcels/1.7.2/
http://192.168.100.19/spark2/parcels/2.1.0/
http://192.168.100.19/sqoop/sqoop-connectors/
```



如下为配置好本地 parcel 存储库 url 后的效果：

![Parcel 存储库设置](../../images/CDH/parcel_config2.png)



### 7. JDK 安装

![JDK 安装](../../images/CDH/jdk.png)



### 8. 单用户模式

<font color=red>不要选择</font>「单用户模式」，直接点击继续：

![单用户模式](../../images/CDH/single_user_mode.png)



### 9. 安装身份验证

- 选择以 root 用户「登录到所有主机」
- 选择身份验证方法为「所有主机接受相同密码」
- 输入密码 p12cHANgepwD
- 点击继续后，即开始集群安装

![身份验证](../../images/CDH/credential.png)



### 10. Cloudera Agent 安装

- 开始 Cloudera Manager Agent 安装

![集群安装](../../images/CDH/install1.png)

- Cloudera Manager Agent 安装完成后点击继续

![集群安装](../../images/CDH/install2.png)



### 11. 组件安装包分发与激活

![集群安装](../../images/CDH/install3.png)

​    组件安装包激活后点击继续

![集群安装](../../images/CDH/install4.png)

### 12. 安装环境检查及可安装组件列表

![集群安装](../../images/CDH/pre_check.png)



![集群安装](../../images/CDH/list.png)



![集群安装](../../images/CDH/list2.png)



### 13. 选择需要安装的服务

> Impala 和 Solr 没有与 CentOS 7.3 相兼容的版本，因此预定义或自定义安装不能选择包含 Impala 或 Solr 的组合

- 选择安装预定义的服务组合

  ![集群安装](../../images/CDH/predefined_combo.png)

  ​

- 选择安装自定义的服务组合

  ![集群安装](../../images/CDH/customize_combo.png)



### 14. 自定义角色分配

选择好服务组合，点击下一步将进入角色分配页面。

- 用户可以选择将指定服务运行在指定主机上，有「选择主机」字样的服务不是必选服务。

  ![集群安装](../../images/CDH/role.png)



- 点击某服务下的主机或「选择主机」，将进入为服务选择主机页面

  ![集群安装](../../images/CDH/host_select.png)

### 15. 数据库设置

为服务选定主机后，点击下一步将进入数据库设置页面。

各服务需要输入的数据库名称和用户名如下图所示，密码均为 `CDH@qingcloud2017`

![集群安装](../../images/CDH/db_config.png)



### 16. 集群服务自定义配置

数据库配置完成后将进入服务参数配置页面，用户基本不需要改动任何配置。

尤其不要改动目录相关设置，否则可能会出现磁盘空间不足的情况。

![集群安装](../../images/CDH/config_review.png)

![集群安装](../../images/CDH/config_review2.png)



### 17. 服务安装

![集群安装](../../images/CDH/service_install.png)



### 18. 安装完成

![集群安装](../../images/CDH/install_complete.png)



点击完成将看到 Cloudera Manager 管理界面

![集群安装](../../images/CDH/cm1.png)



点击左侧相应组件将可以预览服务详情

![集群安装](../../images/CDH/cm-yarn.png)



## 在线伸缩

### 增加节点

CDH on QingCloud 中主节点唯一，用户可以增加从节点。

增加从节点需要如下几步：

- 添加从节点到 CDH on QingCloud 集群中

- 在 Cloudera Manager 里将该新增节点加入 CDH 集群以被 Cloudera Manager 管理

- 创建从节点的主机模版，将需要部署在从节点的角色加入该模版

- 在新加入的从节点主机上应用从节点模版，以部署从节点角色到该主机

- 重启过期服务使新配置生效

  ​

下面将逐一演示每一步：

#### 第 1 步：添加从节点到 CDH on QingCloud 集群中

待新添加节点的「服务状态」变为正常后，继续下一步

> CDH 节点 IP 应该「手动指定」，此为官方推荐做法。

![增加节点](../../images/CDH/add_node1.png)



#### 第 2 步：在 Cloudera Manager 里将该新增节点加入 CDH 集群

进入 Cloudera Manager 的主机->所有主机页面，点击添加新主机

![增加节点](../../images/CDH/add_node2.png)



​	然后按如下图示输入新增节点 IP，密码等配置，依次点「继续」直至完成：



![增加节点](../../images/CDH/add_node3.png)



![增加节点](../../images/CDH/add_node3.png)

![增加节点](../../images/CDH/add_node4.png)

![增加节点](../../images/CDH/add_node5.png)

![增加节点](../../images/CDH/add_node6.png)

![增加节点](../../images/CDH/add_node7.png)

![增加节点](../../images/CDH/add_node8.png)

![增加节点](../../images/CDH/add_node9.png)

![增加节点](../../images/CDH/add_node10.png)

![增加节点](../../images/CDH/add_node11.png)



![增加节点](../../images/CDH/add_node12.png)

#### 第 3 步：创建从节点的主机模版，将需要部署在从节点的角色加入该模版

![增加节点](../../images/CDH/create_template1.png)

![增加节点](../../images/CDH/create_template2.png)



#### 第 4 步：在新加入的从节点主机上应用从节点模版，以部署从节点角色到该主机

![增加节点](../../images/CDH/add_node13.png)

![增加节点](../../images/CDH/add_node14.png)

![增加节点](../../images/CDH/add_node15.png)

#### 第 5 步：重启过期服务使新配置生效

添加节点后，Cloudera Manager 会显示如下「过期配置：需要重新部署客户端配置」的提示，点击该信息将进入过期配置页面

![增加节点](../../images/CDH/add_node16.png)



​	点击过期配置页面的「重启过时服务」，将会重启相关服务使新配置生效

![增加节点](../../images/CDH/restart_outdated1.png)

![增加节点](../../images/CDH/restart_outdated2.png)



### 删除节点

CDH on QingCloud 中主节点唯一且不可删除，从节点最少为 3 个

删除从节点需要如下几步：

- 确保待删除从节点上的数据和服务已经妥善处理，删除该节点不会导致数据丢失

- 在 Cloudera Manager 里将该从节点从 CDH 集群删除

- 将该从节点从 Cloudera Manager 删除

- 重启过期服务使新配置生效

- 将配置参数里的「允许横向缩容」设置为 true 

- 将该节点从 CDH on QingCloud 集群删除

- 将配置参数里的「允许横向缩容」设置为 false 

  ​

下面将逐一演示有关步骤：

#### 第 1 步：确保待删除从节点上的数据和服务已经妥善处理，删除该节点不会导致数据丢失

#### 第 2 步：在 Cloudera Manager 里将该从节点从 CDH 集群删除

![增加节点](../../images/CDH/delete_node1.png)

![增加节点](../../images/CDH/delete_node2.png)

![增加节点](../../images/CDH/delete_node3.png)

#### 第 3 步：将该从节点从 Cloudera Manager 删除

![增加节点](../../images/CDH/delete_node4.png)

​	如下所示，执行该步前需要登录该节点停止 Cloudera Manager Agent 服务后点击确认：

```shell
service cloudera-scm-agent stop
```



![增加节点](../../images/CDH/delete_node6.png)

#### 第 4 步：重启过期服务使新配置生效

> 参考增加节点相关步骤

#### 第 5 步：将配置参数里的「允许横向缩容」设置为 true 

#### 第 6 步：将该节点从 CDH on QingCloud 集群删除



![增加节点](../../images/CDH/delete_node7.png)



#### 第 7 步：将配置参数里的「允许横向缩容」设置为 false 



### 纵向伸缩

CDH 允许分别对各种角色的节点进行纵向的扩容及缩容。

> 扩容/缩容后，Cloudera Manager 服务需要一点时间重新启动，在此期间它的服务暂不可访问
>
> 扩容/缩容或者集群关闭再启动后，Cloudera Manager 的某些服务如 Service Monitor, Host Monitor, Reports Manager, Activity Manager, Event Server 可能需要手动启动



![增加节点](../../images/CDH/scale_vertically1.png)



![增加节点](../../images/CDH/scale_vertically2.png)



## 监控告警

### 资源级别的监控与告警

对 CDH 集群的每个节点提供了资源级别的监控和告警服务，包括 CPU 使用率、内存使用率、硬盘使用率等。



![增加节点](../../images/CDH/monitor.png)

![增加节点](../../images/CDH/alarm.png)



## 配置参数

![更改配置参数](../../images/CDH/change_env.png)


