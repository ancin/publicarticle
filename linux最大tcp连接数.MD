# 详解Linux服务器最大tcp连接数

网络编程
在tcp应用中，server事先在某个固定端口监听，client主动发起连接，经过三路握手后建立tcp连接。那么对单机，其最大并发tcp连接数是多少？
如何标识一个TCP连接
在确定最大连接数之前，先来看看系统如何标识一个tcp连接。系统用一个4四元组来唯一标识一个TCP连接：{local ip, local port, remote ip, remote port}。
client最大tcp连接数
client每次发起tcp连接请求时，除非绑定端口，通常会让系统选取一个空闲的本地端口（local port），该端口是独占的，不能和其他tcp连接共享。tcp端口的数据类型是unsigned short，因此本地端口个数最大只有65536，端口0有特殊含义，不能使用，这样可用端口最多只有65535，所以在全部作为client端的情况下，最大tcp连接数为65535，这些连接可以连到不同的server ip。
server最大tcp连接数
server通常固定在某个本地端口上监听，等待client的连接请求。不考虑地址重用（unix的SO_REUSEADDR选项）的情况下，即使server端有多个ip，本地监听端口也是独占的，因此server端tcp连接4元组中只有remote ip（也就是client ip）和remote port（客户端port）是可变的，因此最大tcp连接为客户端ip数×客户端port数，对IPV4，不考虑ip地址分类等因素，最大tcp连接数约为2的32次方（ip数）×2的16次方（port数），也就是server端单机最大tcp连接数约为2的48次方。
那么单台服务器上的并发TCP连接数可以有多少
上面给出的是理论上的单机最大连接数，在实际环境中，受到机器资源、操作系统等的限制，特别是sever端，其最大并发tcp连接数远不能达到理论上限。在unix/linux下限制连接数的主要因素是内存和允许的文件描述符个数（每个tcp连接都要占用一定内存，每个socket就是一个文件描述符），另外1024以下的端口通常为保留端口。在默认2.6内核配置下，经过试验，每个socket占用内存在15~20k之间。
影响一个socket占用内存的参数包括：
rmem_max
wmem_max
tcp_rmem
tcp_wmem
tcp_mem
grep skbuff /proc/slabinfo
对server端，通过增加内存、修改最大文件描述符个数等参数，单机最大并发TCP连接数超过10万 是没问题的，国外 Urban Airship 公司在产品环境中已做到 50 万并发 。在实际应用中，对大规模网络应用，还需要考虑C10K 问题。
下面再详解一下文件句柄限制和端口限制两大常识
常识一：文件句柄限制
在linux下编写网络服务器程序的朋友肯定都知道每一个tcp连接都要占一个文件描述符，一旦这个文件描述符使用完了，新的连接到来返回给我们的错误是“Socket/File:Can't open so many files”。
这时你需要明白操作系统对可以打开的最大文件数的限制。
进程限制
执行 ulimit -n 输出 1024，说明对于一个进程而言最多只能打开1024个文件，所以你要采用此默认配置最多也就可以并发上千个TCP连接。
临时修改：ulimit -n 1000000 ，但是这种临时修改只对当前登录用户目前的使用环境有效，系统重启或用户退出后就会失效。
重启后失效的修改（不过我在CentOS 6.5下测试，重启后未发现失效）：编辑 /etc/security/limits.conf 文件， 修改后内容为

soft nofile 1000000
hard nofile 1000000

永久修改：编辑/etc/rc.local，在其后添加如下内容
ulimit -SHn 1000000
全局限制
执行 cat /proc/sys/fs/file-nr 输出 9344 0 592026，分别为：

已经分配的文件句柄数，
已经分配但没有使用的文件句柄数，
最大文件句柄数。

在kernel 2.6版本中第二项的值总为0，这并不是一个错误，它实际上意味着已经分配的文件描述符无一浪费的都已经被使用了 。
我们可以把这个数值改大些，用 root 权限修改 /etc/sysctl.conf 文件:
fs.file-max = 1000000

net.ipv4.ip_conntrack_max = 1000000

net.ipv4.netfilter.ip_conntrack_max = 1000000

常识二：端口号范围限制？
操作系统上端口号1024以下是系统保留的，从1024-65535是用户使用的。由于每个TCP连接都要占一个端口号，所以我们最多可以有60000多个并发连接。我想有这种错误思路朋友不在少数吧？（其中我过去就一直这么认为）
我们来分析一下吧
如何标识一个TCP连接：系统用一个4四元组来唯一标识一个TCP连接：{local ip, local port, remote ip, remote port}。好吧，我们拿出《UNIX网络编程：卷一》第四章中对accept的讲解来看看概念性的东西，第二个参数cliaddr代表了客户端的ip地址和端口号。而我们作为服务端实际只使用了bind时这一个端口，说明端口号65535并不是并发量的限制。
server最大tcp连接数：server通常固定在某个本地端口上监听，等待client的连接请求。不考虑地址重用（unix的 SO_REUSEADDR 选项）的情况下，即使server端有多个ip，本地监听端口也是独占的，因此server端tcp连接4元组中只有remote ip（也就是client ip）和remote port（客户端port）是可变的，因此最大tcp连接为客户端ip数×客户端port数，对IPV4，不考虑ip地址分类等因素，最大tcp连接数约为2的32次方（ip数）×2的16次方（port数），也就是server端单机最大tcp连接数约为2的48次方。
