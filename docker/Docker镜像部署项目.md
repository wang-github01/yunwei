# Docker使用

> 以tomcat 上更新war包为例

## 一、上传war包，编写Dockerfile文件

> 首先创建一个webapps 文件，将更新的war包上传（首次上传需要新建一个Dockerfile文件）
>
> Dockerfile文件内容如下：
>
> ```
> from tomcat:v1.0.1 #from 后面是要跟的是制作新镜像的基础的镜像
> 
> ADD sysadmin.war /home/demo/apache-tomcat-7.0.108/webapps
> ```
>

## 二、制作镜像

> ```
> 执行docker build -t 容器名字:版本号 .（docker build -t tomcat:v1.0.2 .）
> ```
>
> 或者执行 docker build -t 域名/仓库名/容器名字:版本号 .
>
> - 域名：harbor仓库地址或者设置的harbor域名
> - 仓库名：在harbor中创建的仓库（存放项目镜像的仓库）

## 三、查看镜像

> ```
> docker images
> ```
>

## 四、启动镜像

> ```
> docker run -d -p 8085:8080 REPOSITORY: TAG（docker run -d -p 8085:8080 tomcat:v1.0.2）
> ```
>
> - -d: 后台运行容器，并返回容器ID；
> - -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
>       （主机端口是外部能访问的端口,容器端口是容器中的端口）
> - REPOSITORY：镜像名字
> - TAG ：镜像版本
>

## 五、查看正在运行镜像

> ```
> docker ps
> ```
>

## 六、向harbor中推送镜像

###  1、制作镜像

> 首先编写Dockerfile文件（模板在Dockerfile/Dockerfile更新war包）
>
> 执行docker build -t 域名/仓库名/容器名字:版本号
>
> 例如： 
>
> ```
> docker build -t 192.168.15.131/tomcat-test/tomcat:v1.0.2 .
> ```
>
> - 域名：harbor仓库地址或者设置的harbor域名
> - 仓库名：在harbor中创建的仓库（存放项目镜像的仓库）
>
> 这里在对仓库进行命名的时候使用“域名/仓库名/容器名字”这种格式的目的是和harbor中的地址一直。后面在想harbor中推送镜像的时候不需要在使用docker tag 命令进行重命名了.

### 2、重命名镜像

> 使用docker build -t 域名/仓库名/容器名字:版本号，制作镜像跳过此步骤
>
> 如果在构建 docker镜像时对镜像进行命名没有使用“域名/仓库名/容器名字:版本号” 这种格式，
>
> ```
> docker build -t tomcat:v1.0.2 .
> ```
>
> 在向harbor推送镜像之前需要是使用docker tag 进行对镜像重命名，
>
> ```
> docker tag tomcat:v1.0.2 192.168.15.131/tomcat-test/tomcat:v1.0.2 .
> ```

### 3、向harbor 中推送镜像 

> ```
> docker push REPOSITORY: TAG
> ```
>
> 例如：
>
> ```
> docker push 192.168.15.131/tomcat-test/tomcat:v1.0.2
> ```

### 4、拉取镜像

> ```
> docker pull harbor地址/仓库地址/镜像名:版本号
> 
> docker pull REPOSITORY: TAG
> ```
