# Linux_时钟设置

1、设置上海时区

```
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

2、安装ntp（用于同步网络时间）

```
yum install -y ntpdate
```

3、同步时间

```
ntpdate time.nist.gov  #time.nist.gov 是一个时间服务器
```

6、查看系统硬件时间

```
hwclock
```

7、将系统时间写入到系统硬件当中，防止系统重启后时间被还原

```
hwclock -w 
或者
hwclock --systohc
```

8、将系统时间调整为与目前硬件时间一致

```
hwclock --hctosys
```