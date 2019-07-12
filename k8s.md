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
   