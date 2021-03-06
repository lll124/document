# DaemonSet 操作实例

DaemonSet 确保全部（或者一些）Node 上运行一个 Pod 的副本。当有 Node 加入集群时，也会为他们新增一个 Pod 。当有 Node 从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod

使用 DaemonSet 的一些典型用法：

- 运行集群存储 daemon，例如在每个 Node 上运行glusterd、ceph

- 在每个 Node 上运行日志收集 daemon，例如fluentd、logstash

- 在每个 Node 上运行监控 daemon，例如 Prometheus Node Exporter、collectd、Datadog 代理、New Relic 代理，或 Ganglia gmond

```
apiVersion: apps/v1
kind: DaemonSet
metadata:  
  name: daemonset-example  
  labels:    
    app: daemonset
spec:  
  selector:    
    matchLabels:      
      name: daemonset-example  
  template:    
    metadata:      
      labels:        
        name: daemonset-example    
    spec:      
      containers:      
      - name: daemonset-example        
        image: wangyanglinux/myapp:v1
```

```
kubectl delete daemonset --all
kubectl get daemonset

```

