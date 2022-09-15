# Linux 配置jdk

## 方法一 修改/etc/profile

首先将jdk包放入/usr/local下，并将其解压（解压后名字jdk1.8.0_191）

配置环境变量vim /etc/profile

增加内容如下：

```
export JAVA_HOME=/usr/local/jdk1.8.0_191

export CLASSPATH=${JAVA_HOME}/lib

export PATH=$PATH:${JAVA_HOME}/bin
```

修改好配置文件后，让其生效：

```
source /etc/profile
```

## 方法二  修改.bashrc 文件

内容同方法一，

生效方法：

```
source .bashrc
```

 