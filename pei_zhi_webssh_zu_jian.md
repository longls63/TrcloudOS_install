# 配置WebSSH组件

---

# 启动WebSSH Server

**注意**

WebSSH Server组件已经集成进TrcloudOS Server镜像，若使用容器方式启动TrcloudOS Server，可跳过这一步骤。

WebSSH Server由开源项目shellinabox修改而来，完成向指定主机上的指定容器发送SSH登录请求，并在Web端模拟终端交互过程，使用容器方式启动。

**镜像**

`172.30.248.190:5000/domeos/shellinabox:1.1`

**启动命令**

`sudo docker run -d --restart=always \ `

 -p <_port>:4200 \

 --name shellinabox \

 172.30.248.190:5000/domeos/shellinabox:1.1`

**参数说明**

`_port: WebSSH服务端口。`

**样例**

sudo docker run -d --restart=always \

 -p 4200:4200 \

 --name shellinabox \

 172.30.248.190:5000/domeos/shellinabox:1.1

**验证**

通过curl -s http:// WebSSH Server服务地址 /domeos/health查看WebSSH Server服务状态，若返回"ok"，说明WebSSH服务运行正常，如:

curl -s 10.10.10.10:4200/domeos/health

ok

# 启动WebSSH Client

WebSSH Client组件已经集成进agent镜像，无需单独部署。启动agent容器时暴露的2222端口即为WebSSH Client对外提供的SSH连接服务端口。




