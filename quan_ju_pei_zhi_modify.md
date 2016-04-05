# 全局配置
---
完成系统的安装及初始化后，访问控制台地址。第一次访问会以admin身份登录。登录后，需要在控制台左侧导航栏点击“全局配置”，进入全局配置页面完成各项配置。

##admin
admin用户是整个DomeOS的系统管理员，拥有所有资源的最高权限。并且只有admin用户能够查看并修改全局配置。
admin用户的电子邮箱和密码，可以在全局配置的“用户管理”页面修改。

下面是全局配置的各项说明。

##1. 用户管理
在用户管理中，您可以看到系统上所有的用户。对于普通用户，您可以修改其密码、电子邮箱或删除该用户。您还可以创建一个新的普通用户账号。

用户类型说明：

**LDAP用户**：通过关联LDAP服务器获得。LDAP用户的管理受LDAP服务器控制。DomeOS控制台只提供展示功能，不能删除或新增LDAP用户。LDAP用户在DomeOS上和普通用户的设定、权限等完全相同。

**普通用户**：DomeOS自身的用户。可以在页面上添加或删除普通用户。

登录DomeOS时，可以选择LDAP用户或普通用户登录。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject1.jpg)

##2. LDAP
通常企业会有内部的LDAP服务器，用来管理公司账号。在LDAP页面上，您可以关联自己公司的LDAP服务器，并告知邮箱后缀。设置LDAP后，该LDAP服务器管理的账号都可以登录DomeOS控制台。

**email后缀：**LDAP服务器可以配置后缀(suffix)，如果您的LDAP服务器有此配置，请在页面上填写。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject2.jpg)

##3. 代码仓库
DomeOS支持关联代码仓库，用于关联代码并构建成项目。目前DomeOS支持关联Gitlab。您需要填写您的Gitlab地址。之后DomeOS会支持更多的代码仓库。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject3.jpg)

##4. 私有仓库
在这里您需要告知系统安装时启动的私有仓库的地址。如果私有仓库需要通过https访问，则需要勾选“https”并输入证书信息。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject4.jpg)

##5. 服务器
在这里您需要填写DomeOS API服务器的访问地址。需要和系统安装时的配置保持一致。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject5.jpg)

##6. 监控配置
您需要填写监控相关的四个组件：Dashboard、transfer、graph、query的地址。需要和系统安装时的配置保持一致。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject6.jpg)

##7. Web SSH
如果希望通过Web前端直接登录容器，请填写Web服务容器的访问地址。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject7.jpg)


