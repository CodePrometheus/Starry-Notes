# StatefulSet

[TOC]

## 简介

statefulset是为了解决**有状态服务**的问题，而产生的一种资源类型（deployment和replicaSets是解决无状态服务而设计的）。

> mysql是有状态服务，但使用的是deploment资源类型，mysql的data数据通过pv的方式存储在第三方文件系统中，也能解决mysql数据存储问题。
>
> 是的，如果mysql是单节点，使用deployment类型确实可以解决数据存储问题。但是如果有状态服务是集群,节点之间有主备或者主从之分，且每个节点分片存储的情况下，deployment则不适用这种场景，因为deployment不会保证pod的有序性，集群通常需要主节点先启动，从节点在加入集群，statefulset则可以保证，其次deployment资源的pod内的pvc是共享存储的，而statefulset下的pod内pvc是不共享存储的，每个pod拥有自己的独立存储空间，正好满足了分片的需求，实现分片的需求的前提是statefulset可以保证pod重新调度后还是能访问到相同的持久化数据。
>
> 适用statefulset常用的服务有elasticsearch集群,mogodb集群,redis集群等等。



StatefulSet就是用来管理某 Pod 集合的部署和扩缩，并为这些 Pod 提供持久存储和持久标识符，和Deployment不同的是，StatefulSet虽然一样管理基于相同容器规约的一组 Pod，但是，StatefulSet为它们的每个Pod维护了一个有粘性的ID，这些Pod是基于相同的规约来创建的，但是不能相互替换：无论怎么调度，每个Pod都有一个永久不变的ID

如果希望使用存储卷为工作负载提供持久存储，可以使用 StatefulSet 作为解决方案的一部分。 尽管 StatefulSet 中的单个 Pod 仍可能出现故障， 但持久的 Pod 标识符使得将现有卷与替换已失败 Pod 的新 Pod 相匹配变得更加容易。



## 应用场景

- 稳定的不共享持久化存储：即每个pod的存储资源是不共享的，且pod重新调度后还是能访问到相同的持久化数据，基于pvc实现。
- 稳定的网络标志：即pod重新调度后其PodName和HostName不变，且PodName和HostName是相同的，基于Headless Service来实现的。
- 有序部署，有序扩展：即pod是有顺序的，在部署或者扩展的时候是根据定义的顺序依次依序部署的（即从0到N-1,在下一个Pod运行之前所有之前的pod必都是Running状态或者Ready状态），是基于init containers来实现的。
- 有序收缩：在pod删除时是从最后一个依次往前删除，即从N-1到0.

基于上面的特性，可以发现statefulset由以下几个部分组成：

- 用于定义网络标志（DNS domain）的headless service
- 用于**创建pvc的volumeClaimTemplates**
- 具体的statefulSet应用



使用statefulset资源类型的服务通常有以下几点特点

- 有状态服务
- 多节点部署，集群部署
- 节点有主从(备)之分。集群通常是主节点先运行，从节点后续运行并加入集群，这里就用**statefulset资源的有序部署的特性**
- 节点之间数据分片存储，**这里使用到了statefulSet资源的存储隔离的特性，以及保证pod重新调度后还是能访问到相同的持久化数据**



## 示例

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
~~~

- 名为 `nginx` 的 Headless Service 用来控制网络域名。
- 名为 `web` 的 StatefulSet 有一个 Spec，它表明将在独立的 3 个 Pod 副本中启动 nginx 容器。
- `volumeClaimTemplates` 将通过 PersistentVolumes 驱动提供的 PersistentVolumes 来提供稳定的存储。



Kubernetes 为每个 VolumeClaimTemplate 创建一个 PersistentVolume。 在上面的 nginx 示例中，每个 Pod 将会得到基于 StorageClass `my-storage-class` 提供的 1 Gib 的 PersistentVolume。

如果没有声明 StorageClass，就会使用默认的 StorageClass。 当一个 Pod 被调度（重新调度）到节点上时，它的 `volumeMounts` 会挂载与其 PersistentVolumeClaims 相关联的 PersistentVolume。 请注意，当 Pod 或者 StatefulSet 被删除时，与 PersistentVolumeClaims 相关联的 PersistentVolume 并不会被删除。要删除它必须通过手动方式来完成。



### 部署和扩缩保证

- 对于包含 N 个 副本的 StatefulSet，当部署 Pod 时，它们是依次创建的，顺序为 `0..N-1`。
- 当删除 Pod 时，它们是逆序终止的，顺序为 `N-1..0`。
- 在将缩放操作应用到 Pod 之前，它前面的所有 Pod 必须是 Running 和 Ready 状态。
- 在 Pod 终止之前，所有的继任者必须完全关闭。

StatefulSet 不应将 `pod.Spec.TerminationGracePeriodSeconds` 设置为 0。 这种做法是不安全的，要强烈阻止。

在上面的 nginx 示例被创建后，会按照 web-0、web-1、web-2 的**顺序**部署三个 Pod。 在 web-0 进入 Running 和 Ready状态前不会部署 web-1。在 web-1 进入 Running 和 Ready 状态前不会部署 web-2。 如果 web-1 已经处于 Running 和 Ready 状态，而 web-2 尚未部署，在此期间发生了 web-0 运行失败，那么 web-2 将不会被部署，要等到 web-0 部署完成并进入 Running 和 Ready 状态后，才会部署 web-2。

如果用户想将示例中的 StatefulSet 收缩为 `replicas=1`，首先被终止的是 web-2。 在 web-2 没有被完全停止和删除前，web-1 不会被终止。 当 web-2 已被终止和删除、web-1 尚未被终止，如果在此期间发生 web-0 运行失败， 那么就不会终止 web-1，必须等到 web-0 进入 Running 和 Ready 状态后才会终止 web-1



### 运行

~~~bash
# 创建 Headless Service 和 StatefulSet
kubectl apply -f web.yaml

# 查看 StatefulSet 的 Pods 的创建情况
kubectl get pods -w -l app=nginx

# 查看是否成功创建
kubectl get service nginx
kubectl get statefulset web

# 扩容
kubectl scale sts web --replicas=5

# 执行 RollingUpdate 更新策略
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'

# 再次改变容器镜像 | Pod 采用和序号相反的顺序更新
kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"gcr.io/google_containers/nginx-slim:0.8"}]'

# 级联删除
kubectl delete statefulset web
~~~

