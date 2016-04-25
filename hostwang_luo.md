# Host网络

##1. 背景介绍
DomeOS 0.2版本提供了host网络模式的部署方式。host网络主要提升了部署的对外服务性能，避免了kube-proxy在ip table转发过程上损失性能。如果想在host网络模式下部署，需要在镜像中添加DomeOS提供的插件——domeize。


##2. Host网络使用流程
1. 用户需要事先写好配置文件模板。配置文件模版是根据自身业务代码编写的，需要在配置文件模板需要使用对外端口的时候，使用AUTO_PORT0,AUTO_PORT1...的变量名代替端口（具体规则见github上domeize项目介绍）。用户需要将写好的配置文件模板放入代码项目或基础镜像内。
2. 在项目配置中填写配置文件模板的位置（镜像内路径），想要放到项目镜像中的位置（镜像内路径）。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject68.jpg)
3. 部署的镜像中需要有domeos开发的插件“domeize”。目前DomeOS提供的基础镜像里都包含了domeize。如果希望自行构建基础镜像，则需要自行添加domeize。

  domeize地址：https://github.com/domeos/domeize
4. 执行构建，项目镜像中包含用户的配置文件模板和domeize
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject17.jpg)
5. 新建部署，选择镜像后，在网络模式中选择“host模式”，并在访问设置中指定对外暴露的端口个数。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject69.jpg)
至此，一个host网络模式的部署就建立成功了。

##3. domeize工作流程
1. doemos server在启动pod的时候，会下发若干个环境变量，包含网络模式、对外暴露端口数、domeos server地址等。
2. 启动时会检测这几个环境变量，如果需要它来动态申请端口，就根据环境变量中的对外暴露端口数去随机在20000~21000之间申请未占用端口，并写入环境变量AUTO_PORT0,AUTO_PORT1...
3. 申请端口后，domeize根据domeos server地址，上报申请到的端口。domeos server把端口信息回写到pod的metadata中的annotations里面。
4. 如果上述步骤执行成功，domeize根据环境变量，修改配置文件模板，生成配置文件（放在指定的镜像内路径下）

这样，一个不会端口冲突的pod就启动成功了。


##4. host模式下外层负载均衡方案：

目前，domeos控制台不提供host网络模式的负载均衡配置，需要用户根据自身业务需求配置。
参考方案：
1. 配置负载均衡器（如nginx）
2. 使用confd读取etcd来生成负载均衡器配置。因为confd代码里定义以AUTO_PORT0来读取服务端口，所以建议配置模板中服务端口以AUTO_PORT0来注入。
3. confd把pod上报的端口信息写到负载均衡器的配置中
4. confd会自动检测pod的端口信息变化，输出到负载均衡器的配置文件中。

confd下载地址：https://github.com/kelseyhightower/confd/releases
支持etcd证书的需要0.12版本以上

confd安装完成后，需要创建一个配置文件和一个模板文件。DomeOS的参考方案中，两个文件如下：

nginx.toml配置文件

    [template]
    src = "upstream.tmpl"
    dest = "/opt/scs/nginx/conf/upstream.conf"
    keys = [
     "/registry/pods/default",
    ]
      
    owner = "root"
    mode = "0644"
    check_cmd = "/usr/sbin/nginx -t -c pathtonginxconf"
    reload_cmd = "/usr/sbin/nginx -s reload"
nginx.tmpl模版文件

    upstream img_cs_group
    {   
        check interval=10000 rise=2 fall=3 timeout=3000 type=http default_down=false;
        check_http_send 'GET /favicon.ico HTTP/1.0\r\nHost: no-monitor.sohu.com\r\n\r\n';
        check_http_expect_alive http_2xx;
        keepalive 80;
                                                                           
        {{ $dir := printf "/registry/pods/default/*" }}
        {{range gets $dir }}{{$data := json .Value }}{{if $data.metadata.annotations.accessType }}{{if eq $data.metadata.annotations.deployName "img-host-test"}}{{if eq $data.metadata.annotations.accessType "DIY"}}{{if $data.status.podIP}}{{if $data.metadata.annotations.AUTO_PORT0}}  server {{$data.status.podIP}}:{{$data.metadata.annotations.AUTO_PORT0}};
        {{end}}{{end}}{{end}}{{end}}{{end}}{{end}}
 
 
    }

##5. 注意事项
使用host网络部署时，申请和分配端口都是针对pod（实例）级别，因此如果一个pod内有多个运行domeize的容器，会出现端口冲突的情况。所以如果一个部署需要使用host网络，需要保证一个pod内只有一个运行domeize的容器。

对应到DomeOS控制台，则如果部署需要用host网络，在新建部署选择镜像时，只能选择一个含domeize的镜像。目前DomeOS提供的基础镜像都包含domeize

host网络的部署可以放心使用日志自动收集功能。因为DomeOS提供的flume镜像不含domeize，所以pod内的flume容器不会引起端口冲突。