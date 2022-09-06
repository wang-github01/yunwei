# nginx安装

## 一、nginx 安装

1、nginx官网下载ningx包http://nginx.org/en/download.html

2、将nginx tar包，上传到服务器

3、安装依赖 （安装过的跳过）

```
 yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-dev
```

4、解压–编译–安装

> - 解压：将包解压到/usr/local下，重命名为nginx
>
> - 编译：进入nginx文中，执行 ./configure
>
>   编译如果提示./configure: error: the HTTP rewrite module requires the PCRE library.，则执行yum -y install pcre-devel后重新编译
>
>   提示./configure: error: the HTTP gzip module requires the zlib library.,则执行yum install -y zlib-devel后重新编译
>
> - 安装：make && make install （安装完成会nginx下会出现sbin目录）

## 二、nginx服务的启动操作

进入目录/usr/local/nginx/sbin

- ​    启动：./nginx
- ​    停止：./nginx -s stop
- ​    重启：./nginx –s reload
- ​    检查：./nginx –t (检查配置文件是否有语法错误)