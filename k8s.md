# Can't connect to 'docker' daemon on building kubernetes from source


The problem you are experiencing is caused by the fact that you are unable to access the Docker socket /var/run/docker.sock as a non-root user. When you run sudo docker version you are running the Docker client as root so it does not experience this problem.

This is a basic Unix permissions problem and there are the standard solutions:

You could run the Kubernetes build as root with sudo make release.
You can fix the permissions on the socket such that you are able to use Docker without sudo.
If you look at the permissions on the Docker socket, you will probably see something like:

- $ ls -l /var/run/docker.sock /var/run/docker.sock
srw-rw----. 1 root docker 0 Mar 17 12:26 /var/run/docker.sock
This shows a socket that is readable by root and by members of the docker group. In this case, I am a member of the docker group so I can run the docker client without sudo. You could set up the same thing in your environment.

Note that of course you always need to start the Docker daemon as root, but in general you would expect to have this configured to start automatically when your system boots, rather than starting it manually.

# centos install kubernetes

## close firewall
  $ systemctl stop firewalld

  $ yum install etcd kubernetes
  把/etc/kubernetes/apiserver --admission_control Service_Account删除

  ### 按照顺序启动
   $ systemctl start etcd
   $ systemctl start docker
   $ systemctl start kube-apiserver
   $ systemctl start kube-controller-manager
   $ systemctl start kube-scheduler
   $ systemctl start kubelet
   $ systemctl start kube-proxy

   ## 定义RC文件



   # 安装etcd
   yum install etcd
   #配置etcd
   系统会自动生成etcd.service文件（路径为/usr/lib/systemd/system/），修改对应的配置
   [Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd \
--name=\"${ETCD_NAME}\" \
--data-dir=\"${ETCD_DATA_DIR}\" \
--listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" \
--advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" \
--initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" \
--initial-cluster=\"${ETCD_INITIAL_CLUSTER}\"  \
--initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\" \
--listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\""
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

## 修改配置 /etc/etcd/etcd.conf
ETCD_NAME=zwetcd_2
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"ETCD_LISTEN_PEER_URLS="http://192.168.37.131:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.37.131:2379,http://127.0.0.1:2379"

ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.37.131:2380"

ETCD_INITIAL_CLUSTER="zwetcd_2=http://192.168.37.131:2380,zwetcd_1=http://192.168.37.130:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.37.131:2379"

### 如果使用firewalld作为防火墙，则需要开放端口：

firewall-cmd --zone=public --add-port=2379/tcp --permanent
firewall-cmd --zone=public --add-port=2380/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all









=============================================================================================

kubernetes角色组成：

1）Pod
Pod是kubernetes的最小操作单元，一个Pod可以由一个或多个容器组成；
同一个Pod只能运行在同一个主机上，共享相同的volumes、network、namespace；
2）ReplicationController（RC）
RC用来管理Pod，一个RC可以由一个或多个Pod组成，在RC被创建后，系统会根据定义好的副本数来创建Pod数量。在运行过程中，如果Pod数量小于定义的，就会重启停止的或重新分配Pod，反之则杀死多余的。当然，也可以动态伸缩运行的Pods规模。
RC通过label关联对应的Pods，在滚动升级中，RC采用一个一个替换要更新的整个Pods中的Pod。
3）Service
Service定义了一个Pod逻辑集合的抽象资源，Pod集合中的容器提供相同的功能。集合根据定义的Label和selector完成，当创建一个Service后，会分配一个Cluster IP，这个IP与定义的端口提供这个集合一个统一的访问接口，并且实现负载均衡。

4）Label
Label是用于区分Pod、Service、RC的key/value键值对； 
Pod、Service、RC可以有多个label，但是每个label的key只能对应一个；
主要是将Service的请求通过lable转发给后端提供服务的Pod集合；
kubernetes组件组成：

1）kubectl
客户端命令行工具，将接受的命令格式化后发送给kube-apiserver，作为整个系统的操作入口。

2）kube-apiserver
作为整个系统的控制入口，以REST API服务提供接口。

3）kube-controller-manager
用来执行整个系统中的后台任务，包括节点状态状况、Pod个数、Pods和Service的关联等。

4）kube-scheduler
负责节点资源管理，接受来自kube-apiserver创建Pods任务，并分配到某个节点。

5）etcd
负责节点间的服务发现和配置共享。

6）kube-proxy
运行在每个计算节点上，负责Pod网络代理。定时从etcd获取到service信息来做相应的策略。

7）kubelet
运行在每个计算节点上，作为agent，接受分配该节点的Pods任务及管理容器，周期性获取容器状态，反馈给kube-apiserver。

8）DNS
一个可选的DNS服务，用于为每个Service对象创建DNS记录，这样所有的Pod就可以通过DNS访问服务了。

 在master 查看 node状态
 kubectl get nodes

配置pod和容器
一定义容器的环境变量

创建pod时，可以为在pod中运行的容器是这环境变量， 要设置环境变量，需要env 字段包含在配置文件中

pod的配置文件定义了一个 名称DEMO_GREETING 和 值为 环境变量 “Hello from the environment”

pod.yaml

apiVersion: v1
kind: Pod
metadata:                           #元数据信息
  name: envar-demo                  #kubectl get  pods 和 登陆容器显示的名字
  labels:                           #标签
    purpose: demonstrate-envars     #标签，可以作为查询条件 kubectl get pods -l purpose=demonsstrate-envars
spec:　　　　　　　　　　　　　　　　　　#规格
  containers:                       #容器
  - name: envar-demo-container      #容器名称
    image: docker.cinyi.com:443/senyint/centos7.3   #使用的镜像
    env:　　　　　　　　　　　　　　　　 #设置env，登陆到容器中查看环境变量， DEME_GREETING 的值是 "hello from the enviroment"
    - name:DEME_GREETING
      value: "hello from the environment"

1. 基于YAML配置文件创建pod
[root@master ~]# kubectl create -f pod.yaml      

2.列出正在运行的pods
[root@master ~]# kubectl get pods 

3.列出标签中 purpose等于demonstrate-envars的pod
[root@master ~]# kubectl get pods -l purpose=demonstrate-envars
   
 
  4.获取一个shell 进入到pod 中运行的容器中
 [root@master ~]# kubectl exec -it envar-demon /bin/bash

 # 定义容器的命令和参数
 创建pod时，可以为在pod中运行的容器定义命令和参数， 要定义命令，需要把command字段包含在配置文件中， 要定义参数，请将该args字段包含在配置文件中，创建pod后， 无法更改定义的命令和参数

您在配置文件中定义的命令和参数会覆盖容器图像提供的默认命令和参数

command_pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: docker.cinyi.com:443/senyint/centos7.3
    command: ["printenv"]
    args: ["HOSTNAME", "KUBENETES_PORT"]

1. 基于yaml配置文件常见pod
[root@master ~]# kubectl create -f  command_pod.yaml

2.列出正在运行的pods
[root@master ~]# kubectl get pods

3. 查看容器中运行的命令的输出，请查看pod中的日志
[root@master ~]# kubectl log  comman-demon

显示的HOSTNAME 和 KUBENETES_PORT环境变量值为

# 想容器分配CPU 和RAM 资源
创建Pod时，可以为在Pod中运行的容器请求CPU和RAM资源。您还可以设置CPU和RAM资源的限制。要请求CPU和RAM资源，请resources:requests在配置文件中包含该字段。要设置CPU和RAM资源的限制，请包括 resources:limits字段。

只有当节点有足够的CPU和RAM可用来满足Pod中所有容器请求的总CPU和RAM时，Kubernetes才会计划一个Pod在节点上运行。此外，当容器在节点上运行时，Kubernetes不允许容器使用的CPU和RAM超过为容器指定的限制。如果容器超过其RAM限制，它将终止。如果容器超过其CPU限制，它将成为其CPU使用受到限制的候选。

Pod的配置文件请求250 milicpu和64 mebibtes的RAM。它还设置了1 cpu和128兆字节RAM的上限

cpu_ram_pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: cpu-ram-demo
spec:
  containers:
  - name: cpu-ram-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"


1. 基于yaml配置文件常见pod

[root@master ~]# kubectl create -f cpu_ram_pod.yaml

2. 查看pods cpu-ram-demon详细信息

# 配置单元以将卷用于存储
容器的文件系统只存在于容器中，因此当容器终止并重新启动时，文件系统的更改将丢失。对于独立于容器的更一致的存储，您可以使用 卷。这对于状态应用程序（例如键值存储和数据库）尤其重要。例如，Redis是一个键值缓存和存储。

创建一个运行一个容器的Pod。这个Pod有一个类型为EmptyDir的卷， 它持续了Pod的生命周期，即使容器终止和重新启动。这里是Pod的配置文件


apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}

