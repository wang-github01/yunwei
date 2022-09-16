# node 节点重新加入mast集群

## 1、从mast上删除node节点

```
kubectl delete node node01
```

##  2、这时如果直接执行加入，会报错。如下：

```
[root@node01 ~]# kubeadm join 192.168.101.102:6443 --token mtaitl.27tmevarpv9xat1z --discovery-token-ca-cert-hash sha256:8da6ec904560c8d91b01860263797f450b07be0cdb7240141c995160260795ca
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "node01" could not be reached
        [WARNING Hostname]: hostname "node01": lookup node01 on 192.168.101.2:53: no such host
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
        [ERROR Port-10250]: Port 10250 is in use
        [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

**解决方案**

根据报错可以看到端口被占用，配置文件可ca证书已经生成，所以需要删除这些配置文件和证书，并且kill掉占用的端口。建议删除之前先备份。

```
[root@node01 ~]# lsof -i:10250
COMMAND PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
kubelet 694 root   30u  IPv6  26021      0t0  TCP *:10250 (LISTEN)
[root@node01 ~]# kill -9 694
[root@node01 ~]# cd /etc/kubernetes/
[root@node01 ~]# ls
bootstrap-kubelet.conf  kubelet.conf  manifests  pki
[root@node01 ~]# mv bootstrap-kubelet.conf bootstrap-kubelet.conf_bk
[root@node01 ~]# mv kubelet.conf kubelet.conf_bk
[root@node01 ~]# cd pki/
[root@node01 ~]# ls
ca.crt
[root@node01 ~]# rm -rf ca.crt 
```

## 3、再次执行加入，又出现报错。

```
[root@node01 ~]# kubeadm join 192.168.101.102:6443 --token mtaitl.27tmevarpv9xat1z --discovery-token-ca-cert-hash  sha256:a3d9827be411208258aea7f3ee9aa396956c0a77c8b570503dd677aa3b6eb6d8 
[preflight] Running pre-flight checks
	[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.12. Latest validated version: 18.09
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.15" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[kubelet-check] Initial timeout of 40s passed.
error execution phase kubelet-start: error uploading crisocket: timed out waiting for the condition
```

**解决方案**

重置 子节点

```
kubeadm reset
```

## 4、最后执行加入，问题解决。

```
[root@node01 pki]# kubeadm join 192.168.101.102:6443 --token mtaitl.27tmevarpv9xat1z --discovery-token-ca-cert-hash sha256:8da6ec904560c8d91b01860263797f450b07be0cdb7240141c995160260795ca
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "node01" could not be reached
        [WARNING Hostname]: hostname "node01": lookup node01 on 192.168.101.2:53: no such host
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:

* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```