# 系统安装及初始化
---
 DomeOS公开镜像仓库： pub.domeos.org
 
安装组件的镜像下载地址（更新于2016-04-21）：

| 安装组件| 镜像地址 |
| -- | -- |
| 私有仓库 | pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0 |
| skyDNS | pub.domeos.org/domeos/skydns:1.5|
| kube2sky | pub.domeos.org/domeos/kube2sky:0.4 |
| agent | pub.domeos.org/domeos/agent:2.4|
| transfer | pub.domeos.org/domeos/transfer:0.0.15 |
| graph | pub.domeos.org/domeos/graph:0.5.7 |
| query | pub.domeos.org/domeos/query:1.5.1 |
| DomeOS server | pub.domeos.org/domeos/server:1.2.3 |

安装步骤：

**步骤1. 配置和启动ETCD集群**

    脚本: start_etcd.sh 
   
  下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/start_etcd.sh

    配置文件: etcd_node_list.conf
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/etcd_node_list.conf
    
    可执行文件: etcd 
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/etcd

说明: 
1.	下载上述三个文件并放在相同目录
2.	将用于部署ETCD集群的主机IP列表写入etcd_node_list.conf中，每行一个IP。
3.	启动前请确认各主机的系统时间一致，如不一致请通过ntpdate进行校准，如 ntpdate ntp.sohu.com。
4.	脚本中默认数据存储目录为脚本所在目录的etcd-data子目录下，默认peer port为4010，默认client port为4012，如果需要可以在脚本文件中的第二步进行修改。
5.	如果主机中已安装过ETCD，则ETCD从低版本升级到高版本时，需要删除老数据文件（ETCD自身bug）。
6.	使用方式：sudo sh start_etcd.sh <当前主机在etcd_node_list.conf中的位置，从1开始。

样例: 

    etcd_node_list.conf文件内容如下:
    ==================================
      10.11.151.97
      10.11.151.100
      10.11.151.101
    ==================================

    则需要:
  
    在10.11.151.97主机上执行sudo sh start_etcd.sh 1
  
    在10.11.151.100主机上执行sudo sh start_etcd.sh 2
  
    在10.11.151.101主机上执行sudo sh start_etcd.sh 3
    
**步骤2：配置和启动kubernetes master端**

    脚本: start_master_centos.sh 
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/start_master_centos.sh
    
    辅助脚本: change_hostname.sh 
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/change_hostname.sh

    程序包: domeos-k8s-master.tar.gz 
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/domeos-k8s-master.tar.gz ，包含: flanneld, mk-docker-opts.sh, kube-apiserver, kube-controller-manager, kube-scheduler, kube-proxy

说明:
  1. 脚本会对hostname进行检查，如果不符合DNS规范，请通过change_hostname.sh脚本对主机hostname进行修改。
  2. 脚本中仅有安装docker时需要连接互联网，如果所在主机无法访问外网，可先本地安装完docker后再执行该脚本。
  3. 脚本成功执行后，将以systemctl的形式启动如下服务: flanneld, docker, kube-apiserver, kube-controller-manager, kube-scheduler, kube-proxy
  4. 摘除master可按如下顺序停止各组件: kube-proxy -> kube-scheduler -> kube-controller-manager -> kube-apiserver -> docker -> flannel
  5. --service-cluster-ip-range和--flannel-network-ip-range地址段不能有重叠

参数说明:
  
  --kube-apiserver-port: kubernetes apiserver 服务端口，默认8080
  
  --etcd-servers: ETCD服务集群地址，各地址间以逗号分隔。**必需**
  
  --service-cluster-ip-range: kubernetes集群service的可用地址范围。默认172.16.0.0/13
  
  --flannel-network-ip-range: Flannel网络可用地址范围。默认172.24.0.0/13
  
  --flannel-subnet-len: 分配给flannel结点的子网长度。默认22
  
  --cluster-dns: 集群DNS服务的IP地址，注意不加端口号，IP地址需落在--service-cluster-ip-range范围内。默认172.16.40.1
  
  --cluster-domain: 集群的search域。默认为domeos.local
  
  --insecure-bind-address: kube-apiserver非安全访问服务地址。默认0.0.0.0
  
  --insecure-port: kube-apiserver非安全访问服务端口。默认8080
  
  --secure-bind-address: kube-apiserver非安全访问服务地址。默认0.0.0.0
  
  --secure-port: kube-apiserver非安全访问服务端口。默认6443
  
  --authorization-mode: 
  
  --authorization-policy-file:
  
  --basic-auth-file: 
  
  以上这三个参数均与kube-apiserver授权认证相关，具体含义参考kubernetes帮助文档。默认不加
  
  --docker-graph-path: docker运行时的根路径，容器、本地镜像等会存储在该路径下，占用空间大，建议设置到大容量磁盘上。默认为脚本所在路径下的docker-graph目录
  
  --insecure-docker-registry: 非安全连接的docker私有仓库，可以先设置该值而不用启动对应的registry。**必需**
  
  --secure-docker-registry: 安全连接的docker私有仓库。默认为空
  
  --docker-registry-crt: 如果设置了--secure-docker-registry，则该值必需，表示私有仓库的证书。

