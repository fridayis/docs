# Host网络

#1. 背景介绍：
DomeOS 0.2版本提供了host网络模式的部署方式。host网络主要提升了部署的对外服务性能，避免了kube-proxy在ip table转发过程上损失性能。如果想在host网络模式下部署，需要在镜像中添加DomeOS提供的插件——domeize。

domeize项目地址：https://github.com/crystalharp/domeize

为了使用host网络模式，需要做以下几件事：
1. 用户需要事先写好配置文件模板。配置文件模版是根据自身业务代码编写的，需要在配置文件模板需要使用对外端口的时候，使用AUTO_PORT0,AUTO_PORT1...的变量名代替端口（具体规则见domeize介绍）。用户需要将写好的配置文件模板放入代码项目内。
2. 在项目配置中填写配置文件模板的位置（代码项目内路径），想要放到项目镜像中的位置（镜像内路径）。在运行过程环境变量中定义生成配置文件的位置（镜像内路径），环境变量名是DOMEIZE_TEMPLATES，环境变量值需要包含镜像内配置文件模板存放路径和配置文件生成路径，如/etc/nginx/nginx.tmpl:/etc/nginx/nginx.conf或/path/to/1.tmpl:/path/to/1.conf,/path/to/2.tmpl:/path/to/2.conf。此外还需要填写domeize的启动命令，如“domeize /path/to/tomcat/startup.sh”。
3. 部署的镜像中需要有domeos开发的插件“domeize”。目前DomeOS提供的基础镜像里都包含了domeize。如果希望自行构建基础镜像，则需要自行添加domeize。

  domeize地址：https://github.com/crystalharp/domeize
4. 执行构建，项目镜像中包含用户的配置文件模板和domeize
5. 新建部署，选择镜像后，在网络模式中选择“host模式”，并指定对外暴露的端口个数。
至此，一个host网络模式的部署就建立成功了。

domeize是在开源项目“dockerize”基础上完成的一个插件。
domeize工作流程：
1. doemos server在启动pod的时候，会下发若干个环境变量，包含网络模式、对外暴露端口数、domeos server地址等。
2. 启动时会检测这几个环境变量，如果需要它来动态申请端口，就根据环境变量中的对外暴露端口数去随机在20000~21000之间申请未占用端口，并写入环境变量AUTO_PORT0,AUTO_PORT1...
3. 申请端口后，domeize根据domeos server地址，上报申请到的端口。domeos server把端口信息回写到pod的metadata中的annotations里面。
4. 如果上述步骤执行成功，domeize根据环境变量，修改配置文件模板，生成配置文件（放在指定的镜像内路径下）

这样，一个不会端口冲突的pod就启动成功了。


host模式下外层负载均衡方案：

目前，domeos控制台不提供host网络模式的负载均衡配置，需要用户根据自身业务需求配置。
参考方案：
1. 配置负载均衡器（如nginx/haproxy）
2. 使用confd读取etcd来生成负载均衡器配置。因为confd代码里定义以AUTO_PORT0来读取服务端口，所以建议配置模板中服务端口以AUTO_PORT0来注入。
3. confd把pod上报的端口信息写到负载均衡器的配置中
4. confd会自动检测pod的端口信息变化，输出到负载均衡器的配置文件中。
