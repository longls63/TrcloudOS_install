#配置监控报警组件

---

TrcloudOS中的监控部分主要使用小米open-falcon框架，并针对TrcloudOS中的监控需求对一些模块进行了二次开发。TrcloudOS中监控模块与其他模块基本独立，每一模块都可以使用镜像方式启动，使用环境变量在各模块之间传递参数信息。其中agent是监控数据上报组件，需要在集群中每台主机部署，其余组件可单独部署。主要模块如下表所示：

| 模块名称| 功能说明| 镜像地址| 是否在falcon原生组件基础上进行二次开发|
|-- |
| agent| 监控信息上报 | 172.30.248.190:5000/domeos/agent:2.5 | 是|
| transfer| 监控信息转发 | 172.30.248.190:5000/domeos/transfer:0.0.15 | 否，仅修改MIN_STEP为10|
| graph| 监控信息存储 | 172.30.248.190:5000/domeos/graph:0.5.7 | 否，仅修改MIN_STEP为10|
| query| 监控信息查询 | 172.30.248.190:5000/domeos/query:1.5.1 | 否|
| hbs| 心跳服务器 | 172.30.248.190:5000/domeos/hbs:1.1.0 | 否|
| judge| 报警事件判断 | 172.30.248.190:5000/domeos/judge:2.0.2 | 否|
| alarm| 报警事件处理 | 172.30.248.190:5000/domeos/alarm:1.0 | 是|
| sender| 报警事件发送 | 172.30.248.190:5000/domeos/sender:1.0 | 是|
| nodata| 假数据填充 | 172.30.248.190:5000/domeos/nodata:0.0.8 | 否|
| redis| 报警事件队列 | 172.30.248.190:5000/domeos/redis:3.0.7 | 否|

**注意**

1.启动容器时，传入环境变量中含有转义双引号的地方引号不可省略，须按照样例格式设定。

2.直接restart组件容器可能导致组件启动失败或运行不正常。须检测各组件运行状态及日志，若不正常须自行重新启动组件。

#domeos/agent

domeos/agent模块是以open-falcon原生agent模块为基础，为适应TrcloudOS监控需求而设计修改的，与原生open-falcon的agent主要区别为：

- 在配置模板中对很大部分监控项设置了忽略，仅保留一些系统基础监控项

- 嵌入了cadvisor代码，实现容器监控，其tag为id={容器64位id}

- 镜像中集成了TrcloudOS WebSSH客户端，默认监听2222端口上的SSH请求，配合WebSSH Server实现Web端容器登录

domeos/agent模块需在集群中所有node节点部署。若使用TrcloudOS给定的脚本添加主机，且指定--start-agent参数为true，默认将自动启动agent容器；否则需要手动启动agent容器。

**启动命令**

`sudo docker run -d --restart=always \ -p 2222:2222 \ -p <_agent_http_port>:1988 \ -e HOSTNAME="\"<_hostname>\"" \ -e TRANSFER_ADDR="[<_transfer_addr>]" \ -e TRANSFER_INTERVAL="<_interval>" \ -e HEARTBEAT_ENABLED="<_heartbeat_enable>" \ -e HEARTBEAT_ADDR="\"<_heartbeat>\"" \ -v /:/rootfs:ro \ -v /var/run:/var/run:rw \ -v /sys:/sys:ro \ -v <_docker_graph_path>:<_docker_graph_path> \ --name agent \ 172.30.248.190:5000/domeos/agent:2.5`

**参数说明**

`_agent_http_port: agent服务http端口，主要用于状态检测、调试等。`

`_hostname: 监控系统中主机的endpoint名，需与kubernetes中添加node时配置的hostname相同。TrcloudOS添加主机脚本中使用主机运行hostname命令的执行结果。 `

`_transfer_addr: transfer的rpc地址，可以配置多个。注意每个IP:Port需加双引号，多个IP:Port之间用逗号分隔。TrcloudOS添加主机脚本中使用TrcloudOS全局配置中的transfer配置。`

`_interval: 监控数据上报时间间隔，单位为秒(s)。TrcloudOS中支持的最小上报时间间隔为10s。TrcloudOS添加主机脚本中默认设置为10s。 `

`_heartbeat_enable: 是否连接心跳服务器(hbs)，可设置为true或false。若需要使用报警服务，必须将其配置为true。TrcloudOS添加主机脚本中默认设置为true。 `

`_heartbeat: 心跳服务器(hbs)rpc监听地址，格式为IP:Port。TrcloudOS添加主机脚本中使用TrcloudOS全局配置中的hbs配置。`

`_docker_graph_path: docker启动时配置的graph目录路径(-g参数)。若没有配置，默认为/var/lib/docker目录`

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

