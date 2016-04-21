# 镜像定制工具

镜像定制工具是一个让用户根据dockerfile或配置文件，在不创建项目的情况下，构建个性化镜像的工具。

点击控制台左侧导航栏的“镜像”，进入页面后在上方点击“镜像定制工具”，进入镜像定制页面。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject73.jpg)
##1. Dockerfile定制
Dockerfile定制让您能够在页面上直接编写Dockerfiel进行构建。编写Dockerfile需要注意的是ADD指令只能通过URL获取文件；COPY指令无效。

在编写完Dockerfile后，您需要进行发布镜像的配置。除了基本了指定镜像名称和版本号。您可以选择是否将镜像发布为基础镜像。基础镜像能够让所有用户查看并使用。此外，您还可以选择是否将镜像作为特定语言的编译或运行镜像；如果启用，则该镜像的镜像名称会使用对应镜像类型的默认名称。目前仅支持作为Java语言的编译或运行镜像。

作为特定编程语言的编译/运行镜像，主要是为后续针对不同语言设置不同的项目配置和构建流程做铺垫。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject74.jpg)
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject75.jpg)

##2. 配置文件定制
配置文件定制方式让您可以选择一个基础镜像，并添加配置文件和环境变量。系统会自动生成Dockerfile并执行构建。

选取基础镜像时，您可以从DomeOS私有镜像库或第三方镜像库中选取镜像。如果使用集群外的第三方镜像库，需要确保镜像库能够被外网访问。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject76.jpg)

##3. 定制记录
定制记录展示了所有的定制历史，类似构建记录。您能够查看每次定制的日志和配置详情。
![](http://881471b33d4f9.cdn.sohucs.com/q_mini/newproject77.jpg)