# 系统安装及初始化
---
 DomeOS公开镜像仓库： pub.domeos.org
 
安装组件的镜像下载地址：

| 安装组件| 镜像地址 |
| -- | -- |
| 私有仓库 | pub.domeos.org/domeos/docker-registry-driver-sohustorage:1.0 |
| skyDNS | pub.domeos.org/domeos/skydns:1.2|
| kube2sky | pub.domeos.org/domeos/kube2sky:1.1 |
| agent | pub.domeos.org/domeos/falcon-agent:1.0|
| transfer | pub.domeos.org/domeos/falcon-transfer:0.0.14 |
| graph | pub.domeos.org/domeos/falcon-graph:0.5.6 |
| query | pub.domeos.org/domeos/falcon-query:1.4.3 |
| dashboard | pub.domeos.org/domeos/falcon-dashboard:0.1 |
| DomeOS server | pub.domeos.org/domeos/server:1.0 |

安装步骤：

1. 如果您的私有仓库通过http访问，在第1步用DomeOS公开镜像仓库提供的镜像或Docker Hub官方镜像启动私有仓库，该私有仓库在您的Kubernetes集群外部。如果您的私有仓库通过https访问，需要在第4步安装Kubernetes时，通过Kubernetes启动私有仓库。

2. 安装并启动etcd集群。详细步骤见etcd官方网站。

3. 给集群内的主机安装Docker。主机的hostname需要符合dns命名规则。每台主机需要连接到私有仓库

4. 安装带flannel网络的Kubernetes集群：

   配置flannel时，需要给出etcd地址并配置etcd-prefix。
   
   启动flannel时，主机上的Docker必须停止。
   
   启动flannel后，需要重新启动Docker。
   
   Kubernetes集群在启动服务端时，需要按顺序启动以下三个组件：api server、kube-controller-manager、kube-scheduler。在api server完全启动后，再启动后两个组件。      
   
   启动主机时，需要启动两个组件：kubelet和kube-proxy                            
   
   更详细的安装步骤详见Kubernetes官方网站。

5. 配置skyDNS，kube2sky。DomeOS的公开镜像仓库中提供了两者的镜像。DomeOS提供安装两者的YAML文件，Kubectl可以根据YAML文件创建两者。您可以根据自身需求修改YAML文件配置项，如：etcd地址、etcd domain、DomeOS server地址。

 YAML文件：
       apiVersion: v1
       kind: ReplicationController
       metadata:
         name: skydns
         labels:
           app: skydns
           version: v9
       spec:
         replicas: 1
         selector:
           app: skydns
           version: v9
         template:
           metadata:
             labels:
               app: skydns
               version: v9
           spec:
             containers:
               - name: skydns
                 image: pub.domeos.org/domeos/skydns:1.2
                 command:
                   - "/skydns"
                 args:
                   - "--machines=http://1.1.1.1:1111"
                   - "--domain=domeos.local"
                   - "--addr=0.0.0.0:53"
                 ports:
                   - containerPort: 53
                     hostPort: 53
                     name: dns-udp
                     protocol: UDP
                   - containerPort: 53
                     hostPort: 53
                     name: dns-tcp
                     protocol: TCP
                 dnsPolicy: ClusterFirst
                 hostNetwork: true
                 restartPolicy: Always
       ---
       apiVersion: v1
       kind: ReplicationController
       metadata:
         name: kube2sky
         labels:
           app: kube2sky
           version: v9
       spec:
         replicas: 1
         selector:
           app: kube2sky
           version: v9
         template:
           metadata:
             labels:
               app: kube2sky
               version: v9
           spec:
             containers:
              - name: kube2sky
                 image: pub.domeos.org/domeos/kube2sky:1.1
                 command:
                   - "/kube2sky"
                 args:
                   - "--etcd-server=http://1.1.1.1:1111"
                   - "--domain=domeos.local"
                   - "--kube_master_url=http://1.1.1.1:8080"
                 dnsPolicy: ClusterFirst
                 restartPolicy: Always

6. 启动MySQL，建议通过传统方式启动而非镜像启动。启动MySQL时，需要创建三个数据库：domeos,graph, dashboard，并创建一个用户拥有这三个库的权限。

7. 启动监控需要的四个组件：transfer、graph、query、dashboard。DomeOS公开镜像仓库提供镜像。 transfer和query中配置的graph信息需要保持一致。

   需要在每台主机上安装agent，每个agent需要配置一个transfer地址。agent镜像在DomeOS公开镜像仓库中提供。
   
   agent和监控四个组件的工作逻辑：
   1. agent上报信息给transfer；
   2. transfer将信息传递给graph存储；
   3. query从graph里查询信息；
   4. dashboard把从query里查询的信息做图形化展示。
   
    更多详细步骤见小米Open-Falcon官方文档

8. 启动DomeOS server 。镜像由DomeOS公开镜像仓库提供。