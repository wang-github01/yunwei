[root@localhost ~]# docker info

```
Containers: 45                  #容器的数量

 Running: 44                    #正在运行的数量
 
 Paused: 0                      #暂停的数量
 
 Stopped: 1                     #已经停止的数量
 
Images: 264                     #镜像数量

Server Version: 1.12.5                   #docker server的版本

Storage Driver: devicemapper             #存储驱动程序

 Pool Name: docker-253:0-537427328-pool  #pool name的值根据Data file的模式改变      # direct-lvm 模式（Pool Name 为 docker-thinpool ）
 
 Pool Blocksize: 65.54 kB                #pool块大小
 
 Base Device Size: 10.74 GB              #基本存储大小
 
 Backing Filesystem: xfs                 #支持的文件系统
 
 Data file: /dev/loop0                   #使用的模式为loop-lvm        生产中不推荐使用（loop-lvm性能比较差）         使用 direct-lvm 模式（配置过程在最后面）
 
 Metadata file: /dev/loop1               #元数据文件位置
 
 Data Space Used: 86.78 GB               #数据使用的空间 direct-lvm 模式 direct-lvm 模式
 
 Data Space Total: 107.4 GB              #数据的总空间
 
 Data Space Available: 20.6 GB           #数据的可用空间
 
 Metadata Space Used: 159.8 MB           #元数据使用的空间
 
 Metadata Space Total: 2.147 GB          #元数据的总空间
 
 Metadata Space Available: 1.988 GB      #元数据的可用空间
 
 Thin Pool Minimum Free Space: 10.74 GB  # Thin pool(瘦供给池)的最小可用空间    *下面有瘦供给的解释    
 
 Udev Sync Supported: true       
 
 Deferred Removal Enabled: false
 
 Deferred Deletion Enabled: false
 
 Deferred Deleted Device Count: 0
 Data loop file: /var/lib/docker/devicemapper/devicemapper/data        #数据loop文件的位置
 WARNING: Usage of loopback devices is strongly discouraged for production use. Use `--storage-opt dm.thinpooldev` to specify a custom block storage device.            #警告信息
 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata    #元数据loop文件的位置
 Library Version: 1.02.135-RHEL7 (2016-09-28)
Logging Driver: json-file        
Cgroup Driver: cgroupfs
Plugins:                                    #插件            
 Volume: local
 Network: host null bridge overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Security Options: seccomp                    #内核安全组件
Kernel Version: 3.10.0-514.el7.x86_64        #内核版本
Operating System: CentOS Linux 7 (Core)      #操作系统
OSType: linux                                #操作系统类型
Architecture: x86_64                         #系统架构
CPUs: 4                                      #CPU数
Total Memory: 30.96 GiB                      #总空间
Name: docker-slave3.ctrm                     #主机名
ID: POZH:PSTG:ULR2:S75Y:OW57:ETGA:Z7RU:WEQA:VGNE:4JMJ:PJ3N:LXZW
Docker Root Dir: /var/lib/docker              #docker根目录
Debug Mode (client): false                    #调试模式（client）
Debug Mode (server): false                    #调试模式（server）
Registry: https://index.docker.io/v1/         #镜像仓库
Insecure Registries:                          #非安全镜像仓库
 15.116.20.134:5000
 15.116.20.104:80
 15.116.20.115:80
 127.0.0.0/8
```

