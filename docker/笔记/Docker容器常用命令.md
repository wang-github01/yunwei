# Docker容器常用命令

> 详情参考网址：http://www.docker.org.cn/dockerppt/106.html

```
systemctl start docker 启动docker服务

systemctl stop docker 关闭docker服务

systemctl restart docker 重启docker 服务

systemctl status docker 查看 docker 运行状态

systemctl enable docker 设置 docker 开机自启

docker version 或者 docker –v 查看docker版本信息

docker info 查看docker 详情信息（内容见附件）

docker ps 查看当前正在运行的容器

docker ps -a 查看所有容器的状态

docker start/stop id/name 启动/停止某个容器

docker exec -ti id bash 进入一个容器中

docker images 查看本地镜像

docker search 镜像名 或者docker images | grep 镜像名 （搜索镜像）

docker rm id/name 删除某个容器

docker rmi id/name 删除某个镜像

docker pull/push 从仓库拉取/向仓库推从镜像

docker cp id:容器路径文件 本地路径 （将容器中的文档复制到本地中）

docker save -o 文件名.tar REPOSITORY:TAG 镜像打包

docker load < 本地镜像文件 （加载本地镜像包到docker中）
```