样例: 
  1. 最简参数
      
          sudo sh start_master_centos.sh --etcd-servers http://10.11.151.97:4012 --insecure-docker-registry 10.11.150.76:5000

  2. 最全参数

          sudo sh start_master_centos.sh --kube-apiserver-port 8080 --etcd-servers http://10.11.151.97:4012,http://10.11.151.100:4012,http://10.11.151.101:4012 --service-cluster-ip-range 172.16.0.0/13 --flannel-network-ip-range 172.24.0.0/13 --flannel-subnet-len 22 --cluster-dns 172.16.40.1 --cluster-domain domeos.local --insecure-bind-address 0.0.0.0 --insecure-port 8080 --secure-bind-address 0.0.0.0 --secure-port 6443 --authorization-mode ABAC --authorization-policy-file /opt/domeos/openxxs/k8s-1.1.7-flannel/authorization --basic-auth-file /opt/domeos/openxxs/k8s-1.1.7-flannel/authentication.csv --docker-graph-path /opt/domeos/openxxs/docker --insecure-docker-registry 10.11.150.76:5000 --secure-docker-registry https://private-registry.sohucs.com --docker-registry-crt /opt/domeos/openxxs/k8s-1.1.7-flannel/registry.crt

**步骤3：配置和启动docker registry**

启动方式: docker容器

镜像: pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0

命令: 

    sudo docker run --restart=always -d -p <_port>:5000 -e REGISTRY_STORAGE_SOHUSTORAGE_ACCESSKEY=<_access_key> -e REGISTRY_STORAGE_SOHUSTORAGE_SECRETKEY=<_secret_key> -e REGISTRY_STORAGE_SOHUSTORAGE_REGION=<_region> -e REGISTRY_STORAGE_SOHUSTORAGE_BUCKET=<_bucket> --name <_name> pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0

参数说明: 
  
  _port: 外部访问端口。
  
  _access_key: 搜狐云台账户的accesskey。
  
  _secret_key: 搜狐云台账户的secretkey。
  
  _region: 搜狐云台存储空间所在地域。
  
  _bucket: 搜狐云台存储空间名称。
  
  _name: 容器名称。

说明: 
  1. 该方式创建的私有仓库将对接到搜狐云存储(cs.sohu.com)上。
  2. 如果docker registry需要使用https安全访问，则加入如下额外参数:
     -v <主机certs目录所在路径>:/certs
     -e REGISTRY_HTTP_TLS_CERTIFICATE=<registry.crt文件路径>
     -e REGISTRY_HTTP_TLS_KEY=<registry.key文件路径>
  3. 若想启本地存储的私有仓库，可以直接使用官方的registry镜像，启动过程请参考https://hub.docker.com/_/registry/

样例:
  
    sudo docker run --restart=always -d -p 5000:5000 -e REGISTRY_STORAGE_SOHUSTORAGE_ACCESSKEY=f5ynwwOZ2k2yT4+qxzmA6A== -e REGISTRY_STORAGE_SOHUSTORAGE_SECRETKEY=MUC9NeF8vXvt0y2f+6dIXA== -e REGISTRY_STORAGE_SOHUSTORAGE_REGION=bjcnc -e REGISTRY_STORAGE_SOHUSTORAGE_BUCKET=registry --name private-registry pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0
**步骤4：配置和启动MySQL，并创建所需数据库和表结构**

数据库: domeos, graph

表结构: 
      
    create-db.sql, graph-db-schema.sql 

下载地址：https://github.com/domeos/server/tree/master/DomeOS/src/main/resources

初始用户: 

    insert-data.sql 
下载地址：https://github.com/domeos/server/tree/master/DomeOS/src/main/resources ，会创建一个DomeOS系统的管理员用户，用户名：admin   ，密码:admin

