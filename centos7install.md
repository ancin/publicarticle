
# downlaod centos from mirrors.163.com

# config

sudo vi  /etc/sysconfig/network-scripts/ifcfg-eth0
第二步：
只需要将最后一项：
    ONBOOT=no 改成 yes
保存退出
第三步：
重启网卡服务：
   /etc/init.d/network restart
# install VBoxLinuxAdditions tools
1. 更新内核
yum update kernel
2.安装依赖包
yum install kernel-headers
yum install kernel-devel
yum install gcc*
yum install make
3.创建/mnt/cdrom目录
mkdir -p /mnt/cdrom
4.挂载cdrom
mount -t auto /dev/cdrom /mnt/cdrom
当出现“mount:block device /dev/cdrom is write-protected,mounting read-only”表示挂载成功。

5.查看cdrom中的内容
ls -l /mnt/cdrom
6.利用mv 或者cp 拷出里面的内容
7.卸载cdrom
umount /mnt/cdrom