# 删除主机

DomeOS提供从集群中删除主机的功能。需要执行DomeOS官方提供的脚本，执行后主机上的kubelet、flannel、kube-proxy、docker会被停止，iptable会被清理。

删除主机脚本地址：http://domeos-script.bjctc.scs.sohucs.com/stop-k8s-node.sh