说明: 

如果以非容器形式启动MySQL，需要在数据库中给172的IP段（容器的IP地址段）以及启动kubernetes master时设置的--service-cluster-ip-rangeIP地址段授权。

**步骤5：配置和启动监控相关组件**

启动方式: docker容器

镜像: 
  1. graph: pub.domeos.org/domeos/falcon-graph:0.5.6
  2. transfer: pub.domeos.org/domeos/falcon-transfer:0.0.14
  3. query: pub.domeos.org/domeos/falcon-query:1.4.3

命令:

1. graph: 

        sudo docker run -d -v <_graph>:/home/work/data/6070 -e DB_DSN="\"<_graph_db_user>:<_graph_db_passwd>@tcp(<_graph_db_addr>)/graph?loc=Local&parseTime=true\"" --name graph -p <_graph_http_port>:6070 -p <_graph_rpc_port>:6071 --restart=always pub.domeos.org/domeos/falcon-graph:0.5.6

2. transfer: 

        sudo docker run -d -e JUDGE_ENABLE="false" -e GRAPH_CLUSTER=<_graph_cluster> -p <_transfer_port>:8433 --name transfer --restart=always pub.domeos.org/domeos/falcon-transfer:0.0.14

3. query: 
        sudo docker run -d -e GRAPH_CLUSTER=<_graph_cluster> -p <_query_port>:9966 --name query --restart=always pub.domeos.org/domeos/falcon-query:1.4.3

参数说明:
  
  _graph: 数据文件存储路径。
  
  _graph_db_user: 用于graph的MySQL数据库的用户名。
  
  _graph_db_passwd: 用于graph的MySQL数据库的密码。
  
  _graph_db_addr: 用于graph的MySQL数据库的地址，格式为IP:Port
  
  _graph_http_port: graph服务http端口。
  
  _graph_rpc_port: graph服务rpc端口。
  
  _graph_cluster: graph服务数据结点，格式为键值对，多个结点间以逗号分隔。
  
  _transfer_port: transfer服务端口。
  
  _query_port: query服务端口。
  

  _graph_db_host:
  
  _graph_db_port:
  
  _graph_db_user:
  
  _graph_db_passwd: 
  
  以上4个配置参数为服务于graph的MySQL数据库的相关参数。
  
  _query_addr: query服务地址，格式必须为http://< ip >:< port >
  

说明:
  1. 参数中需要转义双引号的地方不能省略。
  2. transfer和graph可以启动多个。
  3. transfer和query的GRAPH_CLUSTER参数必须完全一致。
  4. 监控系统是在open-falcon基础上扩展而来，graph、transfer、query三个组件未做修改，详细配置可参考http://book.open-falcon.com/zh/install_from_src/agent.html

样例: 

  1. graph: 
        
          sudo docker run -d -v /opt/my/data:/home/work/data/6070 -e DB_DSN="\"domeos:xxxx@tcp(10.11.10.10:3307)/graph?loc=Local&parseTime=true\"" --name graph -p 6070:6070 -p 6071:6071 --restart=always pub.domeos.org/domeos/falcon-graph:0.5.6

  2. transfer: 
  
          sudo docker run -d -e JUDGE_ENABLE="false" -e GRAPH_CLUSTER="\"node-00\":\"10.11.54.13:6070\",\"node-01\":\"10.11.54.14:6070\"" -p 8433:8433 --name transfer --restart=always pub.domeos.org/domeos/falcon-transfer:0.0.14

  3. query: 
  
          sudo docker run -d -e GRAPH_CLUSTER="\"node-00\":\"10.11.54.13:6070\",\"node-01\":\"10.11.54.14:6070\"" -p 9967:9966 --name query --restart=always pub.domeos.org/domeos/falcon-query:1.4.3

**步骤6：配置和启动WebSSH组件**

启动方式: docker容器

镜像: pub.domeos.org/domeos/shellinabox:1.0

命令: sudo docker run -d --restart=always -p <_port>:4200 --name shellinabox pub.domeos.org/domeos/shellinabox:1.0

参数说明:
  _port: 服务端口。

样例:

    sudo docker run -d --restart=always -p 4200:4200 --name shellinabox pub.domeos.org/domeos/shellinabox:1.0

**步骤7： 配置和启动DomeOS Server**

源码: domeos/server 

下载地址：https://github.com/domeos/server

说明: 编译打包后启动tomcat服务。

**步骤8：DomeOS系统录入参数**

