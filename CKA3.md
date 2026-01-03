# CKA3



25% - Cluster Architecture, Installation & Configuration

* Manage role based access control (RBAC)
* Use kubeadm to install a basic cluster
* Manage a highly-available Kubernetes cluster
* Provision underlying infrastructure to deploy a Kubernetes cluster
* Perform a version upgrade on a Kubernetes cluster using Kubeadm
* Implement etcd backup and restore





15% - Workloads & Scheduling



* Understand deployments and how to perform rolling updates and rollbacks
* Use ConfigMaps and Secrets to conifgure applications
* Know how to scale applications
* Understand the primitives used to create robust, self-healing, application deployments
* Understand how resource limits can affect Pod scheduling
* Awareness of manifest management and common templating tools





20% - Services & Networking

* Understand host networking configuration on the cluster nodes
* Understand connectivity between Pods
* Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
* Know how to use ingress controllers and Ingress resources
* Know how to configure and use CoreDNS
* Choose an appropriate container network interface plugin





10% - Storage

* Understand storage classes, persistent volumes
* Understand volume mode, access modes and reclaim policies for volumes
* Understand persistent volume claims primitive
* Know how to configure applications with persistent storage





30% - Troubleshooting

* Evaluate cluster and node logging
* Understand how to monitor applications
* Manage Container stdout & stderr logs
* Troubleshoot application failure
* Troubleshoot cluster component failure
* Troubleshoot networking



# 一、集群部署

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
# 1.更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包：
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# 2. 下载 Google Cloud 公开签名秘钥：
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# 3. 添加 Kubernetes apt 仓库：
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 4. 更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```



```bash
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yml

kubeadm config images pull --config kubeadm.yml

kubeadm init --config=kubeadm.yml --upload-certs | tee kubeadm-init.log

```









# 二、监控和日志

## Top

kubectl top pod <podname>

```bash
wenyi_sun01@Azure:~$ kubectl top pod aks-helloworld-one-6965865b8b-cnt78 -n=ingress-nginx
NAME                                  CPU(cores)   MEMORY(bytes)   
aks-helloworld-one-6965865b8b-cnt78   1m           49Mi  
wenyi_sun01@Azure:~$ kubectl top node
NAME                                CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
aks-agentpool-37128677-vmss000005   171m         9%     2255Mi          49%
```









# 三、Workload Controller



## Pod

https://kubernetes.io/docs/concepts/workloads/pods/

Details: 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod1
  labels:
    app: nginx
spec:
  containers:
  - image: busybox
    name: bs-xya
    command: ["/bin/sh","-c","sleep 3600000"]
  - name: nginx-wb
    image: nginx:1.14.2
```



```bash
wenyi_sun01@Azure:~/k8sfiles$ kubectl apply -f mypod.yaml 
pod/mypod1 created
wenyi_sun01@Azure:~/k8sfiles$ kubectl get po
NAME                                READY   STATUS      RESTARTS   AGE
mypod1                              2/2     Running     0          6s
```





## Deployment

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Details: 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```





## ReplicasSet

https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

Details: 

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```









## Job

https://kubernetes.io/docs/concepts/workloads/controllers/job/

Details: None

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```



```bash
wenyi_sun01@Azure:~$ kubectl apply -f https://kubernetes.io/examples/controllers/job.yaml
job.batch/pi created
wenyi_sun01@Azure:~$ kubectl get pods
NAME       READY   STATUS      RESTARTS      AGE
dnsutils   1/1     Running     2 (44m ago)   164m
pi-lvjcb   0/1     Completed   0             88s
wenyi_sun01@Azure:~$ kubectl logs pi-lvjcb
3.1415926535
```





## CronJob

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

Details: None

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```



```bash
wenyi_sun01@Azure:~$ kubectl create -f https://abcdefe.blob.core.windows.net/k8s/CronJob.yml
cronjob.batch/hello created
wenyi_sun01@Azure:~$ kubectl get pods
NAME                   READY   STATUS      RESTARTS      AGE
dnsutils               1/1     Running     2 (51m ago)   172m
hello-27608744-sfgcf   0/1     Completed   0             30s
pi-lvjcb               0/1     Completed   0             9m11s
wenyi_sun01@Azure:~$ kubectl logs hello-27608744-sfgcf
Wed Jun 29 17:44:01 UTC 2022
Hello from the Kubernetes cluster
```





## initcontainer

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