# 使用服务访问集群中的应用程序目的

运行Hello World应用程序的两个实例。
创建公开节点端口的Service对象。
使用Service对象访问正在运行的应用程序
1. 在进去中运行Hello World应用程序

[root@kubernetes ~]#  kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080

上述命令将创建一个 Deployment 对象和一个关联的 ReplicaSet 对象。ReplicaSet有两个 Pod，每个都运行Hello World应用程序。
2. 显示有关部署的信息：

复制代码
[root@kubernetes ~]# kubectl get pods
　　NAME 　　　　　　　　　　　　　　　　READY 　　　　STATUS R　　　　　　ESTARTS 　　　　　AGE
　　hello-world-2895499144-d99r4 　　 0/1 　　　　 Running　　　　　　 0 　　　　　　　　 42s
　　hello-world-2895499144-v5wkb 　　 1/1 　　　　 Running　　　　　　 0 　　　　　　　　 2h

 [root@kubernetes ~]# kubectl describe hello-world-2895499144-d99r4

 3.显示有关ReplicaSet 对象的信息：
   kubectl get replicasets
   kubectl describe replicasets
4. 创建公开部署的Service对象

[root@kubernetes ~]# kubectl expose deployment hello-world --type=NodePort --name=example-server
service "example-server" exposed
 