启动DomeOS Server后，你可以在浏览器中输入DomeOS控制台地址，用admin账户登录（用户名和密码都是admin）

如果想正常使用DomeOS的全部功能，您需要在“全局配置”和“集群管理”页面完成相应配置。

在全局配置页面完成各个组件的地址填写：

1. 全局配置->"私有仓库"->私有仓库地址
    
    Docker registry的地址，样例：        http://10.11.150.76:5000 
    如果使用https访问的私有仓库，则勾选"https"并将registry.crt证书文件内容粘贴至"证书信息"中
2. 全局配置->"服务器"->服务器地址
    
    DomeOS Server的地址，样例：http://10.11.150.76:8080
3. 全局配置->"监控配置"-> 控制台地址
    
    dashboard服务地址，样例： http://10.11.150.76:8081
4. 全局配置->"监控配置"-> transfer
    
    transfer服务地址，样例：10.11.150.76:8082
5. 全局配置->"监控配置"-> graph
    
    graph服务地址，样例：10.11.150.76:8083
6. 全局配置->"监控配置"-> query
    
    query服务地址，样例：10.11.150.76:8084
7. 全局配置->"Web SSH"->Web SSH服务地址
    
    WebSSH服务地址，样例：http://10.11.150.76:8085

在集群管理页面把已经启动的kubernetes集群添加进来。

1. 集群管理->管理->集群设置->编辑->api server
    
    kube-apiserver服务地址，样例：10.16.42.200:8080
2. 集群管理->管理->集群设置->编辑->dns服务器
    
    kubernetes集群内DNS服务地址，样例：172.16.40.1
3. 集群管理->管理->集群设置->编辑->etcd
    
    ETCD集群服务地址，注意必须加http前缀 ，样例：http://10.16.42.200:8080
  
4. 集群管理->管理->集群设置->编辑->domain
    
    kubernetes集群内DNS服务search域，样例：domeos.local

其余信息请根据页面提示填写。

如果想成功启动项目构建，该kubernetes集群中的至少一台主机需要被配置成可以用于构建，请在添加完集群后完成如下配置：

1. 全局配置->"构建集群"->api server
    kube-apiserver服务地址，样例:10.16.42.200:8080
  
2. 全局配置->"构建集群"->namespace
    kubernetes集群的namespace，默认为default

**步骤9：添加kubernetes node**

在控制台的“集群管理”的某集群中点击“添加主机”，经过配置后，复制最下面的命令并在主机上运行，即可将主机添加进集群，该操作需要主机能够连接外网。

如果希望手动生成添加主机命令，请看下面的说明。

脚本: 

     start_node_centos.sh 
      
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/start_node_centos.sh

辅助脚本: 
        
     change_hostname.sh 

下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/change_hostname.sh

程序包: 

     domeos-k8s-node.tar.gz 

下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/domeos-k8s-node.tar.gz ，包含: flanneld, mk-docker-opts.sh, kube-proxy, kubelet, kubectl

镜像: pub.domeos.org/domeos/agent:2.3

说明:
  1. 脚本会对hostname进行检查，如果不符合DNS规范，请通过change_hostname.sh脚本对主机hostname进行修改。
  2. 在DomeOS系统中node包含标签PRODENV=HOSTENVTYPE表示可用于生产环境；包含标签TESTENV=HOSTENVTYPE表示可用于测试环境；包含标签BUILDENV=HOSTENVTYPE表示可用于构建镜像。
  3. 在添加node时集群的DNS服务可不启动。
  4. 脚本中安装docker时需要连接互联网，如果所在主机无法访问外网，需要先在本地安装完docker，并将镜像pub.domeos.org/domeos/agent:2.0上传至私有仓库中，同时修改脚本第15步中该镜像对应的名称，再执行该脚本。
  5. 如果设置了私有仓库以https的方式访问，则脚本会从DomeOS Server上下载证书文件，registry.crt，因此需要当前主机可访问DomeOS Server。如果不能访问，则需要将registry.crt文件放置到脚本所在目录，并修改脚本中第7步的"TODO domeos offline"部分。
  6. 脚本成功执行后，将以systemctl的形式启动flanneld、docker、kube-proxy、kubelet，将以docker容器形式启动用于监控的agent，并为该结点打上指定的标签。
  7. 摘除node可按如下顺序停止: agent -> kubelet -> kube-proxy -> docker -> flanneld

