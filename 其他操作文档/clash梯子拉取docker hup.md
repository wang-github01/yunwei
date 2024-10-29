使用clash梯子，配置liunx 服务器。拉取docker hup镜像

1、编写配置文件

```
vim /usr/lib/systemd/system/docker.service

[Service]
Environment="HTTP_PROXY=http://192.168.0.101:7890"
Environment="HTTPS_PROXY=http://192.168.0.101:7890"
Environment="NO_PROXY=localhost,127.0.0.1"

格式：Environment="HTTP_PROXY=http://<ip>:<端口>"

# 加载配置文件
systemctl daemon-reload
# 重启docker
 systemctl restart docker
```

![image-20241009214214208](E:\GitHup\yunwei\其他操作文档\images\image-20241009214214208.png)

![image-20241009214301950](E:\GitHup\yunwei\其他操作文档\images\image-20241009214301950.png)

开启手动代理服务器

![1728532869636](E:\GitHup\yunwei\其他操作文档\images\1728532869636.png)