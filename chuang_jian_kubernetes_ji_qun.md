# 创建Kubernetes集群

----
TrcloudOS底层对接Kubernetes集群，集群配置存储服务选用etcd，容器方案选用Docker，网络方案选用Flannel，用户可以将自己搭建的Kubernetes集群添加进TrcloudOS系统中。我们提供了在Centos 7、Ubuntu 12.04、Ubuntu 14.04、Ubuntu 15.10、Ubuntu 16.04中部署Kubernetes集群的相关脚本以帮助用户创建集群。

###注意：
如果已经有了kubernetes，可以跳过该步骤，进入下一步配置。请确保kubernetes集群的node可以正常访问私有仓库

##部署etcd集群

etcd集群为Kubernetes集群、Flannel、SkyDNS提供key-value存储服务，主要用于存储Kubetnetes所有组件的定义和状态、Flannel的配置及节点状态、SkyDNS记录。我们提供了部署etcd集群的相关脚本，如下表所示：

|文件名	|说明	|下载地址
|--|
|start_etcd.sh	|etcd集群部署脚本	|http://172.30.251.187:12322/start_etcd.sh
###使用方式

* 显示帮助信息
bash start_etcd.sh help
* 部署etcd集群
在各个需要部署etcd的主机节点上执行如下命令

sudo bash start_etcd.sh [options]
###说明

1. 启动前请确认各主机的系统时间一致，如不一致可以通过ntpdate进行校准，如ntpdate ntp.sohu.com。
2. etcd建议使用2.0以上的版本，本脚本提供了2.2.1和2.3.1两个版本的下载和配置。
3. 还没启动所有节点时会报找不到未启动节点的错误，属正常现象，所有节点启动后将不报该错误。
4. 同一etcd集群所有节点的--peer-port参数值和--cluster-nodes参数值（包括IP地址的顺序）必须完全一致。

###参数说明

--cluster-nodes: 集群所有节点的IP列表，各IP间以半角逗号分隔。（必需）

--client-port: 客户端通信监听端口，默认为 4012。

--data-path: 数据目录，默认为 /var/lib/etcd。

--etcd-version: etcd版本，默认为 2.3.1，可选值为{2.2.1, 2.3.1}。

--peer-port: 节点间通信监听端口，默认为 4010。
###样例

若用于部署etcd集群的主机为:

10.11.150.100
10.11.150.101
10.11.150.102

各节点的客户端通信监听端口为4012，数据存储目录为/opt/etcd/data，etcd版本为2.3.1，节点间通信监听端口为4010，则需要在三台主机上分别执行：

sudo sh start_etcd.sh \

    --cluster-nodes 10.11.150.100,10.11.150.101,10.11.150.102 \
    --client-port 4012 \
    --data-path /opt/etcd/data \
    --etcd-version 2.3.1 \
    --peer-port 4010
###验证

通过 curl -L <etcd节点IP>:<client-port>/health 查看各个节点服务状态，若返回true说明该节点上的etcd正常。

curl -L 10.11.150.100:4012/health
{"health": "true"}
###参考

etcd官方地址：https://coreos.com/etcd/

##部署 Kubernetes master

我们提供了在CentOS和Ubuntu中部署Kubernetes master的脚本，通过该脚本将安装和启动如下组件：Flannel、Docker、kube-apiserver、kube-controller-manager、kube-scheduler、kube-proxy。脚本下载地址如下：

|文件名	|说明	|下载地址
|--|
|start_master_centos.sh	|在CentOS中部署Kubernetes master	|http://172.30.251.187:12322/start_master_centos.sh

###使用方式

* 显示帮助信息

CentOS系统中执行如下命令

   bash start_master_centos.sh help


* 部署master
CentOS系统中执行如下命令

sudo bash start_master_centos.sh [options]

###说明

1. 脚本目前支持在CentOS 7、Ubuntu12.04、Ubuntu 14.04、Ubuntu 15.10和Ubuntu 16.04系统中部署。
2. 脚本中安装docker与下载Kubernetes组件和Flannel组件需要连接互联网。如果所在主机无法访问外网，可先将相关组件放到脚本所在目录，注释脚本中下载组件的语句，并在本地安装完docker后再执行该脚本。
3. 脚本成功执行后，将以systemd的形式启动如下服务: flanneld, docker, kube-apiserver, kube-controller-manager, kube-scheduler, kube-proxy。
4. --service-cluster-ip-range和--flannel-network-ip-range地址段不能有重叠。

###参数说明