Details: 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```



```bash
wenyi_sun01@Azure:~$ kubectl apply -f https://abcdefe.blob.core.windows.net/k8s/initcontainer.yml
pod/myapp-pod created

wenyi_sun01@Azure:~$ kubectl get pods
NAME                                READY   STATUS      RESTARTS   AGE
myapp-pod                           0/1     Init:0/2    0          2m27s
```









## 静态Pod



就是把创建pod的yaml文件放到kubelet的目录下，之后由kubelet管理。





## sidecar

和普通container差不多，一般用来跑一些命令。





# 四、调度

## resources.limits/resources.requests



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.17
        name: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```





## nodeSelector

用于将pod调度到匹配label的node上，如果没有匹配的标签会调度失败。

```yaml
apiVersion: v1
kind: Pod
metadata: 
  labels:
    run: nginx
  name: pod-example
spec: 
  nodeSelector:
    disktype: "ssd"
  containers:
  - name: nginx
    image: nginx
```









## nodeAffinity

节点亲和类似于nodeSelector，可以根据节点上的标签来约束Pod可以调度到哪些节点

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      #required和preferred选择一个
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: In
            values:
            - nvidia-tesla
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: gpu
            operator: In
            values:
            - "yes"
  containers:
  - name: web
    image: nginx
```









## Taints 

避免Pod调度到特定Node上（和前面nodeSelector相反）

```yaml
#设置污点
kubectl taint node k8s-node2 gpu=yes:NoSchedule

#查看污点
root@k8s-mas:/usr/local/kubernetes/cluster# kubectl describe node | grep Taint
Taints:             <none>
Taints:             gpu=yes:NoSchedule
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

# 删除污点
kubectl taint node k8s-node2 gpu=yes:NoSchedule-
```





## Tolerations

<font color='red'>重点：Taint的理解——比如node1有taint gpu=yes，意味着所有没有设置容忍gpu-yes的pod都无法创建到node1节点上。</font>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-taints
spec:
  containers:
  - name: pod-taints
    image: busybox:latest
  tolerations:
  # gpu=yes:NoSchedule
  - key: "gpu"
    operator: "Equal"
    value: "yes"
    effect: "NoSchedule“
```







## nodeName

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
  labels:
    app: nginx
spec:
  nodeName: k8s-node2 # 指定node，即使node有污点也能分配过去，因为不经过调度器
  containers:
  - name: nginx
    image: nginx:1.15
```







## DaemonSet

DaemonSet功能：

* 在每一个Node上运行一个Pod
* 新加入的node也同样会自动运行一个pod

应用场景：网络插件（calico），监控agent，日志agent。

```yaml
apiVersion: apps/v1
# kind换成DaemonSet
kind: DaemonSet
metadata:
  name: filebeat
  namespace: kube-system
spec:
  # 也没有Replicas
  selector:
    matchLabels:
      name: filebeat
  template:
    metadata:
      labels:
        name: filebeat
    spec:
      containers:
      - name: log
        image: elastic/filebeat:7.3.2
```







# 五、网络

CNI

## Service

ClusterIP/NodePort/LoadBalancer

https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
      # 默认情况下，为了方便起见，`targetPort` 被设置为与 `port` 字段相同的值。
    - port: 80
      targetPort: 80
      # 可选字段
      # 默认情况下，为了方便起见，Kubernetes 控制平面会从某个范围内分配一个端口号（默认：30000-32767）
      nodePort: 30007
```





## Ingress



