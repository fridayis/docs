# 集群成员

DomeOS分别对项目、部署、集群有独立的成员管理系统。三者的成员管理系统权限独立，结构相同，使用方法和流程基本一致。
在集群详情页的“集群成员”页面，您可以看到集群成员列表和集群创建组列表
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject6.jpg)

##1. 集群创建组
集群创建组在新建集群的时候在“创建者”一项指定。如果您选择用个人身份创建，则该集群没有创建组；如果您选择用某个组的身份创建，则该组就是集群的创建组。创建组的组成员在集群内拥有和组内权限同级的集群权限。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject6.jpg)

如需修改创建组内权限，请点击左侧导航栏“组管理”，找到对应组，进行修改。

##2. 集群成员
集群成员由admin或集群内Master权限的成员添加。点击“编辑”，您可以添加新成员，修改现有集群成员的权限，以及删除现有集群成员。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject6.jpg)

添加成员时，您可以选择导入用户、导入组成员或导入集群成员。选择要添加的成员后，指定集群内权限即可添加新成员了。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject6.jpg)

##3. 角色权限
集群内各角色权限划分如下：

| 角色 |权限 |
| -- | -- |
| master | 集群的最高权限，可以查看集群，修改集群设置，添加namespace，查看主机，修改主机设置，管理集群成员 |
| developer | 可以查看集群，修改集群设置，添加namespace，查看主机，修改主机设置 |
| reporter | 可以查看部署，查看主机|