--cluster-dns: 集群dns服务的ip地址，注意不加端口号，IP地址需落在--service-cluster-ip-range范围内，默认172.16.40.1。注意：此时dns服务尚未配置，应记下这一地址，配置dns服务时使用这一地址作为dns服务的ClusterIP。

--cluster-domain: 集群的search域，默认为domeos.local。注意：此时dns服务尚未配置，应记下这一配置项，配置dns服务是使用这一配置项作为dns服务的domain。

--docker-graph-path: docker运行时的根路径，容器、本地镜像等会存储在该路径下，占用空间大，建议设置到大容量磁盘上，默认为脚本所在路径下的docker-graph目录。

--docker-registry-crt: 如果设置了--secure-docker-registry，则该值必需，表示私有仓库的证书。

--etcd-servers: etcd服务集群地址，各地址间以逗号分隔。(必需)

--flannel-network-ip-range: flannel网络可用地址范围，默认172.24.0.0/13。

--flannel-subnet-len: 分配给flannel节点的子网长度，默认22。

--flannel-version: Flannel版本。默认0.5.5，可选版本为{0.5.5}。

--insecure-bind-address: kube-apiserver非安全访问服务地址，默认0.0.0.0。

--insecure-port: kube-apiserver非安全访问服务端口，默认8080。

--insecure-docker-registry: 非安全连接的docker私有仓库，可以先设置该值而不用启动对应的registry。(必需)

--kube-apiserver-port: kube-apiserver服务端口，默认8080。

--kubernetes-version: Kubernetes版本。默认1.2.0，可选版本为{1.1.3，1.1.7，1.2.0，1.2.4}。

--service-cluster-ip-range: kubernetes集群service的可用地址范围，默认172.16.0.0/13。

--secure-docker-registry: 安全连接的docker私有仓库，默认为空。

###样例

* 最简参数

    sudo bash start_master_centos.sh \
    
        --etcd-servers http://10.11.151.100:4012 \
        --insecure-docker-registry 10.10.10.10:5000
        

* 最全参数

    sudo bash start_master_centos.sh \
    
        --cluster-dns 172.16.40.1 \
        --cluster-domain domeos.local \
        --docker-graph-path /opt/domeos/openxxs/docker \
        --docker-registry-crt /opt/domeos/openxxs/k8s-1.1.7-flannel/registry.crt \
        --etcd-servers http://10.11.150.99:4012,http://10.11.150.100:4012,http://10.11.150.101:4012 \
        --flannel-network-ip-range 172.24.0.0/13 \
        --flannel-subnet-len 22 \
        --flannel-version 0.5.5 \
        --insecure-bind-address 0.0.0.0 \
        --insecure-port 8080 \
        --insecure-docker-registry 10.10.10.10:5000 \
        --kube-apiserver-port 8080 \
        --kubernetes-version 1.2.0 \
        --service-cluster-ip-range 172.16.0.0/13 \
        --secure-docker-registry https://registry.test.com
        
###验证

$ sudo /usr/sbin/domeos/k8s/current/kubectl cluster-info

Kubernetes master is running at http://localhost:8080
###参考

1. Kubernetes官方文档：http://kubernetes.io

2. Docker安装包(rpm)下载地址：https://yum.dockerproject.org/repo/main/centos/7/Packages/

3. Docker安装包(deb)下载地址：https://apt.dockerproject.org/repo/pool/main/d/docker-engine/

##在Kubetnetes集群中添加node

我们提供了在CentOS和Ubuntu中部署Kubernetes node的脚本，通过该脚本将安装和启动如下组件：Flannel、Docker、kube-proxy、kubelet。脚本下载地址如下：

|文件名	|说明	|下载地址
|--|
|start_node_centos.sh	|在CentOS中部署Kubernetes node	|http://172.30.251.187:12322/start_node_centos.sh

###使用方式

* 显示帮助信息
CentOS系统中执行如下命令

bash start_node_centos.sh help

* 部署node

CentOS系统中执行如下命令

sudo bash start_node_centos.sh [options]

###说明

