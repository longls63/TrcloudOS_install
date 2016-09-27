# 启动DNS服务

---
DNS组件在kubernetes集群内部提供了dns解析服务，在获取容器的实时日志时，kubernetes master需要hostname到ip的解析，TrcloudOS的实现方式通过集群内dns组件解析hostname。这个功能也可以通过配置master的host实现。DNS组件还可以为提供集群内域名，node和pod可以直接通过域名访问service。在启动kubernetes相关组件时需要添加DNS信息。

##配置集群内dns服务

在创建kubernetes集群后，一套Kubernetes集群已经创建完毕并加入若干主机，集群内dns服务将通过yaml文件以Kuberbetes Service形式启动。相关文件如下表所示：

|文件名	|说明	|下载地址
|--|
|dns.yaml	|dns服务启动yaml文件	|http://172.30.251.187:12322/dns/dns.yaml
|kubectl	|执行文件	|见上述步骤程序包
###启动命令

kubectl --server <kube-apiserver服务地址> create -f dns.yaml

###参数说明

执行启动命令前需要修改dns.yaml文件中的如下参数：

skydns-svc service中的clusterIP为部署Kubernetes集群时设置的--cluster-dns。

skydns RC中args下的--machines为etcd集群的服务地址，各个地址间以逗号分隔，必须带" http:// "前缀。

skydns RC中args下的--nameservers为集群中的主机使用的外部dns服务(CentOS系统中为/etc/resolv.conf下配置的非集群nameserver)，带端口号，多个DNS服务则以半角逗号分隔。

skydns RC中args下的--domain为部署Kubernetes集群时设置的--cluster-domain。

kube2sky RC中args下的--etcd-server为etcd的服务地址，有且仅能设置一个，且必需在--machines设置的集群中，必须带" http:// "前缀。

kube2sky RC中args下的--domain为部署Kubernetes集群时设置的--cluster-domain。

###说明

注意在执行启动命令前按需修改dns.yaml文件中的参数。

执行完毕后将创建名为skydns-svc的service，名为skydns的RC和名为kube2sky的RC。

###样例

    kubectl --server 10.10.10.10:8080 create -f dns.yaml
    
###验证

1. 通过以下命令获取集群svc列表，确认skydns-svc是否已创建：

 kubectl --server <kube-apiserver服务地址> get svc
2. 通过以下命令获取集群pod列表，确认skydns-为前缀和kube2sky-为前缀的两个pod是否处于Running状态，如skydns-u44ey和kube2sky-2h1b9：

 kubectl --server <kube-apiserver服务地址> get pods
3. 查看node上的/etc/resolv.conf文件，确认前两行是否为如下内容(其中的domeos.local为--cluster-domain参数的值，172.16.40.1为--cluster-dns参数的值):

 search default.svc.domeos.local svc.domeos.local domeos.local
 nameserver 172.16.40.1
若没有，添加如上格式内容

4. 在node上进行dns解析验证，方式有多种，如：

 nslookup skydns-svc.default.svc.domeos.local
如果没有安装nslookup，也可通过ping间接验证：

 ping skydns-svc.default.svc.domeos.local -c 1
解析出IP地址则说明dns服务创建成功。

5. 在通过kubernetes创建的容器内部验证，方法同步骤4。