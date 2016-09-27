# TrcloudOS简介
----

##产品简介

TrcloudOS是基于docker的企业级私有云一站式运维管理系统。

我们致力于研究docker为代表的容器技术，为企业级用户打造持续交付和自动运维平台，解决用户从代码自动编译打包，到线上运行维护的全套需求。TrcloudOS采用私有云模式，实现用户私有集群的容器化管理和资源智能化分配，提供全流程标准化的主机管理、应用持续集成、镜像构建、部署管理、容器运维和多层级监控服务。
##产品服务

###项目管理

* 代码仓库：针对企业级用户的需求，TrcloudOS目前支持关联私有Gitlab代码仓库，可以轻松选择代码项目进行构建。TrcloudOS会陆续推出关联其他代码仓库的功能。

* 持续集成：TrcloudOS支持根据代码项目的tag和branch自动构建。代码仓库的push操作会自动触发一次构建保证项目镜像和开发进度同步。

* 配置dockerfile：TrcloudOS不要求代码项目必须有dockerfile，用户可以在页面上快速配置一个项目的dockerfile，并在项目设置里随时修改。
* 构建记录：TrcloudOS会记录项目的每一次构建，可以查看构建的状态、日志、构建者等全方位信息。
###业务部署

* 快速灵活：您可以用项目镜像或第三方镜像进行部署。可以一次部署多个镜像并为每个镜像的容器设定cpu和内存占用。从配置部署到启动只需要几分钟。
*
* 升级回滚：部署有完整的版本管理，每次升级会生成一个部署版本，可以随意选择一个旧版本进行回滚。部署出现异常时可以指定版本恢复。

* 弹性伸缩：运行中的部署可以随时进行扩容缩容，增减实例个数，满足业务要求。
* 健康检查：可以对部署进行TCP或HTTP检查，贯穿部署的整个生命周期，为业务的稳定健康运行保驾护航。

* 负载均衡：将流量分摊、引导到服务的每个实例，提高服务的整体可用性和吞吐量。
###集群管理与监控
* 集群设置：每个TrcloudOS集群需要配置一套kubernetes，您可以在控制台上将集群的各项设置记录下来方便添加主机、日志收集等。此外可以管理集群的namespace。

* 主机管理：可以查看集群内各个主机的配置、状态；可以随时快速添加主机；可以为主机打标签和划分生产、测试环境。
* 实例管理：可以查看每台主机上的实例列表并查看容器的日志。

* 智能监控：针对物理资源进行主机级别的监控；针对业务进行部署、实例、容器的多层级监控。用户能够从全局和底层单元多角度监控主机和服务的运行状况。
###用户管理

* 成员管理：针对组、项目、部署、集群分别设置独立的成员系统，满足不同维度的成员管理需求。可以根据成员的身份和工作界限设定资源内的权限。

* 组管理：可以创建组并将用户添加到组中。组内用户在创建项目、部署、集群时可以选择以组的身份创建。
* 支持LDAP：企业级用户可以将内部LDAP服务器关联至TrcloudOS，轻松同步所有的企业员工账号，免去注册工作。

##我们的价值

###标准

* 用docker镜像和容器分别标准化业务的集成和交付环节，统一产品开发交付的工作流程。

* 标准化的生产测试环境，避免开发测试过程中环境不统一的问题。

###智能

* 持续集成将代码集成智能化，代码push自动构建。
* 自动化运维提供智能的状态反馈、健康检查功能。

* 智能化监控，及时了解业务和主机运行状况，发现潜在的问题。

###快速

* 分钟级的构建和部署，大大提高开发交付效率。

* 极速的升级回滚和扩容缩容，让业务能够快速迭代和弹性伸缩。
* 便捷的页面操作，规范的使用流程，让用户能够快速上手，高效工作。

###全面

* 覆盖集成、部署、运维、监控等每一个产品开发运维环节，一步到位，省心省力。