参数说明:
  
  --api-server: kubernetes集群kube-apiserver服务地址。**必需**
  
  --cluster-dns: kubernetes集群内DNS服务地址，与kubernetes master启动参数一致。**必需**
  
  --cluster-domain: kubernetes集群内DNS服务的search域，与kubernetes master启动参数一致。**必需**
  
  --monitor-transfer: 监控transfer服务地址，可以填多个，各个地址间以逗号分隔。**必需**
  
  --docker-graph-path: docker运行时的根路径，容器、本地镜像等会存储在该路径下，占用空间大，建议设置到大容量磁盘上。默认为脚本所在路径下的docker-graph目录
  
  --docker-log-level: docker daemon的日志级别，可选值debug/info/warn/error/fatal。kubernetes会定时查询所有容器状态，导致docker daemon输出大量info级别日志到系统日志中(/var/log/message)，建议日志级别设置为warn级别以上。默认为warn
  
  --registry-type: 私有仓库类型，取值为http或https。**必需**
  
  --registry-arg: 私有仓库的地址，可以为域名地址。**必需**
  
  --domeos-server: DomeOS Server服务地址。必需
  
  --etcd-server: ETCD集群服务地址，各个地址间以逗号分隔。**必需**
  
  --node-labels: 为node打上的标签，非必需。

样例:

      sudo sh start_node_centos.sh --api-server http://10.16.42.200:8080 --cluster-dns 172.16.40.1 --cluster-domain domeos.local --monitor-transfer 10.16.42.198:8433,10.16.42.199:8433 --docker-graph-path /opt/domeos/openxxs/docker-graph --docker-log-level warn --registry-type http --registry-arg 10.11.150.76:5000 --domeos-server 10.11.150.76:8080 --etcd-server http://10.16.42.200:4012 --node-labels TESTENV=HOSTENVTYPE,PRODENV=HOSTENVTYPE

**步骤10：创建kubernetes集群内DNS服务**

启动方式: kubernetes service形式

配置文件: 

     dns.yaml 
     
下载地址：http://deploy-domeos.bjcnc.scs.sohucs.com/dns.yaml

执行文件: 
     
     kubectl

镜像: 
1. pub.domeos.org/domeos/skydns:1.4
2. pub.domeos.org/domeos/kube2sky:0.3

命令: 

    kubectl --server <kube-apiserver-addr> create -f dns.yaml

参数说明:
  1. skydns-svc service中的clusterIP为部署时设置的--cluster-dns
  2. skydns RC中args下的--machines为ETCD集群的服务地址，各个地址间以逗号分隔，必须带“ http:// ”前缀
  3. skydns RC中args下的--nameservers为集群中的主机使用的外部DNS服务（centos系统中为/etc/resolv.conf下配置的非集群nameserver），带端口号，多个DNS服务则以半角逗号分隔
  4. skydns RC中args下的--domain为部署时设置的--cluster-domain
  5. kube2sky RC中args下的--etcd-server为ETCD的服务地址，有且仅能设置一个，且必需在--machines设置的集群中，必须带“ http:// ”前缀
  6. kube2sky RC中args下的--domain为部署时设置的--cluster-domain

说明:
  1. 创建前请依据具体部署修改配置文件。
  2. 执行完毕后将创建名为skydns-svc的service，名为skydns的RC和名为kube2sky的RC。

样例:

    kubectl --server 10.16.42.200:8080 create -f dns.yaml

测试:
  1. 通过下方命令获取集群svc列表，确认skydns-svc是否已创建
  
          kubectl --server 10.16.42.200:8080 get svc
    
  2. 通过下方命令获取集群pod列表，确认skydns-为前缀和kube2sky-为前缀的两个pod是否处于Running状态，如skydns-u44ey和kube2sky-2h1b9
  
          kubectl --server 10.16.42.200:8080 get pods
          
  3. 查看node上的/etc/resolv.conf文件，确认前两行是否为如下内容（其中的domeos.local为--cluster-domain参数的值，172.16.40.1为--cluster-dns参数的值）:
  
          search default.svc.domeos.local svc.domeos.local domeos.local
          nameserver 172.16.40.1

  如果不是，添加如上格式内容
  
  4. 在node上进行DNS解析验证，方式有多种，如
   
         nslookup skydns-svc.default.svc.domeos.local
   
   如果没安装nslookup也可通过ping间接验证
   
         ping skydns-svc.default.svc.domeos.local -c 1
         
   解析出IP地址则说明DNS服务创建成功
  5. 在通过kubernetes创建的容器内部验证，方法同第4步
