
> 简单来说, k8s 和 Docker 并不是一个维度的东西，不具有可比性。它们之间是相互依存的关系，Docker 是容器引擎，而 k8s 是用来编排 Docker 等容器的协调器。

## 1 什么是 k8s ？

![K8S 的功能](https://img.quanxiaoha.com/quanxiaoha/166305602868530 "K8S 的功能")

**k8s 是 Kubernetes 的缩写**, 这个单词来自于希腊语，含义是 **舵手** 或 **领航员**，**是一个可以基于容器的集群管理平台。**

k8s 由 Google 开发，它的前身是 Google 自己倒腾了十多年的 Borg 系统，在 2014 年 6 月由 Google 公司正式公布出来并宣布开源。

Google 利用在容器管理多年的经验和专业知识推出了 k8s，主要用于自动化部署应用程序容器，可以支持众多容器化工具包括现在非常流行的 Docker。

目前 k8s 是容器编排市场的领导者，开源并公布了一系列标准化方法，主流的公有云平台都宣布支持。

## 2 K8S 解决了哪些痛点？

随着公司大小业务使用 Docker 越来越多，随之而来，也出现了一系列问题：

- 1、如何**协调、调度和管理**这些容器？
- 2、如何在升级应用程序时 **不中断服务**？
- 3、如何 **监视** 应用程序的运行状况？
- 4、如何**批量重新启动容器**里的程序？
- 5、**负载均衡**
- 6、**鉴权**和**安全性**问题
- 7、**服务之间通信**问题
- 8、**多平台部署**
- ...

于是乎，k8s 出现了，它能够很好的解决这些问题。

> 容器编排除了 k8s 外，还有 Mesos、Docker Swarm, 只不过目前业界最流行的是 k8s 。

## 3 k8s 架构与组件

### 3.1 整体架构

下图清晰表明了 Kubernetes 的架构设计以及组件之间的通信协议。

![Kuberentes 架构](https://img.quanxiaoha.com/quanxiaoha/166313446509796 "Kuberentes 架构")

下面是更抽象的一个视图：
![kubernetes 整体架构示意图](https://img.quanxiaoha.com/quanxiaoha/166313507870065 "kubernetes 整体架构示意图")
### 3.2 Master 架构

![Kubernetes master 架构示意图](https://img.quanxiaoha.com/quanxiaoha/166313519267449 "Kubernetes master 架构示意图")

### 3.3 Node 架构

![kubernetes node 架构示意图](https://img.quanxiaoha.com/quanxiaoha/166313529864147 "kubernetes node 架构示意图")
### 3.4 分层架构

Kubernetes 设计理念和功能其实就是一个类似 Linux 的分层架构，如下图所示。
![Kubernetes 分层架构示意图](https://img.quanxiaoha.com/quanxiaoha/166313554910295 "Kubernetes 分层架构示意图")Kubernetes 分层架构示意图

- `核心层`：Kubernetes 最核心的功能，对外提供 API 构建高层的应用，对内提供插件式应用执行环境
- `应用层`：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS 解析等）、Service Mesh（部分位于应用层）
- `管理层`：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态 Provision 等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）、Service Mesh（部分位于管理层）
- `接口层`：kubectl 命令行工具、客户端 SDK 以及集群联邦
- `生态系统`：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
    - Kubernetes 外部：日志、监控、配置管理、CI/CD、Workflow、FaaS、OTS 应用、ChatOps、GitOps、SecOps 等
    - Kubernetes 内部：CRI、CNI、CSI、镜像仓库、Cloud Provider、集群自身的配置和管理等

## 4 k8s 与 Docker Swarm 比较

![k8s 与 Docker Swarm 比较](https://img.quanxiaoha.com/quanxiaoha/166313602622132 "k8s 与 Docker Swarm 比较")

Docker Swarm 与 k8s 都是容器编排技术。Docker Swarm 是 Docker 官方提供的针对集群化部署管理的解决方案，优点很明显，可以更紧密集成到 Docker 生态系统中。

虽说 Swarm 是 Docker 亲儿子，但依旧没有 k8s 流行，不流行很大程度是因为商业、生态的原因，不多解释。

## 5 生产实践中，关于 k8s 的相关疑问

- 1、使用了 Docker, 一定要上 k8s 吗？
	- **非必须，得结合实际业务综合考量，** 如果是一家小型公司，本身业务并不复杂，没有那么大的用户量，是可以直接仅使用 Docker 的。相反，如果是类似阿里这种用户量大，业务复杂，需要服务器集群，那么上 k8s 就很有必要了。

- 2、没有用到 Docker, 可以使用 k8s 吗？
	- 可以，要知道 k8s 只是一个容器编排工具，需要和容器搭配使用。而目前最流行的容器技术就是 Docker, 所以通常将它们放在一起来说，但是也可以使用其他的容器，如 RunC、Containerted 等。

- 3、Docker Swarm 和 k8s 选哪个？
	- 选 k8s 。2019 年底 Docker Enterprise 已经出售给 Mirantis，Mirantis 声明要逐步淘汰 Docker Swarm，后续会将 k8s 作为默认编排工具。