#环境
$ hostnamectl set-hostname master.example.com

$ yum update -y
$ yum clean all
$ reboot

#安装基础包
$ yum install wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct

$ yum install wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct \
dnsmasq ntp logrotate httpd-tools firewalld libselinux-python conntrack-tools openssl iproute python-dbus PyYAML \
glusterfs-fuse device-mapper-multipath nfs-utils iscsi-initiator-utils ceph-common atomic python-docker-py

# 安装 Docker
$ yum install -y docker
$ systemctl start docker
$ systemctl enable docker
鉴于国内访问 DockerHub 下载镜像的速度过于缓慢，可以使用中国科技大学的 DockerHub 镜像服务器进行加载。编辑 /etc/sysconfig/docker 文件，为 DOCKER_OPTS 变量追加参数 --registry-mirror=https://docker.mirrors.ustc.edu.cn。修改后变量值大致如下：

OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --registry-mirror=https://docker.mirrors.ustc.edu.cn'

add OPTIONS --insecure-registry "172.30.0.0/16" to docker daemon

$ systemctl restart docker

# 安装及启动 OpenShift
## 官方github下载 并解压
将 OpenShift 的相关命令追加至系统的 PATH 环境变量中。编辑 /etc/profile 文件，添加如下文本内容至文件末尾。

PATH=$PATH:/opt/openshift/

$ source /etc/profile

尝试 
$ openshift version

#设置开机启动
在 /etc/rc.d/rc.local 文件末尾，添加如下启动命令。
$ cd /opt/openshift && sudo ./openshift start --master-config=/home/origin/openshift.local.config/master/master-config.yaml --node-config=/home/origin/openshift.local.config/node-master.example.com/node-config.yaml &> /home/logs/all.log
$ chmod +x /etc/rc.d/rc.local

# oc启动

oc cluster up \ –host-data-dir=’HOME/oc/profiles/HOME/oc/profiles/PROFILE/data’ \ –host-config-dir=’HOME/oc/profiles/HOME/oc/profiles/PROFILE/config’
 oc cluster up --public-hostname '192.168.42.65'
 oc cluster up --public-hostname '192.168.8.223'


