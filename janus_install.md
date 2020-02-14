# Janus 安装
yum install -y epel-release && \
yum update -y && \
yum install -y deltarpm && \
yum install -y openssh-server sudo which file curl zip unzip wget && \
yum install -y libmicrohttpd-devel jansson-devel libnice-devel glib22-devel opus-devel libogg-devel pkgconfig  gengetopt libtool autoconf automake make gcc gcc-c++ git cmake libconfig-devel openssl-devel

#upgrade libsrtp 1.5.4
#wget https://github.com/cisco/libsrtp/archive/v1.5.4.tar.gz
#tar xfv v1.5.4.tar.gz
#cd libsrtp-1.5.4
#./configure --prefix=/usr --enable-openssl
#make shared_library && sudo make install

#cd -

wget https://github.com/cisco/libsrtp/archive/v2.0.0.tar.gz
tar xfv v2.0.0.tar.gz
cd libsrtp-2.0.0
./configure --prefix=/usr --enable-openssl
make shared_library && sudo make install

cd -

#install sofia-sip for sip-gateway plugin
wget https://sourceforge.net/projects/sofia-sip/files/sofia-sip/1.12.11/sofia-sip-1.12.11.tar.gz
tar zxf sofia-sip-1.12.11.tar.gz && cd sofia-sip-1.12.11 && ./configure --prefix=/usr CFLAGS=-fno-aggressive-loop-optimizations && make && make install

cd -

#install usrsctp for Data channel support
git clone https://github.com/sctplab/usrsctp && cd usrsctp && \
./bootstrap && \
./configure --prefix=/usr && make && make install

cd -

#install libwebsocket for android or ios instead of http/https
git clone https://github.com/warmcat/libwebsockets && \
mkdir libwebsockets/build && cd libwebsockets/build && \
cmake -DMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_C_FLAGS="-fpic" .. && \
make && make install

cd -

#Janus
#if cannot configure plugin sofia,Perhaps you should add the directory containing `sofia-sip-ua.pc' to the PKG_CONFIG_PATH environment variable,
#for example centos7 :export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
#if cannot load libsofia-sip-ua.so.0 , try ldconfig -v
git clone https://github.com/meetecho/janus-gateway.git && \
cd janus-gateway &&\
sh autogen.sh && \
./configure --prefix=/opt/janus --disable-rabbitmq --disable-docs &&\
make && make install && make configs
-


# 问题解决：

.ibtoolize: AC_CONFIG_MACRO_DIR([./aclocal]) conflicts with ACLOCAL_AMFLAGS=-I ./aclocal

执行libtoolize遇到上面提示的错误时，可能是因为configure.ac和Makefile.am文件是dos格式导致的，使用dos2unix转换一下后再执行，问题可能就解决了。

vim configure.ac
:set fileformat=unix
:wq

## error while loading shared libraries: libGL.so.1: cannot open shared object file
cat /etc/ld.so.conf  

若是地址显示是 默认搜寻目录(/lib和/usr/lib)以及动态库配置文件/etc/ld.so.conf内所列的目录下，那么就只需要执行ldconfig.
命令型模式输入：sudo ldconfig
否则，需要先将库文件所在绝对地址写入到配置文件中，再ldconfig：
命令型模式输入：sudo echo "/.../libXX.so.X'" >> /etc/ld.so.conf

 sudo ldconfig
 https://blog.csdn.net/u011132979/article/details/73201234

 netstat -anp | grep janus

 janus.cfg
可能需要配置下turn，该配置文件提供两种方式turn的配置
1.静态配置的turn server

[nat]
turn_server = 127.0.0.1
turn_port = 3478
turn_type = udp
turn_user = username
turn_pwd = password

2.配置TURN REST API动态获取turn 服务

turn_rest_api = http://yourbackend.com/path/to/api
turn_rest_api_key = anyapikeyyoumayhaveset
turn_rest_api_method = GET

其他相关配置需要的话可以参考对应注释使用

janus.transport.http.cfg
本文使用的消息通道是http，以http配置为例

[general]
json = indented                         ; Whether the JSON messages should be indented (default),
                                                        ; plain (no indentation) or compact (no indentation and no spaces)
base_path = /janus                      ; Base path to bind to in the web server (plain HTTP only)
threads = unlimited                     ; unlimited=thread per connection, number=thread pool
http = yes                                      ; Whether to enable the plain HTTP interface
port = 18000                                    ; Web server HTTP port
;interface = eth0                       ; Whether we should bind this server to a specific interface only
;ip = 192.168.0.1                       ; Whether we should bind this server to a specific IP address (v4 or v6) only
https = yes                                     ; Whether to enable HTTPS (default=no)
secure_port = 18001                     ; Web server HTTPS port, if enabled