5.显示有关服务器的信息

复制代码
[root@kubernetes ~]# kubectl get service
NAME             CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
example-server   10.254.140.175   <nodes>       8080/TCP    45s
kubernetes       10.254.0.1       <none>        443/TCP     2d
nginxserver      10.254.235.210   <none>        11111/TCP   21h

[root@kubernetes ~]# kubectl describe service example-server
Name: 　　　　 example-server
Namespace:	　 default
Labels:	　　  run=load-balancer-example
Selector:	　　run=load-balancer-example
Type:	　　　　	NodePort
IP:	　　　　  	10.254.140.175
Port:	　　　　	<unset>	8080/TCP
NodePort:	　　<unset>	30049/TCP    #自动分配的
Endpoints:	　10.0.224.4:8080
Session Affinity:	None
No events.

记下服务的NodePort值。例如，在前面的输出中，NodePort值为31496。

 http://192.168.20.224:31496
 6.列出运行Hello world 应用程序的pod

 kubectl  get pods --selector="run=load-balancer-example"
NAME                           READY     STATUS             RESTARTS   AGE
hello-world-2895499144-d99r4   0/1       ImagePullBackOff   0          17m
hello-world-2895499144-v5wkb   1/1       Running            0          2h

[root@kubernetes ~]# kubectl  get pods --selector="run=load-balancer-example" --output=wide
NAME                           READY     STATUS             RESTARTS   AGE       IP           NODE
hello-world-2895499144-d99r4   0/1       ImagePullBackOff   0          17m       10.0.225.4   192.168.20.225
hello-world-2895499144-v5wkb   1/1       Running            0          2h        10.0.224.4   192.168.20.224

7. 使用节点地址和节点端口访问Hello World应用程序：

url http://<public-node-ip>:<node-port>

其中<public-node-ip>是您的节点的公共IP地址，<node-port>是您的服务的NodePort值。

[root@kubernetes ~]# curl  192.168.20.224:30049

Hello Kubernetes!
8. 删除服务
kubectl  delete service example-server
service "example-server" deleted

要删除运行Hello World应用程序的Deployment，ReplicaSet和Pod，请输入以下命令
　　[root@kubernetes ~]# kubectl delete deployment hello-world
　　deployment "hello-world" deleted
# 公开外部IP地址以访问集群中的应用程序
运行Hello World应用程序的五个实例。
创建公开外部IP地址的Service对象。
使用Service对象访问正在运行的应用程序。
 

创建5个pod 运行的应用程序

[root@kubernetes ~]# kubectl  run hello-world --replicas=5 --labels="run=load-balncer-example" --image=gcr.io/google-samples/node-hello:1.0 --port=8080

显示有关部署的信息
[root@kubernetes ~]# kubctl get pods
[root@kubernetes ~]# kubectl  describe pods hello-world

显示有关ReplicaSet对象的信息
[root@kubernetes ~]# kubectl  get  replicasets
[root@kubernetes ~]# kubectl  describe replicasets
[root@kubernetes ~]# kubectl  describe pods hello-world


创建部署的Service对象
[root@kubernetes ~]#  kubectl expose deployment hello-world --type=LoadBalancer --name=my-services

显示有关服务的信息
[root@kubernetes ~]#  kubectl get service
[root@kubernetes ~]#  kubectl describe service my-service

# 使用部署运行无状态应用程序
创建nginx部署。
使用kubectl列出有关部署的信息。
更新部署。

创建和探索nginx部署