1. 脚本目前支持在CentOS 7、Ubuntu12.04、Ubuntu 14.04、Ubuntu 15.10和Ubuntu 16.04系统中部署。
2. 脚本会对hostname进行检查，如果不符合dns规范，请通过change_hostname.sh脚本对主机hostname进行修改。
3. 在TrcloudOS系统中，node包含标签PRODENV=HOSTENVTYPE表示可用于生产环境；包含标签TESTENV=HOSTENVTYPE表示可用于测试环境；包含标签BUILDENV=HOSTENVTYPE表示可用于构建镜像。
4. 在添加node时集群的dns服务可不启动。
5. 脚本中安装docker与下载kubernetes相关组件需要连接互联网。如果所在主机无法访问外网，可先将相关组件放到脚本所在目录，注释脚本中下载组件的语句，并在本地安装完docker后再执行该脚本。
6. 若--start-agent设置为true，脚本将下载并启动agent镜像。如果所在主机无法访问外网，可以将该镜像上传至私有镜像仓库，并修改脚本中镜像名称为私有镜像仓库中的相应镜像，再执行脚本。
7. 如果设置了私有仓库以https的方式访问，脚本会从TrcloudOS Server上下载证书文件(registry.crt)，因此需要当前主机可访问TrcloudOS Server。
8. 脚本成功执行后，将以systemctl的形式启动flanneld、docker、kube-proxy、kubelet，将以容器形式启动用于监控数据上报和作为WebSSH Client端的agent，并为该节点打上指定的标签。

###参数说明

--api-server: kubernetes集群kube-apiserver服务地址，必需项，格式为"ip:port"。

--cluster-dns: kubernetes集群内dns服务地址，与kubernetes master启动参数一致，默认为"172.16.40.1"。

--cluster-domain: kubernetes集群内dns服务的search域，与kubernetes master启动参数一致，默认为"domeos.local"。

--docker-graph-path: docker运行时的根路径，容器、本地镜像等会存储在该路径下，占用空间大，建议设置到大容量磁盘上，默认为"/var/lib/docker"。

--docker-log-level: docker daemon的日志级别，可选值debug/info/warn/error/fatal。kubernetes会定时查询所有容器状态，导致docker daemon输出大量info级别日志到系统日志中(/var/log/message)，建议日志级别设置为warn级别以上，默认为warn。

--trcloudos-server: TrcloudOS Server服务地址，当--registry-type=https时为必需项。

--etcd-server: etcd集群服务地址，各个地址间以半角逗号分隔，格式为'http://IP:Port'，必需项。

--flannel-version: Flannel版本。默认0.5.5，可选版本为{0.5.5}。

--heartbeat-addr: 监控hbs服务监听地址，只能配置一个。

--hostname-override: 若设置了该参数，则向Kubernetes集群中注册节点时使用该参数值作为节点的名称。默认值为节点的hostname。

--k8s-data-dir: Kubelet产生的相关文件的存储目录，包括emptyDir，默认为"/var/lib/kubelet"。

--kubernetes-version: Kubernetes版本，必须与对应的Kubernetes master版本相同。默认1.2.0，可选版本为{1.1.3，1.1.7，1.2.0，1.2.4}。

--monitor-transfer: 监控transfer服务地址，可以填多个，各个地址间以逗号分隔。若设置了--heartbeat-addr的值，则此项为必需项。当设置了--monitor-transfer时，节点上会以docker容器形式启动名为agent的容器，用于收集监控信息。

--node-labels: 为node打上的标签，非必需。

--registry-type: 私有仓库类型，取值为http或https，必需项。

--registry-arg: 私有仓库的地址，可以为域名地址，必需项。
###样例

sudo bash start_node_centos.sh \

--api-server http://10.10.10.10:8081 \

--cluster-dns 172.16.40.1 \

--cluster-domain domeos.local \

--docker-graph-path /opt/domeos/openxxs/docker-graph \

--docker-log-level warn \

--trcloudos-server 10.10.10.10:8080 \

--etcd-server http://10.11.150.100:4012,http://10.11.150.101:4012,http://10.11.150.102:4012 \

--flannel-version 0.5.5 \

--heartbeat-addr 10.10.10.10:6030 \

--hostname-override test-host \

--k8s-data-dir /opt/domeos/openxxs/k8s-data \

--kubernetes-version 1.2.0 \

--monitor-transfer 10.10.10.10:8433,10.10.10.11:8433 \

--node-labels TESTENV=HOSTENVTYPE,PRODENV=HOSTENVTYPE \

--registry-type http \

--registry-arg 10.10.10.10:5000
###验证

使用kubectl --server <kube-apiserver服务地址> get node命令查看集群的node节点状态，若显示该节点的状态为"Ready"，说明该node添加成功。如：

kubectl --server 10.10.10.10:8081 get nodes
NAME         STATUS    AGE
tc-151-100   Ready     25d
###参考

1. Kubernetes官方文档：http://kubernetes.io

2. Docker安装包(rpm)下载地址：https://yum.dockerproject.org/repo/main/centos/7/Packages/

3. Docker安装包(deb)下载地址：https://apt.dockerproject.org/repo/pool/main/d/docker-engine/