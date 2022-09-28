# Docker 部署

参考网址：https://blog.csdn.net/u014069688/article/details/100532774/

## 1、查看内核版本

> ```
> uname -a
> ```

## 2、 更新yum源

> 温馨提示：新环境或测试环境可随意操作，生产环境酌情慎重更新
>
> ```
>  yum update   		把 yum 包更新到最新  
> ```

## 3、安装需要的软件包

> yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动
>
> ```
> yum install -y yum-utils device-mapper-persistent-data lvm2
> ```

## 4、设置yum源

> 设置yum源为阿里镜像仓库
>
> ```
> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo			
> ```

## 5、查看docker版本

> 通过查看系统中的所有docker版本，可以安装是时候选择特定版本安装
>
> ```
> yum list docker-ce --showduplicates|sort -r
> ```

##  6、安装Docker

> yum install docker-ce-版本号，我选的是docker-ce-18.03.1.ce
>
> 例如：yum install docker-ce-18.03.1.ce
>
> docker version 或者 docker -v（查看docker版本，如何能看到自己选择安装的docker信息，则Docker安装成功）

## 7、启动Docker

> ```
> systemctl start docker 			（启动docker）
> 
> systemctl enable docker 		(设置开机自启)
> ```

## 8、设置镜像加速

> vim /etc/docker/daemon.json （如果没有就创建）
>
> ```
> {
> "registry-mirrors": ["https://mirror.baidubce.com"]
> }
> ```
>
> ```
> # 加载配置文件
> systemctl daemon-reload
> # 重启 docker 服务
> systemctl restart docker
> ```
