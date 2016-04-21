# 私有仓库对接搜狐云存储
---
虽然用户可以在自己的集群上搭建私有仓库，但往往会面临存储空间小，迁移难，风险大的问题。私有仓库存储了您所有的镜像，如果私有仓库发生事故，会造成巨大损失，严重影响您的业务。因此，DomeOS建议您将私有仓库对接到搜狐云存储上，用稳定、安全的云存储保证你的镜像文件永久可用。

搜狐云台官方网站：cs.sohu.com

如果您希望自己安装对接搜狐云存储的私有仓库，请按照下面的步骤操作。您也可以在DomeOS控制台上的“应用商店”中，选取registry服务并部署。

操作步骤：

1. 从DomeOS公开镜像库中下载私有仓库镜像：

        pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0
2. 用以下命令启动私有仓库镜像：
        sudo docker run --restart=always -d -p <port>:5000
        -e REGISTRY_STORAGE_SOHUSTORAGE_ACCESSKEY=<your access key>
        -e REGISTRY_STORAGE_SOHUSTORAGE_SECRETKEY=<your secret key>
        -e REGISTRY_STORAGE_SOHUSTORAGE_REGION=<your region>
        -e REGISTRY_STORAGE_SOHUSTORAGE_BUCKET=<your bucket>
        --name private-registry
        pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0

 参数说明：

| 参数 | 说明 |
| -- | -- |
| port| 容器对外访问端口 |
| ACCESSKEY | 搜狐云台账户的accesskey |
|  SECRETKEY| 搜狐云台账户的secretkey|
| REGION | 存储空间所在地域。北京联通是bjcnc，北京电信是bjctc，上海电信是shctc，广州电信是gzctc |
| BUCKET | 存储空间的名称 |

3. 如果您的私有仓库需要用https访问，需加入额外参数
        -v /path/to/certs:/certs 
        -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt 
        -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key 

参数说明：

| 参数|说明|
| -- | -- |
| /path/to/certs |主机certs目录的路径 |
| CERTIFICATE | SSL证书|
| KEY | SSL密钥 |