vim  deployment.yaml 

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.cinyi.com:443/nginx:v1.0
        ports:
        - containerPort: 80


[root@kubernetes nginx]#  kubectl  create -f deployment.yam

[root@kubernetes nginx]#  kubectl  describe deployment

[root@kubernetes nginx]#  kubectl  describe pods  nginx-deployment

[root@kubernetes nginx]#  kubectl  exec -it nginx-deployment-253806509-bfwxw /bin/bash 



更新部署,更新images

vim  depolyment_update.yaml


apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.cinyi.com:443/nginx:v1.1          #更新了镜像版本
        ports:
        - containerPort: 80
应用新的yaml文件，更新了nginx版本
[root@kubernetes nginx]# kubectl  apply -f deployment_update.yaml  
通过命令查看更新情况

[root@kubernetes nginx]# kubectl get pods -l app=nginx



 
 
更新部署,更新replica副本数

vim  depolyment_replicas.yaml



apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: docker.cinyi.com:443/nginx:v1.1          #更新了镜像版本
        ports:
        - containerPort: 80

应用新的yaml文件，更新了nginx版本
[root@kubernetes nginx]# kubectl  apply -f deployment_update.yaml  

通过命令查看更新情况


[root@kubernetes nginx]# kubectl get pods -l app=nginx

删除部署

[root@kubernetes nginx]#  kubectl delete deployment nginx-deployment

资源管理

许多应用程序需要创建多个资源，例如deployment和service 。通过将多个资源一起分组在同一文件（由---YAML 分隔）中，可以简化对多个资源的管理

apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80



[root@kubernetes nginx]# kubectl create -f nginx.yaml

[root@kubernetes nginx]# kubectl create -f nginx-1.yaml -f nginx-2.yaml 
deployment "nginx-deployment" created
service "my-nginx-svc" created



缩放应用程序

当应用程序的负载增长或缩小时，可以轻松扩展kubectl。例如，要将nginx副本的数量从3减少到1，请执行以下操作

kubectl scale deployment/my-nginx --replicas=1

# kubernet-dashboard安装
下载dashboard.yaml
kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml

2.修改yaml文件
image 镜像从
gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1
修改成: image: docker.cinyi.com:443/kubernetes-dashboard-amd64:v1.5.1

打开注释，修改成apiserverIP地址和端口
- --apiserver-host=http://192.168.20.226:8080

1.下载dashboard.yaml
kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml

2.修改yaml文件
image 镜像从
gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1
修改成: image: docker.cinyi.com:443/kubernetes-dashboard-amd64:v1.5.1

打开注释，修改成apiserverIP地址和端口
- --apiserver-host=http://192.168.20.226:8080



配置文件如下：
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  labels:
    app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubernetes-dashboard
  template:
    metadata:
      labels:
        app: kubernetes-dashboard
      annotations:
        scheduler.alpha.kubernetes.io/tolerations: |
          [
            {
              "key": "dedicated",
              "operator": "Equal",
              "value": "master",
              "effect": "NoSchedule"
            }
          ]
    spec:
      containers:
      - name: kubernetes-dashboard
        image: docker.cinyi.com:443/kubernetes-dashboard-amd64:v1.5.1
        imagePullPolicy: Always
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
           - --apiserver-host=http://192.168.20.226:8080
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 9090
  selector:
    app: kubernetes-dashboard


3.执行yaml文件
[root@kubernetes dashboard]#  kubectl create -f kubernetes-dashboard.yaml

4. 使用命令kubectl get pod –all-namespaces查看创建的pod

[root@kubernetes dashboard]#  kubectl get pod --all-namespaces
NAMESPACE     NAME                                   READY     STATUS    RESTARTS   AGE
kube-system   kubernetes-dashboard-277061766-cu9vp   1/1       Running   0          22m


5. 使用describe 查看详细信息，IP地址，端口等

[root@kubernetes dashboard]# kubectl describe svc kubernetes-dashboard --namespace=kube-system
Name:            kubernetes-dashboard
Namespace:        kube-system
Labels:            app=kubernetes-dashboard
Selector:                app=kubernetes-dashboard
Type:            NodePort
IP:                   10.254.159.33
Port:                   <unset>    80/TCP
NodePort:               <unset>    31457/TCP
Endpoints:               10.0.223.2:9090
Session Affinity:           None
No events.

IP:10.254.159.33 是cluster-ip，外部无法访问
Endpoint:10.0.223.2:9090 是docker container IP

如果访问kubernetes dashboard， 需要访问nodeport IP 和端口31457
例如 http://192.168.20.223:31457