在这里可以配置监听端口线程数等，支持http和https

janus.plugin.sip.cfg
以sip(sofia)插件为例子

[general]
; Specify which local IP address to use. If not set it will be automatically
; guessed from the system
;local_ip = 1.2.3.4

; Enable local keep-alives to keep the registration open. Keep-alives are
; sent in the form of OPTIONS requests, at the given interval inseconds.
; (0 to disable)
keepalive_interval = 120

; Indicate if the server is behind NAT. If so, the server will use STUN
; to guess its own public IP address and use it in the Contact header of
; outgoing requests
behind_nat = no

; User-Agent string to be used
; user_agent = Cool WebRTC Gateway

; Expiration time for registrations
register_ttl = 3600

; Range of ports to use for RTP/RTCP (default=10000-60000)
rtp_port_range = 40000-60000

可以配置rtp端口范围等，若服务部署在nat网络下可以配置behind_nat = yes，则服务端会通过turn进行网络穿透
## Extension
认demo里面没有配置iceserver，如果服务端报错ice fail可以在客户端代码加上iceserver
vim siptest.js
1
janus = new Janus(
				{
					server: server,
					iceServers: [{urls: "turn:turnip:port", username: "username", credential: "password"}],
					success: function() {
demo中默认没有启用录音功能，但是每个插件都有提供录音接口，可在源码内查看对应REST API说明，在siptest中录音接口说明如下

{
	"request" : "recording",
	"action" : "<start|stop, depending on whether you want to start or stop recording something>"
	"audio" : <true|false; whether or not our audio should be recorded>,
	"video" : <true|false; whether or not our video should be recorded>,
	"peer_audio" : <true|false; whether or not our peer's audio should be recorded>,
	"peer_video" : <true|false; whether or not our peer's video should be recorded>,
	"filename" : "<base path/filename to use for all the recordings>"
}
开始录音
sipcall.send({"message": { "request": "recording", "action":"start","audio":true,"peer_audio":true,"filename":"/opt/janus/share/janus/recordings/" + $('#authuser').val() + "to" + $('#peer').val() + "_" + getNowFormatDate()}});
停止录音
sipcall.send({"message": { "request": "recording", "action":"stop"}});

其他插件如videocall videoroom类似可以录音

janus.js
demo中使用了janus.js。janus.js是和janus服务器进行通信的javascript库，通过使用janus.js简化了webrtc api的使用，以及前端与janus服务器建立连接，交换sdp等功能。不使用janus.js可以自己实现类似功能与janus服务器通信

https://blog.csdn.net/u012231640/article/details/83618274


# Turn部署
git clone https://github.com/coturn/coturn.git
cd coturn
./configure
make && make install

## corturn 配置

## 使用turnadmin添加用户及计算密码
 添加lt-cred-mech用户： 
./turnadmin -a -u XXXX -p 123456ty -r XXXX
其中的参数含义 
-a: 表示使用lt-cred-mech方式连接 
-u：用户名 
-p：密码 
-r ：realm 域。自己试了，似乎随便写，没啥影响。

添加admins用户，应该是使用https时使用吧 
./turnadmin -A -u XXX -p 123456 -r XXXX

其中的参数含义同上， 
-A：表示admin用户。

根据用户名、密码、realm计算出值 
./turnadmin -k -u XXX -p 123456 -r XXX

此时会出来一串数字。 
XXXX

## 配置turnserver.conf文件



turnserver -L  -o -a -b /etc/turnuserdb.conf -f -r shanghai

openssl req -x509 -newkey rsa:2048 -keyout /usr/local/etc/turn_server_pkey.pem -out /usr/local/etc/turn_server_cert.pem -days 99999 -nodes

vim /usr/local/etc/turnuserdb.conf

relay-device=eth0
listening-ip=172.31.148.197
listening-port=3478
tls-listening-port=5349
relay-ip=172.31.148.197
external-ip=47.105.XXX
relay-threads=20
lt-cred-mech
cert=/usr/local/etc/turn_server_cert.pem
pkey=/usr/local/etc/turn_server_pkey.pem
min-port=49152
max-port=65535
userdb=/usr/local/etc/turnuserdb.conf
user=songpeng:0x58f28fffe635382b060eaddc332ed93a
no-loopback-peers  
no-multicast-peers  
no-tcp  
no-tls  
no-cli

turnserver -v -r 47.105.XXXX:3478 -a -o -c /usr/local/etc/turnserver.conf

/usr/local/var/db/turndb


bin/turnserver -v -r 47.105.XXX:3478 -a -o -c /root/live/coturn/turnserver.conf
# centos 防火墙开启
firewall-cmd  --add-port=340079/tcp
systemctl stop firewalld
firewall-cmd  --list-ports