sudo docker run -d --restart=always \

 -p 2222:2222 \

 -p 1988:1988 \

 -e HOSTNAME="\"test-host\"" \

 -e TRANSFER_ADDR[\"10.10.10.10:8433\",\"10.10.10.11:8433\"]" \

 -e TRANSFER_INTERVAL="10" \

 -e HEARTBEAT_EN__ABLED="true" \

 -e HEARTBEAT_ADDR="\"10.10.10.10:6030\"" \

 -v /:/rootfs:ro \

 -v /var/run:/var/run:rw \

 -v /sys:/sys:ro \

 -v /var/lib/docker/:/var/lib/docker:ro \

 --name agent \

 172.30.248.190:5000/domeos/agent:2.5

**验证**

通过curl -s localhost:<_agent_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/transfer

TrcloudOS使用open-falcon原生的transfer组件，仅将g.go文件中的MIN_STEP常量更新为10，最小监控数据取样间隔10s。实际部署时，transfer组件可在多台主机上配置多个实例，需要在agent中配置正确的transfer地址。

**启动命令**

`sudo docker run -d --restart=always \ -p <_transfer_http_port>:6060 \ -p <_transfer_rpc_port>:8433 \ -e JUDGE_CLUSTER="<_judge_cluster>" \ -e GRAPH_CLUSTER="<_graph_cluster>" \ --name transfer \ 172.30.248.190:5000/domeos/transfer:0.0.15`

**参数说明**

`_transfer_http_port: transfer服务http端口，主要用于状态检测、调试等。 `

`_transfer_rpc_port: transfer服务rpc端口，主要用于接收agent上报监控数据。 `

`_judge_cluster: judge服务节点列表，格式为键值对，多个节点之间以逗号分隔。 `

`_graph_cluster: graph服务节点列表，格式为键值对，多个节点之间以逗号分隔。`

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 6060:6060 \ -p 8433:8433 \ -e JUDGE_CLUSTER="\"judge-00\" : \"10.10.10.10:6080\", \"judge-01\" : \"10.10.10.11:6080\"" \ -e GRAPH_CLUSTER="\"graph-00\" : \"10.10.10.10:6070\", \"graph-01\" : \"10.10.10.11:6070\"" \ --name transfer \ 172.30.248.190:5000/domeos/transfer:0.0.15`

**验证**

通过curl -s localhost:<_transfer_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/graph

TrcloudOS使用open-falcon原生的graph组件，仅将g.go文件中的MIN_STEP常量更新为10，最小监控数据取样间隔10s。实际部署时，graph组件可在多台主机上配置多个实例，需要在transfer和query中配置正确的graph地址。

**启动命令**

`sudo docker run -d --restart=always \

 -p <_graph_http_port>:6071 \

 -p <_graph_rpc_port>:6070 \

 -v <_graph_data_path>:/home/work/data/6070 \

 -e DB_DSN="\"<_graph_db_user>:<_graph_db_passwd>@tcp(<_graph_db_addr>)/graph?loc=Local&parseTime=true\"" \

 --name graph \ 172.30.248.190:5000/domeos/graph:0.5.7`

**主要参数说明：**

`_graph_http_port: graph服务http端口，主要用于状态检测、调试等。`

`_graph_rpc_port: graph服务rpc端口，主要用于与transfer和query交互。 `

`_graph_data_path: graph数据文件存储路径。`

`_graph_db_user: TrcloudOS中MySQL数据库的用户名。`

`_graph_db_passwd: TrcloudOS中MySQL数据库的密码。`

`_graph_db_addr: TrcloudOS中MySQL数据库的地址，格式为IP:Port。`

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 6071:6071 \ -p 6070:6070 \ -v /opt/graph/data:/home/work/data/6070 \ -e DB_DSN="\"root:root@tcp(10.10.10.10:3306)/graph?loc=Local&parseTime=true\"" \ --name graph \ 172.30.248.190:5000/domeos/graph:0.5.7`

**验证**

通过curl -s localhost:<_graph_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/query

TrcloudOS使用open-falcon原生的query组件。实际部署时，query组件只需配置单个实例。

**启动命令**

`sudo docker run -d --restart=always \ -p <_query_http_port>:9966 \ -e GRAPH_CLUSTER="<_graph_cluster>" \ --name query \ 172.30.248.190:5000/domeos/query:1.5.1`

**参数说明**

`_query_http_port: query服务http端口，主要用于查询监控数据、状态检测、调试等。 _graph_cluster: graph服务节点列表，格式为键值对，多个节点之间以逗号分隔。注意一定要与transfer中配置的_graph_cluster相同。`

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 9966:9966 \ -e GRAPH_CLUSTER="\"graph-00\" : \"10.10.10.10:6070\", \"graph-01\" : \"10.10.10.11:6070\"" \ --name query \ 172.30.248.190:5000/domeos/query:1.5.1`

**验证**

通过curl -s localhost:<_query_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/hbs

TrcloudOS使用open-falcon原生的心跳服务器(hbs)组件。实际部署时，hbs组件只需配置单个实例。

**启动命令**

`sudo docker run -d --restart=always \ -p <_hbs_http_port>:6031 \ -p <_hbs_rpc_port>:6030 \ -e DATABASE="\"<_portal_db_user>:<_portal_db_passwd>@tcp(<_portal_db_addr>)/portal?loc=Local&parseTime=true\"" \ --name hbs \ 172.30.248.190:5000/domeos/hbs:1.1.0`

**参数说明**

`_hbs_http_port: hbs服务http端口，主要用于状态检测、调试等。 `

`_hbs_rpc_port: hbs服务rpc监听端口，主要用于监听agent的上报数据。 `

`_portal_db_user: TrcloudOS中MySQL数据库的用户名。`

`_portal_db_passwd: TrcloudOS中MySQL数据库的密码。`

`_portal_db_addr: TrcloudOS中MySQL数据库的地址，格式为IP:Port。`

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 6031:6031 \ -p 6030:6030 \ -e DATABASE="\"root:root@tcp(10.10.10.10:3306)/portal?loc=Local&parseTime=true\"" \ --name hbs \ 172.30.248.190:5000/domeos/hbs:1.1.0`

**验证**

通过curl -s localhost:<_hbs_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/judge

TrcloudOS使用open-falcon原生的judge组件。实际部署时，judge组件可在多台主机上配置多个实例。

**启动命令**

`sudo docker run -d --restart=always \ -p <_judge_http_port>:6081 \ -p <_judge_rpc_port>:6080 \ -e HBS_SERVERS="[\"<_hbs_server>\"]" \ -e ALARM_REDIS_DSN="\"<_redis>\"" \ --name judge \ 172.30.248.190:5000/domeos/judge:2.0.2`

**参数说明**

_judge_http_port: judge服务http端口，主要用于状态检测、调试等。 _judge_rpc_port: hbs服务rpc端口，主要用于与transfer的交互。 _hbs_server: 心跳服务器(hbs)rpc监听地址，格式为IP:Port。 _redis: 用于报警的redis服务地址，格式为IP:Port。

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 6081:6081 \ -p 6080:6080 \ -e HBS_SERVERS="[\"10.10.10.10:6030\"]" \ -e ALARM_REDIS_DSN="\"10.10.10.10:6379\"" \ --name judge \ 172.30.248.190:5000/domeos/judge:2.0.2`

**验证**

通过curl -s localhost:<_judge_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/alarm

domeos/alarm模块是以open-falcon原生alarm模块为基础，为适应TrcloudOS监控报警需求而设计修改的。实际部署时，alarm组件只需配置单个实例。在TrcloudOS中，不再单独部署portal模块，报警策略信息由TrcloudOS报警模块提供并写入portal库，供hbs读取。alarm从redis中获取报警event后将保存至TrcloudOS数据库， 以提供TrcloudOS前端展示报警信息。

**启动命令**

`sudo docker run -d --restart=always \ -p <_alarm_http_port>:9912 \ -e DATABASE="\"<_domeos_db_user>:<_domeos_db_passwd>@tcp(<_domeos_db_addr>)/domeos?loc=Local&parseTime=true\"" \ -e REDIS_ADDR="\"<_redis>\"" \ -e API_DOMEOS="\"<_domeos_server>\"" \ --name alarm \ 172.30.248.190:5000/domeos/alarm:1.0`

**参数说明**

_alarm_http_port: alarm服务http端口，主要用于状态检测、调试等。 _domeos_db_user: TrcloudOS中MySQL数据库的用户名。 _domeos_db_passwd: TrcloudOS中MySQL数据库的密码。 _domeos_db_addr: TrcloudOS中MySQL数据库的地址，格式为IP:Port。 _redis: 用于报警的redis服务地址，格式为IP:Port。 _domeos_server: TrcloudOS的server地址。

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 9912:9912 \ -e DATABASE="\"root:root@tcp(10.10.10.10:3306)/domeos?loc=Local&parseTime=true\"" \ -e REDIS_ADDR="\"10.10.10.10:6379\"" \ -e API_DOMEOS="\"http://domeos.example.com\"" \ --name alarm \ 172.30.248.190:5000/domeos/alarm:1.0`

**验证**

通过curl -s localhost:<_alarm_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/sender

domeos/sender模块是以open-falcon原生sender模块为基础，为适应TrcloudOS监控报警需求而设计修改的。实际部署时，sender组件只需配置单个实例。

在TrcloudOS中，api:sms和api:mail改为由TrcloudOS的数据库全局配置中读取得到。当TrcloudOS全局配置改变时TrcloudOS服务器将主动调用sender接口/config/api/reload 从TrcloudOS数据库中更新新的短信和邮件API接口；同时sender也会每隔60s自动向TrcloudOS数据库拉取短信和邮件API接口。

注意：api:sms和api:mail需要满足open-falcon定制的接口规范，具体见官方github中的相关说明。

**启动命令**

`sudo docker run -d --restart=always \ -p <_sender_http_port>:6066 \ -e DATABASE="\"<_domeos_db_user>:<_domeos_db_passwd>@tcp(<_domeos_db_addr>)/domeos?loc=Local&parseTime=true\"" \ -e REDIS_ADDR="\"<_redis>\"" \ --name sender \ 172.30.248.190:5000/domeos/sender:1.0`

**参数说明**

_sender_http_port: sender服务http端口，主要用于状态检测、调试等。 _domeos_db_user: TrcloudOS中MySQL数据库的用户名。 _domeos_db_passwd: TrcloudOS中MySQL数据库的密码。 _domeos_db_addr: TrcloudOS中MySQL数据库的地址，格式为IP:Port。 _redis: 用于报警的redis服务地址，格式为IP:Port。

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 6066:6066 \ -e DATABASE="\"root:root@tcp(10.10.10.10:3306)/domeos?loc=Local&parseTime=true\"" \ -e REDIS_ADDR="\"10.10.10.10:6379\"" \ --name sender \ 172.30.248.190:5000/domeos/sender:1.0`

**验证**

通过curl -s localhost:<_sender_http_port>/health命令查看运行状态，若运行正常将返回ok。

#domeos/nodata

TrcloudOS使用open-falcon原生的nodata组件。实际部署时，nodata组件只需配置单个实例。

**启动命令**

`sudo docker run -d --restart=always \ -p <_nodata_http_port>:6090 \ -e QUERY_QUERYADDR="\"<_query>\"" \ -e CONFIG_DSN="\"<_portal_db_user>:<_portal_db_passwd>@tcp(<_portal_db_addr>)/portal?loc=Local&parseTime=true\"" \ -e SENDER_TRANSFERADDR="\"<_transfer>\"" \ --name nodata \ 172.30.248.190:5000/domeos/nodata:0.0.8`

**参数说明**

`_nodata_http_port: nodata服务http端口，主要用于状态检测、调试等。 `

`_query: query组件的http地址，格式为IP:Port。 `

`_portal_db_user: TrcloudOS中MySQL数据库的用户名。 `

`_portal_db_passwd: TrcloudOS中MySQL数据库的密码。`

`_portal_db_addr: TrcloudOS中MySQL数据库的地址，格式为IP:Port。 `

`_transfer: transfer组件的http地址，若部署多个transfer则填写其中一个，格式为IP:Port。`

其余参数可参照工程中的docker/cfg.template.json文件使用相应环境变量进行修改。

**样例**

`sudo docker run -d --restart=always \ -p 6090:6090 \ -e QUERY_QUERYADDR="\"10.10.10.10:9966\"" \ -e CONFIG_DSN="\"root:root@tcp(10.10.10.10:3306)/portal?loc=Local&parseTime=true\"" \ -e SENDER_TRANSFERADDR="\"10.10.10.10:6060\"" \ --name nodata \ 172.30.248.190:5000/domeos/nodata:0.0.8`

**验证**

通过curl -s localhost:<_nodata_http_port>/health命令查看运行状态，若运行正常将返回ok。

#redis

redis是报警功能正常运行的必要组件。可用容器方式启动redis服务：

`sudo docker run -d \ -p <_redis_port>:6379 \ 172.30.248.190:5000/domeos/redis:3.0.7 \ redis-server --timeout 300 --maxmemory 5gb --maxmemory-policy allkeys-lru --port 6379`

其中_redis_port为redis服务对外提供的服务端口。

#MySQL与redis配置相关说明

部署TrcloudOS说明中已经包含MySQL的相关配置。TrcloudOS的MySQL数据库中包含三张表：domeos、graph与portal。其中domeos是TrcloudOS server主要使用的表，graph与portal是为了与open-falcon兼容而单独创建的表。这三张表在TrcloudOS监控报警组件中都有使用，为确保监控报警功能正常，务必保证TrcloudOS数据库正常运行可访问。

Redis用作报警事件的队列，起到联系judge、alarm和sender组件的关键作用。为确保报警功能正常，务必保证redis组件正常运行。



