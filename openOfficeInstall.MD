https://blog.csdn.net/m0_37039484/article/details/80008519
# 下载
##解压
tar -zxvf Apache_OpenOffice_4.1.4_Linux_x86-64_install-rpm_zh-CN.tar.gz  -C ../softwares/

解压之后会在 /opt/softwares 中生成  文件夹，进入中文件夹。

[root@localhost moudles]# cd /opt/softwares/zh-CN/RPMS

 运行 yum localinstall *.rpm

 成功之后会在当前目录生成 文件夹，运行 yum localinstall 。
  yum localinstall openoffice4.1.4-redhat-menus-4.1.4-9788.noarch.rpm
## 启动服务
  [root@localhost desktop-integration]# nohup /opt/openoffice4/program/soffice -headless -accept="socket,host=127.0.0.1,port=8100;urp;" -nofirststartwizard &
## 查看是否成功

[root@localhost opt]# ps -ef|grep openoffice

## 查看端口
 [root@localhost opt]# netstat -lnp |grep 8100
 
 
 
# centos 乱码解决
## 找到jdk 安装路径。
## 将字体：simhei.ttf 黑体、simsun.ttc 宋体（windows下通过everything找下）上传至/usr/java/jdk1.8.0_91/jre/lib/fonts/fallback路径下。
# 查看系统路径
## root@80ec6 fallback]# cat /etc/fonts/fonts.conf
# 字体拷贝。
## 将 /usr/java/jdk1.8.0_91/jre/lib/fonts的全部内容，拷贝到步骤3查看的路径下， 我的字体路径为：/usr/share/fonts。

# 更新缓存

  执行命令：fc-cache
## kill掉openoffice进程。

[root@80ec6 fonts]# ps -ef | grep openoffice
  kill -9 xxx
## 重启后台运行openoffice
 