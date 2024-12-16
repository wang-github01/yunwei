# k8s 中搭建redis-哨兵集群

本次部署为基于k8s部署[redis](https://link.csdn.net/?target=https%3A%2F%2Fauth.huaweicloud.com%2Fauthui%2Fsaml%2Flogin%3FxAccountType%3Dcsdndev_IDP%26isFirstLogin%3Dfalse%26service%3Dhttps%3A%2F%2Fdeveloper.huaweicloud.com%2Fspace%2Fdevportal%2Fdesktop%3Futm_source%3Dcsdndspace%26utm_adplace%3Dcsdnffnr)一主两从三哨兵集群,本次[服务器](https://link.csdn.net/?target=https%3A%2F%2Fauth.huaweicloud.com%2Fauthui%2Fsaml%2Flogin%3FxAccountType%3Dcsdndev_IDP%26isFirstLogin%3Dfalse%26service%3Dhttps%3A%2F%2Fdeveloper.huaweicloud.com%2Fspace%2Fdevportal%2Fdesktop%3Futm_source%3Dcsdndspace%26utm_adplace%3Dcsdnffnr)为:

```
k8s-master01 192.168.101.102
k8s-master02 192.168.101.103
k8s-master03 192.168.101.104
k8s-node01  192.168.101.105 
```

![image-20241214115531952](E:\GitHup\yunwei\redis\images\image-20241214115531952.png)

## 1、设置nfs存储

**(1) yum下载nfs,rpcbind,四台机器上都执行**

yum install -y nfs-utils rpcbind

**(2) 在192.168.101.102上执行,设置nfs共享目录，在exports文件中新增下列内容**

```
vi /etc/exports

/k8s/redis-sentinel/0 *(rw,sync,no_subtree_check,no_root_squash)
/k8s/redis-sentinel/1 *(rw,sync,no_subtree_check,no_root_squash)
/k8s/redis-sentinel/2 *(rw,sync,no_subtree_check,no_root_squas
```

**(3) 192.168.101.102节点上创建数据存储路径**

```
mkdir -p /k8s/redis-sentinel
```

**(4)进入目录**

```
cd /k8s/redis-sentinel
```

**(5) 192.168.101.102节点上创建数据存储路径**

```
mkdir 0 1 2
```

![image-20241214120911629](E:\GitHup\yunwei\redis\images\image-20241214120911629.png)

**(6) 192.168.101.102节点上执行,设置redis数据存储路径文件权限**

```
chmod -R 755 /k8s/redis-sentinel
```

**(7) 192.168.101.102节点上执行,使新增的nfs共享目录生效**

```
exportfs -arv
```

**(8) 四台机器上都执行,启动rpcbind并设置开机自启**

```
systemctl enable rpcbind && systemctl start rpcbind
```

**(9) 四台机器上都执行,启动nfs并设置开机自启**

```
systemctl enable nfs && systemctl start nfs
```

## 2、编写配置文件

以下操作均在主节点192.168.101.102节点上以root用户操作,可新建目录存放下面所属yml文件,本人在此路径k8s/redis

**(1) vim redis-sentinel-pv.yaml**

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-redis-sentinel-0
spec:
  capacity:
    storage: 4Gi  # 定义 nfs 服务器容量
  accessModes:    # 定义访问模式
    - ReadWriteMany  # 读写权限，可以被多个节点挂载
  volumeMode: Filesystem # 指定存储卷的模式，Filesystem文件系统
  persistentVolumeReclaimPolicy: Recycle # 定义当PV不再被使用时，其的处理方式Retain：保留数据，Recycle：清除PV中的数据
 storageClassName: "redis-sentinel-storage-class"
  # storageClassName字段指定一个资源对象的名称。具有特定类别的PV只能与请求了该类别的PVC进行绑定。未设定类别的PV则只能与不请求任何类别的PVC进行绑定。
  nfs:
    # real share directory
    path: /k8s/redis-sentinel/0   # 定义存储位置
    # nfs real ip
    server: 192.168.101.102   # 定义 nfs 服务器 ip
 
---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-redis-sentinel-1
spec:
  capacity:
    storage: 4Gi  # 定义 nfs 服务器容量
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: "redis-sentinel-storage-class"
  nfs:
    # real share directory
    path: /k8s/redis-sentinel/1
    # nfs real ip
    server: 192.168.101.102


---

 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-redis-sentinel-2
spec:
  capacity:
    storage: 4Gi   # 定义 nfs 服务器容量
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: "redis-sentinel-storage-class"
  nfs:
    # real share directory
    path: /k8s/redis-sentinel/2
    # nfs real ip
    server: 192.168.101.102
```

注:上述配置文件中需要将server:192.168.101.102这个ip改为自己的ip,其他一样即可

**(2) 使redis-sentinel-pv.yml文件生效**

```
kubectl apply -f redis-sentinel-pv.yaml
```

**(3) 查看之前配置文件生成的pv卷是否创建成功**

```
kubectl get pv | grep redis
```

![image-20241214121821387](E:\GitHup\yunwei\redis\images\image-20241214121821387.png)

## 3、创建namespace

**(1) 创建命名空间redis**

```
kubectl create namespace redis
```

**(2) 查看是否创建成功**

```
kubectl get namespace
```

![image-20241214122001827](E:\GitHup\yunwei\redis\images\image-20241214122001827.png)

## 4、创建ConfigMap

**(1) vim redis-sentinel-configmap.yaml**

编写redis 配置文件

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: redis-sentinel-config
  namespace: redis
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
data:
    redis-master.conf: |                    # 主节点配置文件
      port 6379                             # 修改redis监听端口
      tcp-backlog 511
      timeout 0
      tcp-keepalive 0
      loglevel notice
      databases 16
      save 900 1
      save 300 10
      save 60 10000
      stop-writes-on-bgsave-error yes
      rdbcompression yes
      rdbchecksum yes
      dbfilename dump.rdb
      dir /data/                           # 工作目录，需要创建好目录,可自定义
      slave-serve-stale-data yes
      repl-diskless-sync no
      repl-diskless-sync-delay 5
      repl-disable-tcp-nodelay no
      slave-priority 100
      requirepass Idc&57$S6z                # 设置redis密码
      masterauth Idc&57$S6z                 # 主从同步master的密码
      protected-mode no                     # 设置为no，允许外部网络访问
      appendonly no
      appendfilename "appendonly.aof"
      appendfsync everysec
      no-appendfsync-on-rewrite no
      auto-aof-rewrite-percentage 100
      auto-aof-rewrite-min-size 64mb
      aof-load-truncated yes
      lua-time-limit 5000
      slowlog-log-slower-than 10000
      slowlog-max-len 128
      latency-monitor-threshold 0
      notify-keyspace-events ""
      hash-max-ziplist-entries 512
      hash-max-ziplist-value 64
      list-max-ziplist-entries 512
      list-max-ziplist-value 64
      set-max-intset-entries 512
      zset-max-ziplist-entries 128
      zset-max-ziplist-value 64
      hll-sparse-max-bytes 3000
      activerehashing yes
      client-output-buffer-limit normal 0 0 0
      client-output-buffer-limit slave 256mb 64mb 60
      client-output-buffer-limit pubsub 64mb 16mb 60
      hz 10
      aof-rewrite-incremental-fsync yes
    redis-slave.conf: |                        # 从节点配置文件
      port 6379                                # 设置端口
      slaveof redis-sentinel-master-ss-0.redis-sentinel-master-ss.redis.svc.cluster.local 6379
      # slaveof 设置追随主节点成为从节点，
      #redis-sentinel-master-ss-0 是主服务器的一个 Pod 的名称或标识符，表明这是该服务的一个实例。
      #redis-sentinel-master-ss 是服务的名称，主节点的pod name
      #redis.svc.cluster.local 是服务的域名后缀，redis表示pod所在的命名空间其中 svc 表示服务（Service），cluster.local 是集群的本地域名。
      tcp-backlog 511
      timeout 0
      tcp-keepalive 0
      loglevel notice
      databases 16
      save 900 1
      save 300 10
      save 60 10000
      stop-writes-on-bgsave-error yes
      rdbcompression yes
      rdbchecksum yes
      dbfilename dump.rdb
      dir /data/
      slave-serve-stale-data yes
      slave-read-only yes
      repl-diskless-sync no
      repl-diskless-sync-delay 5
      repl-disable-tcp-nodelay no
      slave-priority 100
      appendonly no
      requirepass Idc&57$S6z           # 设置redis密码
      masterauth Idc&57$S6z            # 主从同步master的密码
      protected-mode no
      appendfilename "appendonly.aof"
      appendfsync everysec
      no-appendfsync-on-rewrite no
      auto-aof-rewrite-percentage 100
      auto-aof-rewrite-min-size 64mb
      aof-load-truncated yes
      lua-time-limit 5000
      slowlog-log-slower-than 10000
      slowlog-max-len 128
      latency-monitor-threshold 0
      notify-keyspace-events ""
      hash-max-ziplist-entries 512
      hash-max-ziplist-value 64
      list-max-ziplist-entries 512
      list-max-ziplist-value 64
      set-max-intset-entries 512
      zset-max-ziplist-entries 128
      zset-max-ziplist-value 64
      hll-sparse-max-bytes 3000
      activerehashing yes
      client-output-buffer-limit normal 0 0 0
      client-output-buffer-limit slave 256mb 64mb 60
      client-output-buffer-limit pubsub 64mb 16mb 60
      hz 10
      aof-rewrite-incremental-fsync yes
    redis-sentinel.conf: |                  # 哨兵配置文件
      port 26379                            # 设置端口
      dir /data                             # 工作目录
      protected-mode no
      # Sentinel 监听 redis 主节点, mymaster：master名称可自定义,127.0.0.1 6379 ：redis主节点IP和端口,2 ：表示多少个Sentinel认为redis主节点失效时，才算真正失效
      sentinel monitor mymaster 192.168.101.102 32379 2  
      sentinel down-after-milliseconds mymaster 30000
      sentinel parallel-syncs mymaster 1
      sentinel failover-timeout mymaster 180000
      sentinel auth-pass mymaster Idc&57$S6z
```

注:需更改redis-sentinel.conf下的 sentinel monitor mymaster 192.168.101.102 32379 2 这个设置,192.168.101.102更改为自己的ip 即可.

**(2) 使redis-sentinel-configmap.yaml配置文件生效**

```
kubectl apply -f redis-sentinel-configmap.yaml
```

**(3) 查看configmap配置文件是否创建成功**

```
kubectl get configmap -n redis
```

## 5、创建service

**(1) vim redis-sentinel-service-master.yaml**

（这里为了方便测试，部署了有头的service，正常来说，reids的部署为有序状态，其service部署应为无头的方式部署）

主节点service（ clusterIP: None 为无头部署）

```
kind: Service
apiVersion: v1
metadata:
  labels:
    app: redis-sentinel-master-ss
  name: redis-sentinel-master-ss
  namespace: redis
spec:
  clusterIP: None
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
  selector:
    app: redis-sentinel-master-ss
```

**(2) vim redis-sentinel-service-slave.yaml**

从节点service（ clusterIP: None 为无头部署）

```
kind: Service
apiVersion: v1
metadata:
  labels:
    app: redis-sentinel-slave-ss
  name: redis-sentinel-slave-ss
  namespace: redis
spec:
  clusterIP: None
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
  selector:
    app: redis-sentinel-slave-ss
```

**(3) vim redis-sentinel-service.yml**

有头service主节点部署

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis-sentinel-master-ss
  name: redis-sentinel-master-ss-svc
  namespace: redis
spec:
  externalTrafficPolicy: Cluster
  ports:
  - name: redis
    nodePort: 32379
    port: 6379
    protocol: TCP
    targetPort: 6379
  selector:
    app: redis-sentinel-master-ss
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

**(4) vim sentinel-service.yaml**

（有头哨兵service部署）

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis-sentinel-sentinel-ss
  name: redis-sentinel-sentinel-ss-svc
  namespace: redis
spec:
  ports:
  - name: redis
    port: 26379
    protocol: TCP
    targetPort: 26379
  selector:
    app: redis-sentinel-sentinel-ss
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

**(5) 使上述四个yml文件生效**

```
kubectl apply -f redis-sentinel-service-master.yaml 
kubectl apply -f redis-sentinel-service-slave.yaml 
kubectl apply -f redis-sentinel-service.yml 
kubectl apply -f sentinel-service.yaml
```

## 6、创建StatefulSet

pod的部署采用有状态部署

**(1) vim redis-sentinel-rbac.yaml**

角色的访问控制（）

```
apiVersion: v1
kind: ServiceAccount  # 创建特殊的用户身份
metadata:
  name: redis-sentinel
  namespace: redis
---

kind: Role    # Role 是一种用于定义对Kubernetes资源的操作权限的资源对象
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: redis-sentinel
  namespace: redis
rules:       # 定义Role的权限规则列表
  - apiGroups:  
      - ""     # apiGroups API组的列表，每个API组是一个权限的集合。空字符串（""）表示核心API组
    resources: # 资源类型的列表，例如pods、services等
      - endpoints
    verbs:  # 对资源执行的操作列表，例如get、list、watch、create、update、delete等
      - get

---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: redis-sentinel
  namespace: redis
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role        # kind：必须为Role
  name: redis-sentinel
subjects:            # 指定绑定到Role的实体列表。这些实体可以是用户、组或服务账户。
- kind: ServiceAccount   # 实体的类型，例如User、Group或ServiceAccount。
  name: redis-sentinel
  namespace: redis

```

**(2) vim redis-sentinel-ss-master.yaml**

有状态主节点应用部署

```
kind: StatefulSet
apiVersion: apps/v1
metadata:
  labels:
    app: redis-sentinel-master-ss
  name: redis-sentinel-master-ss
  namespace: redis
spec:          # StatefulSet 资源清单
  replicas: 1
  selector:
    matchLabels:                      # 通过标签选择器匹配Pod
      app: redis-sentinel-master-ss   # 与Pod模板中的标签相匹配。
  serviceName: redis-sentinel-master-ss
  template:                  # pod模板定义
    metadata:
      labels:                # Pod的标签，与StatefulSet的选择器匹配。
        app: redis-sentinel-master-ss
    spec:                    # 定义 pod 所需要的资源清单
      containers:            # Pod中的容器列表
      - args:        #  容器的启动参数。
        - -c         # Shell命令的开始
        - cp /mnt/redis-master.conf /data/ ; redis-server /data/redis-master.conf
        # cp 复制配置文件并启动Redis服务器
        command: # 覆盖容器默认的启动命令。
        - sh     #  使用shell作为启动命令
        image: 192.168.101.110/wang/redis:6.2.5
        imagePullPolicy: IfNotPresent
        name: redis-master
        ports:  # 容器暴露的端口列表
        - containerPort: 6379  #  容器内部端口，这里是6379。
          name: masterport     # 端口的名称，这里是masterport。
          protocol: TCP
        volumeMounts:          # 容器挂载的卷列表。
        - mountPath: /mnt/     # 卷在容器内的挂载路径
          name: config-volume  # 卷的名称
          readOnly: false      # 只读
        - mountPath: /data/
          name: redis-sentinel-master-storage
          readOnly: false
      serviceAccountName: redis-sentinel  # 指定Pod使用的ServiceAccount（指定身份）
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:  # 使用ConfigMap作为卷，将配置文件挂载到容器中
          items:
          - key: redis-master.conf
            path: redis-master.conf
          name: redis-sentinel-config   # ConfigMap的名称
        name: config-volume    # 卷的名称，与volumeMounts中的name匹配
  volumeClaimTemplates:   # 为StatefulSet中的每个Pod创建PersistentVolumeClaim的模板。
  - metadata:
      name: redis-sentinel-master-storage
    spec:
      accessModes:
      - ReadWriteMany
      storageClassName: "redis-sentinel-storage-class" # 存储类的名称，用于指定存储后端。
      resources:
        requests:
          storage: 4Gi
```

**(3) vim redis-sentinel-ss-slave.yaml**

有状态从节点应用部署

```
kind: StatefulSet
apiVersion: apps/v1
metadata:
  labels:
    app: redis-sentinel-slave-ss
  name: redis-sentinel-slave-ss
  namespace: redis
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-sentinel-slave-ss
  serviceName: redis-sentinel-slave-ss
  template:
    metadata:
      labels:
        app: redis-sentinel-slave-ss
    spec:
      containers:
      - args:
        - -c
        - cp /mnt/redis-slave.conf /data/ ; redis-server /data/redis-slave.conf
        command:
        - sh
        image: 192.168.101.110/wang/redis:6.2.5
        imagePullPolicy: IfNotPresent
        name: redis-slave
        ports:
        - containerPort: 6379
          name: slaveport
          protocol: TCP
        volumeMounts:
        - mountPath: /mnt/
          name: config-volume
          readOnly: false
        - mountPath: /data/
          name: redis-sentinel-slave-storage
          readOnly: false
      serviceAccountName: redis-sentinel
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          items:
          - key: redis-slave.conf
            path: redis-slave.conf
          name: redis-sentinel-config
        name: config-volume
  volumeClaimTemplates:
  - metadata:
      name: redis-sentinel-slave-storage
    spec:
      accessModes:
      - ReadWriteMany
      storageClassName: "redis-sentinel-storage-class"
      resources:
        requests:
          storage: 4Gi
```

**(4) 使上述三个配置文件生效**

```
kubectl apply -f redis-sentinel-rbac.yaml 
kubectl apply -f redis-sentinel-ss-master.yaml 
kubectl apply -f redis-sentinel-ss-slave.yaml
```

**(5) 查看命名空间public-service下各pod情况,STATUS皆为running则为正常**

```
kubectl get pods -n redis
```

![image-20241214123052544](E:\GitHup\yunwei\redis\images\image-20241214123052544.png)

至此,一主两从搭建完毕

## 7、创建sentinel

**(1) vim redis-sentinel-ss-sentinel.yaml**

有状态哨兵应用部署

```
kind: StatefulSet
apiVersion: apps/v1
metadata:
  labels:
    app: redis-sentinel-sentinel-ss
  name: redis-sentinel-sentinel-ss
  namespace: redis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis-sentinel-sentinel-ss
  serviceName: redis-sentinel-sentinel-ss
  template:
    metadata:
      labels:
        app: redis-sentinel-sentinel-ss
    spec:
      containers:
      - args:
        - -c
        - cp /mnt/redis-sentinel.conf /data/ ; redis-sentinel /data/redis-sentinel.conf
        command:
        - sh
        image: 192.168.101.110/wang/redis:6.2.5
        imagePullPolicy: IfNotPresent
        name: redis-sentinel
        ports:
        - containerPort: 26379
          name: sentinel-port
          protocol: TCP
        volumeMounts:
        - mountPath: /mnt/
          name: config-volume
          readOnly: false
      serviceAccountName: redis-sentinel
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          items:
          - key: redis-sentinel.conf
            path: redis-sentinel.conf
          name: redis-sentinel-config
        name: config-volume
```

**(2) vim redis-sentinel-service-sentinel.yaml**

无头哨兵service

```
kind: Service
apiVersion: v1
metadata:
  labels:
    app: redis-sentinel-sentinel-ss
  name: redis-sentinel-sentinel-ss
  namespace: redis
spec:
  clusterIP: None
  ports:
  - name: redis
    port: 26379
    targetPort: 26379
  selector:
    app: redis-sentinel-sentinel-ss
```

**(3) 创建配置文件**

```
kubectl apply -f redis-sentinel-ss-sentinel.yaml 
kubectl apply -f redis-sentinel-service-sentinel.yaml
```

**(4) 查看命名空间redis下各个服务状态**

```
kubectl get service -n redis
```

![image-20241214123748455](E:\GitHup\yunwei\redis\images\image-20241214123748455.png)

**(4) 查看命名空间redis pod是否都启动**

![image-20241214123906795](C:\Users\16897\AppData\Roaming\Typora\typora-user-images\image-20241214123906795.png)

至此：一主两从三哨兵集群搭建完毕

搭建完成后，使用kuebctl logs 查看各节点是否有报错

## 8、集群测试

1、进入容器内部执行命令（以登录master为例）

首先查看redsi master 节点的pod ip和 无头的service 设置的可供集群内部访问的端口

![image-20241215235725685](E:\GitHup\yunwei\redis\images\image-20241215235725685.png)

这里ip为 10.224.1.12 端口为 6379

```
redis-cli -c -h 10.224.1.12 -p 6379
# 登录
auth Idc&57$S6z
```

![image-20241216000045859](E:\GitHup\yunwei\redis\images\image-20241216000045859.png)

2、查看节点信息

```
info replication
```

![image-20241214193132857](E:\GitHup\yunwei\redis\images\image-20241214193132857.png)

3、测试数据

```
# 写入数据
set test_key "This is a test data"
# 读取数据
get test_key
```

![image-20241214174215522](E:\GitHup\yunwei\redis\images\image-20241214174215522.png)