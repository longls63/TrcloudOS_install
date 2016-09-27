# 配置私有镜像仓库

---
##启动私有镜像仓库容器

TrcloudOS可直接使用官方的registry镜像，私有仓库存储了所有的镜像，如果私有仓库发生事故，会造成巨大损失，严重影响业务，因此建议将私有仓库挂载到本地目录，保证镜像文件永久可用。启动过程请参考Docker Hub官方说明。


### 注意：
TrcloudOS目前仅支持v2版本registry，请确保部署registry版本正确。如果已经有私有仓库，可以跳过该步骤，进入下一步配置。

###镜像
http://172.30.248.190:5000/registry:2.3.0
###启动命令

sudo docker run --restart=always -d \

    -p <_registry_port>:5000 \
    -v <_local_data>:<_contain_registry>
    --name private-registry \
    http://172.30.248.190:5000/registry:2.3.0
###参数说明

_registry_port：私有仓库对外服务端口

_local_data: 本地挂载目录

_contain_registry: 私有仓库地址

###样例

sudo docker run -d --restart=always \

    -p 5000:5000 \
    --restart=always 
    -v /registries:/var/lib/registry registry:2.3.0
    --name private-registry \
    http://172.30.248.190:5000/registry:2.3.0
    
###验证

通过curl -L http://<私有仓库的服务地址>/v2/查看私有仓库的运行状态，如果显示"{}"说明正常运行，如:

curl -L http://10.10.10.10:5000/v2/

{}
###参考

https://github.com/docker/distribution