simple:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```







```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /hello-world-one(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-one
            port:
              number: 80
      - path: /hello-world-two(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-two
            port:
              number: 80
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-one
            port:
              number: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress-static
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /static/$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /static(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-one
            port: 
              number: 80
```





Iptables



IPVS



Coredns

A record格式：<service name>.<namespace name>.svc.cluster.local



# 六、存储



## emptyDir

**emptyDir卷**：是一个临时数据卷，与Pod生命周期绑定在一起，如果Pod删除了，卷也会被删除

应用场景：Pod中容器之间数据共享



```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: my-pod
spec: 
  containers: 
  - name: write
    image: centos
    command: ["bash","-c","for i in {1..100};do echo $i >> /data/hello;sleep 1;done"]
    volumeMounts:
    - name: data
      mountPath: /data
  - name: read
    image: centos
    command: ["bash","-c","tail -f /opt/hello"]
    volumeMounts: 
    - name: data
      mountPath: /opt
  volumes:
  - name: data
    emptyDir: {}
```





## hostPath

https://kubernetes.io/docs/concepts/storage/volumes/#hostpath

hostPath卷：挂载Node文件系统（Pod所在节点）上文件或者目录到Pod中的容器。

应用场景：<font color="red">Pod中容器需要访问宿主机文件</font>



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-hostpath
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - /bin/sh
    - -c
    - sleep 36000
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    hostPath:
      path: /home/azureuser/tmp # 宿主机的目录
      type: Directory
```







## NFS



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: wwwroot
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
      - name: wwwroot
        nfs:
          server: 172.27.0.5
          path: /home/azureuser/nginx/html
```







## PV



```yaml
apiVersion: v1
kind: PersistentVolume        
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteMany
  nfs:
    path: /home/azureuser/data
    server: 172.27.0.7
```





## PVC



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-pvc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: pvc-m
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
      - name: pvc-m
        persistentVolumeClaim:
          claimName: my-pvc

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```







## StorageClass

https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/

每个 StorageClass 都包含 `provisioner`、`parameters` 和 `reclaimPolicy` 字段， 这些字段会在 StorageClass 需要动态分配 PersistentVolume 时会使用到。

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-retain-sc
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain  # Default is Delete, recommended is retain
volumeBindingMode: WaitForFirstConsumer # Default is Immediate, recommended is WaitForFirstConsumer
allowVolumeExpansion: true  
parameters:
  storageaccounttype: Premium_LRS # or we can use Standard_LRS
  kind: managed # Default is shared (Other two are managed and dedicated)
```



PVC可以绑定到SC：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-retain-sc
  resources:
    requests:
      storage: 5Gi  
```







## StatefulSet

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

几个pod会一个一个创建

Details: 

```yaml
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
  minReadySeconds: 10 # by default is 0
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
      storageClassName: "managed-premium-retain-sc"
      resources:
        requests:
          storage: 1Gi
```





## ConfigMap

https://kubernetes.io/docs/concepts/configuration/configmap/

Details: https://github.com/WY-S/CKA/blob/main/K8S_ConfigMap.md

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfig
  namespace: default
data:
  special.level: info
  special.type: hello
```







## Secret

https://kubernetes.io/docs/concepts/configuration/secret/

Details: https://github.com/WY-S/CKA/blob/main/K8S_Secret.md

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:      
          name: mysecret   
          key: username    
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:      
          name: mysecret   
          key: password
```



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod1
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```







# 七、安全



## role

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```





## rolebinding

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/#rolebinding-example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# 此角色绑定允许 "jane" 读取 "default" 名字空间中的 Pod
# 你需要在该命名空间中有一个名为 “pod-reader” 的 Role
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# 你可以指定不止一个“subject（主体）”
- kind: User
  name: jane # "name" 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" 指定与某 Role 或 ClusterRole 的绑定关系
  kind: Role        # 此字段必须是 Role 或 ClusterRole
  name: pod-reader  # 此字段必须与你要绑定的 Role 或 ClusterRole 的名称匹配
  apiGroup: rbac.authorization.k8s.io
```



RoleBinding 也可以引用 ClusterRole，以将对应 ClusterRole 中定义的访问权限授予 RoleBinding 所在名字空间的资源。这种引用使得你可以跨整个集群定义一组通用的角色， 之后在多个名字空间中复用。

例如，尽管下面的 RoleBinding 引用的是一个 ClusterRole，"dave"（这里的主体， 区分大小写）只能访问 "development" 名字空间中的 Secrets 对象，因为 RoleBinding 所在的名字空间（由其 metadata 决定）是 "development"。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# 此角色绑定使得用户 "dave" 能够读取 "development" 名字空间中的 Secrets
# 你需要一个名为 "secret-reader" 的 ClusterRole
kind: RoleBinding
metadata:
  name: read-secrets
  # RoleBinding 的名字空间决定了访问权限的授予范围。
  # 这里隐含授权仅在 "development" 名字空间内的访问权限。
  namespace: development
subjects:
- kind: User
  name: dave # 'name' 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```





## clusterrole

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/#clusterrole-example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" 被忽略，因为 ClusterRoles 不受名字空间限制
  name: secret-reader
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 Secret 资源的名称为 "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```





## clusterrolebinding

https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/#clusterrolebinding-example

要跨整个集群完成访问权限的授予，你可以使用一个 ClusterRoleBinding。 下面的 ClusterRoleBinding 允许 "manager" 组内的所有用户访问任何名字空间中的 Secrets。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# 此集群角色绑定允许 “manager” 组中的任何人访问任何名字空间中的 Secret 资源
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager      # 'name' 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```





## ServiceAccount

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-service-account/#use-multiple-service-accounts

kubectl get serviceaccounts/build-robot -o yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: 2015-06-16T00:12:59Z
  name: build-robot
  namespace: default
  resourceVersion: "272500"
  uid: 721ab723-13bc-11e5-aec2-42010af0021e
secrets:
- name: build-robot-token-bvbk5
```







## NetworkPolicy

https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/

Details: K8S_NetworkPolicy.md

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```









# 八、集群维护



ETCD数据库备份与恢复

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#snapshot-using-etcdctl-options

```bash
# 备份
ETCDCTL_API=3 etcdctl snapshot save /home/azureuser/snapshotdb 
--endpoints=https://127.0.0.1:2379 
--cacert="/etc/kubernetes/pki/etcd/ca.crt" 
--cert="/etc/kubernetes/pki/etcd/server.crt" 
--key="/etc/kubernetes/pki/etcd/server.key"
```







## kubeadm升级k8s版本

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/



```yaml
apt update
apt-cache madison kubeadm

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.25.0-00 && \
apt-mark hold kubeadm

kubectl drain master --ignore-daemonsets --force

kubeadm version

kubeadm upgrade plan

kubeadm upgrade apply v1.25.0

kubectl uncordon master
systemctl daemon-reload
systemctl restart kubelet
```







## 正确下线节点流程

1、获取节点列表

```yaml
kubectl get node
kubectl get po -owide
```



2、设置不可调度

```yaml
kubectl cordon <node_name>
```



3、驱逐节点上的node

```yaml
kubectl drain <node_name> --ignore-daemonsets --force
```



4/1、移除节点

```yaml
kubectl delete node <node_name>
```



4/2、恢复节点可调度

```yaml
kubectl uncordon <node_name>
```





# 九、Troubleshooting



```bash
kubectl describe TYPE/NAME
kubectl logs TYPE/NAME [-c CONTAINER]
kubectl exec POD [-c CONTAINER] -- COMMAND [args]
```







# 十、考试准备



给出可能用到的标签

https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/
https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
https://kubernetes.io/zh/docs/tasks/administer-cluster/configure-upgrade-etcd/#%E5%A4%87%E4%BB%BD-etcd-%E9%9B%86%E7%BE%A4
https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/#networkpolicy-resource
https://kubernetes.io/zh/docs/concepts/services-networking/service/
https://kubernetes.io/zh/docs/concepts/services-networking/ingress/#the-ingress-resource
https://kubernetes.io/zh/docs/concepts/scheduling-eviction/assign-pod-node/
https://kubernetes.io/zh/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#%E5%88%9B%E5%BB%BA-persistentvolume
https://kubernetes.io/zh/docs/concepts/cluster-administration/logging/#sidecar-container-with-logging-agent
https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/
https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets-as-environment-variables
https://kubernetes.io/zh/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
https://kubernetes.io/zh/docs/tutorials/stateful-application/zookeeper/#%E5%AE%B9%E5%BF%8D%E8%8A%82%E7%82%B9%E6%95%85%E9%9A%9C
https://kubernetes.io/zh/docs/concepts/storage/volumes/#hostpath



cheat sheet：

https://kubernetes.io/docs/reference/kubectl/cheatsheet/



经验总结：

https://blog.csdn.net/red_sky_blue/article/details/123728008

https://blog.csdn.net/u014442879/article/details/113123245?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EESLANDING%7Edefault-6-113123245-blog-86300826.pc_relevant_multi_platform_whitelistv4eslandingrelevant2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EESLANDING%7Edefault-6-113123245-blog-86300826.pc_relevant_multi_platform_whitelistv4eslandingrelevant2&utm_relevant_index=7

https://glory.blog.csdn.net/article/details/102966474?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EESLANDING%7Edefault-5-102966474-blog-86300826.pc_relevant_multi_platform_whitelistv4eslandingrelevant2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EESLANDING%7Edefault-5-102966474-blog-86300826.pc_relevant_multi_platform_whitelistv4eslandingrelevant2&utm_relevant_index=6



真题练习：

https://blog.csdn.net/red_sky_blue/article/details/123728008

https://note.youdao.com/ynoteshare/index.html?id=541a8e5fc51753a1472a7cd2eccdb6cb&type=note&_time=1661705509236

https://blog.csdn.net/deerjoe/article/details/86300826









官方文档：

https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2

https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad





























































