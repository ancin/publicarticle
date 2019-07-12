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
