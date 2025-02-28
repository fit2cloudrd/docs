---
id: version-2.1-userguide-vsphere
title:  在 vSphere 平台上规划、部署及管理 k8s 集群
original_id: userguide-vsphere
---

KubeOperator 支持两种 Kubernetes 集群部署方式，一种是自动模式，另外一种是手动模式，我们推荐使用自动模式。在自动模式下，用户需要准备软件定义的 IaaS 云平台，比如 VMware vSphere 和 Openstack 等。自动模式下 kubernetes 集群的规划、部署和管理包含以下内容：

- 集群规划 （Day 0）
  - 系统设置 
  - 创建部署计划
  - 准备存储
- 集群部署（Day 1）
  - 创建集群
  - 部署集群
  - 服务暴露
- 集群运维和变更（Day 2）
  - 集群运维
  - 集群升级
  - 集群伸缩
  - 集群备份

本章节以 VMware 作为示例，讲解整个部署过程，整个部署示意图如下图所示：

![overview](https://github.com/KubeOperator/docs/blob/master/website/static/img/vmware.png?raw=true)



## 1 登录

KubeOperator 完全启动后，访问 KubeOperator 控制台，进行登录。默认的登录用户名为 admin，默认密码为 kubeoperator@admin123。
KubeOperator 控制台 URL：http://host-ip ，将 host-ip 改为安装 KubeOperator 的主机 IP 地址。

![login-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/KubeOperator-login.png?raw=true)

> 为了保证系统的安全，请在完成登录后，点击控制台右上角的"修改密码"进行密码的重置。

## 2 系统设置

在使用 KubeOperator 之前，需要先对 KubeOperator 进行必要的参数设置。这些系统参数将影响到 Kubernetes 集群的安装及相关服务的访问。

### 2.1 主机 IP 和集群域名后缀

主机 IP 指 KubeOperator 机器自身的 IP。KubeOperator 所管理的集群将使用该 IP 来访问 KubeOperator。

集群域名后缀为集群节点访问地址的后缀，集群暴露出来的对外服务的 URL 都将以该域名后缀作为访问地址后缀。例如: grafana.apps.cluster.f2c.com。

![setting-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/setting-system-1.png?raw=true)

### 2.2 备份

KubeOperator目前的备份功能支持三种不同种类的存储，即 AWS S3、aliyun oss 和 azure 存储。为集群备份和恢复提供存储支持，实现备份和恢复功能。

添加备份账号之前，请首先自行准备好 AWS S3 ，aliyun oss 或者 azure 存储账号信息，包括 AccessKey，SecretKey，endpoint 和桶/容器信息。下图即是添加备份账号详细信息。

以添加 S3 为例，输入名称和 AccessKey，SecretKey 和端点（对应 AWS S3 系统里的 endpoint），单击【获取桶/容器】获取桶名称，建议在 S3 新建一个桶单独使用，最后提交。

![setting-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/setting-backup-add.png?raw=true)

## 3 添加存储

KubeOperator 支持自动创建 NFS 存储和添加自行准备的 NFS 存储，供自动模式和手动模式的 K8s 集群使用。下面先介绍如何添加 KubeOperator 自动创建的 NFS 存储。

### 3.1 新建 NFS

首先要准备一个主机节点单独作为 NFS 存储资源，可以是虚拟机或物理机，操作系统要求：CentOS 7.6 Minimal，最低硬件配置为 2U/2G，磁盘大小建议 xxx G？，在kubeoperator控制台【主机】页面添加该主机。

详细步骤：
  
- 1 KubeOperator 控制台【主机】页面，添加主机，注意这个主机不可以作为 K8s 集群的节点；
- 2 ssh登录该主机节点，为 NFS 新建一个挂载路径，添加存储使用(例如：#mkdir nfs，挂载路径为：/nfs)；
- 3 KubeOpeartor 控制台【存储】，单击【添加】，选中新建 NFS ，在主机下拉列表，选择上述第一步添加的 NFS 主机，如果外部所有网络可访问，白名单选项需要填 ” * “，挂载路径：/nfs，提交。

添加成功后，创建集群时如果选择 NFS 存储，可以看到该 NFS 存储。

![storage-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/nfs-auto.png?raw=true)

### 3.2 录入 NFS

自行准备一个物理机或虚拟机作为 k8s 集群的 NFS 存储服务器。
【存储】，单击【添加】选中“录入 NFS” ，输入存储名称，白名单选项需要填 “ * ”，服务地址输入准备好的 NFS 存储主机 IP 地址，输入设置好的挂载路径，提交
 。
![storage-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/nfs-manu.png?raw=true)



## 4 创建部署计划

### 4.1 创建区域(Region)

Region：与 公有云中的 Region 概念相似，可以简单理解为地理上的区域。在 vSphere 体系中我们使用 DataCenter 实现 Region 的划分。创建区域时，首先选择提供商，目前仅支持 VMware vSphere。

![region-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-region-1.png?raw=true)

配置参数时，需要提供 vSphere 环境信息，包括 vCenter IP，用户名和密码，单击【验证】可以校验 vSphere 信息是否正确。

![region-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-region-conf-1.png?raw=true)

最后一步选择 vCenter 的一个数据中心。

![region-3](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-region-datacenter-1.png?raw=true)

### 4.2 创建可用区(Zone)

Zone: 与 公有云中的 AZ 概念相似，可以简单理解为 Region 中具体的机房。在 vSphere 体系中我们使用不同的 Cluster 或者同个 Cluster 下的不同 Resource Pool 来实现 Zone 的划分。创建可用区时需要选择一个之前添加的区域，如下图：

![zone-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-zone-1.png?raw=true)

选择可用区配置参数时，需要选择计算集群，资源池，存储类型以及网络适配器等信息，这些信息依赖于 vCenter 环境配置。最后单击【检测】按钮，校验输入的起始 IP 地址和子网掩码等信息格式是否正确，检测通过之后才可以单击【完成】。添加成功后会有一个初始化的过程，状态变为就绪后可以选择该可用区创建部署计划。

![zone-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-zone-2.png?raw=true)

![zone-3](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-zone-init.png?raw=true)


### 4.3 创建部署计划(Plan)

Plan: 在 KubeOperator 中用来描述在哪个区域下，哪些可用区中，使用什么样的机器规格，部署什么类型的集群的一个抽象概念。
这里以单主多节点类型举例。

![plan-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-plan-1.png?raw=true)

部署计划配置包括选择可用区，可以单选或多选可用区，并设置 Master 节点，Worker 节点的规格，即 CPU，内存和磁盘大小。

![plan-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/create-plan-2.png?raw=true)

> 注：如果多主多节点集群选择多个可用区的部署计划，创建集群时不支持 vsan 存储。

## 5  创建和部署集群

### 5.1 集群列表
 
在左侧导航菜单中选择【集群】，进入【集群】页后可以看到已添加集群的详细信息，包括 集群部署的 Kubernetes 版本、部署模式、节点数及运行状态等。

![cluster-1](https://github.com/KubeOperator/docs/blob/master/website/static/img-2.1/cluster-list.png?raw=true)

### 5.2 创建集群

> KubeOperator 当前支持 NFS 和 vSAN 作为外部持久化存储，如果使用 NFS 存储，创建集群前，请自行准备 NFS 存储，并可以被集群主机挂载。我们推荐使用专用 NAS 产品，自行搭建的 NFS 服务仅适合在开发测试环境使用。

#### 5.2.1 基本信息

点击【集群】页的【添加】按钮进行集群的创建。在【基本信息】里输入集群的名称，选择该集群所要部署的 Kubernetes 版本和部署模式。
在离线包列表中可以查看 KubeOperator 当前所提供的 Kubernetes 安装版本详细信息。在后续进行 Kubernetes 集群部署时，可以从这些版本中选择其一进行部署（当前支持1.15.0, 1.15.4，后续会继续跟随 Kubernetes 社区发布离线包）。

![cluster-create-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-basicinfo-1.png?raw=true)

离线包列表信息：

![package-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/package-1.png?raw=true)

离线包详情信息：

![package-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/package-2.png?raw=true)

#### 5.2.2 部署计划

选择 Kubernetes 集群的部署计划和 Worker 节点数量，至少 3 个 Worker 节点，Worker 节点配置建议 4 U / 16 G，请保证 vSphere 环境资源充足，尤其是内存资源。

![cluster-create-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-plan-1.png?raw=true)

#### 5.2.3 配置网络

【配置网络】环节，选择集群的网络插件，当前版本支持 Flannel 和 Calico 两种网络方案。

> 对于 Flannel，如果集群节点全部都在同一个二层网络下，请选择"host-gw"。如果不是，则选择"vxlan"。"host-gw" 性能优于 "vxlan"。

![cluster-create-3](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-confnetwork-1.png?raw=true)

#### 5.2.4 配置存储

【添加存储】环节，选择外部持久化存储 vSan 或者 NFS ，如果选择 NFS，支持两种方式的 NFS，一种是 自动创建 NFS 存储，另外一种是用户自行准备的 NFS 存储。 详细描述见3.1和3.2节部分。

![cluster-create-4](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-confstor-1.png?raw=true)

#### 5.2.5 集群配置概览

所有步骤完成后，会有一个集群配置概览页对之前步骤所设参数进行汇总，用户可在此页进行集群配置的最后检查。

![cluster-create-6](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-complate-1.png?raw=true)

### 5.3 部署集群

在集群列表中点击要进行部署的集群名称，默认展示的是该集群的【概览】信息。【概览】页中展示了 Kubernetes 集群的诸多详情，包括集群状态，Worker 状态集群描述信息等。点击【概览】页最下方的【安装】按钮进行 Kubernetes 集群的部署。

![cluster-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-summary-1.png?raw=true)

集群部署开始后，将会自动跳转到【任务】页。在【任务】页里可以看到集群部署当前所执行的具体任务信息。

![cluster-deploy-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-install-1.png?raw=true)

如果是内网环境的话，一个典型的 4 节点集群的部署大概需要10分钟左右的时间,【历史】页可以看到详情部署时间信息。在出现类似下图的信息后，表明集群已部署成功：

![cluster-deploy-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-install-2.png?raw=true)

【历史】页可以看到所有完成的任务详情信息。

![cluster-deploy-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-history-1.png?raw=true)

> 注：通过自动模式创建的集群里所有的主机，包括 master 和 worker 主机默认用户名和密码为：root / KubeOperator@2019。

### 5.4 卸载集群

在集群列表中点击要进行卸载的集群名称，点击【概览】页最下方的【卸载】按钮进行 Kubernetes 集群的卸载。卸载后的集群所有主机将被彻底移除，不会保留在主机列表中。

![cluster-uninstall](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-uninstall-1.png?raw=true)

### 5.5 升级

KubeOperator支持 K8s 升级。

在集群列表中点击要进行升级的集群名称，点击【概览】页最下方的【升级】按钮进行 Kubernetes 集群的升级。

![cluster-upgrade-1](https://github.com/KubeOperator/docs/blob/master/website/static/img-2.1/cluster-upgrade.png?raw=true)

![cluster-upgrade-2](https://github.com/KubeOperator/docs/blob/master/website/static/img-2.1/cluster-upgrade-1.png?raw=true)

![cluster-upgrade-3](https://github.com/KubeOperator/docs/blob/master/website/static/img-2.1/cluster-upgrade-2.png?raw=true)


### 5.6 伸缩

此版本 KubeOperator 支持重点新功能：扩缩容 K8s 集群 worker 节点数量。

KubeOperator 控制台【集群】页，单击一个要扩缩容的集群名称，即【概览】页面，Worker 状态栏左下方单击【伸缩】，在弹出框中选中扩容或者缩容的 worker 节点数量。

![cluster-expand-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-expand.png?raw=true)

确认后，会自动转到【任务】页面，实时查看扩缩容进度，完成后可以看到如下图所示信息。

![cluster-expand-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-expand-add.png?raw=true)


### 5.7 Webkubectl

KubeOperator 新增功能支持 Webkubectl 。

![cluster-webkubectl](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-webkubectl.png?raw=true)


### 5.8 健康状态

在 K8s 集群【健康状态】栏，可以看到整体的集群状态，具体包括 Control Manager，Schedule，etcd 和 nodes 的实时健康状态以及过去半年 K8s 集群运行状态。

![cluster-healthy](https://github.com/KubeOperator/docs/blob/master/website/static/img/cluster-heathy-1.png?raw=true)

### 5.9 备份/恢复

![cluster-backup](https://github.com/KubeOperator/docs/blob/master/website/static/img-2.1/cluster-backup.png?raw=true)

### 5.10 服务暴露

在集群列表中点击集群名称，点击【F5 BIG-IP】添加 F5 BIG-IP，为Kubernetes配置 F5-BIGIP-CONTROLLER 后，我们可以通过 F5 BIGIP 设备向外网暴露服务。

![cluster-f5](https://github.com/KubeOperator/docs/blob/master/website/static/img/f5-bigip-1.png?raw=true)


## 6 管理集群

### 6.1 访问 Dashboard

Dashboard 对应的是 Kubernetes 的控制台，从浏览器中访问 Kubernetes 控制台需要用到【令牌】。点击【概览】页下方的【获取TOKEN】按钮获取令牌信息，将令牌信息复制到粘贴板。

![dashboard-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/dashboard-1.png?raw=true)

输入令牌信息后，点击【登录】，则可进入 Kubernetes 控制台。

![dashboard-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/dashboard-2.png?raw=true)

### 6.2 访问 Grafana

Grafana 对 Prometheus 采集到的监控数据进行了不同维度的图形化展示，更方便用户了解整个 Kubernetes 集群的运行状况。点击 Grafana 下方的【转到】按钮访问 Grafana 控制台。

集群级别的监控面板：

![grafana-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/grafana-1.png?raw=true)

节点级别的监控面板：

![grafana-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/grafana-2.png?raw=true)

### 6.3 访问 Registry

Registry 则用来存放 Kubernetes 集群所使用到的 Docker 镜像。

![regsitry-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/registry-1.png?raw=true)

### 6.4 访问 Prometheus

Prometheus 用来对整个 kubernetes 集群进行监控数据的采集。点击 Prometheus 下方的【转到】按钮即可访问 Prometheus 控制台。

![prometheus-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/prometheus-1.png?raw=true)

### 6.5 访问 Traefik

Traefik 用来作为 kubernetes 集群的HTTP反向代理、负载均衡工具。点击 Trafik 下方的【转到】按钮即可访问 Traefik 控制台。

![prometheus-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/traefik.png?raw=true)

### 6.6 访问 Weave Scope

Weave Scope 用来监控、可视化和管理 kubernetes 集群。点击 Weave Scope 下方的【转到】按钮即可访问 Weave Scope 控制台。点击控制台的顶部【Pod】，会自动生成容器之间的关系图，方便理解容器之间的关系，也方便监控容器化和微服务化的应用。

![weave-scope-1](https://github.com/KubeOperator/docs/blob/master/website/static/img/weave-scope-2.png?raw=true)

点击顶部的【Host】，可以远程shell登录各个节点，还可以看到主机的详细信息。

![weave-scope-2](https://github.com/KubeOperator/docs/blob/master/website/static/img/weave-scope-1.png?raw=true)




